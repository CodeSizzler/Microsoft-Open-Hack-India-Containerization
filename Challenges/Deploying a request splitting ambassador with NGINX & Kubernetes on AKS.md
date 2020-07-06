<h1>Deploying a request splitting ambassador with NGINX & Kubernetes on AKS</h1>

<h2>Lab Overview </h2>
<p>Azure AD integration with AKS-managed Azure AD is designed to simplify the Azure AD integration experience, where users were previously required to create a client app, a server app, and required the Azure AD tenant to grant Directory Read permissions. In the new version, the AKS resource provider manages the client and server apps for you.</p>

<h2>Lab Overview</h2>
<p>In this lab we'll guide you through the steps to deploy a request splitting ambassador that will split 10% of the incoming HTTP requests to an experimental server and the rest to a primary web server using Azure Kubernetes Service (AKS). This pattern is commonly used for testing new features or user experience to a small subset of users.</p>
<img src="https://csgithub.blob.core.windows.net/challengesaks/D01.png"/>

<h2>Demo</h2>
<h2>Learning Objectives </h2>
<p>Understand how to implement the request splitting ambassador pattern with Azure Kubernetes Service (AKS)</p>

<h2>Exercise 1: Accessing Microsoft Azure</h2>
<p>Launch Chrome from the virtual machine desktop and navigate to the URL below. Your Azure Credentials are available by clicking the Cloud Icon at the top of the Lab Player. https://portal.azure.com</p>
<img src="https://csgithub.blob.core.windows.net/challengesaks/D02.png"/>

<h2>Exercise 2: Deploy an Azure Kubernetes Service (AKS) cluster</h2>
<p>In this exercise, an AKS cluster is deployed using the Azure CLI that will be used for deploying the sample application.</p>

<h2>Task 1: Launch the Azure Cloud Shell</h2>
<p>1. From within the Azure portal, click the Cloud Shell icon at the top of the page to launch the cloud shell.</p>
<p>2. Choose Bash and confirm the creation of a storage account.</p>

<h2>Task 2: Create AKS cluster</h2>
<p>1. Use the following commands to first create a service principal to use. Then create an AKS cluster using the service principle information. The following example creates a cluster named codesizzlerAKSCluster with one node of size Standard_DS1_V2. Container health monitoring is also enabled using the –enable-addons monitoring parameter. Be sure to replace the [appID] placeholder text with the appID value shown by the first command and to replace the [password] placeholder text with the password value shown by the first command.</p>
	<b>az ad sp create-for-rbac --skip-assignment</b>
<img src="https://csgithub.blob.core.windows.net/challengesaks/D03.png"/> 
	<b>az aks create --resource-group codesizzlerlb-rg --namecdsakscluster --node-vm-size Standard_B2s --node-count 1 --generate-ssh-keys --service-principal [appID] --client-secret [password]</b>
<img src="https://csgithub.blob.core.windows.net/challengesaks/D04.png"/>  
 
<h2>Task 3: Connect to the cluster</h2>
<p>1. To manage a Kubernetes cluster, use Kubectl, the Kubernetes command-line client. This is installed automatically in the Azure Cloud Shell.</p>
<p>2. To configure kubectl to connect to your Kubernetes cluster, use the az-aks-get-credentials command. This step downloads credentials and configures the Kubernetes CLI to use them.</p>
	<b>az aks get-credentials --resource-group codesizzlerlb-rg --name cdsakscluster</b>
<img src="https://csgithub.blob.core.windows.net/challengesaks/D05.png"/>   	
<p>3 .To verify the connection to your cluster, use the [kubectl get][kubectl-get] command to return a list of the cluster nodes. It can take a few minutes for the nodes to appear.</p>
	<b>kubectl get nodes</b>
<img src="https://csgithub.blob.core.windows.net/challengesaks/D06.png"/>   	

<h2>Exercise 3: Implementing the main web server</h2>
<p>In this exercise, you will deploy the main web server that will accept 90% of the traffic for the application.</p>

<h2>Task 1: Create a custom NGINX Configuration file and ConfigMap</h2>
<p>In this task, you will create an NGINX configuration file will send 50% of the requests to main web server web-deployment, and 50% of the requests to the experimental web server experiment-deployment.</p>
<p>Now, in order for Kubernetes to understand this NGINX-specific configuration, make it readable by creating a ConfigMap object from this file. In essence, a ConfigMap object is a collection of key-value pairs that can be mounted to a volume inside the Kubernetes pod.</p>
<p>1. Create a conf.d sub-folder and change to that directory by executing the following commands:</p>
	<b>mkdir conf.d </b></br>
	<b>cd conf.d </b>  </br>
