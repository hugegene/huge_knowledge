# Getting a Permanent Facebook Page Access Token

How to obtain a non-expiring Page Access Token so a backend service can post to a Facebook Page indefinitely without re-authentication.

## Concepts: Token Types

Facebook's Graph API uses several distinct token types. Knowing which is which is the whole game.

### User Access Token

Authenticates **a person**. Issued after a user logs in and approves your app's requested permissions (scopes) via the FB Login dialog.

- **Acts as:** the user.
- **Can do:** anything the granted scopes allow on resources the user owns or admins (read their profile, list their pages, post on their behalf, etc.).
- **Lifetimes:**
  - **Short-lived:** ~1–2 hours. What you get immediately from `FB.login()` or Graph API Explorer's "Generate Access Token" button.
  - **Long-lived:** ~60 days. Produced by exchanging a short-lived User token via the `fb_exchange_token` endpoint.
- **Cannot be made permanent.** A User token always expires.

### Page Access Token

Authenticates **a Page**, on behalf of one of its admins.

- **Acts as:** the Page itself (posts appear as the Page, not as the user).
- **Can do:** Page-scoped actions — publish to feed, upload photos, publish Reels, read insights — depending on which Page-related scopes the source User token had.
- **Lifetimes:**
  - If derived from a **short-lived** User token → short-lived Page token (~1–2 hours).
  - If derived from a **long-lived** User token → **non-expiring** Page token. ← *this is what you want for backend services.*

### App Access Token

Authenticates **your app itself**, independent of any user. Useful for app-level admin operations and token-debugging calls. Cannot list a user's pages or post on their behalf. Not used in this guide except to authenticate the optional debug step.

---

## The Three-Stage Flow

To end up with a permanent Page Access Token you go through three stages. Each stage *produces a token of a specific type* — keep track of which token you're holding at each step.

| Stage | Input | Output | Token type |
|---|---|---|---|
| 1 | FB Login | Short-lived User token | **User** |
| 2 | Short-lived User token | Long-lived (60-day) User token | **User** |
| 3 | Long-lived User token | Non-expiring Page token (one per page admined) | **Page** |

Steps 1 and 2 are about the **User** token. Step 3 is the only step that produces a **Page** token.

---

## Prerequisites

- A Facebook account that is an **admin** of the target Page.
- A Facebook **App** registered at https://developers.facebook.com/apps/. Note its **App ID** and **App Secret** (App Dashboard → Settings → Basic).
- The app must request these permissions (declared in App Dashboard → App Review → Permissions, or simply request at login if your app is in Development mode and you are the admin):
  - `pages_show_list` — list pages the user manages
  - `pages_manage_posts` — create posts on the page
  - `pages_read_engagement` — read post engagement (often required alongside `pages_manage_posts`)

---

## Stage 1 — Get a Short-Lived User Access Token

Use Graph API Explorer: https://developers.facebook.com/tools/explorer/

1. **Top-right** — select your app from the dropdown.
2. **Right sidebar** — set the token type dropdown to **User Access Token**.
3. In the **Permissions** checklist, tick: `pages_show_list`, `pages_manage_posts`, `pages_read_engagement`.
4. Click **Generate Access Token**.
5. FB login dialog opens. Log in as the user who admins the target Page. Approve the requested scopes.
6. A token string appears in the token field.

**You now hold a short-lived User token.** Lifetime: ~1–2 hours. Do not store it — it's about to expire.

---

## Stage 2 — Extend to a Long-Lived User Access Token

Two options — pick one.

### Option A: Browser (Access Token Debugger)

1. In Graph API Explorer, click the small **ⓘ** icon next to the token field. This opens the Access Token Debugger in a new tab.
2. Scroll to the bottom → click **Extend Access Token**.
3. Enter your FB password to confirm.
4. The page returns a new token. Copy it.

### Option B: curl

```bash
curl -sG "https://graph.facebook.com/v19.0/oauth/access_token" \
  --data-urlencode "grant_type=fb_exchange_token" \
  --data-urlencode "client_id=YOUR_APP_ID" \
  --data-urlencode "client_secret=YOUR_APP_SECRET" \
  --data-urlencode "fb_exchange_token=SHORT_LIVED_USER_TOKEN"
```

Response:
```json
{"access_token": "EAAB...long...", "token_type": "bearer", "expires_in": 5183944}
```

`expires_in` is in seconds — ~60 days.

**You now hold a long-lived User token.** Still a User token, still expires (in ~60 days), but long-lived is the prerequisite for the next step.

---

## Stage 3 — Derive a Non-Expiring Page Access Token

This is the only step that produces a Page token. There is **one Page token per Page** — if the user admins multiple Pages, this call returns all of them.

### Option A: Browser (Graph API Explorer)

