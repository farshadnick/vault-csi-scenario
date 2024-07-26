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
vault auth enable Kubernetes
```

![image](https://github.com/user-attachments/assets/d0f06359-6cd8-4f0c-ab46-61fde601691e)

![image](https://github.com/user-attachments/assets/201940e1-2b9e-4d2a-8104-3c7d8e057210)

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
![image](https://github.com/user-attachments/assets/4706f92d-beb7-42de-990f-81e3b316e966)