<img src="https://csgithub.blob.core.windows.net/challengesaks/D07.png"/>   	
<p>2. Next, create a nginx-ambassador.conf file in the folder and copy the following configuration code block into the new nginx-ambassador.conf file:</p>
<p>Tip: Create a new file using the nano editor by entering nano in the command line, and copy the code given on the labfiles into the editor. Press CTRL-O and then type in nginx-ambassador.conf to specify the file name and press Enter. Then CTRL-X to exit the editor.</p>
<p>Note: The weight=5 parameter is what specifies the distribution of traffic.</p></br>
<img src="https://csgithub.blob.core.windows.net/challengesaks/D08.png"/>   </br>
<img src="https://csgithub.blob.core.windows.net/challengesaks/D09.png"/>   </br>
<p>Tip - Save and exit with :wq</p>
<p>After you have saved the file. You should now have a nginx-ambassador.conf file in the conf.d folder.</p>
<p>Note: Unlike some of the examples on the web, ConfigMaps must be mounted as directories! Not as files. This is why the nginx-ambassador.conf file has to be placed in a folder.</p>
<p>Note: Also, if you are more experienced with NGINX configuration files: NGINX configuration for Kubernetes cannot contain any top level configuration attributes such as http, worker processes, etc. You will need to strip those from your .conf file.</p>
<p>3. Change to the parent directory by executing the following command.</p> 
	<b>cd ..</b></br>
<img src="https://csgithub.blob.core.windows.net/challengesaks/D10.png"/></br>
<p>4. Now, run the command that generates a ConfigMap for the custom NGINX ambassador configuration file:</p></br>
	<b>kubectl create configmap ambassador-config --from-file=conf.d</b></br>
<img src="https://csgithub.blob.core.windows.net/challengesaks/D11.png"/></br>

<h2>Task 2: Create the main web server deployment (90%)</h2>
<p>In order for us to show the benefits of an Ambassador pattern, we need to fire up two web servers that the NGINX can proxy and load-balance between. For our two servers we will fire up Kubernetes pod based on the "Hello World! Docker image from Microsoft’s public Docker repository: https://hub.docker.com/r/microsoft/aci-helloworld/.</p></br>
<p>1. We can easily create a load-balancing Kubernetes pod based on an existing Docker image. Create a new file called web-deployment.yaml with nano using the same instructions as earlier in the lab. Add the configuration downloaded from the labfiles.</p></br>
<img src="https://csgithub.blob.core.windows.net/challengesaks/D12.png"/>  </br>
	<b>- containerPort: 80</b>  </br>
<p>This simple Docker image will process requests on port 80 and return a basic Hello World! HTML response.</p>
<p>Note where the image attribute refers to the Microsoft repository and the aci-helloworld image in that repository. You can find more public Docker images from Microsoft on their Docker hub at: https://hub.docker.com/r/microsoft.</p>
<p>2. Use the following command to bring the Docker image to life by creating the Deployment (and automatically create the Pod in the process):</p></br>
	<b>kubectl create -f web-deployment.yaml</b></br>
<img src="https://csgithub.blob.core.windows.net/challengesaks/D13.png"/>   </br>
<p>3 .Finally, expose the web server to make it accessible to the outside world. To do this, fire up a simple Load Balancer with 2 replicas of our Web Server with the following command:</p></br>
	<b>kubectl expose deployment web-deployment --port=80 --type=ClusterIP --name web-deployment</p></br>
<img src="https://csgithub.blob.core.windows.net/challengesaks/D14.png"/>   </br>
    
<h2>Task 3: Create the experimental web server deployment (10%)</h2>
<p>We can easily create a load-balancing Kubernetes pod based on an existing Docker image. Create a new file called experiment-deployment.yaml with nano using the same instructions as earlier in the lab. Add the code given from the labfiles:</p></b>
<img src="https://csgithub.blob.core.windows.net/challengesaks/D15.png"/>
<p>This simple Docker image will process requests on port 80 and return a basic Hello World! HTML response.</p></br>
<p>Note where the image attribute refers to the Microsoft repository and the aci-helloworld image in that repository. You can find more public Docker images from Microsoft on their Docker hub at: https://hub.docker.com/r/microsoft.</p>
<p>2. Use the following command to bring the Docker image to life by creating the Deployment (and automatically create the Pod in the process):</p>
	<b>kubectl create -f experiment-deployment.yaml</b>
