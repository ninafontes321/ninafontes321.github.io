# TikTok App Review Integration Record

Status date: 2026-08-19

This document records the complete setup used to build, test, record, and submit the TikTok integration for review. It intentionally contains no client secrets, access tokens, passwords, or GitHub tokens.

## 1. Objective

The project is a small personal web application that lets the owner:

1. Sign in with TikTok using Login Kit and OAuth 2.0.
2. See the signed-in TikTok account name.
3. Upload a sample video to the TikTok inbox for review using Content Posting API.
4. Poll the upload status and display the resulting publish ID.

The demonstrated sandbox flow ends with `SEND_TO_USER_INBOX`. It does not publish a public video or read analytics.

## 2. Public GitHub Pages Site

Repository: `ninafontes321/ninafontes321.github.io`

Public site: `https://ninafontes321.github.io/`

Public legal pages:

- Terms of Service: `https://ninafontes321.github.io/tos.html`
- Privacy Policy: `https://ninafontes321.github.io/privacy.html`
- OAuth callback: `https://ninafontes321.github.io/callback.html`

The repository is public and GitHub Pages is enabled on the `main` branch.

### Public files and purpose

| File | Purpose |
| --- | --- |
| `index.html` | Static application UI and OAuth authorization URL construction |
| `callback.html` | Receives the OAuth authorization code and forwards it to the local backend |
| `tos.html` | Terms of Service page supplied to TikTok |
| `privacy.html` | Privacy Policy page supplied to TikTok |
| `sample.mp4` | Six-second vertical sample used by the Content Posting API demo |
| `tiktok-developers-site-verification-S0NnG5BwJhk2HUQKZpp9GSHFJZ7zDEnQ.txt` | TikTok domain-verification asset; public by design and not an API credential |

The app uses a public OAuth client key in `index.html`. A client key identifies the app but is not a secret. The client secret is never placed in the public repository or frontend.

## 3. Developer Portal Configuration

### App identity

- App name: `nina_app`
- App ID: `7675753460318619669`
- Ownership: Individual
- Production status after submission: `In review`
- Sandbox name: `dev-sandbox`
- Sandbox ID: `7675794587449182226`
- Sandbox target user: `ninafontesfit`

### Production draft submitted for review

The production draft was completed with:

- Category: Social Networking
- Description: Personal tool to post videos to my own TikTok account using TikTok official APIs.
- Platform: Web
- Website URL: `https://ninafontes321.github.io/`
- Terms of Service URL: `https://ninafontes321.github.io/tos.html`
- Privacy Policy URL: `https://ninafontes321.github.io/privacy.html`
- Web redirect URI: `https://ninafontes321.github.io/callback.html`
- Products: Login Kit and Content Posting API
- Scopes: `user.info.basic` and `video.upload`
- Demo video: `/home/ubuntu/tiktok_api/tiktok_demo.mp4`

The review submission reason was: `Initial production submission for Login Kit and Content Posting API inbox uploads.`

### Sandbox configuration used for the video

The sandbox was configured with the same products and scopes as the production submission. Login Kit Web configuration used the same redirect URI. Content Posting API was configured for inbox upload. Direct public posting and Display API analytics were not included in the final sandbox flow.

## 4. Frontend Flow

`index.html` is a static GitHub Pages application.

1. The user opens the public site.
2. The `Log in with TikTok` button builds the TikTok authorization URL with `user.info.basic` and `video.upload`.
3. TikTok displays the sandbox consent screen.
4. TikTok redirects to `callback.html` with an authorization code.
5. `callback.html` sends the code to `http://127.0.0.1:8321/exchange`.
6. The backend exchanges the code using the client secret, then stores the token locally.
7. The frontend requests `/me` and displays the TikTok display name.
8. The user enters or accepts a caption and clicks `Upload to TikTok inbox`.
9. The frontend requests `/post` and then polls `/status`.
10. The completed sandbox result is displayed as `SEND_TO_USER_INBOX`.

The frontend never receives or contains the client secret. The loopback backend is intentional for this personal demo and is not a production multi-user backend.

## 5. Local Backend

File: `/home/ubuntu/tiktok_api/demo/backend/server.js`

The Node HTTP backend listens only on `127.0.0.1:8321` and exposes these endpoints:

| Endpoint | Purpose |
| --- | --- |
| `POST /exchange` | Exchanges an OAuth code for TikTok tokens |
| `GET /me` | Reads the signed-in display name and account identifier |
| `POST /post` | Initializes and uploads the sample video |
| `GET /status` | Fetches Content Posting API status |
| `GET /health` | Reports backend availability and token presence |

Credentials are loaded only from the local, uncommitted file:

`/home/ubuntu/tiktok_api/demo/credentials.env`

