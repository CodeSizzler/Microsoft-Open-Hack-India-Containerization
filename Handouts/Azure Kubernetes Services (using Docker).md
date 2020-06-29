<h1>Azure Kubernetes Services (using Docker)</h1>
<h2>Creating Container Images</h2>
<h3>1.	Copy application from git repository</h3>

<p>1.1.	git clone https://github.com/Azure-Samples/azure-voting-app-redis.git</p>
<p>1.2.	cd azure-voting-app-redis</p>

<h3>2.	Create container images using Docker Compose</h3>
<p>2.1.	docker-compose up -d</p>
<p>2.2.	See images when completed docker images</p>
<p>2.3.	See running containers docker ps</p>

<h3>3.	Test application locally</h3>
<p>3.1.	http://localhost:8080</p>

<h3>4.	Clean containers</h3>
<p>4.1.	docker-compose down</p>
