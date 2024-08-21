# Developer Hub Template for Spring boot app + Azure Repos + Azure Devops (CI) + Argo (CD) + Service Mesh

This repository contains the Backstage Template used to create the Kubernetes resources needed to build/deploy a simple Spring Boot application and configures Azure Pipelines (CI) , Argo (Gitops) and also deploys ServiceMesh resources

## Introduction 
This template has been designed to work only with TAP demo (https://demo.redhat.com/catalog?item=babylon-catalog-prod/enterprise.redhat-tap-demo.prod&utm_source=webapp&utm_medium=share-link) it assumes all the infrastructure required are available.

## Azure Requirements :

  1) Azure devops Account and Org
  2) Create a Azure Project with name : "Developer Hub Demo"
  3) Personal Access Token with following 
       ``` 
            Build - Read and Execute
            Code - Full
            Packaging - Read ,Write, & Manage
            Pipeline Resources - Use and Manage
            Pull Request Threads - Read & write
            Release - Read
        ```
   4) Create a NPM registry in Azure Artifacts by `create feed` and name it as `mynpmregistry`

## TAP Demo Changes : 

   1) Upload Dynamic plugin to Azure Artificatory
   
      a) Create .npmrc file from sample file provided from `./TAPDemo-artificats/sample .npmrc file` under home directory. Please make sure you replace all the <<>> with correct values
	    *Note:  Use base64 encoding for password values, an example is: https://www.base64encode.org/ and leave everything as default, copy paste output for azure PAT*
    
      b) Execute the command below to push to registry
 
        ```
            npm publish parfuemerie-douglas-scaffolder-backend-module-azure-pipelines-dynamic-1.2.0.tgz
        ```
2) Add New Dynamic plugin to create Azure Pipeline to `backstage-rhtap-values.yaml` under the `janus-idp-gitops` repo

    This sample file can be gotten from `./TAPDemo-artificats/plugin-add.yaml`
       
    This should be placed in the `backstage-rhtap-values.yaml` under global -> dynamic -> plugins -> package. Entering at the bottom should occur around line 70. You will need to replace/add the SHA integrity line at the top of the newly added content. look for the << SHA Integrity>>	
    
```
# Azure Pipeline
	- package: '@parfuemerie-douglas/scaffolder-backend-module-azure-pipelines-dynamic@1.2.0'
	integrity: '<<SHA Integrity >>'
```

3) Fix the values in the following files     
        ``` 
            1) ./TAPDemo-artificats/dynamic-plugins-npmrc.yaml
            2) ./TAPDemo-artificats/azure-rhdh-secret.yaml
    
    Section below to edit and add base64 encoded values:
    
```
data:
  AZURE_ORG_NAME: <<BASE64 Encoded Azure Org Name>>
  AZURE_PERSONAL_ACCESS_TOKEN: <<BASE64 Encoded Personal Access Token>>
```
    
4) Copy the above fixed files under directory path `janus-idp-gitops/charts/backstage/templates` in repo `janus-idp-gitops` in gitlab of TAP Demo.
    
    ![[Pasted image 20240806133019.png]]
    
5) Update the following env variables in  `backstage-rhtap-values.yaml`. These variables needs to be added under following path in yaml `upstream->backstage->extraEnvVars`
    
    ```
      - name: AZURE_PERSONAL_ACCESS_TOKEN
        valueFrom:
          secretKeyRef:
            key: AZURE_PERSONAL_ACCESS_TOKEN
            name: azure-rhdh-secret
      - name: AZURE_ORG_NAME
        valueFrom:
          secretKeyRef:
            key: AZURE_ORG_NAME
            name: azure-rhdh-secret  
    
    ```

6) Update `backstage-rhtap-values.yaml` one more time under `integrations:` and add the following lines:
```
azure:
- host: dev.azure.com
	credentials:
		- personalAccessToken: ${AZURE_PERSONAL_ACCESS_TOKEN}
			organizations:
				- ${AZURE_ORG_NAME}
```

7) Now you need to force refresh the openshift ArgoCD. After navigating to the Openshift ArgoCD you need to click on the Backstage box.
8) Click the tiny arrow on the Refresh button, and it should open drop down menu and allow "hard Refresh" to be selected.
![[Pasted image 20240806134136.png]]

9) To check it deploying you can open your Openshift console and navigate to workloads 
    -> Pods and select the backstage namespace. You should see new Backstage pods init. Once deployed you can go into the pod terminal and check the env vars with grep limiting to AZURE. This should return the env vars.

10) Connecting Azure repo to ArgoCD is the last item to finish the connections. Go to your Janus ArgoCD (note not the same argoCD above as TAP has two ArgoCDs). 
    - Click the Gear icons on the left menu
    - Select Repositories
    - Select `Connect Repo using HTTPS` button and fill in the values below:
	    - Type - Git
	    - Project - Janus
	    - Repository URL - You Devhub URL ex. ``https://<<AzureOrgName>>@dev.azure.com/<<AzureOrgName>>``
	    - Username - AzureOrgName
	    - Password - Azure PAT 
	    - Leave the rest empty and click the `Save as Credentials Template` button at the top of the window.

## Registering Template.

This new template can be registered on developer hub instance with Register existing component from create section. The URL should be ([https://github.com/arunhari82/azure-developer-hub-template/blob/main/templates/github/spring-boot-backend/template.yaml](https://github.com/arunhari82/azure-developer-hub-template/blob/main/templates/github/spring-boot-backend/template.yaml))

1)  Navigate to you Red Hat Developer Hub then catalog.
2) Click create in the top right
3) Click Register Existing Component 
4) Paste the Select Url:  ([https://github.com/arunhari82/azure-developer-hub-template/blob/main/templates/github/spring-boot-backend/template.yaml](https://github.com/arunhari82/azure-developer-hub-template/blob/main/templates/github/spring-boot-backend/template.yaml)) then click analyze.
5) Once complete navigate back to the Catalog
6) Click create in the top right again
7) You should see a new template labeled `Azure Devops Spring Boot with Service Mesh Template` then click the choose button in the lower right of that box.
   ![[Pasted image 20240806135055.png]]
   
8) The next screen you will need to fill out a few fields. 
    1.  Name - This should be unique 
    2. Azure Org Name 
    3. Azure Repo Name - This should be unique and cannot overwrite a previous repo.
    4. Git Owner - User1:User1
    5. Backstage System Entity - This can be any selection as it does not have any effects.
    6. Click Next

9)   This Screen only has one value to check, the Artifact ID. This needs to match what was entered on the previous screen for the Azure Repo Name. Then click next.
      
10) On this screen leave everything default and click next.
11) Click create.
12) You should see this after each part completes. You also will notice App in your Janus ArgoCD as well.
    ![[Pasted image 20240806141842.png]]