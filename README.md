├── .gitattributes
├── .github
    ├── README-config.md
    └── workflows
    │   └── upload-spark-script-to-bucket.yml
├── .gitignore
├── LICENSE
├── README.md
├── airflow
    ├── .airflowignore
    ├── .env
    ├── .gitignore
    ├── Dockerfile
    ├── README.md
    ├── __init__.py
    ├── dags
    │   ├── common_package
    │   │   ├── __init__.py
    │   │   ├── __pycache__
    │   │   │   ├── __init__.cpython-38.pyc
    │   │   │   └── utils_module.cpython-38.pyc
    │   │   └── utils_module.py
    │   └── custom_dags
    │   │   ├── __init__.py
    │   │   ├── reviews_ingest_dag.py
    │   │   ├── store_and_reviews_ingest_dag.py
    │   │   └── store_ingest_dag.py
    ├── docker-compose.yaml
    ├── requirements.txt
    └── scripts
    │   └── entrypoint.sh
├── dbt
    ├── .gitignore
    ├── README.md
    ├── analyses
    │   └── .gitkeep
    ├── dbt_project.yml
    ├── macros
    │   ├── .gitkeep
    │   ├── fix_bools.sql
    │   └── fix_strings.sql
    ├── models
    │   ├── analysis
    │   │   ├── devs_metacritic.sql
    │   │   ├── dlcs_by_game.sql
    │   │   ├── games_by_genre.sql
    │   │   └── steam_games_analysis.sql
    │   ├── core
    │   │   ├── steam_dlc_data.sql
    │   │   ├── steam_games.sql
    │   │   └── steam_reviews.sql
    │   ├── schema.yml
    │   └── staging
    │   │   ├── categories.sql
    │   │   ├── developers.sql
    │   │   ├── genres.sql
    │   │   ├── publishers.sql
    │   │   ├── steam_spy_scrap.sql
    │   │   └── steam_store_data.sql
    ├── seeds
    │   └── .gitkeep
    ├── snapshots
    │   └── .gitkeep
    └── tests
    │   ├── .gitkeep
    │   └── assert_release_year_is_valid.sql
├── img
    ├── airflow_gant.png
    ├── airflow_graph.png
    ├── airflow_grid.png
    ├── dashboard.png
    ├── dbt_linege.png
    ├── lineage_dbt_1.png
    ├── lineage_dbt_2.png
    ├── lineage_dbt_3.png
    ├── owners_new.png
    ├── owners_old.png
    ├── schema_denorm.png
    ├── steam.jpg
    └── steam_logo.png
├── spark
    ├── .gitignore
    ├── README.md
    ├── all_games_reviews_gcp.ipynb
    └── spark_all_games_reviews.py
└── terraform
    ├── .gitignore
    ├── README.md
    ├── operator-workspace
        ├── main.tf
        └── variables.tf
    └── vault-admin-workspace
        └── main.tf


/.gitattributes:
--------------------------------------------------------------------------------
1 | *.sql linguist-detectable=true


--------------------------------------------------------------------------------
/.github/README-config.md:
--------------------------------------------------------------------------------
 1 | 
 2 | ## Generating OAUTH 2.0 access tokens for Github Actions
 3 | 
 4 | - https://github.com/google-github-actions/auth
 5 | 
 6 | ```{bash}
 7 | export PROJECT_ID="steam-data-engineering-gcp" # update with your value
 8 | 
 9 | gcloud iam service-accounts create "github-actions-service-account" \
10 |   --project "${PROJECT_ID}"
11 | 
12 | gcloud services enable iamcredentials.googleapis.com \
13 |   --project "${PROJECT_ID}"
14 |   
15 | gcloud iam workload-identity-pools create "github-action-pool" \
16 |   --project="${PROJECT_ID}" \
17 |   --location="global" \
18 |   --display-name="Github Actions pool"
19 |   
20 | gcloud iam workload-identity-pools describe "github-action-pool" \
21 |   --project="${PROJECT_ID}" \
22 |   --location="global" \
23 |   --format="value(name)"
24 | ```
25 | 
26 | ```{bash}
27 | export WORKLOAD_IDENTITY_POOL_ID="projects/146724372394/locations/global/workloadIdentityPools/github-action-pool"
28 | ```
29 | 
30 | ```{bash}
31 | gcloud iam workload-identity-pools providers create-oidc "github-action-provider" \
32 |   --project="${PROJECT_ID}" \
33 |   --location="global" \
34 |   --workload-identity-pool="github-action-pool" \
35 |   --display-name="Github Action provider" \
36 |   --attribute-mapping="google.subject=assertion.sub,attribute.actor=assertion.actor,attribute.repository=assertion.repository" \
37 |   --issuer-uri="https://token.actions.githubusercontent.com"
38 | ```
39 | 
40 | ```{bash}
41 | export REPO="VicenteYago/steam-data-engineering" 
42 | ```
43 | 
44 | ```{bash}
45 | gcloud iam service-accounts add-iam-policy-binding "github-actions-service-account@${PROJECT_ID}.iam.gserviceaccount.com" \
46 |   --project="${PROJECT_ID}" \
47 |   --role="roles/iam.workloadIdentityUser" \
48 |   --member="principalSet://iam.googleapis.com/${WORKLOAD_IDENTITY_POOL_ID}/attribute.repository/${REPO}"
49 | ```
50 | 
51 | ```{bash}
52 | gcloud iam workload-identity-pools providers describe "github-action-provider" \
53 |   --project="${PROJECT_ID}" \
54 |   --location="global" \
55 |   --workload-identity-pool="github-action-pool" \
56 |   --format="value(name)"
57 | ```
58 | 
59 | * `projects/146724372394/locations/global/workloadIdentityPools/github-action-pool/providers/github-action-provider`
60 | * `github-actions-service-account@steam-data-engineering-gcp.iam.gserviceaccount.com`
61 | 
62 | ```{bash}
63 | gcloud projects add-iam-policy-binding ${PROJECT_ID} \
64 |     --member=serviceAccount:"github-actions-service-account@steam-data-engineering-gcp.iam.gserviceaccount.com" \
65 |     --role=roles/storage.admin
66 | ```
67 | 
68 | 


--------------------------------------------------------------------------------
/.github/workflows/upload-spark-script-to-bucket.yml:
--------------------------------------------------------------------------------
 1 | name: Upload Spark script to GCP bucket
 2 | on:
 3 |   push:
 4 |     branches:
 5 |     - main
 6 | 
 7 | jobs:
 8 |   upload_spark_script:
 9 |     runs-on: ubuntu-20.04
10 |     permissions:  
11 |       contents: 'read'
12 |       id-token: 'write'
13 |     
14 |     steps:
15 |     - name: checkout
16 |       uses: actions/checkout@v3
17 |       
18 |     - name: auth
19 |       uses: 'google-github-actions/auth@v0'
20 |       with:
21 |         workload_identity_provider: 'projects/146724372394/locations/global/workloadIdentityPools/github-action-pool/providers/github-action-provider'
22 |         service_account: 'github-actions-service-account@steam-data-engineering-gcp.iam.gserviceaccount.com' 
23 |     
24 |     - name: upload-file
25 |       uses: 'google-github-actions/upload-cloud-storage@v0'
26 |       with:
27 |         path: 'spark/spark_all_games_reviews.py'
28 |         destination: 'steam-datalake-reviews/'
29 | 


--------------------------------------------------------------------------------
/.gitignore:
--------------------------------------------------------------------------------
1 | steam-dataset/
2 | reviews/
3 | reviews_lite/
4 | reviews_lite_proc/
5 | steam-dataset.zip
6 | steam-reviews.zip


--------------------------------------------------------------------------------
/LICENSE:
--------------------------------------------------------------------------------
 1 | MIT License
 2 | 
 3 | Copyright (c) 2022 VicenteYago
 4 | 
 5 | Permission is hereby granted, free of charge, to any person obtaining a copy
 6 | of this software and associated documentation files (the "Software"), to deal
 7 | in the Software without restriction, including without limitation the rights
 8 | to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
 9 | copies of the Software, and to permit persons to whom the Software is
10 | furnished to do so, subject to the following conditions:
11 | 
12 | The above copyright notice and this permission notice shall be included in all
13 | copies or substantial portions of the Software.
14 | 
15 | THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
16 | IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
17 | FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
18 | AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
19 | LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
20 | OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
21 | SOFTWARE.
22 | 


--------------------------------------------------------------------------------
/README.md:
--------------------------------------------------------------------------------
 1 | # Steam Data Engineering
 2 | 
 3 | <p align="center">
 4 | <img src="https://user-images.githubusercontent.com/16523144/190527411-9fd2439e-3516-4199-97ef-9fda8fd733b3.png" width="100" height="100">
 5 | </p>
 6 | 
 7 | 
 8 | 
 9 | ## Summary
