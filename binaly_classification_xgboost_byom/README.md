## BYOM リモート推論
### データの準備とモデルの学習
リモート推論を実施するために、SageMaker Notebook インスタンスを使って、データの準備やモデルの学習、推論エンドポイントを作成します。詳細は [こちら](https://github.com/tkazusa/Redshift-ML-practice/blob/master/multi_classification/redshift_ml_byom_customer_churn.ipynb) のノートブックをご確認下さい。


### 推論用データのためにテーブルを作成
```SQL
DROP TABLE IF EXISTS customer_churn_xgb;
CREATE TABLE customer_churn_xgb (
churn int,
day_charge float,
eve_charge float,
night_charge float,
intl_charge float);

COPY customer_churn_xgb
FROM 's3://YYYYMMDD-redshiftml/redshift-ml/customer_churn_validation/'
IAM_ROLE 'arn:aws:iam::XXXXXXXXXXX:role/RedshiftML' IGNOREHEADER 1 CSV;
```


### 学習ジョブから推論用モデルをデプロイする
```SQL
DROP MODEL IF EXISTS remote_customer_churn;
CREATE MODEL remote_customer_churn
FUNCTION remote_fn_customer_churn_predict (float, float, float, float)
RETURNS float
SAGEMAKER '学習ジョブ名'
IAM_ROLE 'arn:aws:iam::XXXXXXXX:role/RedshiftML';
```


### 推論リクエストを行う
```SQL
SELECT remote_fn_customer_churn_predict(
day_charge, 
eve_charge,
night_charge,
intl_charge) AS churn_proba 
FROM customer_churn_xgb 
```
