# AuthAPI

Personal license-key API and dashboard.

## Run locally

1. Install Node.js 18 or newer.
2. Install dependencies with `npm install`.
3. Set the environment variables described below.
4. Start the API with `npm start`.
5. Open `http://localhost:10000`.

## Environment variables

Set secrets in the hosting provider's environment-variable settings. Do not commit real values.

- `JWT_SECRET`: a long, random secret used to sign authentication tokens.
- `HMAC_SECRET`: a long, random secret used to sign license keys. Keep it stable or existing key signatures will no longer validate.

Optional AI provider credentials:

- `OPENAI_API_KEY`
- `ANTHROPIC_API_KEY`

AI features remain unavailable when their provider credential is not configured.

## App package checks

Create and manage app packages in the dashboard's **Gói ứng dụng** tab without an administrator token. Package management endpoints are public: anyone who can reach the server can create, disable, or modify packages and assign keys. Keep this deployment private or add access control before exposing it to other users. Assign each license key to its intended package when creating the key. The app can start a Get Key registration with `POST /api/udid/start` and JSON `{ "key": "<issued-key>", "packageId": "<package-id>" }`. On success, open the returned `registrationUrl` in Safari. The URL contains a random, one-time flow token, not the key; the flow expires after 10 minutes. The user installs the UDID profile there, and the page verifies and links the key to that device automatically. Successful `POST /api/verify-key` calls also link any previously captured UDID to the verified key. Duplicate captures of the same UDID are consolidated into one device record. On iPhone/iPad the page captures the actual UDID. The key list shows captured model and iOS details when available. The app should call `POST /api/app/check-package` with its package ID and key when it starts and periodically while running. It must send that same package ID to `POST /api/verify-key`.

When a package is disabled, the check endpoint returns `allowed: false` with a configured message and optional update URL. The app should show the message and safely block protected features; it should not crash.
