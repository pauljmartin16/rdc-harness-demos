# Vault Demo
- a replay of the RDC Vault demo. Create a 'Hello World' container and demonstrate Vault
    secrets are injected as env variables
- assumes Windows/Powershell. For Mac/Linux replace ` EOLs with \

# Pre-Requisites
```
Install kubectl CLI - https://kubernetes.io/docs/tasks/tools/
Install Helm CLI - https://helm.sh/docs/intro/install/
k8s Cheatsheet for reference - https://kubernetes.io/docs/reference/kubectl/cheatsheet/
```

# Install k8s Environment
- Cloud k8s preferable
- Local - may require Docker Desktop
    - k3d
    - minikube

# Initialize k8s Namespace
```
kubectl create namespace rdc-poc-ns
kubectl config set-context --current --namespace=rdc-poc-ns
```

# Install Helm v3 Operator
- Clone the vault-helm source from GIT
```
https://github.com/hashicorp/vault-helm.git
```

- Create the Vault k8s operator
```
helm install vault `
       --set='server.dev.enabled=true' `
       ./vault-helm
```

- Verify Vault installation
```
helm list
kubectl get pods
```

# Configure k8s Vault Access Policies + rdc-poc-role
- ssh into the Vault Pod
```
kubectl exec -it vault-0 -- /bin/sh
```

- Create a policy and apply to the rdc-poc-role
```
cat <<EOF > /home/vault/vault-read-only-policy.hcl
path "rdc-poc-app/data/kv/dev/*" {
  capabilities = ["read"]
}
EOF

vault policy write rdc-poc-role /home/vault/vault-read-only-policy.hcl
```

- Enable Vault Access in k8s
```
vault auth enable kubernetes

vault write auth/kubernetes/config \
issuer="https://kubernetes.default.svc.cluster.local" \
token_reviewer_jwt="$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" \
kubernetes_host=https://${KUBERNETES_PORT_443_TCP_ADDR}:443 \
kubernetes_ca_cert=@/var/run/secrets/kubernetes.io/serviceaccount/ca.crt \

vault write auth/kubernetes/role/rdc-poc-role \
   bound_service_account_names=rdc-poc-svc \
   bound_service_account_namespaces=rdc-poc-ns \
   policies=rdc-poc-role \
   ttl=1h
```

# Add Values to Vault Instance

- store some test values in $HOME/appsettings.json file
```
{
  "RATESDBSERVER": "k8s-rates-server",
  "RATESDBUSER": "k8s-rates-user",
  "RATESDBPWD": "k8s-rates-password",
  "LOCATORDBSERVER": "k8s-locator-server",
  "LOCATORDBUSER": "k8s-locator-user",
  "LOCATORDBPWD": "k8s-locator-password"
}
```

- create a filesecret in filesecret.json
```
{
  "content": "IyBPcGFxdWUgU2VjcmV0IEZpbGUgRXhhbXBsZQoKVGhpcyBpcyBhbiAib3BhcXVlIiBzZWNyZXQgc3RvcmVkIGFzIGEgZmlsZS4gSXQgd2lsbCBvZnRlbiBiZSBhIGJpbmFyeSwgbm9uLXRleHQKZmlsZS4KCkV4YW1wbGUgc2VjcmV0IGZpbGVzIG9mIHRoaXMgdHlwZSBpbmNsdWRlOgoqIFRMUyBwcml2YXRlIGtleXMKKiBTU0gga2V5cwoqIEtlcmJlcm9zIGtleXRhYiBmaWxlcwo="
}
```

- upload variables and file secret into Vault
```
vault secrets enable -version=2 -path="rdc-poc-app" kv

vault kv put rdc-poc-app/kv/dev/appsecret @appsecret.json

vault kv put rdc-poc-app/kv/dev/filesecret @filesecret.json

```

- verify values tored in Vault
```
vault kv get rdc-poc-app/kv/dev/appsecret
vault kv get rdc-poc-app/kv/dev/filesecret
```

- exit the Pod
```
exit
```

# Load the HelloWorld k8s Application

- load the k8s HelloWorld
```
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f serviceaccount.yaml
```

# Verify Vault Values Injected to HelloWorld Container
- get the Locator Pod and SSH into it
```
kubectl get pods
kubectl exec -it locator-poc-app-## YOUR POD INSTANCE ID ## -- /bin/sh 
```

- verify Vault secrets are loaded
```
cat /vault/secrets/secret.env
ls /vault/secrets/filesecret
```

- set the secrets as variables - simulating startup.sh
```
cd $HOME
vi .profile
```

- add the following to .profile:
```
secretFile="/vault/secrets/secret.env"

if [[ -f "$secretFile" ]]; then

    echo "Detected secret file $secretFile, loading values as environment variables"

    set -o allexport
    source $secretFile
    set +o allexport

fi
```
- execute the profile
```
source .profile
```

- validate the Vault variables are set
```
env | grep RATESDBSERVER
```

- exit the Pod
```
exit
```

# Test Complete
- tear down cloud k8s