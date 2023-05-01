# 6주차 Airflow 고급기능 

## Airflow 와 K8s 

Airflow 의 DAG 를 K8s 에서 돌리고 싶다면 `KubernetesExecutor` 를 쓰면 되고, 특정 테스크만 K8s 에서 돌리고 싶다면 `KubernetesPodOperator` 를 쓰면 된다. 
- 이러면 K8s Pod 에서 DAG 를 다 실행하거나 특정 Task 를 실행하거나 그렇다.

## DBT (Data Build Tool) 소개

데이터웨어하우스에서 ELT 을 잘하게 해주는 오픈소스 솔루션 (상용화도 하는중.)

데이터 웨어하우스는 redshift, snowflake, Bigquery, Spark 를 지원한다.

DBT 의 스냅샷 기능은 유용하다고 함. 
- 데이터의 히스토리 변화를 추적하고 저장하는 기능을 말하며 이 기능을 제공해줌. 
  - 시간에 따라 변화하는 데이터를 추적하는 즉 슬로우리 차원의 변화를 추적하는 기능. 
  - 슬로우리 차원이란 시간애 따라 값이 변화하는 데이터를 말함.

## SqlToS3Operator 

내부적으로 Pandas 를 쓰고있고 이건 메모리에 모든 데이터를 다루는 식이라서 큰 데이터에는 문제가 있을 수 있음. 

큰 데이터가 아니거나 Increment Update 의 경우에는 이걸 써도 괜찮다고함. 

Udemy 에서는 CustomOperator 를 만들어서 Sql -> File 에다가 데이터를 쓰는 식으로 헀다고함. 그리고 하나의 큰 File 이 아니라 File 을 여러개로 만들어서 병렬로 업로드 하는식으로. 
- File 로 만들때는 MySQL 이나 Postgresql 에서 읽을 수 있는 형식의 파일이나 덤프로.

## SQL 테스트 도 있더라 

## Airflow Configuration To Production Usage

- `airflow.cfg` in `/var/lib/airflow/airflow.cfg`
  - airflow 에 대한 설정 정보가 들어있는 파일. 설정을 변경하려면 webserver 와 scheduler 를 restart 해야한다. 
  - `/var/lib/airflow/dags` 에 dag_dir_list_interval 는 dags 폴더를 얼마나 자주 스캔하는지 결정한다. (추가된 dag 나 삭제된 dag 를 반영)
    - dag 가 있는 곳들 `/var/lib/airflow/dags`

- Airflow Database 를 sqlite -> MySQL 이나 Postgresql 로 변경해야한다. 
  - 이 DB 도 주기적으로 Backup 이 되야한다. 기록들을 가지고 있어야함.

- Executor 사용
  - 기본적으로 LocalExecutor 를 쓰지만 K8s 에서는 KubernetesExecutor 를 쓰고 클러스터 환경에서는 CeleryExecutor 를 쓸수도 있다.
  - 이 설정도 `airflow.cfg` 에서 변경

- airflow 2.0 에서는 authentication 장치가 있어야한다. user 와 password 를 통해서 로그인가능. 

- Large Disk Volume for log and local data
  - 최소 100GB 
  - 그리고 이런 데이터는 주기적으로 지워야한다. 

- From Scale Up to Scale out 

- Add Health check for monitoring 

- Cloud Service 
  - GCP 와 AWS 에서 Airflow 를 Managed Service 를 이용가능.

## Airflow API 및 모니터링 

