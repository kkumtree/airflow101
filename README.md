# Airflow 101 프로젝트

이 프로젝트는 Apache Airflow와 Spark를 활용하여 주식 데이터를 수집하고 가공하는 파이프라인 실습을 위한 저장소입니다. Airflow 버전에 따라 각각 구분된 폴더에서 구성 방법을 확인하실 수 있습니다.

## 📁 주요 디렉토리 안내

*   👉 **[Airflow 2.x 버전 실습 (`2_x/README.md`)](./2_x/README.md)**
*   👉 **[Airflow 3.x 버전 실습 (`3_x/README.md`)](./3_x/README.md)**

---

## 🚀 공통 시작하기 (Common Prerequisites)

두 버전의 실습 모두 공통으로 아래 도구들의 설치 및 환경 준비가 필요합니다. 실습을 시작하기 전, 각 폴더(2_x, 3_x) 내부의 README 안내를 꼭 읽어주세요!

### 1. 사전 요구 사항
이 프로젝트를 로컬에서 실행하기 위해서는 다음 도구들이 필수로 설치되어 있어야 합니다.

*   **Docker**: 하위 컨테이너 환경을 실행하기 위해 필요합니다.
    *   [Docker 환경 설치 공식 가이드](https://docs.docker.com/get-docker/)
*   **Astro CLI**: 가장 빠르고 간단하게 로컬 Airflow 환경을 세팅해주는 필수 도구입니다.
    *   [Astro CLI 설치 공식 가이드](https://docs.astronomer.io/astro/cli/install-cli)

### 2. 베이스 Docker 이미지 빌드하기
두 버전 모두 실제 데이터를 분석하거나 분산 스파크(Spark) 환경을 구동하기 위해 컨테이너 이미지를 빌드하는 과정이 필요합니다. 터미널의 현재 위치가 실습 폴더인지 확인한 후 아래 명령어를 실행해야 합니다.

```bash
docker build -t airflow/spark-master ./spark/master && docker build -t airflow/spark-worker ./spark/worker && docker build -t airflow/stock-app ./spark/notebooks/stock_transform  
```
*(위 커맨드는 분산처리를 담당할 `spark-master`, `spark-worker` 인스턴스들과 파이썬 노트북 코드를 보유한 `stock-app` 이미지를 차례대로 모두 빌드해줍니다.)*

---

## 🛠 공통 Astro 워크플로 관리 가이드

Astro CLI를 통해 Airflow 개발 환경을 제어하는 명령어는 어느 폴더에서나 다음과 같이 사용됩니다. 항상 자신이 있는 프로젝트 폴더(`2_x` 또는 `3_x`) 내부에서 명령어를 입력하세요.

*   **`astro dev start`**: 로컬 환경 구동. (Airflow Webserver, Scheduler, DB 등 컨테이너 구동) 
*   **`astro dev stop`**: 작업 환경 임시 정지. 생성된 파일과 데이터는 그대로 유지됩니다.

⚠️ **삭제 주의: `astro dev kill`**
단순 종료가 아니라 **Airflow 메타데이터 DB(Postgres)까지 강제로 완전히 삭제**하는 명령어입니다. 시스템을 구동하며 쌓인 모든 실행 기록(Run History)과 UI에서 저장한 설정(Connections, Variables 등)이 전부 초기화되니 주의해서 사용해 주세요.

---

## 💡 결과 확인: Postgres SQL 접속 안내
실습 폴더별로 제시된 워크플로 파이프라인(DAG)을 무사히 완료한 후에는, 다음과 같이 명령어 터미널에서 `psql` 커맨드로 데이터가 적재되었는지 직접 확인할 수 있습니다.

```bash
# 로컬 저장소 DB에 로그인 우회 접속
PGPASSWORD=postgres psql -h localhost -p 5432 -U postgres -d postgres
```

접속된 인터페이스에서 자유롭게 SQL 쿼리를 활용해보세요.

```sql
-- 최근 데이터 10줄만 읽어오기
SELECT date, open, high, low, close, volume FROM stock_market ORDER BY date DESC LIMIT 10;

-- 저장된 주식 데이터(행) 전체 개수 파악
SELECT COUNT(*) FROM stock_market;

-- 확인 후 데이터베이스 연결 종료
\q
```

**지금 바로 원하는 버전(`2_x` 또는 `3_x`)의 폴더로 이동하여 세부 README.md 가이드를 따라 실습을 진행해보세요!**
