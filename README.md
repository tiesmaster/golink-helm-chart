# golink: helm-chart

Helm chart for Tailscale's golink application

**This chart is not maintained by the [upstream project](https://github.com/tailscale/golink/) and any issues with the chart should be raised [here](https://github.com/tiesmaster/golink-helm-chart/issues/new)**

## Source Code

* <https://github.com/tailscale/golink>

## Requirements

- Helm: 3.8+ (due to OCI registry support)

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

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| affinity | object | `{}` | Assign custom [affinity] rules to the deployment |
| config.authKey.enabled | bool | `false` | Register in the tailnet via the provided TS authkey. If disabled, retrieve the login URL from the container logs. |
| config.authKey.existingSecretName | string | `""` | Or reference an existing secret. Mutually exclusive with key |
| config.authKey.key | string | `""` | Provide the authkey directly, eg. 'tskey-auth-...-...'. Mutually exclusive with existingSecretName |
| config.authKey.keyName | string | `"golink-authkey"` | default: golink-authkey |
| config.hostname | string | `""` | Changes the service name from the default 'go' that golink uses to register itself in the tailnet |
| config.https | bool | `true` | Enable Tailscale's HTTPS support, requires this to be enabled for your tailnet |
| config.verbose | bool | `true` | Enable verbose output logging in the console |
| fullnameOverride | string | `""` | Provide a name to substitute for the full names of resources |
| image.pullPolicy | string | `"IfNotPresent"` | golink image pull policy. One of `Always`, `Never`, `IfNotPresent` |
| image.repository | string | `"ghcr.io/tailscale/golink"` | golink container image name |
| image.tag | string | `""` | golink container image tag |
| imagePullSecrets | list | `[]` | Reference to one or more secrets to be used when pulling images ref: https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/ |
| nameOverride | string | `""` | Provide a name in place of golink for `app:` labels |
| nodeSelector | object | `{}` | [Node selector] for pod assignment |
| persistence.accessModes | list | `["ReadWriteOnce"]` | Persistence access modes |
| persistence.annotations | object | `{}` | Annotations for the persistent volume claim |
| persistence.enabled | bool | `true` | Use persistent volume to store data |
| persistence.extraPvcLabels | object | `{}` | Extra labels to add to the persistent volume claim |
| persistence.selectorLabels | object | `{}` | Selector labels for the persistent volume claim |
| persistence.size | string | `"100Mi"` | Size of persistent volume claim |
| persistence.storageClass | string | `""` | Storage Class to use for the PVC |
| podAnnotations | object | `{}` | Annotations for golink pod |
| podLabels | object | `{}` | Labels for golink pod |
| podSecurityContext | object | `{"fsGroup":65532,"runAsGroup":65532,"runAsUser":65532}` | SecurityContext holds pod-level security attributes and common container settings. This defaults to non root user with uid 65532 and gid 65532 ref: https://kubernetes.io/docs/tasks/configure-pod-container/security-context/ |
| prometheusRule.additionalLabels | object | `{}` | Additional labels to add to the PrometheusRule |
| prometheusRule.enabled | bool | `false` | Enable deploying a PrometheusRule to alert on golink going down |
| resources | object | `{}` | Set container requests and limits for different resources like CPU or memory |
| securityContext | object | `{"capabilities":{"drop":["ALL"]},"readOnlyRootFilesystem":true,"runAsNonRoot":true}` | Container-specific security context configuration ref: https://kubernetes.io/docs/tasks/configure-pod-container/security-context/ |
| serviceAccount | object | `{"annotations":{},"automount":true,"create":true,"name":""}` | Service account for upgrade crd job to use. |
| tolerations | list | `[]` | [Tolerations] for pod assignment |

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

----------------------------------------------
Autogenerated from chart metadata using [helm-docs v1.14.2](https://github.com/norwoodj/helm-docs/releases/v1.14.2)
