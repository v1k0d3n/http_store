# http_store

```
########################################################################
# A PRESCRIPTIVE DEPLOYMENT OF HTTP STORE
########################################################################
```

## Dependencies
- Create and update required configuration files:
  - [`/opt/http_store/environment`](./environment)

- Requires [podman](https://podman.io/)
- Uses [ubi9/httpd-24](https://catalog.redhat.com/software/containers/ubi9/httpd-24) from registry.access.redhat.com (no login required)

## Start http_store (podman)

```
stop_http_store.sh
```

## Stop http_store

```
stop_http_store.sh
```

## OpenShift deployment

For running on OpenShift with a PVC, see [openshift/README.md](./openshift/README.md). The manifests include:
- PVC (configurable size)
- Pod (httpd serving from the PVC)
- Service and Route
- Instructions for uploading ISO files via `oc rsync` or `oc cp`

## References:


