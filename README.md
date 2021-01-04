# DevSetup
```bash
git clone https://github.com/namjals/mlflow.git
cd mlflow
virtualenv venv
source venv/bin/activate
pip install -r requirements.txt
cp env.sh.template env.sh
source env.sh
# for mac
brew install libomp
```
# Tutorial

로컬 or AWS 기반 mlflow 서빙 샘플입니다.

## Local

```bash
# 데이터셋 다운로드
cd sample
wget https://archive.ics.uci.edu/ml/machine-learning-databases/00222/bank-additional.zip
unzip bank-additional.zip

# 학습
python train-xgboost.py

# MODEL_PATH 환경변수 등록
vi ../env.sh
# EX)export MODEL_PATH=/Users/namjals/study/sagemaker-automation/mlflow/direct-marketing-xgboost/mlruns/1/1eaf3db5b68b4d02862d33f6dba7642d/artifacts/direct-marketing-xgboost-model
source env.sh

# mlflow 서빙앱 빌드, --no-push 옵션은 빌드 이미지를 AWS ECR에 푸시하지 않음을 뜻합니다.
# Sagemaker에서 예측을 위해서는 ECR에 push가 필요합니다.
mlflow sagemaker build-and-push-container --no-push

# 예측
python predict-xgboost-local.py

# 확인
mlflow ui
# localhost:5000 접속
```

## SageMaker

- role 권한 얻기
  - musinsa-dev3 계정의 `IAM 서비스` > `역할` 클릭
  - `ds-dev-sagemaker` 검색 및 클릭
  - 역할 ARN 복사

```bash
# 환경 변수에 ROLE 등록
vi env.sh
# ROLE에 역할 ARN을 붙여 넣습니다.
# EX)export ROLE=arn:aws:iam::720388917329:role/ds-dev-sagemaker
source env.sh

# mlflow 서빙앱 빌드 및 ECR에 배포
mlflow sagemaker build-and-push-container

# SageMaker에 모델 배포, 이때 생성되는 ml.m4.xlarge 인스턴스은 시간당 1달러
mlflow sagemaker deploy -a $APP_NAME -m $MODEL_PATH -e $ROLE --region-name $REGION

# 예측
cd sample
python predict-xgboost.py

# 확인
# serving : AWS Sagemaker > EndPoint
# log : AWS CloudWatch > Log Groups > mlflow-xgb-demo
```

# Reference

- 튜토리얼 전반 : https://julsimon.medium.com/deploying-machine-learning-models-with-mlflow-and-amazon-sagemaker-dac2b8244224
- 데이터셋 정보 : [https://archive.ics.uci.edu/ml/datasets/Bank+Marketing#](https://archive.ics.uci.edu/ml/datasets/Bank+Marketing)
- xgboost
  - https://bcho.tistory.com/1354
  - https://xgboost.readthedocs.io/en/latest/python/python_api.html#xgboost.XGBClassifier
- mlflow api 문서 : https://www.mlflow.org/docs/latest/python_api/mlflow.xgboost.html

# TroubleShooting

## 계정 확인 및 변경

- 터미널 환경의 aws 계정이 dev계정이어야 합니다.

```bash
# 현재 적용된 계정을 보여줍니다. profile의 값을 살핍니다.
aws configure list
 
# 변경 가능한 계정들을 보여줍니다.
cat ~/.aws/config
 
위의 두 명령어를 비교하여, dev 계정이 아니라면, dev 계정으로 환경을 변환합니다.
# 저는 ~/.zshrc 에 AWS_DEFAULT_PROFILE로 관리하고 있습니다.

source ~/.zshrc.sh
```

