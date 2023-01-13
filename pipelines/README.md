---
layout: default
title: Airflow
parent: Data Extraction
---
## Airflow

### Setup
To test flows locally, make sure you have the following dependencies installed on your machine:
- docker
- docker-compose (>1.27)
- gcloud

The way we develop using Airflow roughly follows the logic described here:
https://databand.ai/blog/everyday-data-engineering-tips-on-developing-production-pipelines-in-airflow/

You may need to first run

    mkdir -p logs secrets
    echo -e "AIRFLOW_UID=$(id -u)\nAIRFLOW_GID=0" > .env

Go to docker_compose file and change  

    image: ${AIRFLOW_IMAGE_NAME:-X/airflow:X.X.X} 
    to 
    image: ${AIRFLOW_IMAGE_NAME:-oumaima/airflow}

We need to build an image (once)

    docker build -t oumaima/airflow .

Run this command the first time, it will build our custom Airflow dev image and
initialize the metadata database.

    docker-compose up airflow-init

Then to daemonize Airflow

    docker-compose up -d

Finally, you can log into the worker container using `./airflow.sh bash` or
visit `localhost:3000` for the Airflow webserver.

The default credentials are `airflow:airflow`.

### For mac users

Check if all the containers are healthy especially the airflow webserver because the default configuration of Docker desktop for Mac doesn't allocate enough memory to run webserver
that fails and restarts in infinite loop.

    docker ps

Make sure you increase the Docker Desktop memory setting

You can find the instructions here: https://docs.docker.com/desktop/mac/#resources

### DAG versioning

Each DAG should have its version stored in variable in format `<major_version>.<minor_version>.<patch_version>`.
When changes are made to DAG, version also should be updated especially when it effects output data.

- patch_version - any fixes
- minor_version - backward compatible changes
- major_version - not compatible changes

### Run a DAG
Testing that a DAG is correctly formed

    ./airflow.sh python dags/your_dag.py

#### Note 
To no longer have "examples" in your Airflow Workspace
Go to : cd ~/airflow , and change : 
    load_examples = True -> load_examples = False 

And go to docker-compose.yaml , and change :

        AIRFLOW__CORE__LOAD_EXAMPLES: 'true' -> AIRFLOW__CORE__LOAD_EXAMPLES: 'false'