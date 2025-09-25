# Backstage on Local Minikube

> Quick, copy-pasteable README to run Backstage locally on Minikube (Windows WSL / macOS).

---

## What this repo contains

This repository contains manifests and instructions to run Backstage locally inside a Minikube cluster. The README shows step-by-step commands for:

* Starting Minikube (Windows WSL / macOS)
* Preparing GitHub credentials (Personal Access Token + OAuth App)
* Wiring those credentials into Backstage config (safe via Kubernetes Secret)
* Applying the `backstage.yaml` manifest
* Verifying Backstage and Postgres pods are running and port-forwarding Backstage to `localhost:7007`

> **Note:** This README intentionally keeps external prerequisites brief (no long installer guides). If you want full installation steps for any prerequisite, tell me which one and I’ll expand.

---

## Prerequisites (brief)

* A machine with either **Windows + WSL2** (Ubuntu) or **macOS**.
* **Docker Desktop** (running) or a Docker engine accessible from WSL2 / macOS.
* `minikube` installed and on `PATH`.
* `kubectl` installed and on `PATH`.
* A **GitHub account** (you will create a Personal Access Token and an OAuth App).

> Don’t install anything here yet — these are the items you must have available before proceeding.

---

## 1) Start Minikube (minimum resources)

Open a terminal (WSL shell on Windows, or Terminal on macOS). Make sure Docker Desktop (or your Docker daemon) is running first.

**Start Minikube with at least 2 CPUs and 4GB RAM:**

```bash
# start a dedicated profile called `backstage` (works on Docker driver)
minikube start -p backstage --driver=docker --cpus=2 --memory=4096
```

If you prefer another driver on macOS (e.g. `hyperkit`) substitute `--driver=hyperkit`.

**Verify the cluster is healthy:**

```bash
minikube -p backstage status
kubectl cluster-info
kubectl get nodes
```

You should see `minikube` (or the node) ready and `kubectl cluster-info` should report the control plane. If these commands show healthy/ready, continue.

---

## 2) Prepare the GitHub credentials

Backstage needs two things from GitHub:

1. A **Personal Access Token (PAT)** — either classic or fine-grained (used by Backstage for API access / integrations).
2. An **OAuth App** (Client ID + Client Secret) — used for "Sign in with GitHub".

**High-level steps (create these in your GitHub account):**

* PAT: Create a token and copy it (DO NOT commit it). Minimum scopes: `read:user` / `user:email`. Add `repo` scope if you need private-repo access. Use fine-grained token if you prefer.

* OAuth App: GitHub → Settings → Developer settings → OAuth Apps → New OAuth App

  * **Application name:** `backstage-local` (or similar)
  * **Homepage URL:** `http://localhost:7007`
  * **Authorization callback URL:** `http://localhost:7007/api/auth/github/handler/frame`

    * If your Backstage base URL differs, replace `http://localhost:7007` with your base URL.
  * Save and copy **Client ID** and **Client Secret**.

> Keep the PAT, Client ID and Client Secret private. We will load them into Kubernetes Secrets (not into version control).

---

## 3) Create Kubernetes namespace + secrets

```bash
kubectl create namespace backstage

# create a secret that holds the GitHub token and OAuth credentials
kubectl -n backstage create secret generic backstage-github \
  --from-literal=github-token="<YOUR_GITHUB_PAT>" \
  --from-literal=github-client-id="<YOUR_GITHUB_CLIENT_ID>" \
  --from-literal=github-client-secret="<YOUR_GITHUB_CLIENT_SECRET>"
```

Replace `<...>` with your actual values. The secret name `backstage-github` is used in examples below; change it if you prefer.

---

## 4) Update Backstage configuration (manifest / `app-config.yaml`)

Edit your `backstage.yaml` or `app-config.yaml` before applying. Replace inline secrets with **environment variable references** (we’ll inject these from Kubernetes Secrets into the pod).

**Example snippet (insert/replace where appropriate in your Backstage config):**

