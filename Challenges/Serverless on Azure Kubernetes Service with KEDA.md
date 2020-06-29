
<h1>Serverless on Azure Kubernetes Service with KEDA</h1>

<h2>Overview</h2>
<p>The lab consists of 4 steps. This document provides a complete walkthrough of Develop and Deployment of Azure functions on Kubernetes.</p>

<h2>Tasks to Complete</h2>
<h2>Create an AKS Cluster in Azure</h2>
	</p>•Create the Azure Functions locally and test with Queue Trigger</p>
	</p>•Build the Docker Image and deploy it to Kubernetes with the Virtual Nodes</p>
	</p>•Load Test the Storage Queue and watch the Pod Scale out</p>
	</p>•Look at the Functions logs and information in Insights to help Debug Functions issue.</p>

<h2>After completing this lab, you will be able to</h2>
	</p>•Develop and Deploy Azure Functions app on Kubernetes with KEDA</p>

<h2>Pre-requisites</h2>
<p>This document is designed to walk you through the whole lab. However, to take the most out of it you are expected to have</p>
	</p>•Basic knowledge of Azure functions</p>
	</p>•Basic understanding of Docker and Containers</p>
	</p>•Basic understanding of Kubernetes</p>

<h2>Task 1: Create Azure Resources for your Lab</h2>
<p>1. Use the following commands to first create a service principal to use. Then create an AKS cluster using the service principal information. The following example creates a cluster named KubernetesAKS with one node of size Standard_B2s. Container health monitoring is also enabled using the –enable-addons monitoring parameter. Be sure to replace the [appID] placeholder text with the appID value shown by the first command and to replace the [password] placeholder text with the password value shown by the first command.</p>
	<b>az ad sp create-for-rbac --skip-assignment</b>
<img src="https://csgithub.blob.core.windows.net/akswithkeda/01.png"/>
	<b>az aks create --resource-group KubernetesAKS --name myAKSCluster --node-vm-size Standard_B2s  --node-count 1 --generate-ssh-keys --service-principal [appID] --client-secret [password]</b>
<img src="https://csgithub.blob.core.windows.net/akswithkeda/02.png"/>
<p>2. Create an Azure Storage Account. Click + Create a Resource -> Storage Acccount - blob, file, table, queue and add it to the codesizzleraks-rg resource group.</p>
<img src="https://csgithub.blob.core.windows.net/akswithkeda/03.png"/>
<p>3. Install the following resources on your virtual machine:</p>
	<p>https://www.npmjs.com/get-npm</p>
	<p>https://www.npmjs.com/package/azure-functions-core-tools</p>

<h2>Task 1: Create the Azure Functions locally and test with Queue Trigger</h2>
<p>1. Open CMD on the Virtual machine</p>
<p>2. Create a new directory for the function app</p>
	<b>mkdir hello-keda</b>
	<b>cd hello-keda</b>
<img src="https://csgithub.blob.core.windows.net/akswithkeda/04.png"/>
<p>3. Initialize the directory for functions</p>
	<b>func init . –docker</b>
<img src="https://csgithub.blob.core.windows.net/akswithkeda/05.png"/>
	<p>Select dotnet</p>
<img src="https://csgithub.blob.core.windows.net/akswithkeda/06.png"/>
<p>4. Add a new queue triggered function</p>
	<p>func new</p>
<img src="https://csgithub.blob.core.windows.net/akswithkeda/07.png"/>
<p>5. Select Azure Queue Storage Trigger</p>
<p>6. Leave the default of QueueTrigger1 for the name</p>
<img src="https://csgithub.blob.core.windows.net/akswithkeda/08.png"/>
<p>7. Create an Azure storage queue</p>
<img src="https://csgithub.blob.core.windows.net/akswithkeda/09.png"/>
<p>8. Create an Azure storage queue and use name js-queue-items</p>
<img src="https://csgithub.blob.core.windows.net/akswithkeda/10.png"/>
<p>9. Update the function metadata with the storage account info</p>
	<p>Open the hello-keda directory in VSCODE by typing below command in CMD code.</p>
