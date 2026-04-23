# AGENTS.md

## Project Overview

An HTTPS forward proxy for local development with Choreo's Managed Authentication. It sits between a developer's local web app and the Choreo-hosted web app, proxying `/auth` and `/choreo-apis` requests to Choreo while forwarding all other traffic to the local app. Published as the `@choreo/proxy` NPM package.

## Commands

```bash
npm install              # Install dependencies
npm start -- -p <APP_PORT> -u <WEB_APP_URL> # Run the proxy via the package script
npx @choreo/proxy -p <APP_PORT> -u <WEB_APP_URL>  # Run via npx
```

There are no tests or linting configured in this project.

### CLI Arguments

| Flag | Long             | Description                                      | Required | Default |
|------|------------------|--------------------------------------------------|----------|---------|
| -p   | --localAppPort   | Port of the local web app                        | yes      | —       |
| -u   | --choreoAppUrl   | Choreo web app URL for auth proxy                | yes      | —       |
| -f   | --proxyPort      | Port the proxy listens on                        | no       | 10000   |
| -l   | --logLevel       | `info` or `debug`                                | no       | info    |

## Architecture

The entire proxy is a single file: `server.js`. It uses Express with `http-proxy-middleware` to create two proxy targets:

- **choreoProxy** — routes `/auth` and `/choreo-apis` to the Choreo app URL. For `/auth` requests, it injects an `X-Use-Local-Dev-Mode` header set to the proxy's own URL, signaling Choreo to redirect back to the local proxy.
- **localProxy** — routes everything else (`/`) to the developer's local app, including WebSocket upgrades.

The server generates a self-signed TLS certificate at startup using the `selfsigned` package (no files written to disk) and listens over HTTPS.

## Release Process

Releases are triggered manually via the `Release` GitHub Actions workflow (`workflow_dispatch`). It bumps the version (patch/major), pushes a git tag, creates a GitHub release, and publishes to NPM.
