# Sample Website – GitHub → GitHub Actions → Docker Hub → Docker

Static website served by Nginx inside a Docker container.
The Docker image contains **only** the built site files (`index.html`, `style.css`), so visitors never see the source repository.

## Project layout

```
sample-website/
├── index.html
├── style.css
├── nginx.conf
├── Dockerfile
├── .dockerignore
├── .gitignore
└── .github/workflows/docker-publish.yml
```

## 1. Push the code to a PRIVATE GitHub repo

Create an empty **private** repository on GitHub (e.g. `sample-website`), then:

```bash
cd sample-website
git init -b main
git add .
git commit -m "Initial commit"
git remote add origin https://github.com/<your-github-user>/sample-website.git
git push -u origin main
```

A private repo is what keeps the source code hidden from the client.

## 2. Add Docker Hub secrets in GitHub

1. Docker Hub → Account Settings → Security → **New Access Token** (Read & Write). Copy it.
2. GitHub repo → Settings → Secrets and variables → Actions → **New repository secret**:
   - `DOCKERHUB_USERNAME` = your Docker Hub username
   - `DOCKERHUB_TOKEN` = the access token from step 1

Every push to `main` now triggers the workflow, which builds the image and pushes
`<dockerhub-user>/sample-website:latest` to Docker Hub. Check progress in the repo's **Actions** tab.

## 3. Run it in Docker Desktop

```bash
docker pull <dockerhub-user>/sample-website:latest
docker run -d --name sample-website --restart unless-stopped -p 8080:80 <dockerhub-user>/sample-website:latest
```

Open http://localhost:8080

## 4. Make it reachable from the Internet at any time

Docker Desktop runs on your own PC, so the site is only online while the PC is on.
Choose one option:

**A. 24/7 hosting (recommended for a client)** – run the same Docker Hub image on an always-on host:
a small cloud VM (AWS EC2 / Azure VM / any VPS) with Docker installed, or a container service such as
Azure Web App for Containers, AWS App Runner or Render. Point the service at
`<dockerhub-user>/sample-website:latest` and open port 80. Add a domain name and HTTPS (e.g. via Cloudflare).

**B. Temporary public URL from Docker Desktop (only while the PC is on)** – Cloudflare Tunnel:

```bash
cloudflared tunnel --url http://localhost:8080
```

It prints a public `https://....trycloudflare.com` URL.

## Updating the site

Edit the files → `git add . && git commit -m "Update" && git push`.
GitHub Actions rebuilds and pushes a new image; run `docker pull` + `docker run` again (or let your host redeploy).
