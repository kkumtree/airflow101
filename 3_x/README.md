# 프로젝트 안내

본 프로젝트는 로컬 환경에서 Apache Airflow와 Spark를 활용한 주식 데이터 파이프라인을 구축하고 실행하기 위한 가이드입니다. 🇬🇧 [English Version](README.en.md)

## 1. 사전 요구 사항 (Prerequisites)

이 프로젝트를 로컬에서 실행하기 위해서는 다음 도구들이 설치되어 있어야 합니다.

*   **Docker**: 컨테이너 하위 환경을 실행하기 위해 필요합니다.
    *   [Docker 데스크톱 공식 설치 링크](https://docs.docker.com/get-docker/)
*   **Astro CLI**: 로컬 Airflow 환경을 통합적으로 구성하고 간편하게 띄우기 위한 관리 도구입니다.
    *   [Astro CLI 공식 설치 링크](https://docs.astronomer.io/astro/cli/install-cli)

## 2. Docker 이미지 빌드

Airflow 워크플로 중 데이터를 가공하는 작업 등에 사용할 커스텀 Docker 이미지들을 먼저 빌드해야 합니다. 작업 디렉토리(터미널)에서 아래 명령어를 복사하여 붙여넣고 실행하세요.

```bash
docker build -t airflow/spark-master ./spark/master && docker build -t airflow/spark-worker ./spark/worker && docker build -t airflow/stock-app ./spark/notebooks/stock_transform  
```
*(위 명령어는 `spark-master`, `spark-worker`, 그리고 분산 처리를 위한 `stock-app` 총 세 개의 컨테이너 이미지를 각각 빌드합니다.)*

## 3. Airflow 로컬 환경 실행 및 관리 (Astro)

Astro CLI 명령어로 로컬 Airflow를 실행하고 제어할 수 있습니다.

*   **환경 시작하기**
    ```bash
    astro dev start
    ```
    위 명령어를 실행하면 필요한 모든 컨테이너(Webserver, Scheduler, Postgres 등)가 백그라운드에서 구동되며, `http://localhost:8080` 에 접속하여 Airflow UI를 볼 수 있습니다.

*   **환경 중지하기**
    ```bash
    astro dev stop
    ```
    작업 환경을 일시적으로 중단합니다. 저장된 워크플로나 DB 데이터는 보존됩니다.

*   **⚠️ 환경 삭제 및 강제 종료 (주의)**
    ```bash
    astro dev kill
    ```
    **주의:** 이 명령어는 단순히 컨테이너를 종료시키는 것을 넘어서, **Airflow의 메타데이터 DB(Postgres)까지 완전히 삭제**합니다. 모든 대시보드 실행 기록(Run History)과 Connections, Variables가 초기화되니 프로젝트를 완전히 처음부터 깔끔하게 다시 시작하고 싶을 때만 사용하세요.

## 4. 데이터 적재 확인 (워크플로 실행 성공 후)

Airflow UI에서 전체 DAG 파이프라인이 성공적으로 완료되었다면, 터미널에서 `psql` 커맨드로 데이터베이스에 접속해 가공된 주식 데이터 형태를 직접 확인해볼 수 있습니다.

**1) Postgres 터미널 접속하기**
```bash
PGPASSWORD=postgres psql -h localhost -p 5432 -U postgres -d postgres
```

**2) 가공된 주가 데이터 확인하기 (SQL 조회)**

*   **가장 최근 주식 데이터 10건 조회 (내림차순 정렬):**
    ```sql
    SELECT date, open, high, low, close, volume
    FROM stock_market
    ORDER BY date DESC
    LIMIT 10;
    ```
*   **현재 테이블에 적재된 총 데이터 로우(행) 수 확인:**
    ```sql
    SELECT COUNT(*) FROM stock_market;
    ```
*   **데이터베이스 접속 종료:**
    ```text
    \q
    ```
