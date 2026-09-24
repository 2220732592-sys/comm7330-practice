# COM7330 Python Studio — GitHub Pages edition

Personal programming practice and mock exams for BNBU COM7330. English course-style programming questions, Python editor, visible and hidden test results, partial-credit scores, material-driven practice, mistake book, statistics and anonymous autosave.

## Deployment status

This repository contains the GitHub Pages frontend and its Railway Python backend. GitHub Pages is configured for GitHub Actions with HTTPS enforced. Deployment and public integration checks are in progress. The frontend builds successfully, and the backend passes the local execution, grading, isolation and persistence checks described below.

The complete source is in `com7330-source.zip`, uploaded through GitHub's web interface. The Pages workflow extracts this archive before building. Extract it locally to edit the application; upload an updated source archive to redeploy. The archive includes no runtime credentials, user records or installed dependencies.

## Architecture

- GitHub Pages serves the React/Vite frontend. No backend secrets are bundled into the frontend.
- Railway runs CPython 3.13.15 compiled to WASI, using Wasmtime 49. Every test has a separate WebAssembly instance and disposable host process.
- Each test has a 3-second wall-time limit, a 64 MiB guest memory limit, output limits and no access to host environment, user data, sockets or processes. The backend accepts 2 concurrent execution requests, with a bounded queue.
- Anonymous visitors receive a signed random token stored in their browser. No login is needed for practice. API requests use the token to isolate records. Cross-origin access is limited to `https://2220732592-sys.github.io`.
- SQLite and original uploaded files live on a persistent Railway volume. Unsaved answer drafts remain in browser local storage. Clearing the browser's site data loses its anonymous identity; another device starts a separate workspace.
- The original Site remains separately available. Its existing visitor records are not automatically transferred to the new origin.

## GitHub Pages

1. Use repository `2220732592-sys/comm7330-practice`. Upload this source without `.env.local`, `node_modules`, `dist`, or any runtime credentials.
2. Repository Settings → Pages → Build and deployment → Source: **GitHub Actions**.
3. `.github/workflows/pages.yml` builds and deploys on pushes to `main`. The workflow gets the correct project base path from `configure-pages`.
4. Wait for the deployment job to succeed and use the actual URL returned by GitHub. Do not mistake a source repository URL for the live site.

For local development:

```sh
pnpm install --frozen-lockfile
pnpm dev
pnpm build
```

Use Node 24 and pnpm 11.25.0. `VITE_API_ORIGIN` is the public Railway API origin, not an API key. Add a localhost origin explicitly to `PUBLIC_ORIGINS` only for local development, not the production deploy.

## Railway backend

Existing service: `python-sandbox-api`, public origin `https://python-sandbox-api-production.up.railway.app`.

Before deploying the new API, confirm a persistent volume is mounted at `/data`. The bootstrap deliberately refuses to start without this mount to prevent pretending that temporary disk is durable storage.

The existing official image `python:3.13-slim` starts via `services/python-executor/bootstrap.py`, provided as `EXECUTOR_BOOTSTRAP_B64`. `EXECUTOR_BUNDLE_B64` contains gzip-compressed JSON of these five UTF-8 source files: `server.py`, `sandbox.py`, `runner.py`, `case_worker.py`, `visitor_api.py`. `EXECUTOR_KEY` stays exclusively in server configuration. Preserve the existing key so the existing Site can still use the private `/execute` endpoint. Set `PUBLIC_ORIGINS=https://2220732592-sys.github.io` and `PORT=8080`.

The bootstrap pins Python WASI by SHA-256, installs pinned Wasmtime/aiohttp versions, drops root privileges, and gives the HTTP service access only to its own data directory. Cold starts need access to the pinned package downloads. The guest sandbox never receives the backend credentials or storage mount.

Visitor records are limited to 1.8 MB per workspace request. Original files are limited to 20 MB each, 80 MB per visitor, and 256 MB overall; records use at most 128 MB overall. Capacity and rate-limit failures are visible to the user, and unsaved drafts remain local. The 500 MB volume is intended for personal/small-group use; existing Railway usage and quota limits still apply.

Railway is currently on a trial plan. Public Python execution and cloud saves require available Railway credit and an active plan; GitHub Pages only hosts the frontend. The trial must be renewed or upgraded by the account owner before its time or credit runs out. No paid upgrade has been made.

## Verification

`services/python-executor/check_pages_api.py` exercises the real local HTTP API and WASI execution: all 60 reference tests, exactly 10 questions/100 marks, concurrent submissions, anonymous workspace separation, reload, stale-save conflicts, CORS, invalid tokens, uploads, rate limiting and reopening the database after a backend restart.

`services/python-executor/check.py` verifies timeouts, output and memory limits, host-file/environment/network isolation and concurrency. Run with `PYTHON_WASI_ROOT` pointing at the verified CPython WASI runtime and Wasmtime installed.

This is a study tool, not a secure proctored exam platform. Reference solutions and test definitions are shipped to the frontend, although the interface only reveals solutions after submission. Scores and learning records are editable by their owner. Material-driven questions use verified course templates and extracted topic signals; unsupported new task types are not silently invented.
