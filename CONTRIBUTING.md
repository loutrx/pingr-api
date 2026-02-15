# Contributing to Pingr

First off, thanks for taking the time to contribute! Pingr is a community-driven project and every contribution makes a difference.

## Code of Conduct

By participating in this project, you agree to be respectful and constructive in all interactions. We want Pingr to be a welcoming space for everyone, regardless of experience level.

## How Can I Contribute?

### Reporting Bugs

Before creating a bug report, please check existing issues to avoid duplicates.

When filing a bug, include:

- **A clear title** describing the problem
- **Steps to reproduce** the issue
- **Expected behavior** vs **actual behavior**
- **Environment details** — OS, Go/Node version, Docker version, device model (for the app)
- **Logs or screenshots** if applicable

### Suggesting Features

Feature requests are welcome! Open an issue with the `enhancement` label and describe:

- **The problem** you're trying to solve
- **Your proposed solution**
- **Alternatives** you've considered

### Pull Requests

1. **Fork** the repository
2. **Create a branch** from `main` — use a descriptive name:
   - `feat/container-logs` for new features
   - `fix/cpu-metric-accuracy` for bug fixes
   - `docs/api-examples` for documentation
3. **Make your changes** — keep commits focused and atomic
4. **Test your changes** locally
5. **Open a PR** against `main` with a clear description of what and why

## Development Setup

### pingr-api (Go)

```bash
git clone https://github.com/loutrx/pingr-api.git
cd pingr-api
go mod download
go run ./cmd/pingr
```

Requirements:
- Go 1.22+
- Docker (for container monitoring features)
- Access to Docker socket (`/var/run/docker.sock`)

Run tests:

```bash
go test ./...
```

### pingr-app (Expo)

```bash
git clone https://github.com/loutrx/pingr-app.git
cd pingr-app
npm install
npx expo start
```

Requirements:
- Node.js 18+
- Expo Go app on your phone (for quick testing)
- A running Pingr agent to connect to (or use the mock server — see below)

#### Mock Server

For app development without a real server, you can use the mock mode:

```bash
npm run dev:mock
```

This starts a local mock API that returns sample data, so you can work on the UI without deploying an agent.

## Guidelines

### Go (API)

- Follow standard Go conventions (`gofmt`, `go vet`)
- Keep functions short and focused
- Add comments for exported functions and types
- Error handling: always handle errors explicitly, no silent ignores
- New endpoints need tests

### TypeScript (App)

- Use TypeScript strictly — no `any` types
- Follow the existing file structure in `app/` and `components/`
- Components should be functional with hooks
- Keep styles consistent with the existing dark theme and color palette

### Commits

Write clear commit messages:

```
feat: add container restart confirmation dialog

Add a confirmation modal before restarting a container
to prevent accidental restarts. Includes haptic feedback
on the confirm button.
```

Format: `type: short description`

Types: `feat`, `fix`, `docs`, `refactor`, `test`, `chore`

### Documentation

- Update the README if your change affects setup, config, or API endpoints
- Add inline comments for non-obvious logic
- API changes should include request/response examples

## Project Structure

Pingr is split into two repositories:

| Repo | Description | Language |
|------|-------------|----------|
| [pingr-api](https://github.com/loutrx/pingr-api) | Server monitoring agent | Go |
| [pingr-app](https://github.com/loutrx/pingr-app) | Mobile app | TypeScript (Expo) |

If your contribution spans both repos (e.g., a new API endpoint + app screen), please open linked PRs in both repositories.

## First Time Contributing?

Look for issues labeled `good first issue` — these are specifically picked for newcomers. Some ideas:

- Add a new system metric to the API
- Improve error messages in the app
- Add a translation
- Fix a typo in the docs
- Write tests for existing code

## Questions?

Open a discussion in the relevant repository or reach out at [hello@otterium.com](mailto:hello@otterium.com).

---

Thanks for helping make Pingr better! 🏓
