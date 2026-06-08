# Google Cloud OAuth setup (Gmail)

Wren connects to Gmail through the **Gmail API + OAuth 2.0**, not IMAP. Because each person
self-hosts Wren and Gmail's sensitive scopes can't be shipped under shared credentials,
**you create your own Google Cloud OAuth client** and paste the Client ID/Secret into Wren.
This is a one-time manual step. For personal use you do **not** need Google to verify the
app.

## Checklist

1. **Create a Google Cloud project** — https://console.cloud.google.com/projectcreate
2. **Enable the Gmail API** — APIs & Services → Library → search "Gmail API" → Enable.
3. **Configure the OAuth consent screen** — APIs & Services → OAuth consent screen:
   - User type: **External**.
   - Fill in the required app name / support email.
   - Leave it in **Testing** mode (do not publish).
   - Under **Test users**, add the Google account(s) you'll connect. Testing mode means no
     Google verification is required, but only listed test users can authorize.
4. **Create OAuth client credentials** — APIs & Services → Credentials →
   Create credentials → **OAuth client ID**:
   - Application type: **Web application**.
   - Authorized redirect URI — add this **exactly**:
     ```
     http://localhost:8080/accounts/google/callback
     ```
5. **Copy the Client ID and Client Secret** into Wren during onboarding, or later in
   Settings (the app calls `POST /settings/google-client`). The secret is encrypted at
   rest with AES-256-GCM; it is never logged.

## Scopes Wren requests

Least privilege — intentionally **no permanent-delete** capability:

- `openid`
- `https://www.googleapis.com/auth/userinfo.email`
- `https://www.googleapis.com/auth/userinfo.profile`
- `https://www.googleapis.com/auth/gmail.modify` — read, mark read, label, archive, trash
- `https://www.googleapis.com/auth/gmail.send`
- `https://www.googleapis.com/auth/gmail.labels`

"Trash" moves a message to Trash only. Wren does **not** request the full-access
`https://mail.google.com/` scope and does **not** implement permanent delete.

## Notes

- The redirect URI must match exactly, including scheme, host, port, and path. If you run
  the backend somewhere other than `localhost:8080`, add that URI here too and set it in
  Wren's settings.
- Testing-mode refresh tokens can expire after 7 days of inactivity in some Google
  configurations. For a daily-driver client this is rarely an issue; if a connection goes
  stale, reconnect the account.
