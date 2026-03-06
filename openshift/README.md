# http_store on OpenShift

Deploy the HTTP store as a Pod on OpenShift to serve ISO files for Redfish endpoints and bare metal deployments.

## Quick start

```bash
# Create a project (or use existing)
oc new-project http-store

# Create the PVC and deploy
oc apply -f pvc.yaml
oc apply -f pod.yaml
oc apply -f service.yaml
oc apply -f route.yaml

# Wait for the pod to be Ready
oc wait --for=condition=Ready pod/http-store --timeout=120s
```

## Customizing before apply

Edit the YAML files before applying:

| File | What to change |
|------|----------------|
| `pvc.yaml` | `spec.resources.requests.storage` — PVC size (e.g., `50Gi`, `100Gi`) |
| `pod.yaml` | `image` — use `registry.redhat.io` if you prefer (requires `oc login` to registry) |
| `route.yaml` | `spec.host` — set a specific hostname, or omit for auto-generated |

## Uploading ISO files to the PVC

After the pod is running, copy files into `/var/www/html` (the httpd document root):

```bash
# Sync a local directory of ISO files into the pod
oc rsync ./path/to/isos http-store:/var/www/html/

# Or copy a single file
oc cp ./my-image.iso http-store:/var/www/html/my-image.iso

# Or use a running debug pod to copy from another source
oc run curl --rm -it --image=registry.access.redhat.com/ubi9/ubi-minimal --restart=Never -- \
  curl -o /tmp/file.iso https://example.com/image.iso
# Then oc cp from that pod if needed
```

The pod serves files from `/var/www/html` at the Route URL (e.g., `http://http-store-<project>.<cluster-domain>/`).

## OpenShift permissions

The pod runs as UID 1001 (httpd's default). If the pod fails to start with permission errors, grant the `anyuid` SCC to the default service account:

```bash
oc adm policy add-scc-to-user anyuid -z default -n <your-project>
```

## Image registry

The manifests use `registry.access.redhat.com/ubi9/httpd-24`, which does **not** require authentication. For `registry.redhat.io`, log in first:

```bash
oc create secret docker-registry redhat-registry \
  --docker-server=registry.redhat.io \
  --docker-username=<your-username> \
  --docker-password=<your-token> \
  -n <your-project>
```

Then add `imagePullSecrets` to the pod spec and change the image to `registry.redhat.io/ubi9/httpd-24:latest`.