<img src="https://csgithub.blob.core.windows.net/akswithkeda/11.png"/>
<img src="https://csgithub.blob.core.windows.net/akswithkeda/12.png"/>
<p>10. We'll need to update the connection string info for the queue trigger, and make sure the queue trigger capabilities are installed.</p>
<p>11. Copy the current storage account connection string (HINT: don't include the ")</p>
<img src="https://csgithub.blob.core.windows.net/akswithkeda/13.png"/>
<b>local.settings.json: Download the code from lab files</b>
<img src="https://csgithub.blob.core.windows.net/akswithkeda/14.png"/>
<p>12. Open QueueTrigger1.cs which has Queue name and connection settings. Replace the queue name to “js-queue-items" and Connection=”AzureWebJobsStorage"</p>
	<p>QueueTrigger1.cs</p>
<img src="https://csgithub.blob.core.windows.net/akswithkeda/15.png"/>
<p>13. Enable the storage queue bundle on the function runtime Replace the host.json content with the following. This pulls in the extensions to the function runtime.</p>
<b>host.json:Download the code from Lab files</b>
<img src="https://csgithub.blob.core.windows.net/akswithkeda/16.png"/>
<p>13. Debug and test the function locally</p>
	<b>func start</b>
<img src="https://csgithub.blob.core.windows.net/akswithkeda/17.png"/>
<p>14. Go to your Azure Storage account in the Azure portal and open the queues and Select the js-queue-items queue and add a message to send to the function.</p>
<img src="https://csgithub.blob.core.windows.net/akswithkeda/18.png"/>
<p>A screenshot of a cell phone Description automatically generated You should see your function running locally fired correctly immediately.</p>

<h2>TASK 2: Build the Docker Image and deploy it to Kubernetes with the Virtual Nodes</h2>
<p>1. Install KEDA</p>
	<p>•Add Helm repo</p>
	<b>helm repo add kedacore https://kedacore.github.io/charts</p>
	<b>Update Helm repo</b>
	<b>helm repo update</b>
	<b>Install keda Helm chart</b>
	<b>kubectl create namespace keda</b>
	<b>helm install keda kedacore/keda --namespace keda</b>
<p>To confirm that KEDA has successfully installed you can run the following command and should see the following CRD.</p>
	<p>kubectl get customresourcedefinition</p>
<p>2. We need to create Kubernetes secret to fetch image from ACR using secrets.</p>
	<p>kubectl create secret docker-registry acr-auth --docker-server=<acr_name>.azurecr.io --docker-username=<acrusername> --docker-password=<ACR password> --docker-email mailto:userxxx@xxx.onmicrosoft.com)</p>
<p>3. Deploy Function App to KEDA (Virtual Nodes)</p>
	<p>To deploy your function Kubernetes with Azure Virtual Nodes, you need to modify the details of the deployment to allow the selection of virtual nodes.</p>
	<p>Generate a deployment yaml for the function</p>
	<p>In below command add ACR name as <acrname>.azurecr.io</p>
	<p>func kubernetes deploy --name hello-keda --registry <acr_name>.azurecr.io --dotnet --pull-secret acr-auth --dry-run deploy.yaml</p>
<p>Open and modify the created deploy.yaml to tolerate scheduling onto any nodes, including virtual.</p>
	<p>spec:</p>
	<p>containers:
	<p>- name: hello-keda</p>
	<p>image: <acr name>.azurecr.io/hello-keda</p>
	<p>env:</p>
	<p>- name: AzureFunctionsJobHost__functions__</p>
	<p>value: QueueTrigger1</p>
	<p>envFrom:</p>
	<p>-  secretRef:</p>
	<p>name: hello-keda</p>
	<p>imagePullSecrets:</p>
	<p>- name: acr-auth</p>
	<p>tolerations:</p>
	<p>- key: virtual-kubelet.io/provider</p>
	<p>operator: Exists</p>
	<p>- key: azure.com/aci</p>
        <p>effect: NoSchedule</p>
<p>Login into ACR</p>
	<p>az acr login ---name <acr name></p>
<p>Build and deploy the container image and apply the deployment to your cluster.Replace < your-acr-id> with <ACRname>.azurecr.io in below commands.</p>
	<p>docker build -t <your-acr-id>/hello-keda</p>
      	<p>docker push <your-acr-id>/hello-keda</p>
	<p>kubectl apply -f deploy.yaml</p>

<h2>TASK 3: Load Test the Storage Queue and watch the Pod Scale out.</h2>	
<p>1. Add a queue message and validate the function app scales with KEDA</p>
<p>Initially after deploying and with an empty queue you should see 0 pods.</p>
	<p>kubectl get deploy</p>
