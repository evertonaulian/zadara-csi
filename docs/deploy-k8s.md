## Deploying zadara-csi plugin on Kubernetes

### Choose your version

Directory `deploy` contains YAML files organized by K8s versions.

For each version, we provide two options of plugin deployment with different approaches to manage iSCSI connectivity
(see [Node iSCSI Connectivity](README.md#node-iscsi-connectivity) section),
 available in `deploy/<k8s-version-dir>/rootfs` and `deploy/<k8s-version-dir>/client-server`.

For more convenience, we suggest creating a symlink to a directory of your choice:
```shell script
ln -sfT  <k8s-version-dir>/<rootfs|client-server>  deploy/current
# Example:
ln -sfT  k8s-1.16/client-server  deploy/current
```
Or to skip Node iSCSI Connectivity part and choose default:
```shell script
ln -sfT  <k8s-version-dir>/  deploy/current
# Example:
ln -sfT  k8s-1.16  deploy/current
```

---

### Configuration

#### Secrets management

Before deploying Zadara-CSI plugin, edit `deploy/current/secrets.yaml` and set VPSA credentials.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: vpsa-access-token
  namespace: kube-system
stringData:
  access-token: "FAKETOKEN1234567-123"
```

- Get access token from VPSA, and paste into `access-token`.
- Run `kubectl create -f deploy/current/secrets.yaml`. This will create a Secret object in Kubernetes.
- To verify, run `kubectl --namespace kube-system get secrets`, `vpsa-access-token` should appear in output.
- Remove `secrets.yaml`, to keep your secrets safe.

#### Plugin arguments

Edit following parameters in `controller.yaml` and `node.yaml`.
In most cases, only `hostname`, and `secure` need to be changed.

| parameter | description | required | examples |
|-----------|-----------|-----------|----------|
|`name` | Plugin name, to identify plugin instance and use in `provisioner` field of StorageClass  | No. Defaults to `csi.zadara.com` | `us-west.csi.zadara.com`, `on-prem.csi.zadara.com`, `all-flash.csi.zadara.com`
|`nodeid` | Kubernetes Node id | No. Defaults to hostname of a current node, as returned by `uname -n` | `VM-001`, `node42`
| `endpoint` | gRPC UNIX socket | No. Defaults to `unix://tmp/csi.sock` | `unix://csi/csi.sock`
| `hostname` | VPSA hostname, or IP  | Yes | `example.zadaravpsa.com`, `10.0.10.1`
| `token` | API access key | Yes, by default is taken from [secrets](#secrets-management) | `FAKETOKEN1234567-123`
| `secure` | Whether or not to use HTTPS | No. Defaults to `true` | `true`, `false`, `0`, `1`. <br>Pass as `--secure=false`
| `config` | Path to dynamic [config](README.md#extended-configuration) | No. Defaults to `/etc/csi/zadara-csi-config.yaml` | `etc/csi-config.yaml`

---

### Deployment

1. Create [secrets](#secrets-management)
2. Update [plugin arguments](#plugin-arguments)
3. Execute:
    ```
    kubectl create -f deploy/current/csi-driver.yaml
    kubectl create -f deploy/current/node.yaml
    kubectl create -f deploy/current/controller.yaml
    ```