1. Paste the long-lived User token into the token field.
2. Change the request bar to: `GET /me/accounts` → click **Submit**.
3. Response includes a `data` array with one entry per Page you admin:
   ```json
   {
     "data": [
       {
         "access_token": "EAAB...",
         "category": "...",
         "id": "1234567890",
         "name": "My Page Name",
         "tasks": ["ANALYZE", "ADVERTISE", "MESSAGING", "MODERATE", "CREATE_CONTENT", "MANAGE"]
       }
     ]
   }
   ```
4. Copy the `id` and `access_token` for your target Page.

### Option B: curl

```bash
curl -s "https://graph.facebook.com/v19.0/me/accounts?access_token=LONG_LIVED_USER_TOKEN"
```

**You now hold a Page Access Token.** Because the User token in stage 2 was long-lived, this Page token is **non-expiring** (no `expires_at`).

---

## Verify the Page Token Is Truly Non-Expiring

Don't trust — verify. Use the Access Token Debugger.

### Option A: Browser

1. Open https://developers.facebook.com/tools/debug/accesstoken/.
2. Paste the Page token → **Debug**.
3. Look for the **Expires** field. It must say **Never**.

### Option B: curl

```bash
curl -s "https://graph.facebook.com/v19.0/debug_token?input_token=PAGE_TOKEN&access_token=APP_ID|APP_SECRET"
```

In the response, `data.expires_at` must be `0` (Unix epoch zero = never).

If you see a real timestamp instead: the User token in stage 2 wasn't actually long-lived. Redo stage 2 and try again.

---

## Using the Token

Once verified, store the Page ID and Page token as secrets in your backend's environment. Example for a Page post:

```bash
curl -X POST "https://graph.facebook.com/v19.0/${PAGE_ID}/feed" \
  -d "message=Hello world" \
  -d "access_token=${PAGE_TOKEN}"
```

---

## Managing Permanent Page Tokens

Facebook does **not** provide a UI listing "all tokens I have issued." Tokens are not browsable records. Practical management:

| Goal | How |
|---|---|
| **List current page tokens** | Re-run `GET /me/accounts` with any valid long-lived User token. The response IS the canonical list of currently-valid Page tokens for every Page the user admins. You can always re-derive. |
| **Inspect a token** | Access Token Debugger — paste any token, see scopes, app, user, expiry. |
| **Revoke a Page token** | No per-token revoke. To invalidate: (a) user goes to facebook.com/settings → "Apps and Websites" → find your app → **Remove** (kills all tokens that app issued to this user); (b) `DELETE /{user-id}/permissions` to revoke all permissions; (c) `DELETE /{user-id}/permissions/{scope}` to revoke one scope. |
| **Rotate** | Redo stages 1 → 2 → 3, swap in the new token, then revoke the old one via the user's app-settings page if desired. |
| **Monitor app activity** | App Dashboard → usage metrics, error logs, request volume. Won't show individual tokens but shows aggregate misuse signals. |

### Causes of token invalidation

A "permanent" Page token will stop working if:

- The user **changes their FB password**.
- The user **removes the app** from their account.
- The user **revokes the underlying permission** (`pages_manage_posts` etc.).
- The user is **removed as an admin** of the Page.
- Facebook **server-side revokes** the app's permission (rare, typically for policy violations).

When a token breaks, the fix is always: redo the three-stage flow for that one user/page. Other pages' tokens are unaffected.

---

## Common Pitfalls

- **Short-lived User token in stage 3** → produces a short-lived Page token. Always extend (stage 2) first.
- **Wrong app selected** in Graph API Explorer → token will be scoped to a different app and won't work in your backend.
- **Missing `pages_manage_posts` scope** → posting calls return `(#200) Permissions error` even though the token is otherwise valid.
- **Trying to post with an App Access Token** → returns permission errors. App tokens can't act on a user's resources.
- **Using a User token instead of a Page token to post** → post will appear as the user, not as the Page, and may fail if scopes don't cover it.
- **Storing the long-lived User token instead of the Page token** → it expires in 60 days; you have to re-auth a human every two months. Always derive and store the Page token instead.

---

## Reference Endpoints

| Purpose | Endpoint |
|---|---|
| Exchange short-lived → long-lived User token | `GET /oauth/access_token?grant_type=fb_exchange_token&client_id=…&client_secret=…&fb_exchange_token=…` |
| List pages + Page tokens for a user | `GET /me/accounts?access_token={user_token}` |
| Inspect any token | `GET /debug_token?input_token={token}&access_token={app_id}|{app_secret}` |
| Revoke all app permissions for a user | `DELETE /{user-id}/permissions?access_token={user_token}` |
| Revoke a single permission | `DELETE /{user-id}/permissions/{scope}?access_token={user_token}` |
| Publish a Page post | `POST /{page-id}/feed` with `message=…&access_token={page_token}` |