<p>Add a queue message to the queue (using portal in step 8 of TASK 1 above). KEDA will detect the event and add a pod. By default, the polling interval set is 30 seconds on the ScaledObject resource, so it may take up to 30 seconds for the queue message to be detected and activate your function. This can be adjusted on the ScaledObject resource.</p>
<p>2. Load Test the Storage Queue and watch the Pod Scale out.</p>
<p>Using Azure Cloud Shell execute the following PowerShell code. Ensure youre place the placeholder value with your storage account name.</p>
	<p>$resourceGroup = "KubernetesAKS"</p>
	<p>$storageAccountName = "[your storage account name]"</p>
	<p>$queueName = "js-queue-items"</p>
<p>From there, execute the following script to pump some messages into your queue.<p>
	<p>$storageAccount = Get-AzStorageAccount -ResourceGroupName $resourceGroup -Name $storageAccountName<p>
	<p>$ctx = $storageAccount.Context<p>
	<p>$queue = Get-AzStorageQueue –Name $queueName –Context $ctx<p>
	<p>$queueMessage = [Microsoft.Azure.Storage.Queue.CloudQueueMessage]::new("Hello from the PTBC 2020")<p>
	<p>$queue.CloudQueue.AddMessageAsync($queueMessage)<p>
	<p>for($i = 0; $i -lt 1000; $i++)<p>
	<p>{ $queue.CloudQueue.AddMessageAsync($queueMessage)  }<p>
<p>3. Watch the pod scale out</p>
	<p>kubectl get deploy<p>
	<p>kubectl get pods -w<p>
<p>The queue message will be consumed. You can validate the message was consumed by using kubectl logs on the activated pod. New queue messages will be consumed and if enough queue messages are added the function will autoscale. After all messages are consumed and the cooldown period has elapsed (default 300 seconds), the last pod should scale back down to zero.</p>

<h2>TASK 4: Look at the Functions logs and information in Insights to help Debug Functions issue.</h2>
<p>1. Function logs:</p>
<p>We can view function logs in AKS Insight logs</p>
	<p>| where TimeGenerated >= ago(1d)</p>
	<p>| where LogEntry contains "New Queue Message detected"</p>
	<p>| summarize count() by bin(TimeGenerated,5m), ContainerID</p>
	<p>| render timechart</p>
<p>2. View logs</p>
<p>You can view real-time log data as they are generated by the container engine from the Nodes, Controllers, and Containers view. To view log data, perform the following steps.</p>
	<p>1.In the Azure portal, browse to the AKS cluster resource group and select your AKS resource.</p>
	<p>2.On the AKS cluster dashboard, under Monitoring on the left-hand side, choose Insights.</p>
	<p>3.Select either the Nodes, Controllers, or Containers tab.</p>
	<p>4.Select an object from the performance grid, and on the properties pane found on the right side, select View live data (preview) option. If the AKS cluster is configured with single sign-on using Azure AD, you are prompted to authenticate on first use during that browser session. Select your account and complete authentication with Azure.</p>
<p>The pane title shows the name of the pod the container is grouped with.</p>
<p>3. View metrics</p>
<p>You can view real-time metric data as they are generated by the container engine from the Nodes or Controllers view only when a Pod is selected. To view metrics, perform the following steps.</p>
	<p>1.In the Azure portal, browse to the AKS cluster resource group and select your AKS resource.</p>
	<p>2.On the AKS cluster dashboard, under Monitoring on the left-hand side, choose Insights.</p>
	<p>3.Select either the Nodes or Controllers tab.</p>
	<p>4.Select a Pod object from the performance grid, and on the properties pane found on the right side, select View live data (preview) option. If the AKS cluster is configured with single sign-on using Azure AD, you are prompted to authenticate on first use during that browser session. Select your account and complete authentication with Azure.</p>
<p>Congratulations! You made it through the end!</p>
<p>Thank you for taking this lab!</p>
<p>We enjoyed developing the lab and we sincerely hope it met your expectations!</p>
<p>The exercise in the lab were meant to touch base with some new trending technologies in the Azure function app developer’s world.</p>
<p>Many of us have a strong background in developing and deploying Azure Functions app, built upon several years. Today, with all the new exciting scenarios and technologies fostered by the cloud, it is easy to feel left behind. However, even in this brand-new world, we can still rely on our own skills!</p>
<p>It is the approach we use towards developing and deploying that makes the difference!</p>
