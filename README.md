# golink: helm-chart

Helm chart for Tailscale's golink application

**This chart is not maintained by the [upstream project](https://github.com/tailscale/golink/) and any issues with the chart should be raised [here](https://github.com/tiesmaster/golink-helm-chart/issues/new)**

## Source Code

 * app: <https://github.com/tailscale/golink/>
 * helm chart: <https://github.com/tiesmaster/golink-helm-chart>

## Requirements

- Helm: 3.8+ (due to OCI registry support)
- Kubernetes: `>=1.18-0`

## Installing the Chart

To install the chart with the release name `golink`

```sh
helm install golink oci://ghcr.io/tiesmaster/golink
```

## Uninstalling the Chart

To uninstall the `golink` deployment

```sh
helm uninstall golink
```

The command removes all the Kubernetes components associated with the chart **including persistent volumes** and deletes the release. Depending on your storage class, your PV might be retained.

## Configuration

Read through the [values.yaml] file. It has several commented out suggested values.

Specify each parameter using the `--set key=value[,key=value]` argument to `helm install`.

```sh
helm install golink \
  --set config.authKey='tskey-auth-...-...' \
  oci://ghcr.io/tiesmaster/golink
```

Alternatively, a YAML file that specifies the values for the above parameters can be provided while installing the chart.

```console
helm install unifi oci://ghcr.io/tiesmaster/golink -f values.yaml
```
## Values

TBD

# Restore from backup

_If you want to restore the `golink.db` from backup, you can follow the following steps. As the
golink container doesn't have a shell, so `kubectl cp` doesn't work._

Retrieve the golink.db Save the pod spec from below to `golink-restorefrombackup`, and execute the following

```sh
kubectl scale --replicas=0 deploy/golink
kubectl apply -f golink-restorefrombackup.yaml
kubectl wait --for=condition=ready pod/golink-restorefrombackup
kubectl cp golink.db golink-restorefrombackup:/home/nonroot/golink.db
kubectl delete --grace-period=0 --force -f golink-restorefrombackup.yaml
kubectl scale --replicas=1 deploy/golink
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: golink-restorefrombackup
spec:
  containers:
    - name: golink-restorefrombackup
      image: bash
      resources: {}
      command: ['bash', '-c', 'while true; do echo "Sleeping for 1 sec ($(date))"; sleep 1; done']
      volumeMounts:
        - name: data-volume
          mountPath: /home/nonroot
  volumes:
    - name: data-volume
      persistentVolumeClaim:
        claimName: golink
  securityContext:
    # This is needed as its a "non-root" image, and otherwise /home/nonroot cannot be written to
    # https://github.com/tailscale/golink#running-in-production
    # https://github.com/tailscale/golink/issues/6
    # https://github.com/tailscale/golink/pull/12
    runAsUser: 65532
    runAsGroup: 65532
    fsGroup: 65532
```
