
# About the Project
The objective of this project is to explain step by step how to create your own data science work station on your local machine using Docker containers. 
Then docker-compose is used in order to automate the process. In addition, a reference project folder structure is set up

# Getting Started

## Start up on Windows 
1. Start Ubuntu on Widows (check to list all distros: ```wsl -l -v```)
2. Start Docker Desktop
    * Settings/General/ check Use the WSL 2 based engine
    * Settings/Resources/WSL INTEGRATION/Select the correspondent distro (e.g. Ubuntu)
3. Run docker containers
```bash
docker-compose up -d
```
4. Check containers
```bash
docker ps
```

## Architecture
* ML Experiment Tracker: [Scenario 4: MLflow with remote Tracking Server, backend and artifact stores
](https://mlflow.org/docs/latest/tracking.html)
    * Backend: Database of MLFLow entities (Postgres) and artifact store (MinIO)
    * Frontend: MLFlow GUI
* JupyterLab Workbench

## Setting up the tracking server and workbench with docker-compose
```bash
docker-compose --env-file .env up -d
docker ps
docker-compose down

```
In order to update, it may be needed to re build:
```docker-compose --env-file .env build```

## Run local example
This will run a local example that will be tracked by the `ML Experiment Tracker` (containers should be running)

```
cd examples
```  
Set environmnet variables  
```
./demo_env_vars
```  
Activate virtual environment (Ubuntu)  
```
source ./.venv/bin/activate
```  
Run app  
```
python3 demo.py
```  
This script sets up connection to a our ML Experiment Tracker through `localhost` as shown in the code snipet form example/demo.py
```python
remote_server_uri = "http://localhost:5000" # set to your server URI
mlflow.set_tracking_uri(remote_server_uri)
mlflow.set_experiment("testing12345")
```

## Run an experiment from JupyterLab workwbench
When the experiment is run on a docker container instead of in our local (host machine), a slight change should be made. In the mlflow_uri, change localhost to whatever the actual service that runs mlflow server:

```python
remote_server_uri = "http://tracking_server:5000" # set to your server URI
mlflow.set_tracking_uri(remote_server_uri)
mlflow.set_experiment("testing12345-wb")
```

or set as environment variable in Python
```python
MLFLOW_TRACKING_URI = "http://tracking_server:5000" # set to your server URI
os.environ["MLFLOW_TRACKING_URI"] = MLFLOW_TRACKING_URI
```

See an example on: notebooks/testing/test_mlflow_conn.ipynb

## Prerequisites
1. Install WSL2 (On windows) and set WSL as docker engine
2. Install Docker
3. Install docker-compose
4. Create a venv on examples/.venv

## Local project structure
In workbench, a cookiecutter-like project structure is laid out
```
+-- docker
+-- examples
+-- workbench
|   +-- data
|   +-- notebooks
|       +-- nb_config.py
|   +-- src
|   +--.env
+-- .env
+-- docker-compose.yaml
```

## Services
* MLFlow service
* Postgres and pg-admin
* MinIO

# ML Experiment Tracker: MLFlow
On /docker/mlflow/
```bash
docker pull python:3.7.9-slim
docker build -t getting-started-mlflow . 
docker images
docker run -d -t -p 5000:5000 --name mlflow_standalone getting-started-mlflow
docker ps
docker exec -it <container-name>
```
inside container: 
```
mlflow server --host 0.0.0.0
```

# ML Experiment Tracker Backend DB: Postgres
```bash
docker pull postgres:14.1-alpine
docker run -it --rm \
--name postgres_standalone \
-p 5432:5432 \
-e POSTGRES_USER=mlflow \
-e POSTGRES_PASSWORD=mlflow \
-e POSTGRES_DATABASE=mlflow \
-e POSTGRES_PORT=5432 \
-e PGDATA=/var/lib/postgresql/data/ \
-v /tmp:/var/lib/postgresql/data \
postgres:14.1-alpine
docker exec -it postgres_standalone /bin/sh
```
inside container: 
```
psql -username mlflow
\l
\q
```


# ML Experiment Tracker Artifacts Storage: MinIO
```bash
docker run -t -d \
--name minio_standalone \
-p 9000:9000 \
-e MINIO_SECRET_ACCESS_KEY=minio \
-e MINIO_SECRET_ACCESS_KEY=minio123 \
-e MINIO_PORT=9000 \
-v /tmp:/data \
minio/minio:latest \
server /data 
docker ps
```


# Data Science Workbench: JupyterLab
```bash
docker pull jupyter/scipy-notebook:python-3.8.8

docker run -it --rm -d -p 8888:8888 \
-e JUPYTER_ENABLE_LAB=yes \
-v "${PWD}"/workbench:/home/jovyan/workbench \
jupyter/scipy-notebook:python-3.8.8
``` 

In order to get the token: 
```docker exec -it <CONATEINER NAME> sh``` and then ```jupyter server list```

# Runner
```bash
docker build -t runner -f docker/runner/Dockerfile .
docker run --name runner -t -d  runner
docker exec -it runner bash
docker stop runner
```

# Remote VSCode sesion into a running Docker container
1. Install Docker extension
2. Right-click on desired container and select: 'Attach to VSCode`

# Reference:
* [Docker Desktop WSL 2 backend](https://docs.docker.com/desktop/windows/wsl/)
* [Postgres with Docker and Docker compose a step-by-step guide for beginners](https://geshan.com.np/blog/2021/12/docker-postgres/)
* [Configuring a Data Science Workbench](https://www.emilygorcenski.com/post/configuring-a-data-science-workbench/) by emilygorcenski.com/
* [Docker Tutorial part 2 | Docker Compose](https://www.youtube.com/watch?v=UBFiCfbRykE) YouTube. adds a pgadmin service to docker-compose
* [Python Minio Docker Tutorial (Minio SDK) on Docker](https://www.youtube.com/watch?v=huoXr1pbzUQ)
* [Python venvs in VSCode](https://code.visualstudio.com/docs/python/environments)
* [Docker For Data Scientists](https://www.youtube.com/watch?v=0qG_0CPQhpg): Run a train job in docker (configuring GPUs)
* [Cookiecutter Data Science](https://drivendata.github.io/cookiecutter-data-science/#build-from-the-environment-up)

* [How to install virtual environment on ubuntu 16.04](https://gist.github.com/frfahim/73c0fad6350332cef7a653bcd762f08d)