```yaml
integrations:
  github:
    - host: github.com
      token: ${GITHUB_TOKEN}

auth:
  environment: development
  allowGuestAccess: true
  providers:
    github:
      development:
        clientId: ${GITHUB_CLIENT_ID}
        clientSecret: ${GITHUB_CLIENT_SECRET}
        signIn:
          resolvers:
            - resolver: usernameMatchingUserEntityName
              dangerouslyAllowSignInWithoutUserInCatalog: true

enabled:
  github: true
```

**Important:** ` ${GITHUB_TOKEN}`, `${GITHUB_CLIENT_ID}`, and `${GITHUB_CLIENT_SECRET}` are environment variable placeholders. We'll inject them into the Backstage deployment from the Kubernetes Secret.

**Example deployment env injection (deployment or pod spec snippet):**

```yaml
env:
  - name: GITHUB_TOKEN
    valueFrom:
      secretKeyRef:
        name: backstage-github
        key: github-token
  - name: GITHUB_CLIENT_ID
    valueFrom:
      secretKeyRef:
        name: backstage-github
        key: github-client-id
  - name: GITHUB_CLIENT_SECRET
    valueFrom:
      secretKeyRef:
        name: backstage-github
        key: github-client-secret
```

Add those `env:` entries to the Backstage backend deployment in your `backstage.yaml` so the variables are available to the process.

---

## 5) Apply the manifest(s)

```bash
# apply Backstage manifest
kubectl apply -f backstage.yaml -n backstage
```

Give it a minute for images to pull and pods to start.

---

## 6) Verify Backstage and Postgres pods are running

Run:

```bash
kubectl get pods -n backstage
kubectl get deploy -n backstage
```

**Example CLI output (sample):**

```
$ kubectl get pods -n backstage
NAME                                      READY   STATUS    RESTARTS   AGE
backstage-6b7c7d6f6b-abcde                1/1     Running   0          2m
postgres-0                                1/1     Running   0          2m

$ kubectl get deploy -n backstage
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
backstage   1/1     1            1           2m
postgres    1/1     1            1           2m
```

> The outputs above are examples showing *one replica* for each service (READY `1/1`). Your pod names will differ; the key is each important pod should show `READY 1/1` and `STATUS Running`.

If you see `ImagePullBackOff` or `CrashLoopBackOff`, check pod logs:

```bash
kubectl -n backstage logs <pod-name>
```

---

## 7) Port-forward Backstage to localhost:7007

Forward the Backstage service or deployment to your machine so you can open the UI in a browser:

```bash
# preferred: if you have a Service named `backstage`
kubectl -n backstage port-forward svc/backstage 7007:7007

# alternative: port-forward the backend deployment/pod
kubectl -n backstage port-forward deployment/backstage 7007:7007
```

Open your browser to: `http://localhost:7007`

You should see the Backstage UI and a "Sign in with GitHub" option (if OAuth is configured correctly).

---

## 8) Common troubleshooting notes

* **OAuth redirect mismatch**: make sure the **Authorization callback URL** in your GitHub OAuth app exactly matches the URL Backstage sends (for local dev typically `http://localhost:7007/api/auth/github/handler/frame`). Check the exact path in your Backstage backend logs.
* **Bad credentials**: if sign-in fails, double-check Client ID/Secret and the PAT. Make sure the Kubernetes Secret values are correct and pods have been restarted after secret changes.
* **Insufficient resources**: if starting the pod fails due to resources, increase `--cpus`/`--memory` on `minikube start`.
* **Pods stuck Pulling images**: ensure Docker Desktop is running and Minikube can reach the image registry.

---

## 9) Cleanup

```bash
# stop
minikube -p backstage stop

# delete cluster/profile
minikube -p backstage delete
```

---

## Security & best practices

* **Never** commit personal tokens or client secrets to git.
* For production, use a managed database (RDS, Cloud SQL) and external OAuth with proper redirect URIs and HTTPS.
* Use RBAC and limit token scopes to the minimum necessary.

---

## Next steps / optional

* Want this README to include a Helm example, Kustomize overlay, or a sample `Dockerfile` for Backstage? Tell me which and I’ll add it.

---

*End of README*
