# Encrypting Secret Data at Rest
- I have used `Kubeadm` based Cluster
- Version - 1.19
- etcd v3.0 or later is required

### Create `Secret` and Retreive `plain-text` `Secrets` from `ETCD`
<details><summary>show</summary>
<p>

1. Create a new secret called `secretpassword`  in the default namespace with `password=s3cR3t!` data:
```
kubectl create secret generic secretpassword --from-literal=password=s3cR3t!
```

2. Using the `etcdctl` command line, read that `secret` out of `etcd`:
```
ETCDCTL_API=3 etcdctl get /registry/secrets/default/secretpassword \
--cacert /etc/kubernetes/pki/etcd/ca.crt \
--cert /etc/kubernetes/pki/etcd/server.crt \
--key /etc/kubernetes/pki/etcd/server.key 
```
![un-encrypted](https://user-images.githubusercontent.com/28925814/100071395-05444e00-2e61-11eb-81e5-2f2eeefd5c5b.png)

   - Pipe the above command with `hexdump -C`
```
ETCDCTL_API=3 etcdctl get /registry/secrets/default/secretpassword \
--cacert /etc/kubernetes/pki/etcd/ca.crt \
--cert /etc/kubernetes/pki/etcd/server.crt \
--key /etc/kubernetes/pki/etcd/server.key | hexdump -C
```
![un-encrypted-hexdump](https://user-images.githubusercontent.com/28925814/100071459-17be8780-2e61-11eb-9e5f-68ec75d3fbf8.png)

> In both these images, we can see that the secret data is saved as `plain` text. Anyone with access to `etcd` can query and get the data.

</p>
</details>

### Encrypting `Secrets` in `ETCD`
<details><summary>show</summary>
<p>

1. Generate a 32-byte random key and base64 encode it. 
```
head -c 32 /dev/urandom | base64
```
2. Create a new encryption config file and replace the `<BASE 64 ENCODED SECRET>` with the previous step output:
```yaml
#saving this YAML in /etc/kubernetes/pki/encrypt-secrets.yml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
    - secrets
    providers:
    - aescbc:
        keys:
        - name: key1
          secret: <BASE 64 ENCODED SECRET>
    - identity: {}
```
3. Set the `--encryption-provider-config` flag on the `kube-apiserver` to point to the location of the config file.
![kube-apiserver](https://user-images.githubusercontent.com/28925814/100076227-f6f93080-2e66-11eb-847c-f2cd9ba00888.png)

4. Restart your API server. In Kubeadm based cluster saving changes to `/etc/kubernetes/manifests/kube-apiserver.yml` will restart the `kube-apiserver`
> Caution: Your config file contains keys that can decrypt the content in etcd, so you must properly restrict permissions on your masters so only the user who runs the kube-apiserver can read it.

</p>
</details>

### Verifying Encrypted `Secrets` 
<details><summary>show</summary>
<p>

1. After the `kube-apiserver` gets restarted, any newly created `secret` will be encrypted.
2. Data is encrypted when written to etcd. So any previously created `secrets` are still in `plain-text`
3. Performing an update on the existing `secret` will encrypt that content.
```
kubectl get secrets --all-namespaces -o json | kubectl replace -f -
```
4. Using the `etcdctl` command line, read that `secret` out of `etcd`:
```
ETCDCTL_API=3 etcdctl get /registry/secrets/default/secretpassword \
--cacert /etc/kubernetes/pki/etcd/ca.crt \
--cert /etc/kubernetes/pki/etcd/server.crt \
--key /etc/kubernetes/pki/etcd/server.key 
```
![encrypted-secret](https://user-images.githubusercontent.com/28925814/100077317-3a07d380-2e68-11eb-93d3-60c81158d7c4.png)

   - Pipe the above command with `hexdump -C`
```
ETCDCTL_API=3 etcdctl get /registry/secrets/default/secretpassword \
--cacert /etc/kubernetes/pki/etcd/ca.crt \
--cert /etc/kubernetes/pki/etcd/server.crt \
--key /etc/kubernetes/pki/etcd/server.key | hexdump -C
```
![encrypted-secret-hexdump](https://user-images.githubusercontent.com/28925814/100077308-37a57980-2e68-11eb-805c-22691d0c56c2.png)

As seen in the above images, the `secret` is encrypted in `etcd`.

</p>
</details>