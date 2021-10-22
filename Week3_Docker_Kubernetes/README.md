<h1>Containerizing and deploying application into K8S cluster</h1>
<h2>1. Fork repo https://github.com/ingvar-goryainov/python-app</h2>
<h2>2. Build docker image with Python  application</h2>
<h3>•	using “Best practices” for writing Dockerfiles</h3>
<h3>• define non-root user in order to run process inside a container by this user</h3>
<h3>• install all dependencies through pip</h3>
<h3>• build docker image as smaller as possible</h3>
<h2>3. Setup K8S cluster using Minikube(1 master + 1 worker node is enough)</h2>
<h2>4. Deploy the application into the K8S cluster</h2>
<h3>• create Deployment.yaml file with containerized application</h3>
<h3>The deployment requires 3 replicas, “RollingUpdate” strategy. Emulate the “RollingUpdate” strategy by updating docker image. Provide screenshots. Define the liveness and readiness probes to /health endpoint and 8080 port, resources(requests/limits)</h3>
<h3>Create a “Service” object which exposes Pods with application outside the K8S cluster in order to access each of the replicas through the single IP address/DNS name</h3>
<h3>Specify PodDistruptionBudget which defines that only 1 replica can be down</h3>

