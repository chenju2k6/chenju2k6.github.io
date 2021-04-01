---
layout: post
title:  "Fuzzbench"
date:   2021-04-01 10:00:28 -0500
categories: jekyll update
---

### install requirements ###

```
virtualenv venv
source venv/bin/activate
python3 -m pip install -r requirements.txt
```

### Update pip ###

```
python3.8 -m pip install -U pip
```

### Requirements.txt ###

```
find
pybind11==2.6.2
wheel==0.36.2
alembic==1.4.0
google-api-python-client==1.8.2
google-auth==1.24.0
google-cloud-error-reporting==0.33.0
google-cloud-logging==1.14.0
google-cloud-secret-manager==2.1.0
clusterfuzz==0.0.1a0
Jinja2==2.11.3
numpy==1.18.1
Orange3==3.24.1
pandas==1.0.4
psycopg2-binary==2.8.4
pyfakefs==3.7.1
pytest==6.1.2
python-dateutil==2.8.1
pytz==2019.3
PyYAML==5.3.1
redis==3.5.3
rq==1.4.3
scikit-posthocs==0.6.2
scipy==1.4.1
seaborn==0.11.1
sqlalchemy==1.3.19

# Needed for development.
pylint==2.6.0
pytype==2020.11.3
yapf==0.30.0
```

### Build script ###

```
export CFLAGS=
export CXXFLAGS=
export KO_CC=clang-6.0
export KO_CXX=clang++-6.0
export CC=/Kirenenko/bin/ko-clang
export CXX=/Kirenenko/bin/ko-clang++
export KO_DONT_OPTIMIZE=1
wget https://raw.githubusercontent.com/llvm/llvm-project/main/compiler-rt/lib/fuzzer/standalone/StandaloneFuzzTargetMain.c
KO_DONT_OPTIMIZE=1 /Kirenenko/bin/ko-clang -c StandaloneFuzzTargetMain.c -o driver.o
ar r driver.a driver.o
cp driver.a /usr/lib/libFuzzingEngine.a
export FUZZER_LIB=/src/driver.a
```
