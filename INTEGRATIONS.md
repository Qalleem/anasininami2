# Rap Studio v8 Integrations

This app includes in-UI module controls for:
- Build Web Apps
- Android QA
- Vercel
- Netlify
- GitHub
- Hugging Face

## Runtime module behavior
- Module state is stored in `localStorage` key `rapstudio_integrations_v1`.
- Integration profile can be exported/imported from the **Tools** screen.
- Hugging Face module can run live text enhancement via Inference API (requires user token).

## Deployment
- `vercel.json`: root rewrite to `rapstudio_v8.html`
- `netlify.toml`: root redirect to `rapstudio_v8.html`
- `.github/workflows/deploy-static.yml`: verify + conditional deploy to Vercel/Netlify

## Required GitHub secrets (for CI deploy)
- `VERCEL_TOKEN`
- `VERCEL_ORG_ID`
- `VERCEL_PROJECT_ID`
- `NETLIFY_AUTH_TOKEN`
- `NETLIFY_SITE_ID`
