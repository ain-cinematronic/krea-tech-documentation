# Deployment (Sanitized)

This document describes how to deploy the application to Payload Cloud.

## Prerequisites

- The app builds and runs locally:
  - `yarn install`
  - `yarn build`
  - `yarn serve`
- You have a Payload Cloud account.
- A GitHub repository exists with your code pushed.

## Steps

1. Go to Payload Cloud and log in.
2. Click **New Project**.
3. Choose **Import an existing Git codebase**.
4. Select **Continue with GitHub** and authorize if prompted.
5. Pick the GitHub org/user that hosts your repo. Install the GitHub App if needed.
6. Choose the repository and proceed to **Configure your project**.
7. Select a plan and team.
8. In **Project Details**, choose a region and set project name, slug, and default domain.
9. Keep **Build Settings** default, except set **Branch to deploy** to your main branch (e.g., `main`).
10. In **Environment Variables**, add what your app needs. Example set (placeholders only):

    ```conf
    # Server port
    PAYLOAD_PORT=3000

    # Canonical Payload server URL (used for server callbacks & CORS)
    PAYLOAD_PUBLIC_URL=https://<PROJECT_DOMAIN>.payloadcms.app
    # For local testing:
    # PAYLOAD_PUBLIC_URL=http://localhost:3000

    # Public site origin for client code (used in the browser)
    NEXT_PUBLIC_URL=https://<PUBLIC_SITE_DOMAIN>
    # For local testing:
    # NEXT_PUBLIC_URL=http://localhost:3000

    # Preview secret (must match server/client preview route config)
    PAYLOAD_PUBLIC_DRAFT_SECRET=<GENERATE_RANDOM_STRING>

    # API key created in Admin -> API Keys after first deploy
    NEXT_PAYLOAD_API_KEY=<SET_AFTER_FIRST_DEPLOY>

    # App secrets / feature flags
    PAYLOAD_SECRET=<GENERATE_RANDOM_STRING>
    NEXT_PUBLIC_ENABLE_RATING_BANNER=true

    # Optional logging (default: false)
    verbose_logging=false

    # Optional: increase admin safety margin
    PAYLOAD_RATE_LIMIT_MAX=500
    ```

    Optional legacy/back-compat:

    ```conf
    # Preferred for preview URL generation when set
    PAYLOAD_PUBLIC_NEXT_URL=https://<PUBLIC_SITE_DOMAIN>
    ```

11. Agree to the terms and click **Deploy Now**.
12. After deploy completes, open the Admin UI at:
    `https://<PROJECT_DOMAIN>.payloadcms.app/admin` and create the first admin user.
13. In **API Keys**, create a key:
    - Enabled: `true`
    - Name: (your choice)
    - Role: (minimum required for server-to-server routes)
14. Copy the key value into `NEXT_PAYLOAD_API_KEY` in the project’s **Environment Variables** and **Update** to restart.
15. The app is live at the assigned domain. New pushes to the chosen branch trigger automatic deploys.