The OAuth token cache is:

`/home/ubuntu/tiktok_api/demo/token.json`

The token cache was removed after the final recording. Neither file belongs in GitHub.

## 6. Content Posting API Upload

The demo uses `FILE_UPLOAD` rather than `PULL_FROM_URL`.

The sample file is six seconds, H.264, 720x1280, and approximately 6.2 MB. TikTok requires upload chunks of at least 5 MB, so the backend uses a chunk size of up to 10 MB and uploads the file with `Content-Range` headers.

`PULL_FROM_URL` was not used because TikTok required URL ownership verification for that method during setup. The public `sample.mp4` remains in the repository for the site and documentation, but the working demo upload uses the local file-upload path.

## 7. Demo Recording

Recorder: `/home/ubuntu/tiktok_api/demo/rerun.js`

The recorder connects to Chrome over CDP on port `9222`, drives the complete OAuth and upload flow, and uses FFmpeg to capture display `:1` at 1280x800 and 15 FPS.

Final video:

- Path: `/home/ubuntu/tiktok_api/tiktok_demo.mp4`
- Duration: 54.6 seconds
- Video: H.264, 1280x800, 15 FPS
- Size: 331595 bytes
- Result shown: `SEND_TO_USER_INBOX`

The recording includes the public app, TikTok consent screen, signed-in account, upload action, processing state, and completed status.

During recording, TikTok's web authorization page attempted to open a `bytedance://` desktop-app handoff. KDE had no handler and displayed a `Choose Application` dialog. The local machine now has a silent handler at:

`/home/ubuntu/.local/share/applications/ignore-bytedance.desktop`

The handler is a local desktop setting and is not part of the public site. The final video was checked at the reported four-second position and contains no chooser window.

The recorder was also changed to reuse one Chrome tab instead of closing the last tab before creating a new one. This prevents a Chrome CDP race during future recordings.

## 8. Review Submission Text

The production review explanation describes:

- Login Kit OAuth sign-in.
- `user.info.basic` usage to show the signed-in TikTok account.
- Content Posting API `video.upload` usage.
- `FILE_UPLOAD` transfer of the sample video.
- `SEND_TO_USER_INBOX` status handling.
- The public website shown in the video.
- No password collection, analytics scope, or public-posting scope.

This matches the selected production products, selected scopes, sandbox configuration, and attached video.

## 9. Public Repository Security Audit

The audit checked:

1. The current public GitHub tree.
2. All 20 commits currently reachable from the public repository history.
3. All current text assets for secret-like strings.
4. History paths for `.env`, credentials, secrets, tokens, private keys, and certificate files.

Results:

- No client secret found.
- No access token found.
- No refresh token found.
- No GitHub token found.
- No password found.
- No credential file or private-key file found.
- The public OAuth client key in `index.html` is expected and not a secret.
- The TikTok domain-verification file is expected to be public and is not an API credential.

The only GitHub change made for this documentation is the addition of this Markdown file. Existing public HTML pages, the callback, legal pages, sample video, and verification asset were not modified.

## 10. Safe Operating Rules

- Never commit `credentials.env`, `token.json`, `.git_pat`, browser profiles, screenshots containing private portal data, or API responses.
- Keep TikTok client secrets exclusively on a server or local backend.
- Treat any previously shared TikTok password, client secret, OAuth token, or GitHub token as exposed and rotate or revoke it.
- Keep the OAuth client key public only where the TikTok OAuth flow requires it.
- Do not request `video.publish`, `video.list`, or Display API scopes unless the app and demo are updated to use them.
- Keep `tos.html`, `privacy.html`, and `callback.html` publicly reachable while TikTok reviews the app.

## 11. Local Reproduction

Start the backend from the demo directory:

```bash
cd /home/ubuntu/tiktok_api/demo
node backend/server.js
```

Start the dedicated Chrome recording profile with remote debugging on port 9222, then run:

```bash
cd /home/ubuntu/tiktok_api/demo
node rerun.js --record
```

The browser must already be logged into the TikTok sandbox target account. The backend must have valid local credentials, and the GitHub Pages callback URL must remain registered in TikTok Login Kit.

## 12. After TikTok Approval

Approval does not automatically make the current local backend a hosted production service. Before using the app with production users:

1. Replace the frontend sandbox client key with the approved production client key.
2. Replace the backend local credentials with the production credentials, keeping the secret server-side.
3. Keep the approved redirect URI and website URL unchanged unless the portal configuration is updated first.
4. Deploy the backend behind HTTPS with authentication, token encryption, logging controls, and per-user token storage if more than one account will use it.
5. Request additional products or scopes only when the corresponding feature is implemented and recorded for review.

The current submitted review is intentionally limited to Login Kit and inbox video upload.
