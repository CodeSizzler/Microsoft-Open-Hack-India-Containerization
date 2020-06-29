<h1>Geo-replication in Azure Container Registry</h1>

<h2>Introduction</h2>
<p>Companies that want a local presence, or a hot backup, choose to run services from multiple Azure regions. As a best practice, placing a container registry in each region where images are run allows network-close operations, enabling fast, reliable image layer transfers. Geo-replication enables an Azure container registry to function as a single registry, serving multiple regions with multi-master regional registries.</p>

<h2>Prerequisites</h2>
<p>To perform this demo, Contoso runs a public presence website located across the US, Canada, and Europe. To serve these markets with local and network-close content, Contoso runs Azure Kubernetes Service (AKS) clusters in West US, East US, Canada Central, and West Europe.</p>

<h2>Demo</h2>
<h2>Configure geo-replication</h2>
<p>1. Configuring geo-replication is as easy as clicking regions on a map. You can also manage geo-replication using tools including the az acr replication commands in the Azure CLI, or deploy a registry enabled for geo-replication with an Azure Resource Manager template.</p>
<p>2. Geo-replication is a feature of Premium registries. If your registry isn't yet Premium, you can change from Basic and Standard to Premium in the Azure portal</p>
<p>3. To configure geo-replication for your Premium registry, log in to the Azure portal at https://portal.azure.com.</p>
<p>4. Log in to the Azure portal click on search marketplace type for Container Registry</p>
<img src="https://csgithub.blob.core.windows.net/georeplication/Screenshot (3171).png"/>
<p>5. Create a container registry and configure the following settings:</p>
	<p>Subscription: Select on your subscription</p>
	<p>Resource group: Create a new resource</p>
	<p>Registry name: Create on Registry 
	<p>Lovcation: Select on Azure Region
	<p>SKU: Select on Premium
<p>6. Finally click on Review+Create button.
<img src="https://csgithub.blob.core.windows.net/georeplication/Screenshot (3172).png"/>
<img src="https://csgithub.blob.core.windows.net/georeplication/Screenshot (3173).png"/>
<p>7. Navigate to your Azure Container Registry, and select Codesizzlercontainer blade.</p>
<img src="https://csgithub.blob.core.windows.net/georeplication/Screenshot (3174).png"/>
<p>8. Now, move to Codesizzlercontainer on overview tab.select on Update blade.</p>
<img src="https://csgithub.blob.core.windows.net/georeplication/Screenshot (3175).png"/>
<img src="https://csgithub.blob.core.windows.net/georeplication/Screenshot (3176).png"/>
<p>9. Navigate to your Azure Container Registry, and select Replications:</p>
<img src="https://csgithub.blob.core.windows.net/georeplication/Screenshot (3177).png"/>
<p>10. A map is displayed showing all current Azure Regions:</p>
<p>11. To configure additional replicas, select the green hexagons for other regions, then click Create.</p>
<img src="https://csgithub.blob.core.windows.net/georeplication/Screenshot (3178).png"/>
<p>12. ACR begins syncing images across the configured replicas. Once complete, the portal reflects Ready. The replica status in the portal doesn't automatically update. Use the refresh button to see the updated status.</p>
<img src="https://csgithub.blob.core.windows.net/georeplication/Screenshot (3179).png"/>

