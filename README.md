# Wren

A self-hosted email client for **macOS** with a small backend you run yourself. Phase 1 connects to Gmail.

> **Local / personal use.** The backend speaks plain HTTP (no TLS). Run it on your own machine
> or a trusted LAN — don't expose it directly to the public internet.

This repo hosts the **downloads** (macOS app) and **run instructions**. The source code lives in a
private repository; only the prebuilt artifacts are published here.

---

## 1. Run the backend

Requires [Docker](https://www.docker.com/). The backend image is published publicly to GHCR.

```sh
# Grab the compose file + env template from this repo
curl -O https://raw.githubusercontent.com/stefanodecillis/wren-dist/main/docker-compose.yml
curl -o .env.example https://raw.githubusercontent.com/stefanodecillis/wren-dist/main/.env.example
cp .env.example .env

# Generate the required secrets and paste them into .env
openssl rand -base64 32   # MASTER_ENCRYPTION_KEY
openssl rand -base64 48   # JWT_ACCESS_SECRET
openssl rand -base64 48   # JWT_REFRESH_SECRET

docker compose up -d
curl http://localhost:8080/health        # -> {"status":"ok",...,"hasAdmin":false}
```

The image is `ghcr.io/stefanodecillis/wren-backend:latest` (multi-arch: Apple Silicon + Intel/x86).
It runs database migrations automatically on start.

## 2. Install the macOS app

1. Download the latest **`Wren-<version>.dmg`** from [Releases](../../releases).
2. Open the DMG and **drag Wren into Applications**.
3. Launch Wren and set the backend URL to `http://localhost:8080` (or your host's LAN IP, e.g.
   `http://192.168.1.20:8080`). Create the admin account on first run.

You only do this once — Wren keeps itself up to date from then on (see [Updating](#updating)).

> **First launch:** if the build isn't notarized yet, macOS may say it "cannot be opened." Right-click
> the app → **Open** → **Open** (once), or run `xattr -dr com.apple.quarantine /Applications/Wren.app`.

## 3. Connect Gmail

Wren uses the Gmail API with your own Google Cloud OAuth client (Gmail's sensitive scopes can't ship
under shared credentials). Follow [`docs/google-cloud-setup.md`](docs/google-cloud-setup.md) once,
then paste the Client ID/Secret into Wren. Authorize this redirect URI in Google Cloud:

```
http://localhost:8080/accounts/google/callback
```

(Use your `PUBLIC_BASE_URL` instead of `localhost` if the backend is on another host.)

## 4. AI features (optional)

Off by default — the AI UI stays hidden until you configure a provider. See
[`docs/ai-setup.md`](docs/ai-setup.md): "Sign in with ChatGPT" (Codex) or an OpenAI API key.

---

## Updating

```sh
docker compose pull && docker compose up -d   # backend
```

The **macOS app updates itself automatically** (via Sparkle): it checks daily and installs new
versions silently in the background. You can also trigger it anytime from **Wren ▸ Check for Updates…**.
No need to re-download the DMG after the first install.

## Privacy & security

Third-party credentials (Google OAuth secret, Gmail tokens, OpenAI key, ChatGPT tokens) are encrypted
at rest with AES-256-GCM; passwords use Argon2id; refresh tokens rotate and are stored only as hashes.
There is no TLS — keep Wren on your own machine or a trusted LAN.
