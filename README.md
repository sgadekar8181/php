# OpenShift CLI Setup, Token-Based Authentication, and Source-to-Image (S2I) Deployment of a PHP Application

This repository documents the steps for:
1. Installing the OpenShift CLI (`oc`) and logging in using an API token.
2. Deploying a PHP application from GitHub using OpenShift's Source-to-Image (S2I) build strategy.

---

## 📌 Prerequisites

- Access to an OpenShift cluster (e.g., Red Hat OpenShift Service on AWS - ROSA)
- A Linux terminal with `curl` and `tar` installed
- A GitHub repository containing your application source code (PHP app used in this example)

---

## 🛠️ Task 1: Installing the OpenShift CLI (`oc`) and Logging In Using an API Token

### Step 1 — Download the OpenShift CLI archive
```bash
curl -LO https://mirror.openshift.com/pub/openshift-v4/clients/ocp/stable/openshift-client-linux.tar.gz
```

### Step 2 — Extract the archive
```bash
tar -xzf openshift-client-linux.tar.gz
```

### Step 3 — Move the `oc` binary to your system path
> `kubectl` is usually already installed; this step only adds/updates `oc`.
```bash
sudo mv oc /usr/local/bin/
```

### Step 4 — Clean up downloaded files
```bash
rm openshift-client-linux.tar.gz README.md
```

### Step 5 — Get a login token from the OpenShift web console
1. Log in to the OpenShift web console.
2. Click your username in the top-right corner.
3. Select **Copy login command**.

### Step 6 — Copy the token and login command
The console displays your API token and a ready-to-use login command:
```bash
oc login --token=sha256~<token> --server=https://api.<cluster-domain>:6443
```

### Step 7 — Authenticate from the CLI
Paste the copied command into your terminal:
```bash
oc login --token=<your-token> --server=https://api.<cluster-domain>:6443
```

### Step 8 — Verify access
A successful login confirms identity and lists accessible projects:
```text
Logged into "https://api.<cluster-domain>:6443" as "<username>" using the token provided.

You have access to the following projects and can switch between them with 'oc project <projectname>':
    openshift-virtualization-os-images
    sachin8181-dev
    sandbox-shared-models
```

---

## 🚀 Task 2: Source-to-Image (S2I) Build — Deploying a PHP App from GitHub

### Step 1 — Switch to the target project
```bash
oc project sachin8181-dev
```

### Step 2 — Create a new app directly from the Git repository
OpenShift detects the PHP source and automatically builds it using the S2I builder image.
```bash
oc new-app https://github.com/sgadekar8181/php.git --name=php-app
```
This creates an **image stream**, **build config**, **deployment**, and **service** for `php-app`.

### Step 3 — Monitor the build
```bash
oc logs -f buildconfig/php-app
```

### Step 4 — Check pod status
```bash
oc get pods
```

### Step 5 — Check the created service
```bash
oc get svc
```

### Step 6 — Expose the service as a route
```bash
oc expose svc php-app
```

### Step 7 — Get the public route URL
```bash
oc get route
```

### Step 8 — Verify the app in a browser
Open the route hostname from the previous step. You should see the deployed PHP app output.

> ⚠️ **Troubleshooting:** If the build fails with:
> ```text
> mv: cannot overwrite './.github': Directory not empty
> error: build error: building at STEP "RUN /usr/libexec/s2i/assemble": exit status 1
> ```
> This happens because S2I tries to copy the entire repo — including the `.github` folder — into the container.
> **Fix:** Add a `.s2ignore` file to the root of your repository, listing folders/files to exclude:
> ```text
> .github
> ```

---

## 🔄 Rebuilding After a Code Change

### Step 9 — Check existing build configs
```bash
oc get buildconfig
```

### Step 10 — Update the code in the Git repository
Edit and commit your changes (e.g., `index.php`) directly in GitHub.

### Step 11 — Trigger a new build manually
```bash
oc start-build php-app
```

### Step 12 — Watch the new build/pod progress
```bash
oc get pods -w
```

### Step 13 — View the new build logs
```bash
oc logs -f php-app-3-build
```
A successful rebuild ends with:
```text
Push successful
```

### Step 14 — Refresh the app in your browser
The existing route automatically serves the updated code — no need to re-expose the service.

---

## 📂 Useful Commands Reference

| Command | Purpose |
|---|---|
| `oc login --token=<token> --server=<url>` | Log in using an API token |
| `oc project <name>` | Switch to a project/namespace |
| `oc new-app <git-url> --name=<app-name>` | Create a new app from a Git repo using S2I |
| `oc get pods` | List pods and their status |
| `oc get svc` | List services |
| `oc expose svc <name>` | Create a route to expose a service externally |
| `oc get route` | List routes and their URLs |
| `oc get buildconfig` | List build configurations |
| `oc start-build <name>` | Manually trigger a new build |
| `oc logs -f buildconfig/<name>` | Stream build logs |

---

#

---

## 📜 License

This documentation is for personal learning and reference purposes.
