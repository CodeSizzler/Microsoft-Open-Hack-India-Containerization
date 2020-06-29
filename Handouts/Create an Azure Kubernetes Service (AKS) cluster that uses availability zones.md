<h1>Create an Azure Kubernetes Service (AKS) cluster that uses availability zones</h1>

<h2>Introduction</h2>
<p>An Azure Kubernetes Service (AKS) cluster distributes resources such as nodes and storage across logical sections of underlying Azure infrastructure. This deployment model when using availability zones, ensures nodes in a given availability zone are physically separated from those defined in another availability zone. AKS clusters deployed with multiple availability zones configured across a cluster provide a higher level of availability to protect against a hardware failure or a planned maintenance event.</p>

<h2>Prerequisites</h2>
<p>To perform this demo, You need the Azure CLI version 2.0.76 or later installed and configured. Run az --version to find the version. If you need to install or upgrade, see Install Azure CLI.</p>

<h2>Demo</h2>
<h2>Create an AKS cluster across availability zones</h2>
<p>1. When you create a cluster using the az aks create command, the --zones parameter defines which zones agent nodes are deployed into. The control plane components such as etcd is spread across three zones if you define the --zones parameter at cluster creation time. The specific zones which the control plane components are spread across are independent of what explicit zones are selected for the initial node pool.</p>
<p>2. The following example creates an AKS cluster named myAKSCluster in the resource group named myResourceGroup. A total of 3 nodes are created - one agent in zone 1, one in 2, and then one in 3.</p>
	<b>az group create --name CodeSizzlerAKS-Rg --location CentralUS</b>
<img src="https://csgithub.blob.core.windows.net/createanazure/Screenshot (3164).png"/>
	<b>az aks create --resource-group CodeSizzlerAKS-Rg --name CodeSizzlerAKS --generate-ssh-keys  --vm-set-type VirtualMachineScaleSets --load-balancer-sku standard --node-count 3 --zones 1 2 3</b>
<img src="https://csgithub.blob.core.windows.net/createanazure/Screenshot (3165).png"/>
<p>3. It takes a few minutes to create the AKS cluster.</p>

<h2>Verify node distribution across zones</h2>
<p>1. When the cluster is ready, list the agent nodes in the scale set to see what availability zone they're deployed in.</p>
<p>2. First, get the AKS cluster credentials using the az aks get-credentials command:</p>
	<b>az aks get-credentials --resource-group CodeSizzler-Rg --name CodeSizzlerAKS</b>
<img src="https://csgithub.blob.core.windows.net/createanazure/Screenshot (3166).png"/>

<h2>Verify pod distribution across zones</h2>
<p>1. As documented in Well-Known Labels, Annotations and Taints, Kubernetes uses the failure-domain.beta.kubernetes.io/zone label to automatically distribute pods in a replication controller or service across the different zones available. In order to test this, you can scale up your cluster from 3 to 5 nodes, to verify correct pod spreading:</p>
	<b>az aks scale --resource-group CodeSizzlerAKS-Rg --name myAKSCluster --node-count 5</b>
<img src="https://csgithub.blob.core.windows.net/createanazure/Screenshot (3167).png"/>
<p>2. We now have two additional nodes in zones 1 and 2. You can deploy an application consisting of three replicas. We will use NGINX as an example:</p>
	<b>kubectl run nginx --image=nginx --replicas=3</b>
<img src="https://csgithub.blob.core.windows.net/createanazure/Screenshot (3168).png"/>
<p>3. By viewing nodes where your pods are running, you see pods are running on the nodes corresponding to three different availability zones. For example, with the command <b>kubectl describe pod | grep -e "^Name:" -e "^Node:"</b> you would get an output similar to this:</p>
<img src="https://csgithub.blob.core.windows.net/createanazure/Screenshot (3170).png"/>
