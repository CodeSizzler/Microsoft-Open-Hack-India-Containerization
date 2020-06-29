<h1>Integrate AKS-managed Azure AD</h1>

<h2>Introduction</h2>
<p>Azure AD integration with AKS-managed Azure AD is designed to simplify the Azure AD integration experience, where users were previously required to create a client app, a server app, and required the Azure AD tenant to grant Directory Read permissions. In the new version, the AKS resource provider manages the client and server apps for you.</p>

<h2>Prerequisites</h2>
<p>To perform this demo, Locate your Azure Account tenant ID by navigating to the Azure portal and select Azure Active Directory > Properties > Directory ID.</p>

<h2>Demo</h2>
<p>1. You must have the following resources installed:</p>
	<li>The Azure CLI, version 2.5.1 or later</li>
	<li>The aks-preview 0.4.38 extension</li>
	<li>Kubectl with a minimum version of 1.18</li>
	<br>
<p>2. To install/update the aks-preview extension or later, use the following Azure CLI commands:</p>
	<b>az extension add --name aks-preview</b>
<img src="https://csgithub.blob.core.windows.net/lntergrate/Screenshot (3147).png"/>
<p>3. To install kubectl, use the following commands:</p>
	<b>kubectl version --client</b>
<img src="https://csgithub.blob.core.windows.net/lntergrate/Screenshot (3149).png"/>
<p>4. Use the below command for registering ACS</p>
	<b>az feature register --name AAD-V2 --namespace Microsoft.ContainerService</b>
<img src="https://csgithub.blob.core.windows.net/lntergrate/Screenshot (3150).png"/>
<p>5. It might take several minutes for the status to show as Registered. You can check the registration status by using the az feature list command:</p>
	<b>az feature list -o table --query "[?contains(name, 'Microsoft.ContainerService/AAD-V2')].{Name:name,State:properties.state}"</b>
<img src="https://csgithub.blob.core.windows.net/lntergrate/Screenshot (3151).png"/>
<img src="https://csgithub.blob.core.windows.net/lntergrate/Screenshot (3152).png"/>
<p>6. When the status shows as registered, refresh the registration of the Microsoft.ContainerService resource provider by using the az provider register command:</p>
	<b>az provider register --namespace Microsoft.ContainerService</b>
<img src="https://csgithub.blob.core.windows.net/lntergrate/Screenshot (3153).png"/>

<h2>Create an AKS cluster with Azure AD enabled</h2>
<p>1. Create an AKS cluster by using the following CLI commands.</p>
<p>2. Create an Azure resource group:</p>
	<b>az group create --name CodeSizzlerAKS-Rg --location CentralUS</b>
<img src="https://csgithub.blob.core.windows.net/lntergrate/Screenshot (3154).png"/>
<p>3. You can use an existing Azure AD group, or create a new one. You need the object ID for your Azure AD group.</p>
	<b>az ad group list</b>
<img src="https://csgithub.blob.core.windows.net/lntergrate/Screenshot (3155).png"/>
<p>4. To create a new Azure AD group for your cluster administrators, use the following command:</p>
	<b>az ad group create --display-name MyADCS --mail-nickname MyADCS</b>
<img src="https://csgithub.blob.core.windows.net/lntergrate/Screenshot (3156).png"/>
<p>5. Create an AKS cluster, and enable administration access for your Azure AD group</p>
	<b>az aks create -g CodeSizzlerAKS-RG -n CodeSizzlerAKS --enable-aad --aad-admin-group-object-ids a7bb1ec1-9274-46ec-ac73-7e3996d96e57 --aad-tenant-id replace-your-tenant-id</b>
<p>6. A successful creation of an AKS-managed Azure AD cluster has the following section in the response body.</p>
<img src="https://csgithub.blob.core.windows.net/lntergrate/Screenshot (3158).png"/>
<p>7. The cluster is created within a few minutes.</p>

<h2>Access an Azure AD enabled cluster</h2>
<p>1. You'll need the Azure Kubernetes Service Cluster User built-in role to perform the following steps.</p>
<p>2. Get the user credentials to access the cluster:</p>
	<b>az aks get-credentials --resource-group CodeSizzlerAKS-Rg --name CodeSizzlerAKS</b>
<img src="https://csgithub.blob.core.windows.net/lntergrate/Screenshot (3159).png"/>
<p>3. Follow the instructions to sign in.</p>
<p>4. Use the kubectl get nodes command to view nodes in the cluster:</p>
	<b>kubectl get nodes</b>
<img src="https://csgithub.blob.core.windows.net/lntergrate/Screenshot (3160).png"/>
<img src="https://csgithub.blob.core.windows.net/lntergrate/Screenshot (3161).png"/>
<img src="https://csgithub.blob.core.windows.net/lntergrate/Screenshot (3132).png"/>