<img src="https://csgithub.blob.core.windows.net/challengesaks/D16.png"/>   
<p>3. Finally, expose the web server to make it accessible to the outside world. To do this, fire up a simple Load Balancer with 2 replicas of our Web Server with the following command:</p>
	<b>kubectl expose deployment experiment-deployment --port=80 --type=ClusterIP --name experiment-deployment</b>
<img src="https://csgithub.blob.core.windows.net/challengesaks/D17.png"/>  

<h2>Exercise 4: Create a Deployment in Kubernetes</h2>
<p>In this exercise, you will create a simple load balancing service that returns a fixed string for an HTTP request. In the previous steps you learned how to create a custom NGINX configuration to return a fixed string from an NGINX implementation and how to create a ConfigMap that can be read by Kubernetes when deploying your service.</p>

<h2>Task 1: Create the load balancing service</h2>
<p>1. Create a new file called ambassador-deployment.yaml with nano using the same instructions as earlier in the lab. Add the code from the lab files given.</p>
<img src="https://csgithub.blob.core.windows.net/challengesaks/D18.png"/> 
<p>Use the following command to have Kubernetes deploy the pods:</p>
	<b>kubectl create -f ambassador-deployment.yaml</b>
<img src="https://csgithub.blob.core.windows.net/challengesaks/D19.png"/> 
<p>2. The deployment is created, however it is not accessible from the outside world. To make the deployment accessible from the outside world, expose the deployment with this command:</p>
	<b>kubectl expose deployment ambassador-deployment --port=80 --type=LoadBalancer</b>
<img src="https://csgithub.blob.core.windows.net/challengesaks/D20.png"/> 
<p>3.  Check to see if the pods and deployments actually exist and have succeeded, by running these commands:</p>
	<b>kubectl get pods --output=wide</b>
<img src="https://csgithub.blob.core.windows.net/challengesaks/D21.png"/>
<p>The web-deployment-xxxx-xxxx items refer to the two replicas as defined in your ambassador-deployment.yaml file.</p>
<p>The ambassador-deployment-xxxx-xxxx refers to the pod we created with ambassador-pod.yaml.</p>
<p>4. Execute the following command, to receive notifications when AKS has issued an IP address for our pod so we can test it from the outside world.</p>
	<b>kubectl get services --watch</b>
<img src="https://csgithub.blob.core.windows.net/challengesaks/D22.png"/>
<p>Note: the –watch flag will wait for the IP address to be populated and show the assigned IP address once available.</p> 
<p>This results in an output similar to this:</p>
<img src="https://csgithub.blob.core.windows.net/challengesaks/D23.png"/>
<p> Note: It may take a few minutes for the EXTERNAL-IP of the ambassador-deployment to change from to an IP address.</p>
<p>Press CTRL + C to stop the watch task. Be sure to make note of the IP address.</p>

<h2>Exercise 5: Test and Debug the Kubernetes Pod Deployment</h2>
<p>In this exercise, you will learn how to test your deployment as well as common commands to run to view logs and further debug your Kubernetes Pod Deployment.</p>

<h2>Task 1: Test the Kubernetes Pod Deployment</h2>
<p>1. Now that AKS has issued an EXTERNAL-IP (as we can see from the output of our kubectl get services statement), go to a web browser and navigate to the public IP or use cURL to see the output of the load balanced custom NGINX implementation:</p2>
<p>Note: Replace [public ip] with the actual IP address.</p>
	<b>curl "http://[public ip]"</b>
<img src="https://csgithub.blob.core.windows.net/challengesaks/D24.png"/>
<p>2. If we repeat the above step a number of times, we should eventually see that 50% of the calls are made to the web-deployment container, and the remaining 50% to the experiment-deployment container. You can verify that by looking at the logs of both containers.</p2>

</h2>Task 2: Debugging your Kubernetes Pod Deployment</h2>
<p>Deploying a pod, deployment or a service with Kubernetes can be a daunting task. However, here are a few commands that will be able to guide you to potential problems with your configuration.</p>
<p>1. Inspect the active configuration of a pod - The command kubectl get pods returns a list of pods and their unique names</p>
<img src="https://csgithub.blob.core.windows.net/challengesaks/D25.png"/>
<p>2. To return the the YAML configuration used to deploy a particular pod, execute this command, where [ambassador-deployment-515597033-dklz1] is the name of your pod. Whichever name you choose, make sure to make note of it in Notepad.</p> 
	<b>kubectl get pod [ambassador-deployment-515597033-dklz1] -o yaml</b>   
