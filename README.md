Docker Image Publisher — CI/CD Pipeline with GitHub Actions

A Node.js application with a fully automated CI/CD pipeline that builds and publishes Docker images to Docker Hub on every release tag.

---

Description

This project demonstrates a production-ready CI/CD pipeline built with GitHub Actions. Every time a new version tag (e.g. `v1.0.0`) is pushed, the pipeline automatically runs tests, builds a Docker image, and publishes it to Docker Hub with two tags: the version tag and `latest`.

---

Features

- ✅ Automated testing on Node.js 18 and 20 (matrix strategy)
- ✅ Dependency caching with `~/.npm` for faster runs
- ✅ Concurrency control — cancels outdated runs automatically
- ✅ Skip CI support via `[skip ci]` in commit messages
- ✅ Automatic Docker image build and push on tag release
- ✅ Custom composite GitHub Action for Docker build logic
- ✅ Secrets management for Docker Hub credentials
- ✅ Log upload on failure for easy debugging

---

Project Structure

```
dockerapp/
  .github/
    workflows/
      ci.yml           # Runs tests on every push to master
      publish.yml      # Builds and pushes Docker image on tags
    actions/
      docker-build/
        action.yml     # Custom composite action for Docker build
  src/
    index.js           # Express app
    __tests__/
      index.test.js    # Jest tests
  package.json
  package-lock.json
  Dockerfile
  .dockerignore
```

---

Workflows

### `ci.yml` — Continuous Integration

Triggers on every push and pull request to `master`.

- Skips if commit message contains `[skip ci]`
- Skips if triggered by `dependabot`
- Cancels previous runs on the same branch (concurrency)
- Runs tests on **Node 18** and **Node 20** in parallel (matrix)
- Caches `~/.npm` using `hashFiles('package-lock.json')` for speed
- Uploads logs automatically if any step fails

### `publish.yml` — Docker Image Publisher

Triggers only when a tag starting with `v` is pushed (e.g. `v1.0.0`).

- Runs tests first (`needs: test`) before publishing
- Logs into Docker Hub using repository secrets
- Calls the custom `docker-build` composite action
- Pushes two tags to Docker Hub: `:v1.0.0` and `:latest`
- Outputs the full image name for reference

### `actions/docker-build/action.yml` — Custom Action

A reusable composite action that:

- Takes `image_name`, `tag`, and `dockerfile_path` as inputs
- Builds the Docker image with both version and latest tags
- Pushes both tags to Docker Hub
- Returns `full_image_name` as output

---

Required Secrets

Add these in **GitHub → Settings → Secrets and variables → Actions**:

|Secret|Description|
|---|---|
|`DOCKER_USERNAME`|Your Docker Hub username|
|`DOCKER_PASSWORD`|Your Docker Hub access token|

---

Docker Hub

The image is available at:

```
docker pull hamzabkr/dockerapp:latest
docker pull hamzabkr/dockerapp:v1.0.0
```

Run the app locally:

```bash
docker run -p 3000:3000 hamzabkr/dockerapp:latest
```

Then open: `http://localhost:3000`

---

## 📦 API Endpoints

|Method|Endpoint|Description|
|---|---|---|
|GET|`/`|Returns app info and version|
|GET|`/health`|Health check endpoint|

---

Local Development

```bash
# Install dependencies
npm install

# Run tests
npm test

# Start the app
npm start
```

---

Release a New Version

```bash
# Create and push a new tag
git tag v1.0.1
git push origin v1.0.1
```

This automatically triggers the pipeline and publishes a new Docker image.

---

GitHub Actions Concepts Used

|Concept|Where Used|
|---|---|
|Matrix strategy|Node 18 and 20 in `ci.yml`|
|Dependency caching|`~/.npm` cache with `hashFiles`|
|Concurrency|Cancel in-progress runs|
|Job outputs|Pass image tag between jobs|
|Custom composite action|`docker-build` action|
|Secrets|Docker Hub credentials|
|`continue-on-error`|Lint step|
|`if: failure()`|Upload logs on failure|
|`needs:`|Test must pass before publish|
|Tag trigger|`on: push: tags: v*`|
|`startsWith()` function|Validate tag ref|

