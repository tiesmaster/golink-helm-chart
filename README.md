# golink: helm-chart

Helm chart for Tailscale's golink application

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
