# AI setup (optional)

Wren's AI features are **off by default** and hidden in the UI until you configure a
provider. AI calls run on the backend, never from the apps directly.

## What the AI does

- **Writing profile (per account).** Wren studies the account's own *sent* mail to learn
  your identity, tone, formality, and typical greetings/sign-offs, and stores a compact
  profile used by the other features. It regenerates automatically about every two months;
  you can force it from **Settings → AI writing profile → Regenerate**.
- **Auto-drafts.** In the background, Wren looks at unread inbox threads where someone is
  waiting on you, asks the model whether a reply is warranted, and if so pre-writes one in
  your voice. When you open the reply composer for that thread, a banner — *"Auto draft to
  <name>"* — offers **Use** (insert and edit) or **Delete**. Newsletters, notifications,
  no-reply, and receipts are skipped.
- **Proofread / Improve.** Select text in the composer and run **Proofread** (fix errors)
  or **Improve** (a business-editor rewrite) from the bottom-bar buttons or the right-click
  / long-press menu. The result replaces your selection (or the whole body if nothing is
  selected).
- **Summarize.** Condense a thread into bullet points from the reading pane.

All of these work on **either provider** — your ChatGPT subscription ("Sign in with
ChatGPT") or an OpenAI API key. On the API-key provider, proofread/improve and the
background features are pinned to a cheap model (`gpt-4o-mini` by default, independent of
the model you set for summaries) to keep cost predictable; on the ChatGPT subscription they
use your subscription's model. The background work (writing profile + auto-drafts) runs on a
bounded poller (a small batch per account every few minutes) so it stays gentle on
subscription rate limits.

There are two providers, both behind one interface. Pick one in onboarding or Settings.

## Option A (recommended): Sign in with ChatGPT

Uses your existing ChatGPT subscription via the same OAuth flow the Codex CLI uses, then
calls OpenAI's Codex backend with the resulting token. No API key to manage.

How it works in Wren:
1. On the **Mac**, choose "Sign in with ChatGPT". The app opens OpenAI's consent page in your browser.
2. After you approve, the **Mac app** captures the redirect locally on
   `http://localhost:1455/auth/callback` and sends the code to the backend. This works no matter
   where the backend runs (remote / behind a reverse proxy) — no port needs to be published.
3. **If port 1455 is busy** (the real Codex CLI is running) or you're on iOS, Wren shows the
   redirect URL and you paste it (or the `code`) back. This always works.
4. The backend exchanges + stores the tokens encrypted (AES-256-GCM), refreshes them, and runs AI
   for every device on your account — so you only sign in once, on the Mac.

### Important caveats (please read)

- This reuse of ChatGPT/Codex credentials for a non-Codex app is **undocumented** by
  OpenAI. It is framed across the community as **personal use only**: your own
  subscription, a single user, no resale, no multi-user serving, and respect the rate
  limits. You are responsible for complying with OpenAI's Terms of Use.
- **Shared-account logout caveat:** if the *same* ChatGPT account is also signed in via the
  Codex CLI or another tool, refresh-token rotation can intermittently log one of them out.
  Wren treats its own store as the single canonical token sink.
- This provider is **fragile by nature** — if OpenAI changes the flow it may break. It is
  kept fully isolated in one module so only it breaks, and the API-key provider below
  remains as a standing fallback.

## Option B (fallback): OpenAI API key

Paste an OpenAI API key. Wren calls `${baseUrl}/chat/completions` (default
`https://api.openai.com/v1`) with your configured model. Because it's OpenAI-compatible,
you can point `baseUrl` at a local model server or another gateway instead.

This is the durable path: if the ChatGPT sign-in flow ever changes, this keeps working.

## Skipping AI

Choose "Skip for now". The AI endpoints return `409` while unconfigured and the apps hide
all AI UI. You can enable it later in Settings.

## Privacy

Prompt contents and tokens are never written to logs.
