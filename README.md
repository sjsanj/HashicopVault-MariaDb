# HashicopVault-MariaDb

Deploy MariaDB in any Kubernetes infrastructure and integrate it with the hashicorp vault for managing the database credential.

1. Deploy hashicorp vault(consider 2 replication)

step1: we are using vault-hem repo for creating hashicrop vault on Kubernetes
step2: first clone the repo using git clone https://github.com/hashicorp/vault-helm.git 
step3: next edit the values.yaml file to access vault UI
step4: here we edit the service type to NodePort to access change the replicas and save the file
step5: for deploying the vault we created a namespace as the vault
step6: using the command helm install -f values.yaml vault hashicorp/vault --namespace vault --version 0.14.0 for deployment
step7: check the pods by using kubectl get po -n vault and services by using kubectl get svc -n vault
step8: get inside the pod by using kubectl exec -it for Unsealing the vault (Important) and store the keys in separate folder.
step9: now we can login vault UI on 30000 port.

2. Deploy MariaDB in Kubernetes(consider 3 replication)

step1: For deploying the MariaDB helm repo add bitnami https://charts.bitnami.com/bitnami
step2: for install in Helm 3 using helm install my-release bitnami/<chart>
step3:edit the slave replicas to 3 in values.yml
step4:helm install my-release stable/mariadb for installing the chart

3. Integrate the DB with vault to store database credential

step1: mount the mysql using the command vault mount mysql
step2: run the vault secrets enable database
step3: to configure vault with database plugin run the command vault write database/config/my-mysql-database \


REFERENCES

https://github.com/helm/charts/blob/master/stable/mariadb/

https://dwops.com/blog/deploy-hashicorp-vault-on-kubernetes-using-helm/

https://www.strongdm.com/connect/mariadb-hashicorp-vault

https://spring.io/blog/2016/08/15/managing-your-database-secrets-with-vault