API Example 
- 특정 DAG 를 트리거하기 
- 모든 DAG 리스트하기 
- 모든 Variable 리스트하기 
- [API 문서 참조](https://airflow.apache.org/docs/apache-airflow/stable/stable-rest-api-ref.html#tag/DAG)

## 구글 스프레드 시트와 Airflow 의 연동

```python
"""
 - 구글 스프레드시트에서 읽기를 쉽게 해주는 모듈입니다. 아직은 쓰는 기능은 없습니다만 쉽게 추가 가능합니다.
 - 메인 함수는 get_google_sheet_to_csv입니다.
  - 이는 google sheet API를 통해 구글 스프레드시트를 읽고 쓰는 것이 가능하게 해줍니다.
  - 읽으려는 시트(탭)가 있는 스프레드시트 파일이 구글 서비스 어카운트 이메일과 공유가 되어있어야 합니다.
  - Airflow 상에서는 서비스어카운트 JSON 파일의 내용이 google_sheet_access_token이라는 이름의 Variable로 저장되어 있어야 합니다.
    - 이 이메일은 iam.gserviceaccount.com로 끝납니다.
    - 이 Variable의 내용이 매번 파일로 쓰여지고 그 파일이 구글에 권한 체크를 하는데 사용되는데 이 파일은 local_data_dir Variable로 지정된 로컬 파일 시스템에 저장된다. 이 Variable은 보통 /var/lib/airflow/data/로 설정되며 이를 먼저 생성두어야 한다 (airflow 사용자)
  - JSON 기반 서비스 어카운트를 만들려면 이 링크를 참고하세요: https://denisluiz.medium.com/python-with-google-sheets-service-account-step-by-step-8f74c26ed28e
 - 아래 2개의 모듈 설치가 별도로 필요합니다.
  - pip3 install oauth2client
  - pip3 install gspread
 - get_google_sheet_to_csv 함수:
  - 첫 번째 인자로 스프레드시트 링크를 제공. 이 시트를 service account 이메일과 공유해야합니다.
  - 두 번째 인자로 데이터를 읽어올 tab의 이름을 지정합니다.
  - 세 번째 인자로 지정된 test.csv로 저장합니다.
gsheet.get_google_sheet_to_csv(
    'https://docs.google.com/spreadsheets/d/1hW-_16OqgctX-_lXBa0VSmQAs98uUnmfOqvDYYjuE50/',
    'Test',
    'test.csv',
)
 - 여기 예제에서는 아래와 같이 테이블을 만들어두고 이를 구글스프레드시트로부터 채운다
CREATE TABLE keeyong.spreadsheet_copy_testing (
    col1 int,
    col2 int,
    col3 int,
    col4 int
);
"""
from airflow import DAG
from airflow.operators.python import PythonOperator
from airflow.providers.amazon.aws.transfers.s3_to_redshift import S3ToRedshiftOperator
from airflow.models import Variable

from datetime import datetime
from datetime import timedelta
from plugins import gsheet
from plugins import s3

import requests
import logging
import psycopg2
import json


def download_tab_in_gsheet(**context):
    url = context["params"]["url"]
    tab = context["params"]["tab"]
    table = context["params"]["table"]
    data_dir = Variable.get("local_data_dir")

    gsheet.get_google_sheet_to_csv(
        url,
        tab,
        data_dir+'{}.csv'.format(table)
    )
     

def copy_to_s3(**context):
    table = context["params"]["table"]
    s3_conn_id = "aws_conn_id"
    s3_bucket = "grepp-data-engineering"
    s3_key = table
    data_dir = Variable.get("local_data_dir")
    local_files_to_upload = [ data_dir+'{}.csv'.format(table) ]
    replace = True

    s3.upload_to_s3(s3_conn_id, s3_bucket, s3_key, local_files_to_upload, replace)


dag = DAG(
    dag_id = 'Gsheet_to_Redshift',
    start_date = datetime(2021,11,27), # 날짜가 미래인 경우 실행이 안됨
    schedule = '0 9 * * *',  # 적당히 조절
    max_active_runs = 1,
    max_active_tasks = 2,
    catchup = False,
    default_args = {
        'retries': 1,
        'retry_delay': timedelta(minutes=3),
    }
)

sheets = [
    {
        "url": "https://docs.google.com/spreadsheets/d/1hW-_16OqgctX-_lXBa0VSmQAs98uUnmfOqvDYYjuE50/",
        "tab": "Test",
        "schema": "keeyong",
        "table": "spreadsheet_copy_testing"
    }
]

for sheet in sheets:
    download_tab_in_gsheet = PythonOperator(
        task_id = 'download_{}_in_gsheet'.format(sheet["table"]),
        python_callable = download_tab_in_gsheet,
        params = sheet,
        dag = dag)

    copy_to_s3 = PythonOperator(
        task_id = 'copy_{}_to_s3'.format(sheet["table"]),
        python_callable = copy_to_s3,
        params = {
            "table": sheet["table"]
        },
        dag = dag)

    run_copy_sql = S3ToRedshiftOperator(
        task_id = 'run_copy_sql_{}'.format(sheet["table"]),
        s3_bucket = "grepp-data-engineering",
        s3_key = sheet["table"],
        schema = sheet["schema"],
        table = sheet["table"],
        copy_options=['csv', 'IGNOREHEADER 1'],
        method = 'REPLACE',
        redshift_conn_id = "redshift_dev_db",
        dag = dag
    )

    download_tab_in_gsheet >> copy_to_s3 >> run_copy_sql
```

## Dag Dependencies 

Trigger 하는 방법은 크게 두 가지가 있다. 
- Explicit Trigger (A -> B 테스크일 때 A 가 B 를 트리거하는 것)
  - TriggerDagOperator 로 실행할 수 있다. 
- Reactive Trigger (A -> B 테스크일 때 B 가 A 를 끝났는지 조회해서 트리거하는 것.)
  - ExternalTaskSensor 를 통해 모니터링할 테스크를 지정해놔야한다. 모니터링을 하는 테스트크는 계속 실행된다.

- BranchPythonOperator 도 있다. 상황에 따라서 트리거 할 지 말지를 결정하는 애 

### TriggerDagOperator 

```python
trigger_b = TriggerDagOperator(
  task_id = "taskId_B", 
  trigger_dag_id = "트리거하려는 DAG ID",
  conf = {} # 넘겨주고 싶은 정보들,
  execution_date = "{{ ds }}"
)
```

### ExternalTaskSensor

```python
waiting_for_end_of_dag_a = ExternalTaskSensor(
  task_id = "task Id", 
  external_dag_id = "기다릴 DAG ID", 
  external_task_id = "기다릴 task id",
  timeout = 300 # 기다릴 시간, 
  mode = "reschedule"
)
```