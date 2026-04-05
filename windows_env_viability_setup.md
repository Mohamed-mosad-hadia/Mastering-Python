# Windows Environment Viability Setup (Direct Links + Exact Commands)

This is the exact setup checklist to make the project runnable on a new Windows machine with minimal dependency/runtime errors.

## 1) Install Required Software (with links)

Install these in order:

1. **Python 3.10.11 (64-bit)**
- Release page: https://www.python.org/downloads/release/python-31011/
- Direct installer: https://www.python.org/ftp/python/3.10.11/python-3.10.11-amd64.exe
- During install: check `Add Python to PATH`.

2. **Git for Windows**
- https://git-scm.com/download/win

3. **JDK 11 (Temurin)**
- https://adoptium.net/temurin/releases/?version=11
- Install x64 MSI for Windows.

4. **Microsoft Visual C++ Redistributable (x64)** (recommended for ML wheels)
- https://aka.ms/vs/17/release/vc_redist.x64.exe

## 2) Verify Core Tools

Open PowerShell and run:

```powershell
python --version
git --version
java -version
```

Expected:
- Python `3.10.x`
- Java `11.x`

## 3) Create Virtual Environment (name: `hub`)

From repo root:

```powershell
cd C:\ml\jules_hub_and_spikes_system_5

python -m venv hub
.\hub\Scripts\Activate.ps1

python -m pip install --upgrade pip setuptools wheel
```

If script activation is blocked:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
.\hub\Scripts\Activate.ps1
```

## 4) Install Core ML Package Set (30 libraries)

Run this once:

```powershell
python -m pip install `
  pandas==2.2.2 `
  numpy==1.26.4 `
  scipy==1.12.0 `
  scikit-learn==1.6.1 `
  lightgbm==4.6.0 `
  mlflow==3.8.1 `
  pyspark==3.3.0 `
  py4j==0.10.9.5 `
  pyarrow==18.1.0 `
  pyyaml==6.0.3 `
  pytest==8.4.2 `
  portalocker==3.2.0 `
  fastapi==0.115.0 `
  uvicorn==0.30.6 `
  shap==0.46.0 `
  tqdm==4.67.1 `
  joblib==1.4.2 `
  requests==2.32.3 `
  python-dateutil==2.9.0.post0 `
  pytz==2024.2 `
  packaging==24.2 `
  typing-extensions==4.12.2 `
  pydantic==2.9.2 `
  cloudpickle==3.1.0 `
  sqlalchemy==2.0.35 `
  alembic==1.13.3 `
  click==8.1.7 `
  protobuf==5.28.2 `
  psutil==6.0.0 `
  matplotlib==3.9.2
```

## 5) Install Project Requirements (authoritative)

Even after the 30-library pack, install project files to avoid missing transitive deps:

```powershell
python -m pip install -r requirements.txt
python -m pip install -r ml_hub\actuarial_pricing_model_v5\requirements.txt
python -m pip install -r ml_hub\stp_automation_model_v1\requirements.txt
python -m pip install -r ml_hub\medical_approval_model_v1\requirements.txt
python -m pip install -r ml_hub\financial_approval_model_v1\requirements.txt
python -m pip install -r ml_hub\member_churn_model_v1\requirements.txt
python -m pip install -r ml_hub\fwa_detection_model_v1\requirements.txt
python -m pip install -r ml_hub\network_value_model_v1\requirements.txt
```

Install local shared package:

```powershell
python -m pip install -e ml_hub\medins_ml_utils
```

## 6) Set Runtime Environment Variables

In the same PowerShell session:

```powershell
$env:MEDINS_ENV = "development"
$env:MEDINS_AUDIT_SALT = "DEV-ONLY-NOT-FOR-PRODUCTION"
$env:MEDINS_DIAGNOSTIC_MODE = "0"
$env:MLFLOW_TRACKING_URI = "sqlite:///mlflow.db"
$env:SPARK_LOCAL_IP = "127.0.0.1"
```

Set `JAVA_HOME` (edit path to your JDK location):

```powershell
$env:JAVA_HOME = "C:\Program Files\Eclipse Adoptium\jdk-11.0.27.6-hotspot"
$env:PATH = "$env:JAVA_HOME\bin;$env:PATH"
$env:PYSPARK_PYTHON = (Get-Command python).Source
```

## 7) Viability Checks (must pass)

### 0) Confirm you are inside env `hub`
```powershell
python -c "import sys; print(sys.prefix)"
```
Expected output ends with `\hub`.

### A) Import check
```powershell
python -c "import pandas,numpy,scipy,sklearn,lightgbm,mlflow,pyspark,pyarrow,pytest,portalocker,fastapi,uvicorn,shap; print('IMPORTS_OK')"
```

### A2) Version assurance check (strict pins)
```powershell
python -c "import importlib.metadata as m; req={'pandas':'2.2.2','numpy':'1.26.4','scipy':'1.12.0','scikit-learn':'1.6.1','lightgbm':'4.6.0','mlflow':'3.8.1','pyspark':'3.3.0','py4j':'0.10.9.5','pyarrow':'18.1.0','pyyaml':'6.0.3','pytest':'8.4.2','portalocker':'3.2.0','fastapi':'0.115.0','uvicorn':'0.30.6','shap':'0.46.0','tqdm':'4.67.1','joblib':'1.4.2','requests':'2.32.3','python-dateutil':'2.9.0.post0','pytz':'2024.2','packaging':'24.2','typing-extensions':'4.12.2','pydantic':'2.9.2','cloudpickle':'3.1.0','sqlalchemy':'2.0.35','alembic':'1.13.3','click':'8.1.7','protobuf':'5.28.2','psutil':'6.0.0','matplotlib':'3.9.2'}; bad=[]; [bad.append((k,m.version(k),v)) for k,v in req.items() if m.version(k)!=v]; print('VERSION_CHECK_OK' if not bad else 'VERSION_MISMATCH'); [print(x) for x in bad]; raise SystemExit(1 if bad else 0)"
```

### B) Spark check
```powershell
python -c "from pyspark.sql import SparkSession; s=SparkSession.builder.appName('preflight').getOrCreate(); print('SPARK_OK', s.range(1).count()); s.stop()"
```

### C) Project smoke tests
```powershell
python -m pytest -q ml_hub\tests\test_trial_program.py
python -m pytest -q ml_hub\tests\test_train_serve_parity.py
python -m pytest -q ml_hub\tests\test_production_etl.py
```

If all pass, your environment is viable.

## 8) If Something Fails

- `MEDINS_ENV is not set`
  - Set `$env:MEDINS_ENV="development"` before running scripts.

- `Spark Java gateway process exited before sending its port number`
  - Java is missing/wrong. Recheck JDK 11 and `JAVA_HOME`.

- `ModuleNotFoundError: medins_ml_utils`
  - Run `python -m pip install -e ml_hub\medins_ml_utils`.

- `lightgbm` install/build errors
  - Ensure VC++ redistributable installed, then reinstall `lightgbm==4.6.0`.

- `NO_GO` in pre-training proof due `low_join_retention` or `zero_join_overlap`
  - Use coherent sample build from real snapshot before running cycle.

## 9) Ready State Definition

Environment is ready only when:
- Tool versions are correct (Python 3.10, Java 11).
- Active env is `hub`.
- Import check passes.
- Strict version assurance check passes.
- Spark check passes.
- 3 smoke tests pass.
- Runtime env vars are set.
