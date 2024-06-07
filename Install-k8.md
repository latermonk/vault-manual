# k8s install

https://developer.hashicorp.com/vault/tutorials/kubernetes/kubernetes-sidecar      

https://killercoda.com/playgrounds/scenario/kubernetes       

1. clone demo repo     

```bash
git clone https://github.com/hashicorp-education/learn-vault-kubernetes-sidecar.git
```



```bash
cd learn-vault-kubernetes-sidecar
```



2.  install vault using helm   

   ```bash
   helm repo add hashicorp https://helm.releases.hashicorp.com
   ```

   ```bash
   helm repo update
   ```

   ```bash
   helm install vault hashicorp/vault --set "server.dev.enabled=true"
   ```

   

3. Configure  vault 

   Start an interactive shell session on the `vault-0` pod.      

   ```bash
   kubectl exec -it vault-0 -- /bin/sh
   ```

   Enable kv-v2 secrets at the path `internal`  

   ```bash
   vault secrets enable -path=internal kv-v2
   ```

   Create a secret at path `internal/database/config` with a `username` and `password`.    

   ```bash
   vault kv put internal/database/config username="db-readonly-username" password="db-secret-password"
   ```

   Verify that the secret is defined at the path `internal/database/config`   

   ```bash
   vault kv get internal/database/config
   ```

   

4. ## Configure Kubernetes authentication   

   ```
   kubectl exec -it vault-0 -- /bin/sh
   ```

   Enable the Kubernetes authentication method

   ```bash
   vault auth enable kubernetes
   ```

   Configure the Kubernetes authentication method to use the location of the Kubernetes API.    

   ```bash
   vault write auth/kubernetes/config \
         kubernetes_host="https://$KUBERNETES_PORT_443_TCP_ADDR:443"
   ```

5. Create policy and role

   ```bash
   vault policy write internal-app - <<EOF
   path "internal/data/database/config" {
      capabilities = ["read"]
   }
   EOF
   ```

   ```bash
   vault write auth/kubernetes/role/internal-app \
         bound_service_account_names=internal-app \
         bound_service_account_namespaces=default \
         policies=internal-app \
         ttl=24h
   ```

6. exit

   ```bash
   exit
   ```

7. Test 

   ```bash
   kubectl create sa internal-app
   ```

   ```bash
   kubectl apply --filename pod-payroll.yaml
   ```

   ```bash
   kubectl exec \
         payroll \
         --container payroll -- cat /vault/secrets/database-config.txt
   ```

   
