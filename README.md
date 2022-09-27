# HashicopVault-MariaDb

Deploy MariaDB in any Kubernetes infrastructure and integrate it with the hashicorp vault for managing the database credential.

1. Deploy hashicorp vault(consider 2 replication)

step1: Clone the Helm Chart for the HashiCorp Vault
           git clone https://github.com/hashicorp/vault-helm.git

step2: Edit the values.yaml file to access the Vault UI
          cd vault-helm
          -->Open the values.yaml files in an editor.
          -->Then find the “service” in the file and uncomment “nodePort” in the file
          -->Then find “Vault UI” in the file and enable ui.
          --> Make the value “true” for “activeVaultPodOnly”
          --> Change the “serviceType” to “NodePort”.
          --> Give the same port number in “serviceNodePort” used above. i.e 30000 and save the file

step3: Deploying the Vault.
           -->First Create a namespace for vault
           kubectl create namespace vault
           -->Add hashicorp repository in helm and check the chard is added successfully
           helm repo add hashicorp https://helm.releases.hashicorp.com
           -->Install the specific version in the vault namespace with values.yaml file
           helm install -f values.yaml vault hashicorp/vault --namespace vault --version 0.14.0
           -->Check the pods, services and other components.
           kubectl get po -n vault
           kubectl get svc -n vault

step4: Unsealing the Vault. (Important step)
           To unseal the vault, we need to initialise the vault operator that will provide some Unseal keys that is going to be used to unseal the vault. Save the Unseal keys and Initial Root Token in a file for later use.
           kubectl exec -ti vault-0 -n vault -- vault operator init

step5: Login in the Vault UI on Exposed Port
         -->now we can login vault UI on 30000 port.

2. Deploy MariaDB in Kubernetes(consider 3 replication)

step1: For deploying the MariaDB helm repo using https://github.com/helm/charts.git
          helm repo add bitnami https://charts.bitnami.com/bitnami
          helm install --name my-release bitnami/<chart>  
          -->To update an exisiting stable deployment with a chart hosted in the bitnami repository you can execute (Note:Edit the slave replicas to 3 in values.yml)
step2:helm repo add bitnami https://charts.bitnami.com/bitnami
          helm upgrade my-release bitnami/<chart>
          helm install my-release stable/mariadb

step2: To install the chart with the release name my-release

step3: Edit the slave replicas to 3 in values.yml

step4:helm install my-release stable/mariadb for installing the chart

3. Integrate the DB with vault to store database credential.

step1: Enable the database secrets engine run the command "vault secrets enable database"

step2:Configure Vault with the proper plugin and connection information using the command 
          vault write database/config/my-mysql-database \
    	plugin_name=mysql-database-plugin \
   	connection_url="{{username}}:{{password}}@tcp(127.0.0.1:3306)/" \
    	allowed_roles="my-role" \
   	username="vaultuser" \
   	password="vaultpass"


step3: Configure a role that maps a name in Vault to an SQL statement to execute to create the database credential
           vault write database/roles/my-role \
    	db_name=my-mysql-database \
    	creation_statements="CREATE USER '{{name}}'@'%' IDENTIFIED BY '{{password}}';GRANT SELECT ON *.* TO '{{name}}'@'%';" \
  	default_ttl="1h" \
   	max_ttl="24h"





REFERENCES

https://github.com/helm/charts/blob/master/stable/mariadb/

https://dwops.com/blog/deploy-hashicorp-vault-on-kubernetes-using-helm/

https://www.strongdm.com/connect/mariadb-hashicorp-vault

https://spring.io/blog/2016/08/15/managing-your-database-secrets-with-vault
