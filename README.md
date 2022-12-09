# Introduction

## About the Project
The objective of this project is to explain step by step how go through a comprehensive MLOps cycle on your local machine using Docker containers, in order to comprehensively understand and MLOps System (platform side + processes)
Then docker-compose is used in order to automate the process.

Therefore, several technlogies are introduced (like MLFlow) but also it is necessary to define and standardize the processes involved, to do son, a set of best practices are set:
* Project folder structure
* Software architecture
* Naming convention


## Overview
A data science workbench, fully opearting with advanced MLOps capabilities must (at least) contain:
* A data science workbench: This will allow to the data scienstist to work with an environment that can be easily wrapped in a docker image in order to being able to:
    * Share the image with peers to standardize analytics tools 
    * Wrap their source code into an image in order to run it in a runner environment environment (like kubernetes) 
    * Templatize the source code  
    This environment will rely on jupyterlab to provide a GUI to develop source code and mlflow to manage and wrap environments, orchestrate runs and create train/serving (batch or REST-API) services

* MLOps Platform: Set of services and tools used to provide (at least) the following capabilities:
    * Experiment tracking accesible from ds-workbench and runner environments
    * Model (and artifacts) store  
    * Model metadata to provide lineage
    So as to allow reproductibility and ease model building. This must be connected to Serving Services  

Other modules can be part of the MLOps Platform, like the runner environment itself (like a kubernetes cluster) orchestration and automation, monitoring and data versioning; however, on this simplified local approach we will keep things simple to demonstrate the whole cycle and the combination between technologies and actual processes


# Getting Started 

## Prerequisites
### Windows
1. Install WSL2 
2. Install Docker Desktop
3. Set WSL as docker engine 
3. Install docker-compose on WSL
4. Create a venv on examples/.venv on WSL

### Linux
1. Install docker
2. Install docker-compose
3. Create a venv on examples/.venv

## Steps
### Get on with MLFlow
MLflow will be the main tool. The role of this techlogy is explained in `Architecture`. In `project_exp0_local_mlflow` an overview of MLOps processes and MLFlow running in local is examplified

### Spin up a MLFlow Service
Running MLFlow locally is simple, however, it cannot be leveraged in both `data science workbench` and runner environments and not every feature is available in local mode. Therefore, docker-compose is used in order to set a MLFlow Tracking server that will be the core of the `MLOps Platform`. This process implemented in `tracking_server_mlflow` and it is a portable (to other projects) folder 

## Full MLOps Cycle Example
In `project_exp1_jupyter` the complete cycle is demostrated in the following way:
* `data science workbench` is spin up in order to provide a tool to develop source code and subsequently build a image that will contain the train and prediction pipeline
* Track experiments and define a training/prediction policy leveraing tools in `MLOps Platform`
    * Each time a new model (algorithm, set of features or new hyperameters) is trained, each experiment is tracked in MLFlow by logging each run (individual experiments)
    * Therefore, a model version is obtained when model kind, set of features and hyperparameters configuration is set
    * A given model version is periodically re-trained on new data to create a new model instance. Then a data date is suffixed to model name
    <center>`model_name = $instance_name_$version_$data_date`
</center>

* When a model instance is obtained it is stored in MLFlow server and then deployed in a prediction service



## Architecture
* MLOps Platform:
    * Experiment tracking and artifact store MLFlow sever: [Scenario 4: MLflow with remote Tracking Server, backend and artifact stores
](https://mlflow.org/docs/latest/tracking.html)
        * Backend: Database of MLFLow entities (Postgres) and artifact store (MinIO)
        * Frontend: MLFlow GUI
* Development environment (set up specifically for a given project)
* Train Service: An image that runs a re-training job, connected to MLFLow server
* Serving Service
    * Model Backend
        * Batch: Runs a prediction job
        * REST-API: Spins up a Inference REST-API
    * Monitoring backend: Stores predictions and analyze data drift


## Services:
* Tracking Server
    * MLFlow at port 5000
    * PGAdmin at port 80  
* Developement Environment
    * Jupyterlab at port 8000  
    
    Ports are defined in ./tracking_serer_mlflow/.env and .project_exp1_jupyter/.env
* Run Train Job (TODO)
* Run Batch Inference Job (TODO)
* Run REST-API Inference Service (TODO)

## Examples
See `examples` folder in `tracking_server_mlflow` to run mlflow train logging and prediction serving (batch and REST-API)  examples

In addition, check `project_exp1_jupyter/workbench` to get a glimpse of proposed project structure. 


# References
(Connect PGAdmin to PG DB in docker-compose)[https://belowthemalt.com/2021/06/09/run-postgresql-and-pgadmin-in-docker-for-local-development-using-docker-compose/]