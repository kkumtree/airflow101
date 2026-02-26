# 프로젝트 안내

본 프로젝트는 로컬 환경에서 Apache Airflow와 Spark를 활용한 주식 데이터 파이프라인을 구축하고 실행하기 위한 가이드입니다. 🇬🇧 [English Version](README.en.md)

## 1. 사전 요구 사항 (Prerequisites)

이 프로젝트를 로컬에서 실행하기 위해서는 다음 도구들이 설치되어 있어야 합니다.

*   **Docker**: 컨테이너 환경을 구동하기 위해 필요합니다.
    *   [Docker 공식 설치 페이지](https://docs.docker.com/get-docker/)
*   **Astro CLI**: 로컬 Airflow 환경을 쉽게 구성하고 관리하기 위한 필수 도구입니다.
    *   [Astro CLI 공식 설치 페이지](https://docs.astronomer.io/astro/cli/install-cli)

## 2. Docker 이미지 필수 빌드

Airflow 작업(태스크)에서 사용할 커스텀 Spark Docker 이미지들을 먼저 빌드해야 합니다. 터미널의 현재 위치가 프로젝트 폴더인지 확인한 후, 아래 명령어를 차례대로 실행하거나 한 줄로 복사하여 실행하세요.

```bash
docker build -t airflow/spark-master ./spark/master && docker build -t airflow/spark-worker ./spark/worker && docker build -t airflow/stock-app ./spark/notebooks/stock_transform  
```
*(이 명령어는 코드 처리에 필요한 `spark-master`, `spark-worker`, 그리고 실제 데이터를 변환하는 `stock-app` 컨테이너 이미지를 각각 생성합니다.)*

## 3. Airflow 로컬 환경 관리 (Astro 명령어)

Astro CLI를 사용하여 로컬 Airflow 환경을 제어합니다.

*   **환경 시작하기 (Start)**
    ```bash
    astro dev start
    ```
    이 명령어를 실행하면 필요한 모든 컨테이너(웹서버, 스케줄러, Postgres DB 등)가 구동되며 로컬 Airflow 환경(`http://localhost:8080`)에 접속할 수 있습니다.

*   **환경 중지하기 (Stop)**
    ```bash
    astro dev stop
    ```
    작업을 잠시 멈추고 리소스를 절약하기 위해 컨테이너를 중지합니다. **기존 데이터베이스와 설정 정보는 그대로 보존됩니다.**

*   **환경 삭제하기 (Kill) - ⚠️ 주의**
    ```bash
    astro dev kill
    ```
    **주의:** 이 명령어는 단순히 실행 중인 컨테이너를 종료하는 것을 넘어서 **Airflow 메타데이터 DB(Postgres)까지 완전히 삭제**합니다. 기존의 워크플로 실행 기록(런 히스토리)과 설정해둔 Connections, Variables 정보가 전부 초기화되므로 완전히 초기 상태부터 다시 시작해야 할 때만 사용하세요.

## 4. 데이터 정상 적재 확인하기

Airflow UI에서 DAG(워크플로) 실행이 성공적으로 완료되었다면, 시스템에 내장된 로컬 Postgres DB에 접속하여 가공된 주식 데이터가 제대로 들어갔는지 확인할 수 있습니다.

**1) 터미널 접속 명령어 실행:**
```bash
PGPASSWORD=postgres psql -h localhost -p 5432 -U postgres -d postgres
```

**2) 쿼리를 통한 데이터 확인 (SQL):**

*   **최근 10일치 주가 테이블 데이터 조회:**
    ```sql
    SELECT date, open, high, low, close, volume
    FROM stock_market
    ORDER BY date DESC
    LIMIT 10;
    ```
*   **현재 테이블에 들어있는 총 데이터 건수 확인하기:**
    ```sql
    SELECT COUNT(*) FROM stock_market;
    ```
*   **DB 접속 종료하고 터미널로 돌아오기:**
    ```text
    \q
    ```
