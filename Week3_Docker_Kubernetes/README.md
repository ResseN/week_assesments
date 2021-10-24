<h1>Containerizing and deploying application into K8S cluster</h1>
<h2>1. Fork repo https://github.com/ingvar-goryainov/python-app</h2>
<p>Make fork of provided repository. Forked repository address https://github.com/ResseN/python-app</p>
<h2>2. Build docker image with Python  application</h2>
<h3>•	using “Best practices” for writing Dockerfiles</h3>
<h3>• define non-root user in order to run process inside a container by this user</h3>
<h3>• install all dependencies through pip</h3>
<h3>• build docker image as smaller as possible</h3>
<p>Final <a href="https://github.com/ResseN/python-app/blob/main/Dockerfile">Dockerfile</a> contains next instuctions</p>
<blockquote>
   <pre> 
FROM python:3.6 as base
FROM base as builder
COPY application/  /opt/app/
WORKDIR /opt/app
RUN  grep -oPz "install_requires=\[(.|\n)*?\]" setup.py | \
     grep -oPz "\'(.*)\',?" | tr ',' '\n' | tr "'" '\n' | \
     sed 's/\x0//g' | awk 'NF' > requirements.txt && \
     pip install -r requirements.txt && \
     python3 setup.py install && \
     rm -rf .eggs demo.eggs-info build dist setup.py
FROM python:3.6-alpine
RUN adduser --disabled-password pythonuser
USER pythonuser
WORKDIR /opt/app
COPY --from=builder /opt/app/ .
RUN pip install --no-cache-dir -r requirements.txt
CMD ["python3", "-m", "demo"]
   </pre>
</blockquote>
<p>Docker build <a href="https://github.com/ResseN/week_assesments/blob/main/Week3_Docker_Kubernetes/resources/docker_build.log">log</a></p>
<p>Docker image size equal 49.1 MB</p> 
<img src="https://github.com/ResseN/week_assesments/blob/main/Week3_Docker_Kubernetes/resources/docker_image_size.png" width=100%>
<p>Start docker container, check availability of python_app, show container logs and status running container on image below</p>
<img src="https://github.com/ResseN/week_assesments/blob/main/Week3_Docker_Kubernetes/resources/docker_ps_log.png" width=100%>
<h2>3. Setup K8S cluster using Minikube(1 master + 1 worker node is enough)</h2>
<p>Minikube cluster was installed on MacBook with VirtualBox. </p>
<blockquote>
   <pre> 
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64
sudo install minikube-darwin-amd64 /usr/local/bin/minikube
minikube version
minikube start -n 2
   </pre>
</blockquote>
<img src="https://github.com/ResseN/week_assesments/blob/main/Week3_Docker_Kubernetes/resources/minikube_cluster_create.png" width=100%>
<p>Checking nodes of cluster</p>
<img src="https://github.com/ResseN/week_assesments/blob/main/Week3_Docker_Kubernetes/resources/minikube_cluster_nodes.png" width=100%>
<h2>4. Deploy the application into the K8S cluster</h2>
<h3>• create Deployment.yaml file with containerized application</h3>
<p>Writed <a href="https://github.com/ResseN/python-app/blob/main/Deployment.yaml">Deployment</a> file to deploy app on the cluster</p>
<blockquote>
   <pre> 
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: pythonapp
  name: pythonapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: pythonapp
  template:
    metadata:
      labels:
        app: pythonapp
    spec:
      containers:
      - image: admressen/python_app:v0.1
        name: pythonapp
   </pre>
</blockquote>
<p><a href="https://github.com/ResseN/week_assesments/blob/main/Week3_Docker_Kubernetes/resources/k8s_deployment_log">Log</a> with output of entred commands show us that deploy is created, pod upped on non-master node but app service accessed from external IP of both nodes</p>
<h3>*The deployment requires 3 replicas, “RollingUpdate” strategy. Emulate the “RollingUpdate” strategy by updating docker image. Provide screenshots. Define the liveness and readiness probes to /health endpoint and 8080 port, resources(requests/limits)</h3>
<p>To realise that requirement need use that <a href="https://github.com/ResseN/python-app/blob/main/Deployment_replica3.yaml">Deployment</a> file</p>
<p>Demonstrating rollingupdate strategy executing next command</p>
<blockquote>
   <pre>
   kubectl set image deployments/pythonapp pythonapp=admressen/python_app:v0.2
   </pre>
</blockquote>
<img src="https://github.com/ResseN/week_assesments/blob/main/Week3_Docker_Kubernetes/resources/rolling_update_log.png">
<p>LivenessProbe and readinessProbe working we can see in pods log</p>
<img src="https://github.com/ResseN/week_assesments/blob/main/Week3_Docker_Kubernetes/resources/pod_probes_log.png">
<h3>*Create a “Service” object which exposes Pods with application outside the K8S cluster in order to access each of the replicas through the single IP address/DNS name</h3>
<p>Creating service we can with next command:</p>
<blockquote>
   <pre>
   kubectl expose deploy pythonapp --port=8080 --target-port=8080 --type=NodePort
   </pre>
</blockquote>
<p>Multiple requests to ip:port of service provide answers from different pods. Working as internal loadbalancer</p>
<img src="https://github.com/ResseN/week_assesments/blob/main/Week3_Docker_Kubernetes/resources/services_answer.png">
<h3>*Specify PodDistruptionBudget which defines that only 1 replica can be down</h3>
<p>To create PDB with 1 unavailable replica we need execute next command</p>
<blockquote>
   <pre>
kubectl create poddisruptionbudget pythonapp --selector=pythonapp --max-unavailable=1
   </pre>
</blockquote>
<img src="https://github.com/ResseN/week_assesments/blob/main/Week3_Docker_Kubernetes/resources/pdb.png">
