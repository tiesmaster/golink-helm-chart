# Update README.md

```sh
helm-docs .
```

# How to release?

 1. Set a valid `GITHUB_TOKEN` with `write:package` scope
 2. Bump the version in `Chart.yaml`
 3. Run the following commands from the terminal:

```sh
helm package .
echo $GITHUB_TOKEN | helm registry login ghcr.io -u tiesmaster --password-stdin
helm push golink-$(yq .version Chart.yaml).tgz oci://ghcr.io/tiesmaster
```