10 | 
11 | This project creates a pipeline for processing and analyze the raw data scraped from the videogame digitial distribution platform [**Steam**](https://store.steampowered.com/). The data is procesed in an **ELT** pipeline with the objective of provide insight about the best selling videogames since 2006. 
12 | 
13 | 
14 | ## Dataset
15 | 
16 | The dataset is originary from [**Kaggle**](https://www.kaggle.com/datasets/souyama/steam-dataset), a repository of free datasets for data science. The dataset is composed as a mix of directly scraped records from **Steam** itself and [**Steamspy**](https://steamspy.com/), a website dedicated to collect actual and speculative data about the former platform. All the files are enconded as raw jsons and need and extensive work of transformation in order to make then usable. Additionally all the **reviews** for each game are also retrieved from [**another Kaggle dataset**](https://www.kaggle.com/datasets/souyama/steam-reviews).
17 | 
18 | ## Tools & Technologies
19 | * Cloud - [**Google Cloud Platform**](https://cloud.google.com/)
20 | * Infraestructure as Code - [**Terraform**](https://www.terraform.io/)
21 | * Containerization - [**Docker**](https://www.docker.com/), [**Docker Compose**](https://docs.docker.com/compose/)
22 | * Orchestration - [**Airflow**](https://airflow.apache.org/)
23 | * Transformation - [**Spark**](https://spark.apache.org/)
24 | * Transformation - [**dbt**](https://www.getdbt.com/)
25 | * Data Lake - [**Google Cloud Storage**](https://cloud.google.com/storage)
26 | * Data Warehouse - [**BigQuery**](https://cloud.google.com/bigquery)
27 | * Data Visualization - [**PowerBI**](https://powerbi.microsoft.com/en-au/)
28 | * Languages - **Python**, **SQL**
29 | 
30 | ## Architecture
31 | 
32 | To add more realism the datasets are placed in a **S3 bucket in AWS**. The **trasfer from AWS to GCP** is done trough **Terraform** as part of the initialization tasks, a one-time operation. 
33 | 
34 | ![](https://github.com/VicenteYago/steam-data-engineering/blob/main/img/steam.jpg)
35 | 
36 | Since the **reviews** datasets is ~ 40 GB and ~ 500k files a special processing in **Spark** is performed. 
37 | The **Airflow** server is located in a vitual machine deployed in **Compute Engine GCP** .
38 | 
39 | ## Dashboard
40 | 
41 | For a more compacted visualization only games with metacritic score are displayed (3.5k out of 51k).
42 | 
43 | ![](https://github.com/VicenteYago/steam-data-engineering/blob/main/img/dashboard.png)
44 |  
45 |  A note about **Revenue**: 
46 |  The revenue is calculated as a product of `owners * price`. This is a pretty naive approximation, see [Steamspy guidelines about "Owned" and "Sold" equivalence](https://steamspy.com/about).
47 |  
48 |  
49 | ## Orchestration
50 | 
51 | The **DAG** has to main branches : 
52 | 
53 | * **Store branch** : selected JSON files are converted into parquet and then loaded as tables in **Bigquery** in parallel.
54 | * **Reviews branch** : JSON files are compacted before the cluster is created, then processed in dataproc as a **pyspark job**. Finally the results are injected into **Bigquery**, where they are merged with other tables using **dbt**. Cluster creation and deletion is fully automated.
55 | 
56 | Both branches start by retrieving the respective data from the **GCS Buckets**, and they converge in the **dbt** task wich builds the definitive tables in **BigQuery**.
57 | 
58 | ![](https://github.com/VicenteYago/steam-data-engineering/blob/main/img/airflow_graph.png)
59 | 
60 | 
61 | ## Transformation
62 | Transformations are done with Spark (20%) and dbt (80%).
63 | 
64 | ### Spark 
65 | 
66 | The spark job consists in process all 500k JSON files of reviews in a temporal **Dataproc** cluster and write them in a **Bigquery** table.
67 | 
68 | ### dbt 
69 | All the transformation for the steam store data is done in **dbt** using **SQL** intensively. The main table `steam_games` is built following the [google recommendations](https://cloud.google.com/blog/topics/developers-practitioners/bigquery-explained-working-joins-nested-repeated-data), i.e. is a **denormalized table** featuring nested an repeated structers to avoid the cost of performing joins.
70 | 
71 | ![](https://github.com/VicenteYago/steam-data-engineering/blob/main/img/lineage_dbt_1.png)
72 | 
73 | In order to built the dashboard a custom unnested tables are also built from `steam_games`. 
74 | 
75 | ## Tests
76 | 
77 | In the final stage of the pipeline, inside the **dbt** task some checks are performed: 
78 | 
79 | * Checks for uniqueness and non-null for the `gameid` column in all affected tables. 
80 | * Checking for consistency of `release_date` column in table `steam_games`
81 | 
82 | ## Improvements 
83 | - ~partition core tables, for performance~
84 | - Add a cost analysis of a full pipeline run
85 | - extract more value from steam reviews --> new dashboard ? 
86 | - USE CASE: matrix factorization for recommendations
87 | - ~github action for upload spark script to bucket~
88 | - More tests on the pipeline
89 | - Fully normalize the tables as exercise 
90 | 
91 | 
92 | ## Special Mentions
93 | Thanks to Alexey and his community from [datatalks club](https://datatalks.club/), create this project without the course of [Data Engineering](https://github.com/DataTalksClub/data-engineering-zoomcamp) would have been much more difficult.  
94 | 


--------------------------------------------------------------------------------
/airflow/.airflowignore:
--------------------------------------------------------------------------------
1 | airflow/dags/common_package


--------------------------------------------------------------------------------
/airflow/.env:
--------------------------------------------------------------------------------
1 | AIRFLOW_UID=1000
2 | 


--------------------------------------------------------------------------------
/airflow/.gitignore:
--------------------------------------------------------------------------------
1 | logs/
2 | plugins/
3 | dags/__pycache__
4 | .kaggle/kaggle.json
5 | dags/common_package/__pycache__
6 | dags/custom_dags/__pycache__


--------------------------------------------------------------------------------
/airflow/Dockerfile:
--------------------------------------------------------------------------------
 1 | # First-time build can take upto 10 mins.
 2 | 
 3 | #FROM apache/airflow:2.3.4
 4 | 
 5 | FROM apache/airflow:2.3.4-python3.8
 6 | 
 7 | ENV AIRFLOW_HOME=/opt/airflow
 8 | 
 9 | USER root
10 | RUN apt-get update -qq && apt-get install vim -qqq && apt-get install unzip
11 | # git gcc g++ -qqq
12 | 
13 | USER airflow
14 | COPY requirements.txt .
15 | RUN pip install --no-cache-dir -r requirements.txt
16 | #RUN mkdir ~/.kaggle/
17 | #COPY .kaggle /home/airflow/.kaggle
18 | 
19 | # Ref: https://airflow.apache.org/docs/docker-stack/recipes.html
20 | SHELL ["/bin/bash", "-o", "pipefail", "-e", "-u", "-x", "-c"]
21 | 
22 | ARG CLOUD_SDK_VERSION=322.0.0
23 | ENV GCLOUD_HOME=/home/google-cloud-sdk
24 | ENV PYTHON_VERSION=3.8.13
25 | 
26 | USER root
27 | ENV PATH="${GCLOUD_HOME}/bin/:${PATH}"
28 | RUN DOWNLOAD_URL="https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-sdk-${CLOUD_SDK_VERSION}-linux-x86_64.tar.gz" \
29 |     && TMP_DIR="$(mktemp -d)" \
30 |     && curl -fL "${DOWNLOAD_URL}" --output "${TMP_DIR}/google-cloud-sdk.tar.gz" \
31 |     && mkdir -p "${GCLOUD_HOME}" \
32 |     && tar xzf "${TMP_DIR}/google-cloud-sdk.tar.gz" -C "${GCLOUD_HOME}" --strip-components=1 \
33 |     && "${GCLOUD_HOME}/install.sh" \
34 |        --bash-completion=false \
35 |        --path-update=false \
36 |        --usage-reporting=false \
37 |        --quiet \
38 |     && rm -rf "${TMP_DIR}" \
39 |     && gcloud --version
40 | 
41 | WORKDIR $AIRFLOW_HOME
42 | 
43 | COPY scripts scripts
44 | RUN chmod +x scripts
45 | 
46 | USER $AIRFLOW_UID


--------------------------------------------------------------------------------
/airflow/README.md:
--------------------------------------------------------------------------------
 1 | # Airflow
 2 | 
 3 | 
 4 | ## Description
 5 | 
 6 | * **Execution graph (DAG)** : 
 7 | ![](https://github.com/VicenteYago/steam-data-engineering/blob/main/img/airflow_graph.png)
 8 | 
 9 | * **Grid** : 
10 | ![](https://github.com/VicenteYago/steam-data-engineering/blob/main/img/airflow_grid.png)
11 | 
12 | * **Gantt** : 
13 | ![](https://github.com/VicenteYago/steam-data-engineering/blob/main/img/airflow_gant.png)
14 | 
15 | 
16 | ## **Config**
17 | 
18 | ### dbt config necessary to run the final task 
19 | 
20 | 1. Create a service account in GCP and export credentials into `~/.google/credentials/google_credentials.json`
21 | 2. Create `~/.dbt/profiles.yml`: 
22 | 
23 | ```{yml}
24 | #profiles.yml
25 | default:
26 |   target: dev
27 |   outputs:
28 |     dev:
29 |       type: bigquery
30 |       method: service-account
31 |       project: steam-data-engineering-gcp
32 |       dataset: dbt_development
33 |       threads: 1
34 |       keyfile: '/home/vyago/.google/credentials/google_credentials.json'
35 |       location: europe-southwest1
36 | 
37 | airflow:
38 |   target: dev
39 |   outputs:
40 |     dev:
41 |       type: bigquery
42 |       method: service-account
43 |       project: steam-data-engineering-gcp
44 |       dataset: dbt_development
45 |       threads: 1
46 |       keyfile: '/.google/credentials/google_credentials.json'
47 |       location: europe-southwest1
48 | ```
49 | 
50 | 
51 | ### Building the Airflow service
52 | 
53 | - https://airflow.apache.org/docs/apache-airflow/stable/howto/docker-compose/index.html
54 | 
55 | Built & run : 
56 | ```{bash}
57 | sudo docker compose build --no-cache
58 | sudo docker compose up airflow-init
59 | sudo docker compose up -d 
60 | chmod 777 dbt
61 | ```
62 | 
63 | Clean: 
64 | ```{bash}
65 | sudo docker compose down --volumes --rmi all
66 | ```
67 | 


--------------------------------------------------------------------------------
/airflow/__init__.py:
--------------------------------------------------------------------------------
https://raw.githubusercontent.com/VicenteYago/steam-data-engineering/d991376ae7a2ec16970b65c81f724830704221c1/airflow/__init__.py


--------------------------------------------------------------------------------
/airflow/dags/common_package/__init__.py:
--------------------------------------------------------------------------------
https://raw.githubusercontent.com/VicenteYago/steam-data-engineering/d991376ae7a2ec16970b65c81f724830704221c1/airflow/dags/common_package/__init__.py


--------------------------------------------------------------------------------
/airflow/dags/common_package/__pycache__/__init__.cpython-38.pyc:
--------------------------------------------------------------------------------
https://raw.githubusercontent.com/VicenteYago/steam-data-engineering/d991376ae7a2ec16970b65c81f724830704221c1/airflow/dags/common_package/__pycache__/__init__.cpython-38.pyc


--------------------------------------------------------------------------------
/airflow/dags/common_package/__pycache__/utils_module.cpython-38.pyc:
--------------------------------------------------------------------------------
https://raw.githubusercontent.com/VicenteYago/steam-data-engineering/d991376ae7a2ec16970b65c81f724830704221c1/airflow/dags/common_package/__pycache__/utils_module.cpython-38.pyc


--------------------------------------------------------------------------------
/airflow/dags/common_package/utils_module.py:
--------------------------------------------------------------------------------
  1 | import os
  2 | import json
  3 | import glob
  4 | import sys
  5 | import dask.dataframe as dd
  6 | from google.api_core import page_iterator
  7 | from google.cloud import storage
  8 | 
  9 | from airflow import DAG
 10 | from airflow.utils.dates import days_ago
 11 | from airflow.utils.dates import days_ago
 12 | from airflow.operators.bash import BashOperator
 13 | from airflow.operators.python import PythonOperator
 14 | from airflow.providers.google.cloud.hooks.gcs import GCSHook
 15 | from airflow.providers.google.cloud.operators.dataproc import DataprocDeleteClusterOperator
 16 | from airflow.providers.google.cloud.operators.dataproc import DataprocCreateClusterOperator
 17 | from airflow.providers.google.cloud.operators.bigquery import BigQueryCreateExternalTableOperator
 18 | from airflow.providers.google.cloud.operators.dataproc import DataprocSubmitPySparkJobOperator
 19 | 
 20 | 
 21 | 
 22 | def download_gcs(bucket, local_dir, prefix=None):
 23 | 
 24 | #        os.makedirs(local_dir)
 25 | 
 26 |     hook = GCSHook()
 27 |     gcs_objs_list = hook.list(bucket_name = bucket, prefix = prefix)
 28 |     for obj in gcs_objs_list : 
 29 |         if (obj.endswith(".json")):
 30 |             local_obj = os.path.join(local_dir, obj)
 31 |             if not os.path.exists(local_obj):
 32 |                 os.makedirs(os.path.dirname(local_obj), exist_ok=True)
 33 | 
 34 |             hook.download(bucket_name = bucket,
 35 |                          object_name = obj,
 36 |                          filename = local_obj)
 37 | 
 38 | def modify_path(path, old_prefix, new_prefix) : 
 39 |    return path.replace(old_prefix, new_prefix)
 40 | 
 41 | 
 42 | def upload_gcs(bucket, local_dir):
 43 |     """
 44 |     Ref: https://cloud.google.com/storage/docs/uploading-objects#storage-upload-object-python
 45 |     :param bucket: GCS bucket name
 46 |     :param object_name: target path & file-name
 47 |     :param local_file: source path & file-name
 48 |     :return:
 49 |     """
 50 |     hook = GCSHook()
 51 |     for path, _, files in os.walk(local_dir):
 52 |         for file in files:
 53 |             if file.endswith(".parquet"):
 54 |                 filename = os.path.join(path, file)
 55 | 
 56 |                 object_name = modify_path(filename, "raw", "proc")
 57 |                 object_name = os.path.relpath(object_name, local_dir)
 58 | 
 59 |                 hook.upload(bucket_name = bucket,
 60 |                             filename = filename,
 61 |                             object_name = object_name)
 62 | 
 63 | def format_to_parquet(local_dir):
 64 | 
 65 |     for path, _, files in os.walk(local_dir):
 66 |         for file in files:
 67 |             if file.endswith(".json"):
 68 |                 file_path = os.path.join(path, file)
 69 |                 file_path_parquet = file_path.replace('.json', '.parquet')
 70 |                 ddf = dd.read_json(file_path, orient='index')
 71 |                 ddf = ddf.astype('string') 
 72 |                 ddf.to_parquet(file_path_parquet, compression='snappy')
 73 | 
 74 | #https://stackoverflow.com/questions/37074977/how-to-get-list-of-folders-in-a-given-bucket-using-google-cloud-api
 75 | def _item_to_value(iterator, item):
 76 |     return item
 77 | 
 78 | def list_directories(bucket_name, prefix):
 79 |     if prefix and not prefix.endswith('/'):
 80 |         prefix += '/'
 81 | 
 82 |     extra_params = {
 83 |         "projection": "noAcl",
 84 |         "prefix": prefix,
 85 |         "delimiter": '/'
 86 |     }
 87 | 
 88 |     gcs = storage.Client()
 89 | 
 90 |     path = "/b/" + bucket_name + "/o"
 91 | 
 92 |     iterator = page_iterator.HTTPIterator(
 93 |         client=gcs,
 94 |         api_request=gcs._connection.api_request,
 95 |         path=path,
 96 |         items_key='prefixes',
 97 |         item_to_value=_item_to_value,
 98 |         extra_params=extra_params,
 99 |     )
100 |     return [x for x in iterator]
101 | 
102 | 
103 | def n_files_in_bucket_dir(bucket, prefix, client):
104 |     blob_list = client.list_blobs(bucket_or_name=bucket, prefix = prefix)
105 |     return(sum([1 for i in blob_list]))
106 | 
107 | def compact_dir(input_dir, output_dir, filename):
108 |     glob_data = []
109 |     for file in glob.glob(os.path.join(input_dir, "*.json")):
110 |         with open(file) as json_file:
111 |             data = json.load(json_file)
112 |             glob_data.append(data)
113 |     with open(os.path.join(output_dir, filename), 'w') as f:
114 |         json.dump(glob_data, f, indent=4)
115 | 
116 | def filter_dir_by_nfiles(dir_list, bucket, gcs_client): 
117 |     review_pgs_by_game_filter = { 
118 |         dir : v  for dir in dir_list if (v := n_files_in_bucket_dir(bucket, dir, gcs_client)) > 1
119 |     }
120 |     return review_pgs_by_game_filter
121 | 
122 | 
123 | 
124 | def upload_gcs_reviews(bucket, local_dir):  # shared function used in dag_data_ingestion.py
125 |     """
126 |     Ref: https://cloud.google.com/storage/docs/uploading-objects#storage-upload-object-python
127 |     :param bucket: GCS bucket name
128 |     :param object_name: target path & file-name
129 |     :param local_file: source path & file-name
130 |     :return:
131 |     """
132 |     hook = GCSHook()
133 |     for path, _, files in os.walk(local_dir):
134 |         for file in files:
135 |             if file.endswith(".json"): # !!!! MODIFIED ".parquet"
136 |                 filename = os.path.join(path, file)
137 | 
138 |                 object_name = os.path.join(BUCKET_SUBDIR_PROC, file)
139 | 
140 |                 hook.upload(bucket_name = bucket,
141 |                             filename = filename,
142 |                             object_name = object_name)
143 | 
144 | 
145 | def build_compacted_reviews_dataset(bucket,
146 |                                     bucket_subdir,
147 |                                     reviews_local_dir,
148 |                                     reviews_local_dir_proc): 
149 | 
150 |     # list all directories in gcp bucket 
151 |     gcs_client = storage.Client()
152 |     dir_list = list_directories(bucket, prefix=bucket_subdir)
153 | 
154 |     # calculate dict w/ dirs that contains more than 1 file
155 |     dir_list_filter_dict = filter_dir_by_nfiles(dir_list, bucket, gcs_client)
156 | 
157 |     # download selected dirs in local
158 |     for dir in dir_list_filter_dict.keys() : 
159 |         download_gcs(bucket, local_dir = reviews_local_dir, prefix=dir)
160 | 
161 |     # compress local dirs into 1 json file per dir 
162 |     if not os.path.exists(reviews_local_dir_proc):
163 |         os.makedirs(reviews_local_dir_proc)
164 |     
165 |     for root, dirs, files in os.walk(os.path.join(reviews_local_dir, bucket_subdir)):
166 |         for dir in dirs:
167 |             dir_path = os.path.join(os.path.join(reviews_local_dir, bucket_subdir), dir)
168 |             compact_dir(dir_path, reviews_local_dir_proc, f'{dir}-compacted.json')
169 | 
170 |     # upload dirs to bucket in /proc folder
171 |     upload_gcs_reviews(bucket = bucket, local_dir = reviews_local_dir_proc)


--------------------------------------------------------------------------------
/airflow/dags/custom_dags/__init__.py:
--------------------------------------------------------------------------------
https://raw.githubusercontent.com/VicenteYago/steam-data-engineering/d991376ae7a2ec16970b65c81f724830704221c1/airflow/dags/custom_dags/__init__.py


--------------------------------------------------------------------------------
/airflow/dags/custom_dags/reviews_ingest_dag.py:
--------------------------------------------------------------------------------
  1 | import os
  2 | import json
  3 | import glob
  4 | import sys
  5 | import dask.dataframe as dd
  6 | from google.api_core import page_iterator
  7 | from google.cloud import storage
  8 | 
  9 | from airflow import DAG
 10 | from airflow.utils.dates import days_ago
 11 | from airflow.utils.dates import days_ago
 12 | from airflow.operators.bash import BashOperator
 13 | from airflow.operators.python import PythonOperator
 14 | from airflow.providers.google.cloud.hooks.gcs import GCSHook
 15 | from airflow.providers.google.cloud.operators.dataproc import DataprocDeleteClusterOperator
 16 | from airflow.providers.google.cloud.operators.dataproc import DataprocCreateClusterOperator
 17 | from airflow.providers.google.cloud.operators.bigquery import BigQueryCreateExternalTableOperator
 18 | from airflow.providers.google.cloud.operators.dataproc import DataprocSubmitPySparkJobOperator
 19 | 
 20 | 
 21 | sys.path.append("airflow/dags/common_package")                                                    
 22 | from common_package.utils_module import *
 23 | 
 24 | AIRFLOW_HOME = os.environ.get("AIRFLOW_HOME", "/opt/airflow/")
 25 | PROJECT_ID = os.environ.get("GCP_PROJECT_ID")
 26 | BUCKET_STORE = os.environ.get("GCP_GCS_BUCKET_STORE")
 27 | BUCKET_REVIEWS = os.environ.get("GCP_GCS_BUCKET_REVIEWS")
 28 | BUCKET_SUBDIR = "raw"
 29 | BUCKET_SUBDIR_PROC = "proc"
 30 | BIGQUERY_DATASET = os.environ.get('GCP_BQ_DATASET')
 31 | DATASETS_LOCAL_DIR = os.path.join(AIRFLOW_HOME, "datasets/") 
 32 | STEAM_STORE_LOCAL_DIR = os.path.join(DATASETS_LOCAL_DIR, "steamstore")
 33 | REVIEWS_LOCAL_DIR = os.path.join(DATASETS_LOCAL_DIR, "reviews")
 34 | REVIEWS_LOCAL_DIR_PROC = os.path.join(DATASETS_LOCAL_DIR, "reviews_proc")
 35 | DBT_DIR = os.path.join(AIRFLOW_HOME, "dbt")
 36 | 
 37 | BQ_TABLE = 'steam_reviews'
 38 | TEMP_BUCKET = 'steam-datalake-store-alldata'
 39 | JOB_ARGUMENTS = [PROJECT_ID, BIGQUERY_DATASET, BQ_TABLE, BUCKET_REVIEWS, BUCKET_SUBDIR_PROC, TEMP_BUCKET]
 40 | 
 41 | CLUSTER_NAME = "steam-reviews-spark-cluster"
 42 | CLUSTER_REGION = "europe-west1"
 43 | PYSPARK_FILE = "spark_all_games_reviews.py" # decouple in a separate (previous) task
 44 | 
 45 | CLUSTER_CONFIG = {
 46 |     "master_config": {
 47 |         "num_instances": 1,
 48 |         "machine_type_uri": "n1-standard-2",
 49 |         "disk_config": {"boot_disk_type": "pd-standard", "boot_disk_size_gb": 50},
 50 |     },
 51 |     "worker_config": {
 52 |         "num_instances": 2,
 53 |         "machine_type_uri": "n1-standard-2",
 54 |         "disk_config": {"boot_disk_type": "pd-standard", "boot_disk_size_gb": 50},
 55 |     },
 56 | }
 57 | 
 58 | default_args = {
 59 |     "owner": "airflow",
 60 |     "start_date": days_ago(1),
 61 |     "depends_on_past": False,
 62 |     "retries": 1,
 63 | }
 64 | 
 65 | with DAG(
 66 |     dag_id="steam_eviews_ingestion_dag",
 67 |     schedule_interval="@weekly",
 68 |     default_args=default_args,
 69 |     catchup=False,
 70 |     max_active_runs=1,
 71 |     tags=['steam-de'],
 72 | ) as dag:
 73 | 
 74 |     #-------------------------------------------REVIEWS
 75 | 
 76 |     download_steam_reviews_from_datalake_task = PythonOperator(
 77 |         task_id="download_steam_reviews_from_datalake_task",
 78 |         python_callable=build_compacted_reviews_dataset,
 79 |         op_kwargs={
 80 |             "bucket": BUCKET_REVIEWS,
 81 |             "bucket_subdir": BUCKET_SUBDIR,
 82 |             "reviews_local_dir" : REVIEWS_LOCAL_DIR,
 83 |             "reviews_local_dir_proc" : REVIEWS_LOCAL_DIR_PROC
 84 |         }
 85 |     )
 86 | 
 87 |     create_cluster_dataproc_task = DataprocCreateClusterOperator(
 88 |         task_id="create_cluster_dataproc_task",
 89 |         project_id=PROJECT_ID,
 90 |         cluster_config=CLUSTER_CONFIG,
 91 |         region=CLUSTER_REGION,
 92 |         trigger_rule="all_success",
 93 |         cluster_name=CLUSTER_NAME
 94 |     )
 95 | 
 96 |     submit_spark_job_task = DataprocSubmitPySparkJobOperator(
 97 |         task_id = "submit_dataproc_spark_job_task",
 98 |         main = f"gs://{BUCKET_REVIEWS}/{PYSPARK_FILE}",
 99 |         arguments = JOB_ARGUMENTS,
100 |         cluster_name = CLUSTER_NAME,
101 |         region = CLUSTER_REGION,
102 |         dataproc_jars = ["gs://spark-lib/bigquery/spark-3.1-bigquery-0.27.0-preview.jar"]
103 |     )
104 | 
105 |     delete_cluster_dataproc_task = DataprocDeleteClusterOperator(
106 |         task_id="delete_cluster_dataproc_task",
107 |         project_id=PROJECT_ID,
108 |         cluster_name=CLUSTER_NAME,
109 |         region=CLUSTER_REGION,
110 |         trigger_rule="all_done"
111 |     )
112 | 
113 |     rm_reviews_task = BashOperator(
114 |         task_id='remove_reviews_files_from_local',
115 |         bash_command=f'rm -rf {REVIEWS_LOCAL_DIR} {REVIEWS_LOCAL_DIR_PROC}',
116 |         trigger_rule="all_done"
117 |     )
118 | 
119 |     #-------------------------------------------DBT
120 |     run_dbt_task = BashOperator(
121 |     task_id='run_dbt',
122 |     bash_command=f'cd {DBT_DIR} && dbt run --profile airflow',
123 |     trigger_rule="all_success"
124 |     )
125 | 
126 | 
127 |     download_steam_reviews_from_datalake_task >> create_cluster_dataproc_task >> submit_spark_job_task >> delete_cluster_dataproc_task >> rm_reviews_task >> run_dbt_task
128 |     


--------------------------------------------------------------------------------
/airflow/dags/custom_dags/store_and_reviews_ingest_dag.py:
--------------------------------------------------------------------------------
  1 | import os
  2 | import json
  3 | import glob
  4 | import sys
  5 | import dask.dataframe as dd
  6 | from google.api_core import page_iterator
  7 | from google.cloud import storage
  8 | 
  9 | from airflow import DAG
 10 | from airflow.utils.dates import days_ago
 11 | from airflow.utils.dates import days_ago
 12 | from airflow.operators.bash import BashOperator
 13 | from airflow.operators.python import PythonOperator
 14 | from airflow.providers.google.cloud.hooks.gcs import GCSHook
 15 | from airflow.providers.google.cloud.operators.dataproc import DataprocDeleteClusterOperator
 16 | from airflow.providers.google.cloud.operators.dataproc import DataprocCreateClusterOperator
 17 | from airflow.providers.google.cloud.operators.bigquery import BigQueryCreateExternalTableOperator
 18 | from airflow.providers.google.cloud.operators.dataproc import DataprocSubmitPySparkJobOperator
 19 | 
 20 | sys.path.append("airflow/dags/common_package")                                                    
 21 | from common_package.utils_module import *
 22 | 
 23 | AIRFLOW_HOME = os.environ.get("AIRFLOW_HOME", "/opt/airflow/")
 24 | PROJECT_ID = os.environ.get("GCP_PROJECT_ID")
 25 | BUCKET_STORE = os.environ.get("GCP_GCS_BUCKET_STORE")
 26 | BUCKET_REVIEWS = os.environ.get("GCP_GCS_BUCKET_REVIEWS")
 27 | BUCKET_SUBDIR = "raw"
 28 | BUCKET_SUBDIR_PROC = "proc"
 29 | BIGQUERY_DATASET = os.environ.get('GCP_BQ_DATASET')
 30 | DATASETS_LOCAL_DIR = os.path.join(AIRFLOW_HOME, "datasets/") 
 31 | STEAM_STORE_LOCAL_DIR = os.path.join(DATASETS_LOCAL_DIR, "steamstore")
 32 | REVIEWS_LOCAL_DIR = os.path.join(DATASETS_LOCAL_DIR, "reviews")
 33 | REVIEWS_LOCAL_DIR_PROC = os.path.join(DATASETS_LOCAL_DIR, "reviews_proc")
 34 | DBT_DIR = os.path.join(AIRFLOW_HOME, "dbt")
 35 | 
 36 | BQ_TABLE = 'steam_reviews'
 37 | TEMP_BUCKET = 'steam-datalake-store-alldata'
 38 | JOB_ARGUMENTS = [PROJECT_ID, BIGQUERY_DATASET, BQ_TABLE, BUCKET_REVIEWS, BUCKET_SUBDIR_PROC, TEMP_BUCKET]
 39 | 
 40 | CLUSTER_NAME = "steam-reviews-spark-cluster" #"spark-cluster-reviews"
 41 | CLUSTER_REGION = "europe-west1"
 42 | PYSPARK_FILE = "spark_all_games_reviews.py" # decouple in a separate (previous) task
 43 | 
 44 | CLUSTER_CONFIG = {
 45 |     "master_config": {
 46 |         "num_instances": 1,
 47 |         "machine_type_uri": "n1-standard-2",
 48 |         "disk_config": {"boot_disk_type": "pd-standard", "boot_disk_size_gb": 50},
 49 |     },
 50 |     "worker_config": {
 51 |         "num_instances": 2,
 52 |         "machine_type_uri": "n1-standard-2",
 53 |         "disk_config": {"boot_disk_type": "pd-standard", "boot_disk_size_gb": 50},
 54 |     },
 55 | }
 56 | 
 57 | default_args = {
 58 |     "owner": "airflow",
 59 |     "start_date": days_ago(1),
 60 |     "depends_on_past": False,
 61 |     "retries": 1,
 62 | }
 63 | 
 64 | with DAG(
 65 |     dag_id="steam_store_and_reviews_ingestion_dag",
 66 |     schedule_interval="@weekly",
 67 |     default_args=default_args,
 68 |     catchup=False,
 69 |     max_active_runs=1,
 70 |     tags=['steam-de'],
 71 | ) as dag:
 72 | 
 73 |     #-------------------------------------------STORE    
 74 |     download_steam_dataset_from_datalake_task = PythonOperator(
 75 |         task_id="download_steam_dataset_from_datalake_task",
 76 |         python_callable=download_gcs,
 77 |         op_kwargs={
 78 |             "bucket": BUCKET_STORE,
 79 |             "local_dir": STEAM_STORE_LOCAL_DIR 
 80 |         }
 81 |     )
 82 | 
 83 |     format_to_parquet_inlocal_task = PythonOperator(
 84 |         task_id="format_to_parquet_inlocal_task",
 85 |         python_callable=format_to_parquet,
 86 |         op_kwargs={
 87 |             "local_dir": STEAM_STORE_LOCAL_DIR
 88 |         }
 89 |     )
 90 | 
 91 |     rm_files_store_task = BashOperator(
 92 |         task_id="remove_needless_files_store_inlocal_task",
 93 |         bash_command=f"find {STEAM_STORE_LOCAL_DIR} -type d -name news_data -prune -exec rm -rf {{}} \; &&  find {STEAM_STORE_LOCAL_DIR} -type d -name steam_charts -prune -exec rm -rf {{}} \; && find {STEAM_STORE_LOCAL_DIR} -name 'missing.json' -delete"
 94 |     )
 95 | 
 96 |     upload_steam_dataset_proc_task = PythonOperator(
 97 |         task_id="upload_steam_dataset_proc_task",
 98 |         python_callable=upload_gcs,
 99 |         op_kwargs={
100 |             "bucket": BUCKET_STORE,
101 |             "local_dir": STEAM_STORE_LOCAL_DIR 
102 |         }
103 |     )
104 | 
105 |     hook = GCSHook()
106 |     bq_parallel_tasks = list()
107 |     gcs_objs_list = hook.list(bucket_name = BUCKET_STORE, prefix = "proc")
108 |     for obj in gcs_objs_list: 
109 |         TABLE_NAME = obj.split("/")[-2].replace('.parquet', '')
110 |         bigquery_external_table_task = BigQueryCreateExternalTableOperator(
111 |             task_id=f"bigquery_external_table_{TABLE_NAME}_task",
112 |             table_resource={
113 |                 "tableReference": {
114 |                     "projectId": PROJECT_ID,
115 |                     "datasetId": BIGQUERY_DATASET,
116 |                     "tableId": TABLE_NAME,
117 |                 },
118 |                 "externalDataConfiguration": {
119 |                     "sourceFormat": "PARQUET",
120 |                     "sourceUris": [f"gs://{BUCKET_STORE}/{obj}"]
121 |                 },
122 |             }
123 |         )
124 |         bq_parallel_tasks.append(bigquery_external_table_task)
125 | 
126 |     rm_store_task = BashOperator(
127 |         task_id='remove_store_files_from_local',
128 |         bash_command=f'rm -rf {STEAM_STORE_LOCAL_DIR}',
129 |         trigger_rule="all_done"
130 |     )
131 | 
132 |     #-------------------------------------------REVIEWS
133 | 
134 |     download_steam_reviews_from_datalake_task = PythonOperator(
135 |         task_id="download_steam_reviews_from_datalake_task",
136 |         python_callable=build_compacted_reviews_dataset,
137 |         op_kwargs={
138 |             "bucket": BUCKET_REVIEWS,
139 |             "bucket_subdir": BUCKET_SUBDIR,
140 |             "reviews_local_dir" : REVIEWS_LOCAL_DIR,
141 |             "reviews_local_dir_proc" : REVIEWS_LOCAL_DIR_PROC
142 |         }
143 |     )
144 | 
145 |     create_cluster_dataproc_task = DataprocCreateClusterOperator(
146 |         task_id="create_cluster_dataproc_task",
147 |         project_id=PROJECT_ID,
148 |         cluster_config=CLUSTER_CONFIG,
149 |         region=CLUSTER_REGION,
150 |         trigger_rule="all_success",
151 |         cluster_name=CLUSTER_NAME
152 |     )
153 | 
154 |     submit_spark_job_task = DataprocSubmitPySparkJobOperator(
155 |         task_id = "submit_dataproc_spark_job_task",
156 |         main = f"gs://{BUCKET_REVIEWS}/{PYSPARK_FILE}",
157 |         arguments = JOB_ARGUMENTS,
158 |         cluster_name = CLUSTER_NAME,
159 |         region = CLUSTER_REGION,
160 |         dataproc_jars = ["gs://spark-lib/bigquery/spark-3.1-bigquery-0.27.0-preview.jar"]
161 |     )
162 | 
163 |     delete_cluster_dataproc_task = DataprocDeleteClusterOperator(
164 |         task_id="delete_cluster_dataproc_task",
165 |         project_id=PROJECT_ID,
166 |         cluster_name=CLUSTER_NAME,
167 |         region=CLUSTER_REGION,
168 |         trigger_rule="all_done"
169 |     )
170 | 
171 |     rm_reviews_task = BashOperator(
172 |         task_id='remove_reviews_files_from_local',
173 |         bash_command=f'rm -rf {REVIEWS_LOCAL_DIR} {REVIEWS_LOCAL_DIR_PROC}',
174 |         trigger_rule="all_done"
175 |     )
176 | 
177 |     #-------------------------------------------DBT
178 |     run_dbt_task = BashOperator(
179 |     task_id='run_dbt',
180 |     bash_command=f'cd {DBT_DIR} && dbt run --profile airflow',
181 |     trigger_rule="all_success"
182 |     )
183 | 
184 |     download_steam_dataset_from_datalake_task >> format_to_parquet_inlocal_task >> rm_files_store_task >> upload_steam_dataset_proc_task >> bq_parallel_tasks >> rm_store_task >> run_dbt_task
185 |     download_steam_reviews_from_datalake_task >> create_cluster_dataproc_task >> submit_spark_job_task >> delete_cluster_dataproc_task >> rm_reviews_task >> run_dbt_task
186 |     


--------------------------------------------------------------------------------
/airflow/dags/custom_dags/store_ingest_dag.py:
--------------------------------------------------------------------------------
  1 | import os
  2 | import json
  3 | import glob
  4 | import sys
  5 | import dask.dataframe as dd
  6 | from google.api_core import page_iterator
  7 | from google.cloud import storage
  8 | 
  9 | from airflow import DAG
 10 | from airflow.utils.dates import days_ago
 11 | from airflow.utils.dates import days_ago
 12 | from airflow.operators.bash import BashOperator
 13 | from airflow.operators.python import PythonOperator
 14 | from airflow.providers.google.cloud.hooks.gcs import GCSHook
 15 | from airflow.providers.google.cloud.operators.dataproc import DataprocDeleteClusterOperator
 16 | from airflow.providers.google.cloud.operators.dataproc import DataprocCreateClusterOperator
 17 | from airflow.providers.google.cloud.operators.bigquery import BigQueryCreateExternalTableOperator
 18 | from airflow.providers.google.cloud.operators.dataproc import DataprocSubmitPySparkJobOperator
 19 | 
 20 | sys.path.append("airflow/dags/common_package")                                                    
 21 | from common_package.utils_module import *
 22 | 
 23 | AIRFLOW_HOME = os.environ.get("AIRFLOW_HOME", "/opt/airflow/")
 24 | PROJECT_ID = os.environ.get("GCP_PROJECT_ID")
 25 | BUCKET_STORE = os.environ.get("GCP_GCS_BUCKET_STORE")
 26 | BUCKET_REVIEWS = os.environ.get("GCP_GCS_BUCKET_REVIEWS")
 27 | BUCKET_SUBDIR = "raw"
 28 | BUCKET_SUBDIR_PROC = "proc"
 29 | BIGQUERY_DATASET = os.environ.get('GCP_BQ_DATASET')
 30 | DATASETS_LOCAL_DIR = os.path.join(AIRFLOW_HOME, "datasets/") 
 31 | STEAM_STORE_LOCAL_DIR = os.path.join(DATASETS_LOCAL_DIR, "steamstore")
 32 | REVIEWS_LOCAL_DIR = os.path.join(DATASETS_LOCAL_DIR, "reviews")
 33 | REVIEWS_LOCAL_DIR_PROC = os.path.join(DATASETS_LOCAL_DIR, "reviews_proc")
 34 | DBT_DIR = os.path.join(AIRFLOW_HOME, "dbt")
 35 | 
 36 | BQ_TABLE = 'steam_reviews'
 37 | TEMP_BUCKET = 'steam-datalake-store-alldata'
 38 | JOB_ARGUMENTS = [PROJECT_ID, BIGQUERY_DATASET, BQ_TABLE, BUCKET_REVIEWS, BUCKET_SUBDIR_PROC, TEMP_BUCKET]
 39 | 
 40 | CLUSTER_NAME = "steam-reviews-spark-cluster" #"spark-cluster-reviews"
 41 | CLUSTER_REGION = "europe-west1"
 42 | PYSPARK_FILE = "spark_all_games_reviews.py" # decouple in a separate (previous) task
 43 | 
 44 | CLUSTER_CONFIG = {
 45 |     "master_config": {
 46 |         "num_instances": 1,
 47 |         "machine_type_uri": "n1-standard-2",
 48 |         "disk_config": {"boot_disk_type": "pd-standard", "boot_disk_size_gb": 50},
 49 |     },
 50 |     "worker_config": {
 51 |         "num_instances": 2,
 52 |         "machine_type_uri": "n1-standard-2",
 53 |         "disk_config": {"boot_disk_type": "pd-standard", "boot_disk_size_gb": 50},
 54 |     },
 55 | }
 56 | 
 57 | '''
 58 | PYSPARK_JOB = {
 59 |     "reference": {"project_id": PROJECT_ID},
 60 |     "placement": {"cluster_name": CLUSTER_NAME},
 61 |     "pyspark_job": {"main_python_file_uri": f"gs://{BUCKET_REVIEWS}/{PYSPARK_FILE}",
 62 |                     "jar_file_uris": ["gs://spark-lib/bigquery/spark-3.1-bigquery-0.27.0-preview.jar"]},
 63 |     "arguments":JOB_ARGUMENTS
 64 | 
 65 | }
 66 | '''
 67 | 
 68 | 
 69 | default_args = {
 70 |     "owner": "airflow",
 71 |     "start_date": days_ago(1),
 72 |     "depends_on_past": False,
 73 |     "retries": 1,
 74 | }
 75 | 
 76 | with DAG(
 77 |     dag_id="steam_store_ingestion_dag",
 78 |     schedule_interval="@weekly",
 79 |     default_args=default_args,
 80 |     catchup=False,
 81 |     max_active_runs=1,
 82 |     tags=['steam-de'],
 83 | ) as dag:
 84 | 
 85 |     #-------------------------------------------STORE    
 86 |     download_steam_dataset_from_datalake_task = PythonOperator(
 87 |         task_id="download_steam_dataset_from_datalake_task",
 88 |         python_callable=download_gcs,
 89 |         op_kwargs={
 90 |             "bucket": BUCKET_STORE,
 91 |             "local_dir": STEAM_STORE_LOCAL_DIR 
 92 |         }
 93 |     )
 94 | 
 95 |     format_to_parquet_inlocal_task = PythonOperator(
 96 |         task_id="format_to_parquet_inlocal_task",
 97 |         python_callable=format_to_parquet,
 98 |         op_kwargs={
 99 |             "local_dir": STEAM_STORE_LOCAL_DIR
100 |         }
101 |     )
102 | 
103 |     rm_files_store_task = BashOperator(
104 |         task_id="remove_needless_files_store_inlocal_task",
105 |         bash_command=f"find {STEAM_STORE_LOCAL_DIR} -type d -name news_data -prune -exec rm -rf {{}} \; &&  find {STEAM_STORE_LOCAL_DIR} -type d -name steam_charts -prune -exec rm -rf {{}} \; && find {STEAM_STORE_LOCAL_DIR} -name 'missing.json' -delete"
106 |     )
107 | 
108 |     upload_steam_dataset_proc_task = PythonOperator(
109 |         task_id="upload_steam_dataset_proc_task",
110 |         python_callable=upload_gcs,
111 |         op_kwargs={
112 |             "bucket": BUCKET_STORE,
113 |             "local_dir": STEAM_STORE_LOCAL_DIR 
114 |         }
115 |     )
116 | 
117 |     hook = GCSHook()
118 |     bq_parallel_tasks = list()
119 |     gcs_objs_list = hook.list(bucket_name = BUCKET_STORE, prefix = "proc")
120 |     for obj in gcs_objs_list: 
121 |         TABLE_NAME = obj.split("/")[-2].replace('.parquet', '')
122 |         bigquery_external_table_task = BigQueryCreateExternalTableOperator(
123 |             task_id=f"bigquery_external_table_{TABLE_NAME}_task",
124 |             table_resource={
125 |                 "tableReference": {
126 |                     "projectId": PROJECT_ID,
127 |                     "datasetId": BIGQUERY_DATASET,
128 |                     "tableId": TABLE_NAME,
129 |                 },
130 |                 "externalDataConfiguration": {
131 |                     "sourceFormat": "PARQUET",
132 |                     "sourceUris": [f"gs://{BUCKET_STORE}/{obj}"]
133 |                 },
134 |             }
135 |         )
136 |         bq_parallel_tasks.append(bigquery_external_table_task)
137 | 
138 |     rm_store_task = BashOperator(
139 |         task_id='remove_store_files_from_local',
140 |         bash_command=f'rm -rf {STEAM_STORE_LOCAL_DIR}',
141 |         trigger_rule="all_done"
142 |     )
143 | 
144 |     #-------------------------------------------DBT
145 |     run_dbt_task = BashOperator(
146 |     task_id='run_dbt',
147 |     bash_command=f'cd {DBT_DIR} && dbt run --profile airflow',
148 |     trigger_rule="all_success"
149 |     )
150 | 
151 |     download_steam_dataset_from_datalake_task >> format_to_parquet_inlocal_task >> rm_files_store_task >> upload_steam_dataset_proc_task >> bq_parallel_tasks >> rm_store_task >> run_dbt_task
152 | 


--------------------------------------------------------------------------------
/airflow/docker-compose.yaml:
--------------------------------------------------------------------------------
  1 | # Licensed to the Apache Software Foundation (ASF) under one
  2 | # or more contributor license agreements.  See the NOTICE file
  3 | # distributed with this work for additional information
  4 | # regarding copyright ownership.  The ASF licenses this file
  5 | # to you under the Apache License, Version 2.0 (the
  6 | # "License"); you may not use this file except in compliance
  7 | # with the License.  You may obtain a copy of the License at
  8 | #
  9 | #   http://www.apache.org/licenses/LICENSE-2.0
 10 | #
 11 | # Unless required by applicable law or agreed to in writing,
 12 | # software distributed under the License is distributed on an
 13 | # "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
 14 | # KIND, either express or implied.  See the License for the
 15 | # specific language governing permissions and limitations
 16 | # under the License.
 17 | #
 18 | 
 19 | # Basic Airflow cluster configuration for CeleryExecutor with Redis and PostgreSQL.
 20 | #
 21 | # WARNING: This configuration is for local development. Do not use it in a production deployment.
 22 | #
 23 | # This configuration supports basic configuration using environment variables or an .env file
 24 | # The following variables are supported:
 25 | #
 26 | # AIRFLOW_IMAGE_NAME           - Docker image name used to run Airflow.
 27 | #                                Default: apache/airflow:2.3.4
 28 | # AIRFLOW_UID                  - User ID in Airflow containers
 29 | #                                Default: 50000
 30 | # Those configurations are useful mostly in case of standalone testing/running Airflow in test/try-out mode
 31 | #
 32 | # _AIRFLOW_WWW_USER_USERNAME   - Username for the administrator account (if requested).
 33 | #                                Default: airflow
 34 | # _AIRFLOW_WWW_USER_PASSWORD   - Password for the administrator account (if requested).
 35 | #                                Default: airflow
 36 | # _PIP_ADDITIONAL_REQUIREMENTS - Additional PIP requirements to add when starting all containers.
 37 | #                                Default: ''
 38 | #
 39 | # Feel free to modify this file to suit your needs.
 40 | ---
 41 | version: '3'
 42 | x-airflow-common:
 43 |   &airflow-common
 44 |   # In order to add custom dependencies or upgrade provider packages you can use your extended image.
 45 |   # Comment the image line, place your Dockerfile in the directory where you placed the docker-compose.yaml
 46 |   # and uncomment the "build" line below, Then run `docker-compose build` to build the images.
 47 |   #image: ${AIRFLOW_IMAGE_NAME:-apache/airflow:2.3.4}
 48 |   build: 
 49 |     context: .
 50 |     dockerfile: ./Dockerfile
 51 |   environment:
 52 |     &airflow-common-env
 53 |     AIRFLOW__CORE__EXECUTOR: CeleryExecutor
 54 |     #AIRFLOW__CORE__EXECUTOR: LocalExecutor
 55 |     AIRFLOW__DATABASE__SQL_ALCHEMY_CONN: postgresql+psycopg2://airflow:airflow@postgres/airflow
 56 |     # For backward compatibility, with Airflow <2.3
 57 |     AIRFLOW__CORE__SQL_ALCHEMY_CONN: postgresql+psycopg2://airflow:airflow@postgres/airflow
 58 |     AIRFLOW__CELERY__RESULT_BACKEND: db+postgresql://airflow:airflow@postgres/airflow
 59 |     AIRFLOW__CELERY__BROKER_URL: redis://:@redis:6379/0
 60 |     AIRFLOW__CORE__FERNET_KEY: ''
 61 |     AIRFLOW__CORE__DAGS_ARE_PAUSED_AT_CREATION: 'true'
 62 |     AIRFLOW__CORE__LOAD_EXAMPLES: 'false'
 63 |     AIRFLOW__API__AUTH_BACKENDS: 'airflow.api.auth.backend.basic_auth'
 64 |     _PIP_ADDITIONAL_REQUIREMENTS: ${_PIP_ADDITIONAL_REQUIREMENTS:-}
 65 | 
 66 |     GOOGLE_APPLICATION_CREDENTIALS: /.google/credentials/google_credentials.json
 67 |     AIRFLOW_CONN_GOOGLE_CLOUD_DEFAULT: 'google-cloud-platform://?extra__google_cloud_platform__key_path=/.google/credentials/google_credentials.json'
 68 | 
 69 |     GCP_PROJECT_ID: 'steam-data-engineering-gcp'
 70 |     GCP_GCS_BUCKET_STORE: 'steam-datalake-store-alldata'
 71 |     GCP_GCS_BUCKET_REVIEWS: 'steam-datalake-reviews'
 72 |     GCP_BQ_DATASET: 'steam_raw'
 73 |   volumes:
 74 |     - ./dags:/opt/airflow/dags
 75 |     - ./logs:/opt/airflow/logs
 76 |     - ./plugins:/opt/airflow/plugins
 77 |     - ../dbt:/opt/airflow/dbt
 78 |     - /home/vyago-gcp/.google/credentials/:/.google/credentials:ro
 79 |     - /home/vyago-gcp/.dbt:/home/airflow/.dbt:ro
 80 | 
 81 |   user: "${AIRFLOW_UID:-50000}:0"
 82 |   depends_on:
 83 |     &airflow-common-depends-on
 84 |     #redis:
 85 |     #  condition: service_healthy
 86 |     postgres:
 87 |       condition: service_healthy
 88 | 
 89 | services:
 90 |   postgres:
 91 |     image: postgres:13
 92 |     environment:
 93 |       POSTGRES_USER: airflow
 94 |       POSTGRES_PASSWORD: airflow
 95 |       POSTGRES_DB: airflow
 96 |     volumes:
 97 |       - postgres-db-volume:/var/lib/postgresql/data
 98 |     healthcheck:
 99 |       test: ["CMD", "pg_isready", "-U", "airflow"]
100 |       interval: 5s
101 |       retries: 5
102 |     restart: always
103 | 
104 |   redis:
105 |     image: redis:latest
106 |     expose:
107 |       - 6379
108 |     healthcheck:
109 |       test: ["CMD", "redis-cli", "ping"]
110 |       interval: 5s
111 |       timeout: 30s
112 |       retries: 50
113 |     restart: always
114 | 
115 |   airflow-webserver:
116 |     <<: *airflow-common
117 |     command: webserver
118 |     ports:
119 |       - 8080:8080
120 |     healthcheck:
121 |       test: ["CMD", "curl", "--fail", "http://localhost:8080/health"]
122 |       interval: 10s
123 |       timeout: 10s
124 |       retries: 5
125 |     restart: always
126 |     depends_on:
127 |       <<: *airflow-common-depends-on
128 |       airflow-init:
129 |         condition: service_completed_successfully
130 | 
131 |   airflow-scheduler:
132 |     <<: *airflow-common
133 |     command: scheduler
134 |     healthcheck:
135 |       test: ["CMD-SHELL", 'airflow jobs check --job-type SchedulerJob --hostname "${HOSTNAME}"']
136 |       interval: 10s
137 |       timeout: 10s
138 |       retries: 5
139 |     restart: always
140 |     depends_on:
141 |       <<: *airflow-common-depends-on
142 |       airflow-init:
143 |         condition: service_completed_successfully
144 | 
145 |   airflow-worker:
146 |     <<: *airflow-common
147 |     command: celery worker
148 |     healthcheck:
149 |       test:
150 |         - "CMD-SHELL"
151 |         - 'celery --app airflow.executors.celery_executor.app inspect ping -d "celery@${HOSTNAME}"'
152 |       interval: 10s
153 |       timeout: 10s
154 |       retries: 5
155 | 
156 |     environment:
157 |       <<: *airflow-common-env
158 |       # Required to handle warm shutdown of the celery workers properly
159 |       # See https://airflow.apache.org/docs/docker-stack/entrypoint.html#signal-propagation
160 |       DUMB_INIT_SETSID: "0"
161 |     restart: always
162 |     depends_on:
163 |       <<: *airflow-common-depends-on
164 |       airflow-init:
165 |         condition: service_completed_successfully
166 | 
167 |   airflow-triggerer:
168 |     <<: *airflow-common
169 |     command: triggerer
170 |     healthcheck:
171 |       test: ["CMD-SHELL", 'airflow jobs check --job-type TriggererJob --hostname "${HOSTNAME}"']
172 |       interval: 10s
173 |       timeout: 10s
174 |       retries: 5
175 |     restart: always
176 |     depends_on:
177 |       <<: *airflow-common-depends-on
178 |       airflow-init:
179 |         condition: service_completed_successfully
180 | 
181 |   airflow-init:
182 |     <<: *airflow-common
183 |     entrypoint: /bin/bash
184 |     # yamllint disable rule:line-length
185 |     command:
186 |       - -c
187 |       - |
188 |         function ver() {
189 |           printf "%04d%04d%04d%04d" ${1//./ }
190 |         }
191 |         airflow_version=$(AIRFLOW__LOGGING__LOGGING_LEVEL=INFO && gosu airflow airflow version)
192 |         airflow_version_comparable=$(ver ${airflow_version})
193 |         min_airflow_version=2.2.0
194 |         min_airflow_version_comparable=$(ver ${min_airflow_version})
195 |         if (( airflow_version_comparable < min_airflow_version_comparable )); then
196 |           echo
197 |           echo -e "\033[1;31mERROR!!!: Too old Airflow version ${airflow_version}!\e[0m"
198 |           echo "The minimum Airflow version supported: ${min_airflow_version}. Only use this or higher!"
199 |           echo
200 |           exit 1
201 |         fi
202 |         if [[ -z "${AIRFLOW_UID}" ]]; then
203 |           echo
204 |           echo -e "\033[1;33mWARNING!!!: AIRFLOW_UID not set!\e[0m"
205 |           echo "If you are on Linux, you SHOULD follow the instructions below to set "
206 |           echo "AIRFLOW_UID environment variable, otherwise files will be owned by root."
207 |           echo "For other operating systems you can get rid of the warning with manually created .env file:"
208 |           echo "    See: https://airflow.apache.org/docs/apache-airflow/stable/start/docker.html#setting-the-right-airflow-user"
209 |           echo
210 |         fi
211 |         one_meg=1048576
212 |         mem_available=$(($(getconf _PHYS_PAGES) * $(getconf PAGE_SIZE) / one_meg))
213 |         cpus_available=$(grep -cE 'cpu[0-9]+' /proc/stat)
214 |         disk_available=$(df / | tail -1 | awk '{print $4}')
215 |         warning_resources="false"
216 |         if (( mem_available < 4000 )) ; then
217 |           echo
218 |           echo -e "\033[1;33mWARNING!!!: Not enough memory available for Docker.\e[0m"
219 |           echo "At least 4GB of memory required. You have $(numfmt --to iec $((mem_available * one_meg)))"
220 |           echo
221 |           warning_resources="true"
222 |         fi
223 |         if (( cpus_available < 2 )); then
224 |           echo
225 |           echo -e "\033[1;33mWARNING!!!: Not enough CPUS available for Docker.\e[0m"
226 |           echo "At least 2 CPUs recommended. You have ${cpus_available}"
227 |           echo
228 |           warning_resources="true"
229 |         fi
230 |         if (( disk_available < one_meg * 10 )); then
231 |           echo
232 |           echo -e "\033[1;33mWARNING!!!: Not enough Disk space available for Docker.\e[0m"
233 |           echo "At least 10 GBs recommended. You have $(numfmt --to iec $((disk_available * 1024 )))"
234 |           echo
235 |           warning_resources="true"
236 |         fi
237 |         if [[ ${warning_resources} == "true" ]]; then
238 |           echo
239 |           echo -e "\033[1;33mWARNING!!!: You have not enough resources to run Airflow (see above)!\e[0m"
240 |           echo "Please follow the instructions to increase amount of resources available:"
241 |           echo "   https://airflow.apache.org/docs/apache-airflow/stable/start/docker.html#before-you-begin"
242 |           echo
243 |         fi
244 |         mkdir -p /sources/logs /sources/dags /sources/plugins
245 |         chown -R "${AIRFLOW_UID}:0" /sources/{logs,dags,plugins}
246 |         exec /entrypoint airflow version
247 |     # yamllint enable rule:line-length
248 |     environment:
249 |       <<: *airflow-common-env
250 |       _AIRFLOW_DB_UPGRADE: 'true'
251 |       _AIRFLOW_WWW_USER_CREATE: 'true'
252 |       _AIRFLOW_WWW_USER_USERNAME: ${_AIRFLOW_WWW_USER_USERNAME:-airflow}
253 |       _AIRFLOW_WWW_USER_PASSWORD: ${_AIRFLOW_WWW_USER_PASSWORD:-airflow}
254 |       _PIP_ADDITIONAL_REQUIREMENTS: ''
255 |     user: "0:0"
256 |     volumes:
257 |       - .:/sources
258 | 
259 |   airflow-cli:
260 |     <<: *airflow-common
261 |     profiles:
262 |       - debug
263 |     environment:
264 |       <<: *airflow-common-env
265 |       CONNECTION_CHECK_MAX_COUNT: "0"
266 |     # Workaround for entrypoint issue. See: https://github.com/apache/airflow/issues/16252
267 |     command:
268 |       - bash
269 |       - -c
270 |       - airflow
271 | 
272 |   # You can enable flower by adding "--profile flower" option e.g. docker-compose --profile flower up
273 |   # or by explicitly targeted on the command line e.g. docker-compose up flower.
274 |   # See: https://docs.docker.com/compose/profiles/
275 |   
276 |   flower:
277 |     <<: *airflow-common
278 |     command: celery flower
279 |     profiles:
280 |       - flower
281 |     ports:
282 |       - 5555:5555
283 |     healthcheck:
284 |       test: ["CMD", "curl", "--fail", "http://localhost:5555/"]
285 |       interval: 10s
286 |       timeout: 10s
287 |       retries: 5
288 |     restart: always
289 |     depends_on:
290 |       <<: *airflow-common-depends-on
291 |       airflow-init:
292 |         condition: service_completed_successfully
293 | 
294 | volumes:
295 |   postgres-db-volume:


--------------------------------------------------------------------------------
/airflow/requirements.txt:
--------------------------------------------------------------------------------
1 | apache-airflow-providers-google
2 | pyarrow
3 | kaggle
4 | dbt-bigquery
5 | 


--------------------------------------------------------------------------------
/airflow/scripts/entrypoint.sh:
--------------------------------------------------------------------------------
 1 | #!/usr/bin/env bash
 2 | export GOOGLE_APPLICATION_CREDENTIALS=${GOOGLE_APPLICATION_CREDENTIALS}
 3 | export AIRFLOW_CONN_GOOGLE_CLOUD_DEFAULT=${AIRFLOW_CONN_GOOGLE_CLOUD_DEFAULT}
 4 | 
 5 | airflow db upgrade
 6 | 
 7 | airflow users create -r Admin -u admin -p admin -e admin@example.com -f admin -l airflow
 8 | # "$_AIRFLOW_WWW_USER_USERNAME" -p "$_AIRFLOW_WWW_USER_PASSWORD"
 9 | 
10 | airflow webserver


--------------------------------------------------------------------------------
/dbt/.gitignore:
--------------------------------------------------------------------------------
1 | 
2 | target/
3 | dbt_packages/
4 | logs/
5 | 


--------------------------------------------------------------------------------
/dbt/README.md:
--------------------------------------------------------------------------------
 1 | # dbt
 2 | 
 3 | The heavy transformations are performed in **dbt**, it takes as inputs an array of dirty tables and outputs a fully curated dataset ready to be ingested by BI analysts.
 4 | 
 5 | ![](https://github.com/VicenteYago/steam-data-engineering/blob/main/img/lineage_dbt_1.png)
 6 | ![](https://github.com/VicenteYago/steam-data-engineering/blob/main/img/lineage_dbt_2.png)
 7 | ![](https://github.com/VicenteYago/steam-data-engineering/blob/main/img/lineage_dbt_3.png)
 8 | 
 9 | ## Some transformations
10 | 
11 | - some tables, like `developers` do not have an ID, so the `developers.name` has to be used for this purpose, with all the problems it have, since same developers are not always named equally. For example `"FromSoftware, Inc."` and `"Fromsoftware inc ®"` are both transformed into `fromsoftware` using **macros**.
12 | - extracting dates from text
13 | - some columns needed to be splitted,for example `owners` is splitted into `owners.high` and `owners.low`  : 
14 | 
15 |   <p align="center">
16 |   <img src="https://github.com/VicenteYago/steam-data-engineering/blob/main/img/owners_old.png" >
17 |   </p>
18 |   
19 |   <p align="center">
20 |   <img src="https://github.com/VicenteYago/steam-data-engineering/blob/main/img/owners_new.png" >
21 |   </p>
22 | 
23 | - multiple type casts.
24 | 
25 | ## Denormalization
26 | Denormalized tables with nested and repeated files are implemented following  [google recommendations](https://cloud.google.com/blog/topics/developers-practitioners/bigquery-explained-working-joins-nested-repeated-data).
27 | 
28 |   <p align="center">
29 |   <img src="https://github.com/VicenteYago/steam-data-engineering/blob/main/img/schema_denorm.png" >
30 |   </p>
31 | 
32 | ## 2 enviroments
33 | As dbt recommends a **development** and **production** enviroments are used for safety.
34 | 
35 | 
36 | ## Tests
37 | In progess
38 | 


--------------------------------------------------------------------------------
/dbt/analyses/.gitkeep:
--------------------------------------------------------------------------------
https://raw.githubusercontent.com/VicenteYago/steam-data-engineering/d991376ae7a2ec16970b65c81f724830704221c1/dbt/analyses/.gitkeep


--------------------------------------------------------------------------------
/dbt/dbt_project.yml:
--------------------------------------------------------------------------------
 1 | 
 2 | # Name your project! Project names should contain only lowercase characters
 3 | # and underscores. A good package name should reflect your organization's
 4 | # name or the intended use of these models
 5 | name: 'steam_dbt_project'
 6 | version: '1.0.0'
 7 | config-version: 2
 8 | 
 9 | # This setting configures which "profile" dbt uses for this project.
10 | profile: 'default'
11 | 
12 | # These configurations specify where dbt should look for different types of files.
13 | # The `source-paths` config, for example, states that models in this project can be
14 | # found in the "models/" directory. You probably won't need to change these!
15 | model-paths: ["models"]
16 | analysis-paths: ["analyses"]
17 | test-paths: ["tests"]
18 | seed-paths: ["seeds"]
19 | macro-paths: ["macros"]
20 | snapshot-paths: ["snapshots"]
21 | 
22 | target-path: "target"  # directory which will store compiled SQL files
23 | clean-targets:         # directories to be removed by `dbt clean`
24 |   - "target"
25 |   - "dbt_packages"
26 | 
27 | 
28 | # Configuring models
29 | # Full documentation: https://docs.getdbt.com/docs/configuring-model
30 | models:
31 |   steam_dbt_project:
32 |   
33 |     staging:
34 |       materialized: view
35 | 
36 |     core: 
37 |       materialized: table
38 |       +schema: core
39 | 
40 |     analysis: 
41 |       materialized: table
42 |       +schema: analysis
43 | 
44 | 
45 | 


--------------------------------------------------------------------------------
/dbt/macros/.gitkeep:
--------------------------------------------------------------------------------
https://raw.githubusercontent.com/VicenteYago/steam-data-engineering/d991376ae7a2ec16970b65c81f724830704221c1/dbt/macros/.gitkeep


--------------------------------------------------------------------------------
/dbt/macros/fix_bools.sql:
--------------------------------------------------------------------------------
1 | {% macro fix_bools(column_json) %}
2 |     
3 |     REPLACE(
4 |         REPLACE({{column_json}}, 'True', 'true')
5 |     , 'False', 'false')
6 | 
7 | {% endmacro %}


--------------------------------------------------------------------------------
/dbt/macros/fix_strings.sql:
--------------------------------------------------------------------------------
1 | {% macro fix_strings(col_str) %}
2 |     
3 | -- regexp_replace(normalize_and_casefold({{col_str}}), r'\W+','')
4 | regexp_replace(normalize_and_casefold({{col_str}}),
5 |                r'llc|inc\.|inc|s\.a\.|ltd\.|co\.|\.|\,|\s|\(mac\)|\(linux\)|\(mac/linux\)|™|®',
6 |                '')
7 | {% endmacro %}


--------------------------------------------------------------------------------
/dbt/models/analysis/devs_metacritic.sql:
--------------------------------------------------------------------------------
 1 | 
 2 | SELECT 
 3 |   developers_unnest,
 4 |   appid,
 5 |   metacritic
 6 | FROM {{ref('steam_games')}},
 7 |   UNNEST(developers) as developers_unnest
 8 | WHERE metacritic is not null
 9 | 
10 | 


--------------------------------------------------------------------------------
/dbt/models/analysis/dlcs_by_game.sql:
--------------------------------------------------------------------------------
1 | select appid, metacritic, name, ARRAY_LENGTH(dlcs) as n_dlcs
2 | FROM {{ref('steam_games')}}
3 | WHERE metacritic is not null
4 | ORDER BY n_dlcs DESC


--------------------------------------------------------------------------------
/dbt/models/analysis/games_by_genre.sql:
--------------------------------------------------------------------------------
 1 | SELECT 
 2 |   appid,
 3 |   genres_unnest as genre,
 4 |   name,
 5 |   metacritic 
 6 | 
 7 | FROM {{ref('steam_games')}},
 8 |   UNNEST(genres.name) as genres_unnest
 9 | 
10 | WHERE metacritic is not null


--------------------------------------------------------------------------------
/dbt/models/analysis/steam_games_analysis.sql:
--------------------------------------------------------------------------------
 1 | SELECT appid,
 2 |        type,
 3 |        name,
 4 |        metacritic,
 5 |        required_age,
 6 |        is_free,
 7 |        release_year,
 8 |        coming_soon, 
 9 |        controller_support,
10 |        demos_appid,
11 |        revenue.low as revenue_low,
12 |        revenue.high as revenue_high,
13 |        drm_notice,
14 |        recommendations,
15 |        negative,
16 |        positive,
17 |        price,
18 |        initialprice,
19 |        discount_percentage,
20 |        ccu
21 | FROM {{ref('steam_games')}} as steam_games
22 | 
23 | WHERE metacritic is not null
24 | 


--------------------------------------------------------------------------------
/dbt/models/core/steam_dlc_data.sql:
--------------------------------------------------------------------------------
 1 | {{ config(materialized='table') }}
 2 | 
 3 | select
 4 |     SAFE_CAST(steam_appid as integer) as appid,
 5 |     type,
 6 |     name,
 7 |    JSON_EXTRACT_STRING_ARRAY(developers) as developers,
 8 |    JSON_EXTRACT_STRING_ARRAY(publishers) as publishers,
 9 | 
10 |    STRUCT<currency STRING, price_final INTEGER,  price_discount INTEGER>(
11 |     JSON_EXTRACT_SCALAR(price_overview, '$.currency'),
12 |     CAST(JSON_EXTRACT(price_overview, '$.final') as integer),
13 |     CAST(JSON_EXTRACT(price_overview, '$.discount_percent') as integer)
14 |    ) as price, 
15 | 
16 |    CAST(JSON_EXTRACT(metacritic, '$.score') as integer) as score,
17 | 
18 |    CAST(JSON_EXTRACT({{ fix_bools('release_date') }}, '$.coming_soon') as boolean) as coming_soon,
19 |    CAST(REGEXP_EXTRACT(JSON_EXTRACT( {{ fix_bools('release_date') }}, '$.date'), r"(\d\d\d\d)") as integer) as release_year,
20 |    CAST(JSON_EXTRACT(recommendations, "$.total")as integer)  as recommendations,
21 | 
22 | from  {{source('raw', 'steam_dlc_data')}}
23 | 


--------------------------------------------------------------------------------
/dbt/models/core/steam_games.sql:
--------------------------------------------------------------------------------
 1 | {{ config(
 2 |     materialized='incremental',
 3 |     unique_key='appid',
 4 |     partition_by={
 5 |       "field": "release_year",
 6 |       "data_type": "int64",
 7 |       "range": {
 8 |         "start": 1997,
 9 |         "end": 2023,
10 |         "interval": 3
11 |       }
12 |     }
13 | )}}
14 | 
15 | with t as 
16 | (
17 |   SELECT 
18 |         steam_store_data.appid,
19 |         steam_store_data.type,
20 |         steam_store_data.name,
21 |         steam_store_data.developers,
22 |         steam_store_data.publishers,
23 |         steam_store_data.dlcs,
24 |         steam_store_data.metacritic,
25 |         steam_store_data.required_age,
26 |         steam_store_data.is_free,
27 |         steam_store_data.release_year,
28 |         steam_store_data.coming_soon, 
29 |         steam_store_data.controller_support,
30 |         steam_store_data.demos_appid,
31 |         steam_store_data.drm_notice,
32 |         steam_store_data.recommendations,
33 |         steam_store_data.categories as categories,
34 |         steam_store_data.genres as genres,
35 |         STRUCT<low integer, high integer>(steam_spy_scrap.owners_low, steam_spy_scrap.owners_high) as owners,
36 |         STRUCT<low decimal, high decimal> (steam_spy_scrap.owners_low * steam_spy_scrap.price,
37 |                                         steam_spy_scrap.owners_high*steam_spy_scrap.price) as revenue,
38 |         STRUCT<windows boolean,
39 |                 mac     boolean,
40 |                 linux   boolean>(steam_store_data.platform_windows, steam_store_data.platform_mac, steam_store_data.platform_linux) as platforms,
41 |         steam_spy_scrap.negative,
42 |         steam_spy_scrap.positive,
43 |         steam_spy_scrap.price,
44 |         steam_spy_scrap.initialprice,
45 |         steam_spy_scrap.discount_percentage,
46 |         
47 |         steam_spy_scrap.ccu, 
48 |         ROW_NUMBER() over(partition by steam_store_data.appid) as rnk
49 | 
50 |   FROM {{ref('steam_store_data')}} as steam_store_data LEFT JOIN  
51 |             {{ref('steam_spy_scrap')}}  as steam_spy_scrap ON steam_spy_scrap.appid = steam_store_data.appid
52 |   WHERE release_year <=2025
53 | )
54 | 
55 | 
56 | SELECT 
57 |   appid,
58 |   type,
59 |   name,
60 |   developers,
61 |   publishers,
62 |   dlcs,
63 |   metacritic,
64 |   required_age,
65 |   is_free,
66 |   release_year,
67 |   coming_soon, 
68 |   controller_support,
69 |   demos_appid,
70 |   drm_notice,
71 |   recommendations,
72 |   categories as categories,
73 |   genres as genres,
74 |   owners,
75 |   revenue,
76 |   platforms,
77 |   negative,
78 |   positive,
79 |   price,
80 |   initialprice,
81 |   discount_percentage,
82 |   ccu
83 | FROM t WHERE rnk = 1
84 | 


--------------------------------------------------------------------------------
/dbt/models/core/steam_reviews.sql:
--------------------------------------------------------------------------------
 1 | {{ config(materialized='table') }}
 2 | 
 3 | with t as (
 4 |     SELECT 
 5 |         SAFE_CAST(recommendationid as INTEGER) as reviewid,			
 6 |         comment_count,			
 7 |         language,			
 8 |         received_for_free,			
 9 |         review,			
10 |         steam_purchase,		
11 |         timestamp_created,		
12 |         timestamp_updated,			
13 |         voted_up,			
14 |         votes_funny,			
15 |         votes_up,			
16 |         SAFE_CAST(weighted_vote_score as DECIMAL) as weighted_vote_score,			
17 |         written_during_early_access,		
18 |         SAFE_CAST(gameid as INTEGER) as gameid,			
19 |         author_last_played,		
20 |         author_num_games_owned,			
21 |         author_num_reviews,			
22 |         author_playtime_at_review,			
23 |         author_playtime_forever,			
24 |         author_playtime_last_two_weeks,			
25 |         CAST(author_steamid as INTEGER) as author_steamid, 
26 |         ROW_NUMBER() over(partition by recommendationid) as rnk
27 |     FROM  {{source('raw', 'steam_reviews')}}
28 | )
29 |  
30 | select 
31 |     reviewid,			
32 |     comment_count,			
33 |     language,			
34 |     received_for_free,			
35 |     review,			
36 |     steam_purchase,		
37 |     timestamp_created,		
38 |     timestamp_updated,			
39 |     voted_up,			
40 |     votes_funny,			
41 |     votes_up,			
42 |     weighted_vote_score,			
43 |     written_during_early_access,		
44 |     gameid,			
45 |     author_last_played,		
46 |     author_num_games_owned,			
47 |     author_num_reviews,			
48 |     author_playtime_at_review,			
49 |     author_playtime_forever,			
50 |     author_playtime_last_two_weeks,			
51 |     author_steamid
52 | from t where rnk = 1
53 | 
54 | 


--------------------------------------------------------------------------------
/dbt/models/schema.yml:
--------------------------------------------------------------------------------
 1 | version: 2
 2 | 
 3 | sources : 
 4 | 
 5 |     - name: raw
 6 |       description: aaaa
 7 |       database: steam-data-engineering-gcp
 8 |       schema: steam_raw
 9 |       tables: 
10 |         - name : steam_spy_scrap
11 |         - name : steam_store_data
12 |         - name : steam_dlc_data
13 |         - name : steam_reviews
14 | 
15 |     - name: staging
16 |       description: aaa
17 |       database: steam-data-engineering-gcp
18 |       schema: dbt_development
19 |       tables: 
20 |         - name : categories
21 |         - name : genres
22 |         - name : developers
23 |         - name : publishers
24 |         
25 | models: 
26 |   - name: steam_spy_scrap
27 |   - name: steam_store_data
28 |   - name: categories
29 |   - name: developers
30 |   - name: publishers
31 |   - name: genres
32 | 
33 |   - name: steam_games
34 |     columns:
35 |       - name: appid
36 |         tests:
37 |           - unique
38 |           - not_null
39 | 
40 |   - name: steam_reviews
41 |     columns: 
42 |       - name: author_steamid
43 |         tests: 
44 |           - not_null
45 |       - name: gameid
46 |         tests: 
47 |           - not_null
48 |       - name: reviewid
49 |         tests: 
50 |           - not_null
51 |           - unique
52 | 
53 |   - name: steam_dlc_data
54 |     columns:
55 |       - name: appid
56 |         tests:
57 |           - unique
58 |           - not_null


--------------------------------------------------------------------------------
/dbt/models/staging/categories.sql:
--------------------------------------------------------------------------------
 1 | {{ config(materialized='view') }}
 2 | 
 3 | with fixed as (
 4 |   select 
 5 |   steam_appid,
 6 |   JSON_EXTRACT_SCALAR(categories, '$.description') as categories_name,
 7 |   CAST(JSON_QUERY(categories, '$.id') as integer) as categories_id
 8 | 
 9 |   from `steam-data-engineering-gcp`.`steam_raw`.`steam_store_data`,
10 |     unnest(json_query_array(categories)) as categories
11 | )
12 | 
13 | 
14 | select CAST(original.steam_appid as integer ) as appid,
15 |         STRUCT <name ARRAY<STRING>,
16 |                 id   ARRAY<INTEGER>> (ARRAY_AGG(fixed.categories_name),
17 |                                       ARRAY_AGG(fixed.categories_id)) as categories
18 | 
19 | FROM {{source('raw', 'steam_store_data')}} as original
20 |        JOIN fixed ON original.steam_appid = fixed.steam_appid
21 | 
22 | GROUP BY original.steam_appid


--------------------------------------------------------------------------------
/dbt/models/staging/developers.sql:
--------------------------------------------------------------------------------
 1 | {{ config(materialized='view') }}
 2 | 
 3 | select CAST(original.steam_appid as integer ) as appid,
 4 |             ARRAY_AGG( distinct {{ fix_strings('fixed.devs') }}) as devs
 5 | 
 6 | FROM {{source('raw', 'steam_store_data')}} as original
 7 |        JOIN 
 8 |             (
 9 |             select 
10 |                 steam_appid,
11 |                 JSON_EXTRACT_SCALAR(devs) as devs
12 |             from {{source('raw', 'steam_store_data')}},
13 |                 unnest(json_query_array(developers)) as devs
14 |             ) fixed ON original.steam_appid = fixed.steam_appid
15 | 
16 | GROUP BY original.steam_appid


--------------------------------------------------------------------------------
/dbt/models/staging/genres.sql:
--------------------------------------------------------------------------------
 1 | {{ config(materialized='view') }}
 2 | 
 3 | select CAST(original.steam_appid as integer ) as appid,
 4 |         STRUCT <name ARRAY<STRING>,
 5 |                 id   ARRAY<INTEGER>> (ARRAY_AGG(fixed.genres_name),
 6 |                                       ARRAY_AGG(fixed.genres_id)) as genres
 7 | 
 8 | FROM {{source('raw', 'steam_store_data')}} as original
 9 |        JOIN 
10 |             (
11 |             select 
12 |                 steam_appid,
13 |                 JSON_EXTRACT_SCALAR(genres, '$.description') as genres_name,
14 |                 CAST( REGEXP_EXTRACT(genres, r"(\d+)") as integer) as genres_id 
15 | 
16 |             from {{source('raw', 'steam_store_data')}},
17 |                 unnest(json_query_array(genres)) as genres
18 |             ) fixed ON original.steam_appid = fixed.steam_appid
19 | 
20 | GROUP BY original.steam_appid


--------------------------------------------------------------------------------
/dbt/models/staging/publishers.sql:
--------------------------------------------------------------------------------
 1 | {{ config(materialized='view') }}
 2 | 
 3 | select CAST(original.steam_appid as integer ) as appid,
 4 |             ARRAY_AGG( distinct {{ fix_strings('fixed.pubs') }}) as pubs
 5 | 
 6 | FROM {{source('raw', 'steam_store_data')}} as original
 7 |        JOIN 
 8 |             (
 9 |             select 
10 |                 steam_appid,
11 |                 JSON_EXTRACT_SCALAR(pubs) as pubs
12 |             from {{source('raw', 'steam_store_data')}},
13 |                 unnest(json_query_array(publishers)) as pubs
14 |             ) fixed ON original.steam_appid = fixed.steam_appid
15 | 
16 | GROUP BY original.steam_appid


--------------------------------------------------------------------------------
/dbt/models/staging/steam_spy_scrap.sql:
--------------------------------------------------------------------------------
 1 | {{ config(materialized='view') }}
 2 | 
 3 | 
 4 | select
 5 |     SAFE_CAST(appid as integer) as appid,
 6 |     name,
 7 |     developer,
 8 |     publisher,
 9 |     score_rank,
10 |     SAFE_CAST(positive as integer) as positive,
11 |     SAFE_CAST(negative as integer) as negative,
12 |     SAFE_CAST(userscore as integer) as userscore,
13 |     SAFE_CAST( REPLACE( REGEXP_EXTRACT(owners, r'(.*)\.\.'), ",", "") AS INTEGER) AS owners_low,
14 |     SAFE_CAST( REPLACE( REGEXP_EXTRACT(owners, r'\.\.(.*)'), ",", "") AS INTEGER) AS owners_high,
15 |     SAFE_CAST(average_forever as integer) as average_forever,
16 |     SAFE_CAST(average_2weeks as integer) as average_2weeks,
17 |     SAFE_CAST(median_forever as integer) as median_forever,
18 |     SAFE_CAST(median_2weeks as integer) as median_2weeks,
19 |     SAFE_CAST(price as decimal)/100 as price,
20 |     SAFE_CAST(initialprice as decimal)/100 as initialprice,
21 |     SAFE_CAST( REPLACE(discount, '.0', '' ) as integer) as discount_percentage,
22 |     SAFE_CAST(ccu as integer) as ccu
23 | from  {{source('raw', 'steam_spy_scrap')}}


--------------------------------------------------------------------------------
/dbt/models/staging/steam_store_data.sql:
--------------------------------------------------------------------------------
 1 | {{ config(
 2 |    materialized='incremental', 
 3 |    unique_key='appid'
 4 | ) }}
 5 | 
 6 | 
 7 | SELECT
 8 |    type, 
 9 |    name, 
10 |    SAFE_CAST(ssd.steam_appid as integer) as appid, 
11 |    JSON_QUERY_ARRAY(dlc) as dlcs,
12 |    SAFE_CAST(required_age as integer) as required_age,
13 |    SAFE_CAST(is_free as BOOL) as is_free,
14 |    developers.devs as developers,
15 | 
16 |    CAST(JSON_EXTRACT({{ fix_bools('platforms') }}, '$.windows') as boolean) as platform_windows,
17 |    CAST(JSON_EXTRACT({{ fix_bools('platforms') }}, '$.mac') as boolean) as platform_mac,
18 |    CAST(JSON_EXTRACT({{ fix_bools('platforms') }}, '$.linux') as boolean) as platform_linux,
19 |    
20 |    CAST(JSON_EXTRACT(metacritic, '$.score') as integer) as metacritic , 
21 |    CAST(JSON_EXTRACT(recommendations, '$.total') as integer ) as num_recommendations, 
22 | 
23 |    CAST(JSON_EXTRACT({{ fix_bools('release_date') }}, '$.coming_soon') as boolean) as coming_soon,
24 |    CAST(REGEXP_EXTRACT(JSON_EXTRACT( {{ fix_bools('release_date') }}, '$.date'), r"(\d\d\d\d)") as integer) as release_year,
25 |    CASE WHEN controller_support = 'full' THEN TRUE 
26 |              ELSE FALSE END as controller_support,
27 |    drm_notice,
28 |    CAST(JSON_EXTRACT(recommendations, "$.total")as integer)  as recommendations,
29 |    CAST(JSON_EXTRACT(TRIM(demos, '[]'), '$.appid') as integer) as demos_appid,
30 |    categories.categories as categories,
31 |    genres.genres as genres,
32 |    publishers.pubs as publishers
33 |    
34 | FROM 
35 | 
36 |    (
37 |       (
38 |          (
39 |             {{source('raw', 'steam_store_data')}} ssd LEFT JOIN
40 |             {{source('staging', 'genres')}} genres ON CAST(ssd.steam_appid as integer) = genres.appid
41 |          )  LEFT JOIN {{source('staging', 'developers')}} developers ON genres.appid = developers.appid  
42 |       )  LEFT JOIN {{source('staging', 'categories')}} categories ON genres.appid = categories.appid
43 |    ) LEFT JOIN {{source('staging', 'publishers')}} publishers ON categories.appid = publishers.appid
44 | 
45 | 
46 | 
47 | 
48 | 


--------------------------------------------------------------------------------
/dbt/seeds/.gitkeep:
--------------------------------------------------------------------------------
https://raw.githubusercontent.com/VicenteYago/steam-data-engineering/d991376ae7a2ec16970b65c81f724830704221c1/dbt/seeds/.gitkeep


--------------------------------------------------------------------------------
/dbt/snapshots/.gitkeep:
--------------------------------------------------------------------------------
https://raw.githubusercontent.com/VicenteYago/steam-data-engineering/d991376ae7a2ec16970b65c81f724830704221c1/dbt/snapshots/.gitkeep


--------------------------------------------------------------------------------
/dbt/tests/.gitkeep:
--------------------------------------------------------------------------------
https://raw.githubusercontent.com/VicenteYago/steam-data-engineering/d991376ae7a2ec16970b65c81f724830704221c1/dbt/tests/.gitkeep


--------------------------------------------------------------------------------
/dbt/tests/assert_release_year_is_valid.sql:
--------------------------------------------------------------------------------
1 | SELECT 
2 | 
3 | release_year
4 | 
5 | FROM {{ ref('steam_games') }}
6 | 
7 | WHERE release_year > 2025
8 |  OR   release_year < 1997


--------------------------------------------------------------------------------
/img/airflow_gant.png:
--------------------------------------------------------------------------------
https://raw.githubusercontent.com/VicenteYago/steam-data-engineering/d991376ae7a2ec16970b65c81f724830704221c1/img/airflow_gant.png


--------------------------------------------------------------------------------
/img/airflow_graph.png:
--------------------------------------------------------------------------------
https://raw.githubusercontent.com/VicenteYago/steam-data-engineering/d991376ae7a2ec16970b65c81f724830704221c1/img/airflow_graph.png


--------------------------------------------------------------------------------
/img/airflow_grid.png:
--------------------------------------------------------------------------------
https://raw.githubusercontent.com/VicenteYago/steam-data-engineering/d991376ae7a2ec16970b65c81f724830704221c1/img/airflow_grid.png


--------------------------------------------------------------------------------
/img/dashboard.png:
--------------------------------------------------------------------------------
https://raw.githubusercontent.com/VicenteYago/steam-data-engineering/d991376ae7a2ec16970b65c81f724830704221c1/img/dashboard.png


--------------------------------------------------------------------------------
/img/dbt_linege.png:
--------------------------------------------------------------------------------
https://raw.githubusercontent.com/VicenteYago/steam-data-engineering/d991376ae7a2ec16970b65c81f724830704221c1/img/dbt_linege.png


--------------------------------------------------------------------------------
/img/lineage_dbt_1.png:
--------------------------------------------------------------------------------
https://raw.githubusercontent.com/VicenteYago/steam-data-engineering/d991376ae7a2ec16970b65c81f724830704221c1/img/lineage_dbt_1.png


--------------------------------------------------------------------------------
/img/lineage_dbt_2.png:
--------------------------------------------------------------------------------
https://raw.githubusercontent.com/VicenteYago/steam-data-engineering/d991376ae7a2ec16970b65c81f724830704221c1/img/lineage_dbt_2.png


--------------------------------------------------------------------------------
/img/lineage_dbt_3.png:
--------------------------------------------------------------------------------
https://raw.githubusercontent.com/VicenteYago/steam-data-engineering/d991376ae7a2ec16970b65c81f724830704221c1/img/lineage_dbt_3.png


--------------------------------------------------------------------------------
/img/owners_new.png:
--------------------------------------------------------------------------------
https://raw.githubusercontent.com/VicenteYago/steam-data-engineering/d991376ae7a2ec16970b65c81f724830704221c1/img/owners_new.png


--------------------------------------------------------------------------------
/img/owners_old.png:
--------------------------------------------------------------------------------
https://raw.githubusercontent.com/VicenteYago/steam-data-engineering/d991376ae7a2ec16970b65c81f724830704221c1/img/owners_old.png


--------------------------------------------------------------------------------
/img/schema_denorm.png:
--------------------------------------------------------------------------------
https://raw.githubusercontent.com/VicenteYago/steam-data-engineering/d991376ae7a2ec16970b65c81f724830704221c1/img/schema_denorm.png


--------------------------------------------------------------------------------
/img/steam.jpg:
--------------------------------------------------------------------------------
https://raw.githubusercontent.com/VicenteYago/steam-data-engineering/d991376ae7a2ec16970b65c81f724830704221c1/img/steam.jpg


--------------------------------------------------------------------------------
/img/steam_logo.png:
--------------------------------------------------------------------------------
https://raw.githubusercontent.com/VicenteYago/steam-data-engineering/d991376ae7a2ec16970b65c81f724830704221c1/img/steam_logo.png


--------------------------------------------------------------------------------
/spark/.gitignore:
--------------------------------------------------------------------------------
1 | lib/
2 | 


--------------------------------------------------------------------------------
/spark/README.md:
--------------------------------------------------------------------------------
1 | # Spark
2 | 
3 | [In progess ...]
4 | 


--------------------------------------------------------------------------------
/spark/all_games_reviews_gcp.ipynb:
--------------------------------------------------------------------------------
1 | {"cells":[{"cell_type":"markdown","metadata":{},"source":["## All reviews from all games"]},{"cell_type":"code","execution_count":1,"metadata":{},"outputs":[{"data":{"text/plain":["'/opt/conda/miniconda3/bin/python'"]},"execution_count":1,"metadata":{},"output_type":"execute_result"}],"source":["import sys\n","import pyspark\n","import os\n","from pyspark.sql import SparkSession\n","from pyspark.conf import SparkConf\n","from pyspark import SparkContext\n","from pyspark.sql.functions import input_file_name\n","from pyspark.sql import functions as F\n","from pyspark.sql import Row\n","from pyspark.sql.functions import row_number,lit\n","from pyspark.sql.window import Window\n","import re\n","from pyspark.sql.types import StringType\n","from google.api_core import page_iterator\n","from google.cloud import storage\n","\n","sys.executable"]},{"cell_type":"code","execution_count":9,"metadata":{},"outputs":[],"source":["#CREDENTIALS = '/home/vyago/.google/credentials/google_credentials.json'\n","#GCS_JAR = \"./lib/gcs-connector-hadoop3-latest.jar\"\n","\n","BQ_JAR = \"gs://spark-lib/bigquery/spark-3.1-bigquery-0.27.0-preview.jar\"\n","BQ_PROJECT_ID = \"steam-data-engineering-gcp\"\n","BQ_DATASET = 'steam_raw'\n","BQ_TABLE = 'reviews'\n","\n","TEMP_BUCKET = 'steam-datalake-dataset'\n","BUCKET = \"steam-datalake-reviews\"\n","BUCKET_SUBDIR = \"proc\"\n","\n","conf = SparkConf() \\\n","    .setAppName('steam-gcp-dataproc') \\\n","    .set(\"spark.jars\", f\"{BQ_JAR}\") \\\n","    .set(\"spark.hadoop.google.cloud.auth.service.account.enable\", \"true\")\n","\n","#   .setMaster('local[*]') \\\n","#   .set(\"spark.jars\", f\"{GCS_JAR}, {BQ_JAR}\") \\\n"]},{"cell_type":"code","execution_count":10,"metadata":{},"outputs":[{"name":"stderr","output_type":"stream","text":["Setting default log level to \"WARN\".\n","To adjust logging level use sc.setLogLevel(newLevel). For SparkR, use setLogLevel(newLevel).\n","22/09/26 17:12:37 INFO org.apache.spark.SparkEnv: Registering MapOutputTracker\n","22/09/26 17:12:37 INFO org.apache.spark.SparkEnv: Registering BlockManagerMaster\n","22/09/26 17:12:37 INFO org.apache.spark.SparkEnv: Registering BlockManagerMasterHeartbeat\n","22/09/26 17:12:37 INFO org.apache.spark.SparkEnv: Registering OutputCommitCoordinator\n"]}],"source":["sc = SparkContext(conf=conf)\n","\n","#hadoop_conf = sc._jsc.hadoopConfiguration()\n","#hadoop_conf.set(\"fs.AbstractFileSystem.gs.impl\",  \"com.google.cloud.hadoop.fs.gcs.GoogleHadoopFS\")\n","#hadoop_conf.set(\"fs.gs.impl\", \"com.google.cloud.hadoop.fs.gcs.GoogleHadoopFileSystem\")\n","#hadoop_conf.set(\"fs.gs.auth.service.account.json.keyfile\", CREDENTIALS)\n","#hadoop_conf.set(\"fs.gs.auth.service.account.enable\", \"true\")"]},{"cell_type":"code","execution_count":11,"metadata":{},"outputs":[],"source":["spark = SparkSession.builder \\\n","    .config(conf=sc.getConf()) \\\n","    .getOrCreate()"]},{"cell_type":"markdown","metadata":{},"source":["Connect to GCS"]},{"cell_type":"code","execution_count":12,"metadata":{},"outputs":[],"source":["# https://stackoverflow.com/questions/49538327/pyspark-string-pattern-from-columns-values-and-regexp-expression\n","\n","def flatten_df(nested_df):\n","    flat_cols = [c[0] for c in nested_df.dtypes if c[1][:6] != 'struct']\n","    nested_cols = [c[0] for c in nested_df.dtypes if c[1][:6] == 'struct']\n","\n","    flat_df = nested_df.select(flat_cols +\n","                               [F.col(nc+'.'+c).alias(nc+'_'+c)\n","                                for nc in nested_cols\n","                                for c in nested_df.select(nc+'.*').columns])\n","    return flat_df\n","\n","\n","def proc_json(raw_df, field)  :\n","    rows = raw_df[field]\n","    return(rows)\n","\n","def get_previous_word(text):\n","    matches = re.search('.*/(\\d+)-.*', text)\n","    return matches.group(1)\n","\n","extract_game_id = F.udf(\n","    lambda text: get_previous_word(text),\n","    StringType()\n",")\n"]},{"cell_type":"markdown","metadata":{},"source":["Read data"]},{"cell_type":"code","execution_count":14,"metadata":{},"outputs":[{"name":"stdout","output_type":"stream","text":["root\n"," |-- cursor: string (nullable = true)\n"," |-- query_summary: struct (nullable = true)\n"," |    |-- num_reviews: long (nullable = true)\n"," |    |-- review_score: long (nullable = true)\n"," |    |-- review_score_desc: string (nullable = true)\n"," |    |-- total_negative: long (nullable = true)\n"," |    |-- total_positive: long (nullable = true)\n"," |    |-- total_reviews: long (nullable = true)\n"," |-- reviews: array (nullable = true)\n"," |    |-- element: struct (containsNull = true)\n"," |    |    |-- author: struct (nullable = true)\n"," |    |    |    |-- last_played: long (nullable = true)\n"," |    |    |    |-- num_games_owned: long (nullable = true)\n"," |    |    |    |-- num_reviews: long (nullable = true)\n"," |    |    |    |-- playtime_at_review: long (nullable = true)\n"," |    |    |    |-- playtime_forever: long (nullable = true)\n"," |    |    |    |-- playtime_last_two_weeks: long (nullable = true)\n"," |    |    |    |-- steamid: string (nullable = true)\n"," |    |    |-- comment_count: long (nullable = true)\n"," |    |    |-- language: string (nullable = true)\n"," |    |    |-- received_for_free: boolean (nullable = true)\n"," |    |    |-- recommendationid: string (nullable = true)\n"," |    |    |-- review: string (nullable = true)\n"," |    |    |-- steam_purchase: boolean (nullable = true)\n"," |    |    |-- timestamp_created: long (nullable = true)\n"," |    |    |-- timestamp_updated: long (nullable = true)\n"," |    |    |-- voted_up: boolean (nullable = true)\n"," |    |    |-- votes_funny: long (nullable = true)\n"," |    |    |-- votes_up: long (nullable = true)\n"," |    |    |-- weighted_vote_score: string (nullable = true)\n"," |    |    |-- written_during_early_access: boolean (nullable = true)\n"," |-- success: long (nullable = true)\n"," |-- filename: string (nullable = false)\n","\n"]},{"name":"stderr","output_type":"stream","text":["                                                                                \r"]}],"source":["all_games = spark.read.json(f\"gs://{BUCKET}/{BUCKET_SUBDIR}/*\", multiLine=True) \\\n","                      .withColumn(\"filename\", input_file_name())\n","all_games.printSchema()"]},{"cell_type":"code","execution_count":15,"metadata":{},"outputs":[{"name":"stderr","output_type":"stream","text":["[Stage 3:>                                                          (0 + 1) / 1]\r"]},{"name":"stdout","output_type":"stream","text":["+--------------------+--------------------+--------------------+-------+--------------------+\n","|              cursor|       query_summary|             reviews|success|            filename|\n","+--------------------+--------------------+--------------------+-------+--------------------+\n","|AoJ4qo/0rPoCfp6b6QI=|{100, null, null,...|[{{1656197665, 74...|      1|gs://steam-datala...|\n","|AoJw5pWI/v0CetHvlgM=|{100, null, null,...|[{{1643432700, 14...|      1|gs://steam-datala...|\n","|AoJ4qaHj9vICfueVkAI=|{100, null, null,...|[{{1596365869, 69...|      1|gs://steam-datala...|\n","|AoJwrseHlf0Ce/HBiAM=|{100, null, null,...|[{{1640537142, 27...|      1|gs://steam-datala...|\n","|AoJw9Y+FhIEDe/P3ugM=|{100, null, null,...|[{{1656626369, 36...|      1|gs://steam-datala...|\n","|AoJ47pP/yPACerap9QE=|{100, null, null,...|[{{1660175584, 10...|      1|gs://steam-datala...|\n","|AoJwwqSC+MgCdZSsLw==|{100, null, null,...|[{{1590322827, 27...|      1|gs://steam-datala...|\n","|AoJ49q3f3/ECfNaBhAI=|{100, null, null,...|[{{1646058950, 13...|      1|gs://steam-datala...|\n","|AoJw86KGwuoCcNHHwAE=|{100, null, null,...|[{{1650057700, 17...|      1|gs://steam-datala...|\n","|    AoJ4z7WCh7MCeLhF|{100, null, null,...|[{{1385003206, 32...|      1|gs://steam-datala...|\n","|AoJw8dOOid0Cfbv0fg==|{100, null, null,...|[{{1658370659, 66...|      1|gs://steam-datala...|\n","|AoJ404fc5esCeLrIzQE=|{100, null, null,...|[{{1561623030, 28...|      1|gs://steam-datala...|\n","|AoJ4ofHLuNICeMiCTw==|{100, null, null,...|[{{1461632695, 45...|      1|gs://steam-datala...|\n","|AoJw+uXFjOMCeL2ynwE=|{100, null, null,...|[{{1530380861, 31...|      1|gs://steam-datala...|\n","|AoJwirPN0+sCf7DRxwE=|{100, null, null,...|[{{1562413908, 52...|      1|gs://steam-datala...|\n","|AoJ4z8eN78YCcq++KQ==|{100, null, null,...|[{{1432432089, 35...|      1|gs://steam-datala...|\n","|AoJ4uq3o4NACfPLfRw==|{100, null, null,...|[{{1447654605, 36...|      1|gs://steam-datala...|\n","|AoJ4gsmG2tsCerDqdw==|{100, null, null,...|[{{1590094507, 21...|      1|gs://steam-datala...|\n","|AoJ4weLVmNQCc5f3VQ==|{100, null, null,...|[{{1537736669, 53...|      1|gs://steam-datala...|\n","|AoJ4zpfV/uYCeqS9rgE=|{100, null, null,...|[{{1566471273, 76...|      1|gs://steam-datala...|\n","+--------------------+--------------------+--------------------+-------+--------------------+\n","only showing top 20 rows\n","\n"]},{"name":"stderr","output_type":"stream","text":["                                                                                \r"]}],"source":["all_games.show() # compute intesive WARNING !"]},{"cell_type":"code","execution_count":16,"metadata":{},"outputs":[],"source":["rdd = all_games.rdd\n","rdd = rdd.repartition(5)"]},{"cell_type":"code","execution_count":17,"metadata":{},"outputs":[{"name":"stderr","output_type":"stream","text":["[Stage 4:>                                                          (0 + 2) / 2]\r"]},{"name":"stdout","output_type":"stream","text":["Number of files processed: 97\n"]},{"name":"stderr","output_type":"stream","text":["                                                                                \r"]}],"source":["print(f\"Number of files processed: {rdd.count()}\")"]},{"cell_type":"code","execution_count":18,"metadata":{},"outputs":[{"name":"stdout","output_type":"stream","text":["Nº partitons:  5\n"]}],"source":["print(\"Nº partitons: \", rdd.getNumPartitions())"]},{"cell_type":"markdown","metadata":{},"source":["### Reviews "]},{"cell_type":"code","execution_count":19,"metadata":{},"outputs":[{"name":"stdout","output_type":"stream","text":["root\n"," |-- author: struct (nullable = true)\n"," |    |-- last_played: long (nullable = true)\n"," |    |-- num_games_owned: long (nullable = true)\n"," |    |-- num_reviews: long (nullable = true)\n"," |    |-- playtime_at_review: long (nullable = true)\n"," |    |-- playtime_forever: long (nullable = true)\n"," |    |-- playtime_last_two_weeks: long (nullable = true)\n"," |    |-- steamid: string (nullable = true)\n"," |-- comment_count: long (nullable = true)\n"," |-- language: string (nullable = true)\n"," |-- received_for_free: boolean (nullable = true)\n"," |-- recommendationid: string (nullable = true)\n"," |-- review: string (nullable = true)\n"," |-- steam_purchase: boolean (nullable = true)\n"," |-- timestamp_created: long (nullable = true)\n"," |-- timestamp_updated: long (nullable = true)\n"," |-- voted_up: boolean (nullable = true)\n"," |-- votes_funny: long (nullable = true)\n"," |-- votes_up: long (nullable = true)\n"," |-- weighted_vote_score: string (nullable = true)\n"," |-- written_during_early_access: boolean (nullable = true)\n","\n"]},{"name":"stderr","output_type":"stream","text":["22/09/26 17:14:27 WARN org.apache.spark.scheduler.TaskSetManager: Lost task 0.0 in stage 7.0 (TID 11) (cluster-11ee-w-1.europe-west1-b.c.steam-data-engineering-gcp.internal executor 2): org.apache.spark.util.TaskCompletionListenerException: refCnt: 0, decrement: 1\n","\tat org.apache.spark.TaskContextImpl.invokeListeners(TaskContextImpl.scala:145)\n","\tat org.apache.spark.TaskContextImpl.markTaskCompleted(TaskContextImpl.scala:124)\n","\tat org.apache.spark.scheduler.Task.run(Task.scala:147)\n","\tat org.apache.spark.executor.Executor$TaskRunner.$anonfun$run$3(Executor.scala:498)\n","\tat org.apache.spark.util.Utils$.tryWithSafeFinally(Utils.scala:1439)\n","\tat org.apache.spark.executor.Executor$TaskRunner.run(Executor.scala:501)\n","\tat java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)\n","\tat java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)\n","\tat java.lang.Thread.run(Thread.java:750)\n","\n"]}],"source":["df_reviews = rdd.flatMap(lambda item : proc_json(item, field = \"reviews\")) \\\n","                .toDF()\n","df_reviews.printSchema()"]},{"cell_type":"markdown","metadata":{},"source":["### Game ID "]},{"cell_type":"code","execution_count":20,"metadata":{},"outputs":[{"name":"stdout","output_type":"stream","text":["root\n"," |-- gameid: string (nullable = true)\n","\n"]}],"source":["df_gameid = rdd.map(  lambda item : [ proc_json(item, field = \"reviews\"),\n","                                      proc_json(item, field = \"filename\")] ) \\\n","               .flatMap(lambda item:  [item[1] for i in item[0] ]) \\\n","               .map(lambda item : Row(gameid  = item, )).toDF()\n","\n","df_gameid = df_gameid.withColumn('gameid', extract_game_id('gameid'))\n","df_gameid.printSchema()"]},{"cell_type":"markdown","metadata":{},"source":["### Join dataframes"]},{"cell_type":"code","execution_count":21,"metadata":{},"outputs":[],"source":["w = Window().orderBy(lit('A'))\n","df_reviews = df_reviews.withColumn(\"row_num\", row_number().over(w))\n","df_gameid = df_gameid.withColumn(\"row_num\", row_number().over(w))\n","\n","df_reviews = df_reviews.join(df_gameid, on = [\"row_num\"], how = \"inner\")"]},{"cell_type":"code","execution_count":22,"metadata":{},"outputs":[{"name":"stdout","output_type":"stream","text":["root\n"," |-- row_num: integer (nullable = true)\n"," |-- author: struct (nullable = true)\n"," |    |-- last_played: long (nullable = true)\n"," |    |-- num_games_owned: long (nullable = true)\n"," |    |-- num_reviews: long (nullable = true)\n"," |    |-- playtime_at_review: long (nullable = true)\n"," |    |-- playtime_forever: long (nullable = true)\n"," |    |-- playtime_last_two_weeks: long (nullable = true)\n"," |    |-- steamid: string (nullable = true)\n"," |-- comment_count: long (nullable = true)\n"," |-- language: string (nullable = true)\n"," |-- received_for_free: boolean (nullable = true)\n"," |-- recommendationid: string (nullable = true)\n"," |-- review: string (nullable = true)\n"," |-- steam_purchase: boolean (nullable = true)\n"," |-- timestamp_created: long (nullable = true)\n"," |-- timestamp_updated: long (nullable = true)\n"," |-- voted_up: boolean (nullable = true)\n"," |-- votes_funny: long (nullable = true)\n"," |-- votes_up: long (nullable = true)\n"," |-- weighted_vote_score: string (nullable = true)\n"," |-- written_during_early_access: boolean (nullable = true)\n"," |-- gameid: string (nullable = true)\n","\n"]}],"source":["df_reviews.printSchema()"]},{"cell_type":"code","execution_count":null,"metadata":{},"outputs":[],"source":["df_reviews.count() # compute intesive WARNING !"]},{"cell_type":"code","execution_count":null,"metadata":{},"outputs":[],"source":["df_reviews.take(1) # compute intesive WARNING !"]},{"cell_type":"code","execution_count":23,"metadata":{},"outputs":[{"name":"stdout","output_type":"stream","text":["root\n"," |-- row_num: integer (nullable = true)\n"," |-- comment_count: long (nullable = true)\n"," |-- language: string (nullable = true)\n"," |-- received_for_free: boolean (nullable = true)\n"," |-- recommendationid: string (nullable = true)\n"," |-- review: string (nullable = true)\n"," |-- steam_purchase: boolean (nullable = true)\n"," |-- timestamp_created: long (nullable = true)\n"," |-- timestamp_updated: long (nullable = true)\n"," |-- voted_up: boolean (nullable = true)\n"," |-- votes_funny: long (nullable = true)\n"," |-- votes_up: long (nullable = true)\n"," |-- weighted_vote_score: string (nullable = true)\n"," |-- written_during_early_access: boolean (nullable = true)\n"," |-- gameid: string (nullable = true)\n"," |-- author_last_played: long (nullable = true)\n"," |-- author_num_games_owned: long (nullable = true)\n"," |-- author_num_reviews: long (nullable = true)\n"," |-- author_playtime_at_review: long (nullable = true)\n"," |-- author_playtime_forever: long (nullable = true)\n"," |-- author_playtime_last_two_weeks: long (nullable = true)\n"," |-- author_steamid: string (nullable = true)\n","\n"]}],"source":["df_reviews_flat = flatten_df(df_reviews) \n","df_reviews_flat.printSchema()"]},{"cell_type":"markdown","metadata":{},"source":["### Write into BigQuery"]},{"cell_type":"markdown","metadata":{},"source":["- https://github.com/GoogleCloudDataproc/spark-bigquery-connector"]},{"cell_type":"code","execution_count":25,"metadata":{},"outputs":[{"name":"stderr","output_type":"stream","text":["22/09/26 17:16:07 WARN org.apache.spark.sql.execution.window.WindowExec: No Partition Defined for Window operation! Moving all data to a single partition, this can cause serious performance degradation.\n","22/09/26 17:16:07 WARN org.apache.spark.sql.execution.window.WindowExec: No Partition Defined for Window operation! Moving all data to a single partition, this can cause serious performance degradation.\n","22/09/26 17:16:08 WARN org.apache.spark.sql.execution.window.WindowExec: No Partition Defined for Window operation! Moving all data to a single partition, this can cause serious performance degradation.\n","22/09/26 17:16:08 WARN org.apache.spark.sql.execution.window.WindowExec: No Partition Defined for Window operation! Moving all data to a single partition, this can cause serious performance degradation.\n","22/09/26 17:16:08 WARN org.apache.spark.sql.execution.window.WindowExec: No Partition Defined for Window operation! Moving all data to a single partition, this can cause serious performance degradation.\n","22/09/26 17:16:09 WARN org.apache.spark.sql.execution.window.WindowExec: No Partition Defined for Window operation! Moving all data to a single partition, this can cause serious performance degradation.\n","22/09/26 17:16:09 WARN org.apache.spark.sql.execution.window.WindowExec: No Partition Defined for Window operation! Moving all data to a single partition, this can cause serious performance degradation.\n","22/09/26 17:16:09 WARN org.apache.spark.sql.execution.window.WindowExec: No Partition Defined for Window operation! Moving all data to a single partition, this can cause serious performance degradation.\n","                                                                                \r"]}],"source":["# Saving the data to BigQuery\n","df_reviews_flat.write \\\n","  .format('bigquery') \\\n","  .option('table', f'{BQ_PROJECT_ID}.{BQ_DATASET}.{BQ_TABLE}') \\\n","  .mode(\"append\") \\\n","  .option(\"temporaryGcsBucket\",TEMP_BUCKET) \\\n","  .save()"]}],"metadata":{"kernelspec":{"display_name":"Python 3","language":"python","name":"python3"},"language_info":{"codemirror_mode":{"name":"ipython","version":3},"file_extension":".py","mimetype":"text/x-python","name":"python","nbconvert_exporter":"python","pygments_lexer":"ipython3","version":"3.8.13"},"vscode":{"interpreter":{"hash":"b60d76cc204c3a2caf44ab1915661f58ff09a8919b71fd1dfd9e0ece822b469f"}}},"nbformat":4,"nbformat_minor":2}


--------------------------------------------------------------------------------
/spark/spark_all_games_reviews.py:
--------------------------------------------------------------------------------
  1 | 
  2 | import sys
  3 | import re
  4 | from pyspark.sql import SparkSession
  5 | from pyspark.conf import SparkConf
  6 | from pyspark import SparkContext
  7 | from pyspark.sql.functions import input_file_name
  8 | from pyspark.sql import functions as F
  9 | from pyspark.sql import Row
 10 | from pyspark.sql.functions import row_number,lit
 11 | from pyspark.sql.window import Window
 12 | from pyspark.sql.types import StringType
 13 | 
 14 | 
 15 | def flatten_df(nested_df):
 16 |     flat_cols = [c[0] for c in nested_df.dtypes if c[1][:6] != 'struct']
 17 |     nested_cols = [c[0] for c in nested_df.dtypes if c[1][:6] == 'struct']
 18 | 
 19 |     flat_df = nested_df.select(flat_cols +
 20 |                                [F.col(nc+'.'+c).alias(nc+'_'+c)
 21 |                                 for nc in nested_cols
 22 |                                 for c in nested_df.select(nc+'.*').columns])
 23 |     return flat_df
 24 | 
 25 | def proc_json(raw_df, field)  :
 26 |     rows = raw_df[field]
 27 |     return(rows)
 28 | 
 29 | def get_previous_word(text):
 30 |     matches = re.search('.*/(\d+)-.*', text)
 31 |     return matches.group(1)
 32 | 
 33 | extract_game_id = F.udf(
 34 |     lambda text: get_previous_word(text),
 35 |     StringType()
 36 | )
 37 | 
 38 | 
 39 | GCP_PROJECT_ID = sys.argv[1]
 40 | BQ_DATASET     = sys.argv[2]
 41 | BQ_TABLE       = sys.argv[3]
 42 | BUCKET         = sys.argv[4] 
 43 | BUCKET_SUBDIR  = sys.argv[5]
 44 | TEMP_BUCKET    = sys.argv[6]
 45 | 
 46 | def main():
 47 | 
 48 |     conf = SparkConf() \
 49 |         .setAppName('steam-gcp-dataproc') \
 50 |         .set("spark.hadoop.google.cloud.auth.service.account.enable", "true") 
 51 | 
 52 |     sc = SparkContext(conf=conf)
 53 | 
 54 |     spark = SparkSession.builder \
 55 |         .config(conf=sc.getConf()) \
 56 |         .getOrCreate()
 57 | 
 58 | 
 59 |     # READ DATA
 60 |     all_games = spark.read.json(f"gs://{BUCKET}/{BUCKET_SUBDIR}/*", multiLine=True) \
 61 |                         .withColumn("filename", input_file_name())
 62 | 
 63 |     rdd = all_games.rdd
 64 |     rdd = rdd.repartition(5)
 65 | 
 66 | 
 67 |     # REVIEWS
 68 |     df_reviews = rdd.flatMap(lambda item : proc_json(item, field = "reviews")) \
 69 |                     .toDF()
 70 | 
 71 |     # GAMEID
 72 |     df_gameid = rdd.map(  lambda item : [ proc_json(item, field = "reviews"),
 73 |                                         proc_json(item, field = "filename")] ) \
 74 |                 .flatMap(lambda item:  [item[1] for i in item[0] ]) \
 75 |                 .map(lambda item : Row(gameid  = item, )).toDF()
 76 | 
 77 |     df_gameid = df_gameid.withColumn('gameid', extract_game_id('gameid'))
 78 | 
 79 |     # JOIN DATAFRAMES
 80 |     w = Window().orderBy(lit('A'))
 81 |     df_reviews = df_reviews.withColumn("row_num", row_number().over(w))
 82 |     df_gameid = df_gameid.withColumn("row_num", row_number().over(w))
 83 | 
 84 |     df_reviews = df_reviews.join(df_gameid, on = ["row_num"], how = "inner")
 85 | 
 86 |     df_reviews_flat = flatten_df(df_reviews) 
 87 | 
 88 |     # BIGQUERY WRITE
 89 |     df_reviews_flat.write \
 90 |     .format('bigquery') \
 91 |     .option('table', f'{GCP_PROJECT_ID}.{BQ_DATASET}.{BQ_TABLE}') \
 92 |     .mode("append") \
 93 |     .option("temporaryGcsBucket",TEMP_BUCKET) \
 94 |     .save()
 95 | 
 96 | 
 97 | 
 98 | if __name__ == "__main__":
 99 |     main()
100 | 


--------------------------------------------------------------------------------
/terraform/.gitignore:
--------------------------------------------------------------------------------
1 | .terraform/
2 | terraform.tfstate
3 | terraform.tfstate.backup
4 | .terraform-version
5 | .terraform.lock.hcl
6 | .terraform.tfstate.lock.info


--------------------------------------------------------------------------------
/terraform/README.md:
--------------------------------------------------------------------------------
 1 | # Terraform
 2 | 
 3 | ## Description
 4 | `terraform apply` create de following resources: 
 5 | - GCP bucket  `steam-datalake-store-alldata`
 6 | - GCP bucket  `steam-datalake-reviews`
 7 | - BigQuery dataset `steam_raw`
 8 | - Storage transfer job from AWS from S3 bucket to GCP bucket `steam-datalake-store-alldata`
 9 | - Storage transfer job from AWS from S3 bucket to GCP bucket `steam-datalake-reviews`
10 | 
11 | 
12 | ## Config
13 | 
14 | The **Terraform** service uses **Vault** to safely provide temporal **AWS** credentials
15 | 
16 | Install terraform, valut: 
17 | 
18 | - https://learn.hashicorp.com/tutorials/terraform/install-cli
19 | - https://learn.hashicorp.com/tutorials/terraform/secrets-vault
20 | 
21 | Start server:
22 | 
23 | ```{bash}
24 | cd terraform
25 | vault server -dev -dev-root-token-id="steam"
26 | ```
27 | Export credentials
28 | ```{bash}
29 | export TF_VAR_aws_access_key=<AWS_ACCESS_KEY_ID>
30 | export TF_VAR_aws_secret_key=<AWS_SECRET_ACCESS_KEY>
31 | export VAULT_ADDR=http://127.0.0.1:8200
32 | export VAULT_TOKEN=steam
33 | ```
34 | 
35 | Initialize admin workspace: 
36 | 
37 | ```{bash}
38 | cd vault-admin-workspace/
39 | terraform init
40 | terraform apply
41 | ```
42 | Initialize admin workspace: 
43 | ```{bash}
44 | cd ../operator-workspace/
45 | terraform init
46 | terraform apply
47 | ```
48 | 
49 | 
50 | 
51 | 


--------------------------------------------------------------------------------
/terraform/operator-workspace/main.tf:
--------------------------------------------------------------------------------
  1 | terraform {
  2 |   required_version = ">= 1.0"
  3 |   backend "local" {
  4 |         path = "terraform.tfstate"
  5 |   } 
  6 |   required_providers {
  7 |     google = {
  8 |       source = "hashicorp/google"
  9 |     }
 10 |   }
 11 | }
 12 | 
 13 | provider "google" {
 14 |   project     = var.project
 15 |   region      = var.region
 16 |   credentials = file(var.gcp_credentials) # Use this if you do not want to set env-var GOOGLE_APPLICATION_CREDENTIALS
 17 | }
 18 | 
 19 | #--------------------------ADMIN VAULT----------------------#
 20 | data "terraform_remote_state" "admin" {
 21 |   backend = "local"
 22 | 
 23 |   config = {
 24 |     path = var.terraform_admin_path
 25 |   }
 26 | }
 27 | 
 28 | data "vault_aws_access_credentials" "creds" {
 29 |   backend = data.terraform_remote_state.admin.outputs.backend
 30 |   role    = data.terraform_remote_state.admin.outputs.role
 31 | }
 32 | 
 33 | provider "aws" {
 34 |   region     = var.aws_vault_region
 35 |   access_key = data.vault_aws_access_credentials.creds.access_key
 36 |   secret_key = data.vault_aws_access_credentials.creds.secret_key
 37 | }
 38 | #-----------------------------------------------------------#
 39 | 
 40 | resource "google_storage_bucket" "steam-dataset" {
 41 |   name     =  var.gcp_bucket_dataset
 42 |   location = var.region
 43 | 
 44 |   # Optional, but recommended settings:
 45 |   storage_class               = var.storage_class
 46 |   uniform_bucket_level_access = true
 47 | 
 48 |   versioning {
 49 |     enabled = false
 50 |   }
 51 | 
 52 |   lifecycle_rule {
 53 |     action {
 54 |       type = "Delete"
 55 |     }
 56 |     condition {
 57 |       age = 365 // days
 58 |     }
 59 |   }
 60 | 
 61 |   force_destroy = true
 62 | }
 63 | 
 64 | resource "google_storage_bucket" "steam-reviews" {
 65 |   name     =  var.gcp_bucket_reviews
 66 |   location = var.region
 67 | 
 68 |   # Optional, but recommended settings:
 69 |   storage_class               = var.storage_class
 70 |   uniform_bucket_level_access = true
 71 | 
 72 |   versioning {
 73 |     enabled = false
 74 |   }
 75 | 
 76 |   lifecycle_rule {
 77 |     action {
 78 |       type = "Delete"
 79 |     }
 80 |     condition {
 81 |       age = 365 // days
 82 |     }
 83 |   }
 84 | 
 85 |   force_destroy = true
 86 | }
 87 | 
 88 | resource "google_bigquery_dataset" "steam-raw" {
 89 |   dataset_id                 = var.gcp_bq_dataset_steam_raw
 90 |   project                    = var.project
 91 |   location                   = var.region
 92 |   delete_contents_on_destroy = true
 93 | }
 94 | 
 95 | 
 96 | 
 97 | # TRANSFER AWS -> GCP
 98 | 
 99 | data "google_storage_transfer_project_service_account" "default" {
100 |   project = var.project
101 | }
102 | 
103 | resource "google_storage_bucket_iam_member" "s3-transfer-dataset" {
104 |   bucket     = google_storage_bucket.steam-dataset.name
105 |   role       = "roles/storage.admin"
106 |   member     = "serviceAccount:${data.google_storage_transfer_project_service_account.default.email}"
107 | }
108 | 
109 | resource "google_storage_bucket_iam_member" "s3-transfer-reviews" {
110 |   bucket     = google_storage_bucket.steam-reviews.name
111 |   role       = "roles/storage.admin"
112 |   member     = "serviceAccount:${data.google_storage_transfer_project_service_account.default.email}"
113 | }
114 | 
115 | resource "google_storage_transfer_job" "steam-dataset" {
116 |   description = "Nightly backup of S3 bucket"
117 |   project     = var.project
118 | 
119 |   transfer_spec {
120 |     
121 |     transfer_options {
122 |       delete_objects_unique_in_sink = false
123 |     }
124 |     aws_s3_data_source {
125 |       bucket_name = var.aws_s3_bucket_dataset
126 |       aws_access_key {
127 |       access_key_id     = data.vault_aws_access_credentials.creds.access_key # !! 
128 |       secret_access_key = data.vault_aws_access_credentials.creds.secret_key # !!
129 |       }
130 | 
131 |     }
132 |     gcs_data_sink {
133 |       bucket_name = var.gcp_bucket_dataset
134 |       path = "raw/"
135 |     }
136 |   }
137 | 
138 |   schedule {
139 |     schedule_start_date {
140 |       year  = 2022
141 |       month = 9
142 |       day   = 21
143 |     }
144 |   }
145 | 
146 |   depends_on = [google_storage_bucket_iam_member.s3-transfer-dataset]
147 | }
148 | 
149 | 
150 | resource "google_storage_transfer_job" "steam-reviews" {
151 |   description = "Nightly backup of S3 bucket"
152 |   project     = var.project
153 | 
154 |   transfer_spec {
155 |     
156 |     transfer_options {
157 |       delete_objects_unique_in_sink = false
158 |     }
159 |     aws_s3_data_source {
160 |       bucket_name = var.aws_s3_bucket_reviews
161 |       aws_access_key {
162 |       access_key_id     = data.vault_aws_access_credentials.creds.access_key # !! 
163 |       secret_access_key = data.vault_aws_access_credentials.creds.secret_key # !!
164 |       }
165 | 
166 |     }
167 |     gcs_data_sink {
168 |       bucket_name = var.gcp_bucket_reviews
169 |       path = "raw/"
170 |     }
171 |   }
172 | 
173 |   schedule {
174 |     schedule_start_date {
175 |       year  = 2022
176 |       month = 9
177 |       day   = 21
178 |     }
179 |   }
180 | 
181 |   depends_on = [google_storage_bucket_iam_member.s3-transfer-reviews]
182 | }


--------------------------------------------------------------------------------
/terraform/operator-workspace/variables.tf:
--------------------------------------------------------------------------------
 1 | 
 2 | 
 3 | #locals {
 4 | #  data_lake_bucket = "steam-bucket"
 5 | #}
 6 | 
 7 | variable "project" {
 8 |   description = "Your GCP Project ID"
 9 |   default = "steam-data-engineering-gcp"
10 | }
11 | 
12 | variable "gcp_credentials" {
13 |   description = "JSON service account credentials"
14 |   default     = "/home/vyago-gcp/.google/credentials/google_credentials.json"
15 |   type        = string
16 | }
17 | 
18 | variable "aws_credentials" {
19 |   description = "JSON service account credentials"
20 |   default     = "/home/vyago/.aws/credentials"
21 |   type        = string
22 | }
23 | 
24 | variable "aws_conf" {
25 |   description = "JSON service account credentials"
26 |   default     = "/home/vyago/.aws/conf"
27 |   type        = string
28 | }
29 | 
30 | #--------------------------ADMIN VAULT----------------------#
31 | variable "name" { 
32 |   default = "dynamic-aws-creds-operator" 
33 | }
34 | 
35 | variable "aws_vault_region" { 
36 |   default = "us-east-1" # change
37 | }
38 | 
39 | variable "terraform_admin_path" { 
40 |   default = "../vault-admin-workspace/terraform.tfstate" 
41 | }
42 | 
43 | variable "ttl" { 
44 |   default = "1" 
45 | }
46 | #------------------------------------------------#
47 | variable "region" {
48 |   description = "Region for GCP resources. Choose as per your location: https://cloud.google.com/about/locations"
49 |   default     = "europe-southwest1"
50 |   type        = string
51 | }
52 | 
53 | variable "gcp_bucket_dataset" {
54 |   description = "Bucket name for storage steam appdata, steamspy & scraped"
55 |   default     = "steam-datalake-store-alldata"
56 |   type        = string
57 | }
58 | 
59 | variable "gcp_bucket_reviews" {
60 |   description = "Bucket name for storage steam appdata, steamspy & scraped"
61 |   default     = "steam-datalake-reviews"
62 |   type        = string
63 | }
64 | 
65 | variable "aws_s3_bucket_dataset"{
66 |   description = "S3 Bucket steam dataset"
67 |   default     = "steam-store-alldata"
68 |   type        = string
69 | }
70 | 
71 | variable "aws_s3_bucket_reviews"{
72 |   description = "S3 Bucket steam reviews"
73 |   default     = "steam-reviews-lite" # !!!! Reduced version, "steam-reviews" full
74 |   type        = string
75 | }
76 | 
77 | variable "storage_class" {
78 |   description = "Storage class type for your bucket. Check official docs for more info."
79 |   default     = "STANDARD"
80 | }
81 | 
82 | variable "gcp_bq_dataset_steam_raw" {
83 |   description = "Storage class type for your bucket. Check official docs for more info."
84 |   default     = "steam_raw"
85 |   type        = string
86 | }
87 | 


--------------------------------------------------------------------------------
/terraform/vault-admin-workspace/main.tf:
--------------------------------------------------------------------------------
 1 | variable "aws_access_key" {}
 2 | variable "aws_secret_key" {}
 3 | variable "name" { default = "dynamic-aws-creds-vault-admin" }
 4 | 
 5 | terraform {
 6 |   backend "local" {
 7 |     path = "terraform.tfstate"
 8 |   }
 9 | }
10 | 
11 | provider "vault" {}
12 | 
13 | resource "vault_aws_secret_backend" "aws" {
14 |   access_key = var.aws_access_key
15 |   secret_key = var.aws_secret_key
16 |   path       = "${var.name}-path"
17 | 
18 |   default_lease_ttl_seconds = "120"
19 |   max_lease_ttl_seconds     = "240"
20 | }
21 | 
22 | resource "vault_aws_secret_backend_role" "admin" {
23 |   backend         = vault_aws_secret_backend.aws.path
24 |   name            = "${var.name}-role"
25 |   credential_type = "iam_user"
26 | 
27 |   policy_document = <<EOF
28 | {
29 |   "Version": "2012-10-17",
30 |   "Statement": [
31 |     {
32 |       "Effect": "Allow",
33 |       "Action": [
34 |         "iam:*", "s3:*"
35 |       ],
36 |       "Resource": "*"
37 |     }
38 |   ]
39 | }
40 | EOF
41 | }
42 | 
43 | output "backend" {
44 |   value = vault_aws_secret_backend.aws.path
45 | }
46 | 
47 | output "role" {
48 |   value = vault_aws_secret_backend_role.admin.name
49 | }


--------------------------------------------------------------------------------