<img src="https://csgithub.blob.core.windows.net/challengesaks/D26.png"/>
<p>3. The following statement returns the access logs of a pod. Replace [ambassador-deployment-515597033-dklz1] with the pod name you chose in the previous step. If you do not see any output, try using a different pod.</p>
	<b>kubectl logs [ambassador-deployment-515597033-dklz1]</b>
<img src="https://csgithub.blob.core.windows.net/challengesaks/D27.png"/>
<p>When the pod is running correctly, it returns an output similar to the above one.</p>
<p>4. If you do not want to wait for an external IP address to be issued to your pod by Azure, you can also enter the pod yourself and perform curl statements within the pod:</p>
	<b>kubectl run curl-ambassador-deployment --image=radial/busyboxplus:curl -i --tty –rm</b>
<p>This will fire up a CLI that is running within the pod. Now you can execute a few curl statements against the load balancer in the pod:If you don’t see a command prompt, try pressing enter.</p>
<img src="https://csgithub.blob.core.windows.net/challengesaks/D28.png"/>	  
	<b>curl "http://[public ip]" </b>
<p>This will show you the response from the Microsoft Hello World Docker image:</p>
<img src="https://csgithub.blob.core.windows.net/challengesaks/D29.png"/>	 
<p>5. Exit the pod by entering exit.</p>
<img src="https://csgithub.blob.core.windows.net/challengesaks/D30.png"/> 

<h2>Exercise 6: Monitoring</h2>
<p>In this exercise, you will explore the configuration of your AKS cluster from the Azure Portal.</p>

<h2>Task 1: Access the AKS Resource</h2>
<p>1. Within the Azure Portal, click Show portal menu to expand the left navigation then click Resource Groups.</p>
<p>Note: Notice there are two resource groups specifically for your AKS cluster. The codesizzlerlb-rg resource group holds your Azure Kubernetes Service instance, the MC_codesizzlerlb-rg* resource group contains the resources it orchestrates such as virtual machines, virtual networks, disks and network interfaces.</p>
<img src="https://csgithub.blob.core.windows.net/challengesaks/D31.png"/> 
<p>2. Open the codesizzlerlb-rg resource group and click the cdsakscluster resource.</p>
<img src="https://csgithub.blob.core.windows.net/challengesaks/D32.png"/>  
<p>3. Click the Monitor containers tile.</p>
<img src="https://csgithub.blob.core.windows.net/challengesaks/D33.png"/> 
<img src="https://csgithub.blob.core.windows.net/challengesaks/D34.png"/> 
<p>Explore the different views of the Cluster, its Nodes, Controllers and Containers.</p>
<img src="https://csgithub.blob.core.windows.net/challengesaks/D35.png"/>     
<p>4. Explore the AKS Logs by going back to the myAKSCluster dashboard and click View Logs.</p>
<img src="https://csgithub.blob.core.windows.net/challengesaks/D36.png"/> 
<p>5. Tip: Run the default query, and explore further by clicking Query explorer to view other example queries.</p>
<img src="https://csgithub.blob.core.windows.net/challengesaks/D37.png"/>   
<img src="https://csgithub.blob.core.windows.net/challengesaks/D38.png"/> 

<h2>Task 2: Scale the AKS Cluster </h2>
<p>1. Click Scale on the left navigation of your AKS cluster and notice your current node count.</p>
<img src="https://csgithub.blob.core.windows.net/challengesaks/D39.png"/> 
<p>2. In the CLI, enter the following to scale your cluster to a node count of 2:</p>  
	<b>az aks scale --resource-group codesizzlerlb-rg --name cdsakscluster --node-count 1 --nodepool-name nodepool1</b>
<img src="https://csgithub.blob.core.windows.net/challengesaks/D40.png"/> 
<p>3. Navigate to the other resource group and open the virtual machine scale set. Note that the virtual machine scale set is being provisioned to add capacity.</p>
<img src="https://csgithub.blob.core.windows.net/challengesaks/D41.png"/> 
<img src="https://csgithub.blob.core.windows.net/challengesaks/D42.png"/>  

<h2>Summary</h2>
<p>In this lab you implemented the ambassador pattern with NGINX and configured it as a proxy to split a portion of the requests to an “experimental” service. You then deployed Docker images inside a Kubernetes pod to serve simple HTTP responses. Then you made requests to validate the requests were split appropriately to the different Docker images. Finally, you explored the AKS cluster from the Azure portal and increased the scale to two nodes.</p2>



















 
