<h1>Managed identities in AKS</h1>
<h2>Overview:</h2>
<p>Currently, an Azure Kubernetes Service (AKS) cluster (specifically, the Kubernetes cloud provider) requires an identity to create additional resources like load balancers and managed disks in Azure. This identity can be either a managed identity or a service principal. If you use a service principal, you must either provide one or AKS creates one on your behalf. If you use managed identity, this will be created for you by AKS automatically. Clusters using service principals eventually reach a state in which the service principal must be renewed to keep the cluster working. Managing service principals adds complexity, which is why it's easier to use managed identities instead. The same permission requirements apply for both service principals and managed identities.</p>
<p>Managed identities are essentially a wrapper around service principals, and make their management simpler. Credential rotation for MI happens automatically every 46 days according to Azure Active Directory default. AKS uses both system-assigned and user-assigned managed identity types. These identities are currently immutable.</p>

<h2>Before you begin</h2>
<p>You must have the following resource installed:</p>
<li>The Azure CLI, version 2.2.0 or later</li>
<h2>Limitations</h2>
<li>Bring your own managed identities is not currently supported.</li>
<li>AKS clusters with managed identities can be enabled only during creation of the cluster.</li>
<li>Existing AKS clusters cannot be updated or upgraded to enable managed identities.</li>
<li>During cluster upgrade operations, the managed identity is temporarily unavailable.M</li>

<h2>Create an AKS cluster with managed identities</h2>
<p>You can now create an AKS cluster with managed identities by using the following CLI commands.</p>
<p>First, login with your azure account using: <b>az login</b></p>
<p>Next, create a resource group using the below command:</p>
<b>az group create --name codesizzlerrg --location westus2</b>
<p>Now, create an AKS cluster:</p>
<b>az aks create -g CodesizzlerRG -n CodeSizzlerCluster --enable-managed-identity –generate-ssh-keys</b>
<p>Use the following command to query objectid of your control plane managed identity:</p>
<b>az aks show -g CodesizzlerRG -n CodesizzlerCluster --query "identity"</b>
<p>Finally, get credentials to access the cluster:</p>
<b>az aks get-credentials --resource-group CodesizzlerRG --name CodesizzlerCluster</b>
<p>The cluster will be created in a few minutes. You can then deploy your application workloads to the new cluster and interact with it just as you've done with service-principal-based AKS clusters.</p>

