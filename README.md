# CI/CD with Docker and GitHub Actions (Backend)

This repo includes a simple Node.js backend (Express + Mongoose) and a CI/CD pipeline using Docker and GitHub Actions.

## What you get
- Dockerfile for the backend service (port 3001)
- `.dockerignore` to reduce image size and speed up builds
- CI workflow: builds and pushes image to GitHub Container Registry (GHCR)
- CD demo workflow: prints a safe, copy-pasteable deploy plan (no real server)

## Prerequisites
- The repository must be on GitHub
- Ensure `Actions` are enabled
- GHCR uses the built-in `GITHUB_TOKEN` for authentication

## Image name
Images push to: `ghcr.io/<owner>/<repo>-backend`

## How to run locally
```bash
# From repo root
cd backend
npm install
npm start
# Service will listen on http://localhost:3001
```

Note: The app currently connects to MongoDB Atlas via hardcoded URI in `backend/src/index.js`. For production, switch to environment variables (e.g., `MONGODB_URI`).

## CI: Build and Push
- Trigger: push to `main` (or run manually) touching `backend/**`
- Workflow: `.github/workflows/ci.yml`
- Produces tags: `latest` on default branch, `sha`, and branch/tag names

## CD: Demo Deploy
- Trigger: manual (`workflow_dispatch`)
- Workflow: `.github/workflows/cd-demo.yml`
- Prints out commands to deploy the image on any Linux host with Docker. No real server is contacted.

### Example deploy on a server
```bash
docker login ghcr.io -u <GITHUB_USER> -p <YOUR_GITHUB_TOKEN>
docker pull ghcr.io/<owner>/<repo>-backend:latest
docker stop backend || true && docker rm backend || true
docker run -d --name backend -p 3001:3001 \
  -e PORT=3001 \
  -e MONGODB_URI=<your_mongodb_uri> \
  ghcr.io/<owner>/<repo>-backend:latest
```

### Optional: enable real SSH deploy
1. Add secrets in repo settings: `SSH_HOST`, `SSH_USER`, `SSH_KEY`, `MONGODB_URI`.
2. In `.github/workflows/cd-demo.yml`, set the SSH step `if:` to `true`.

## Switching to Docker Hub (optional)
If you prefer Docker Hub:
- Add repo secrets: `DOCKERHUB_USERNAME`, `DOCKERHUB_TOKEN`.
- Replace the GHCR login step with Docker Hub:

```yaml
- uses: docker/login-action@v3
  with:
    username: ${{ secrets.DOCKERHUB_USERNAME }}
    password: ${{ secrets.DOCKERHUB_TOKEN }}
```

- Change `REGISTRY`/`IMAGE_NAME` to something like `docker.io/<username>/<repo>-backend`.

## Notes
- For production, avoid hardcoding credentials. Use secrets and env vars.
- Consider adding a `package-lock.json` for reproducible builds and `npm ci`.
