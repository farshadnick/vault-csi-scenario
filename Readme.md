![image](https://github.com/user-attachments/assets/eb233f90-2038-452b-aab4-ecd39a640aba)

# 1- Install SecretAccount for Vault
```
kubectl apply -f sa-valut.yaml
```
# 1-1 Verify 
```
kubectl get sa vault-auth-sa
kubectl get clusterrolebinding secret-reader-binding
kubectl describe clusterrolebinding secret-reader-binding
kubectl get secret vault-auth-secret
```

# 2- Edit Kubernetes Authentication method to Integrate vault to your kubernetes
## 2-1 install vault cli and enable kubernetes plugin for vault
```
export VAULT_ADDR=http://192.168.6.150:8200
# enter your Vault's Root Token
vault login  
vault auth enable kubernetes
```

# 2-2 we need kubernetes CA , Token and kubernetes API URL
## Kubernetes CA
```
cat /etc/kubernetes/pki/ca.crt
```
## Token
```
export K8S_TOKEN=$(k get secret vault-auth-secret -o jsonpath="{.data.token}" | base64 -d)
echo $K8S_TOKEN
```
## Vault Secret 
```
export K8S_SECRET=$(kubectl get secrets --output=json \
    | jq -r '.items[].metadata | select(.name|startswith("vault-auth-")).name')
echo $K8S_SECRET
```

![image](https://github.com/user-attachments/assets/d0f06359-6cd8-4f0c-ab46-61fde601691e)

## Kubernetes CA 
![image](https://github.com/user-attachments/assets/201940e1-2b9e-4d2a-8104-3c7d8e057210)
## TOKEN 
![image](https://github.com/user-attachments/assets/1f02139c-e096-4793-bc21-4ade6ba34e33)

![image](https://github.com/user-attachments/assets/55658981-875f-4aa1-a6a6-16e39bc74bf2)

# 3-Create a Secret in Vault
```
vault secrets enable -path=/secret kv
vault kv put secret/db-pass pwd="admin@123"
vault kv get secret/db-pass
```
# 3-1 Create Policy for Accessing to that secret
```
vault policy write internal-app - <<EOF
path "secret/db-pass" {
  capabilities = ["read"]
}
EOF
vault policy read internal-app
```
# 3-2 Connect Kubernetes to Vault by Vault-sa Service Account
```

vault write auth/kubernetes/role/database \
    bound_service_account_names=vault-auth-sa \
    bound_service_account_namespaces=default \
    policies=internal-app \
    ttl=20m

vault read auth/kubernetes/role/database
```
# 4- Install Vault CSI Driver 
```
# Install Secret store CSI driver
helm repo add secrets-store-csi-driver https://kubernetes-sigs.github.io/secrets-store-csi-driver/charts
helm upgrade -i csi secrets-store-csi-driver/secrets-store-csi-driver --set syncSecret.enabled=true \
    --set enableSecretRotation=true --set rotationPollInterval=30s --set syncSecret.enabled=true

k get ds
k api-resources | grep -i csi
k get csidriver
k get csinodes
k get po

# Install Valut CSI Provider
helm repo add hashicorp https://helm.releases.hashicorp.com
helm install vault hashicorp/vault \
    --set "server.enabled=false" \
    --set "injector.enabled=false" \
    --set "csi.enabled=true"

```
# 5- Create Secret Provider Class to point to your secret on vault 
```
kubectl apply -f spc-crd-vault.yaml
kubectl api-resources | grep -i secret
kubectl get secretproviderclasses
kubectl describe secretproviderclasses vault-database
```
# 6- Create pod that Mount to Your secret on Kubernetes
```
kubectl apply -f  webapp.yaml
```
![image](https://github.com/user-attachments/assets/4706f92d-beb7-42de-990f-81e3b316e966)
