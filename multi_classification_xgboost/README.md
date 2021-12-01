# Amazon Redshift ML でのモデル学習と推論

### データのインポート
[チュートリアル:顧客解約モデルの作成](https://docs.aws.amazon.com/ja_jp/redshift/latest/dg/tutorial_customer_churn.html) を参考に[Abalone file (アワビファイル)](https://s3.amazonaws.com/redshift-downloads/redshift-ml/abalone_xg/abalone.csv) 下記の CLI コマンドで自身の S3 バケットにコピーします。今回は `redshiftml` というバケット名を使っています。

```
aws s3 cp s3://redshift-downloads/redshift-ml/abalone_xgb/abalone_xgb.csv s3://YYYYMMDD-redshiftml/redshift-ml/abalone_xgb/
```

テーブルを作成して、S3 からデータをインポートします。

```SQL
DROP TABLE IF EXISTS abalone_xgb;

CREATE SCHEMA demo_ml;

CREATE TABLE demo_ml.abalone_xgb (
length_val float, 
diameter float, 
height float,
whole_weight float, 
shucked_weight float, 
viscera_weight float,
shell_weight float, 
rings int,
record_number int);

COPY demo_ml.abalone_xgb FROM 's3://redshift-downloads/redshift-ml/abalone_xgb/' IAM_ROLE 'arn:aws:iam::XXXXXXXXXXXX:role/Redshift-ML' IGNOREHEADER 1 CSV;
```

## Redshift Create Model を使ったモデルの構築
`AutoML` ではなく、`XGBOOST` をモデルタイプに指定して学習を実施。

```SQL
DROP MODEL IF EXISTS demo_ml.abalone_xgboost_multi_predict_age;

CREATE MODEL demo_ml.abalone_xgboost_multi_predict_age
FROM ( SELECT length_val,
              diameter,
              height,
              whole_weight,
              shucked_weight,
              viscera_weight,
              shell_weight,
              rings 
        FROM abalone_xgb WHERE record_number < 2500 )
TARGET rings FUNCTION ml_fn_abalone_xgboost_multi_predict_age
IAM_ROLE 'arn:aws:iam::815969174475:role/RedshiftML'
AUTO OFF
MODEL_TYPE XGBOOST
OBJECTIVE 'multi:softmax'
PREPROCESSORS 'none'
HYPERPARAMETERS DEFAULT EXCEPT (NUM_ROUND '100', NUM_CLASS '30')
settings (S3_BUCKET '20211117-redshiftml', max_runtime 1800);
```

作成したモデルを下記クエリで確認します。


```SQL
SHOW MODEL ALL;
SHOW MODEL demo_ml.abalone_xgboost_multi_predict_age;
```

### 推論エンドポイントへのクエリ

モデルを作成すると、推論エンドポイントが準備されているので、下記のクエリで推論リクエストを行い結果を確認します。

```SQL
select ml_fn_abalone_xgboost_multi_predict_age(length_val, 
                                               diameter, 
                                               height, 
                                               whole_weight, 
                                               shucked_weight, 
                                               viscera_weight, 
                                               shell_weight)+1.5 as age 
from abalone_xgb where record_number > 2500;
```


## BYOM (リモート推論) を用いたモデルの構築
詳細は[独自のモデルを持参 (BYOM)](https://docs.aws.amazon.com/ja_jp/redshift/latest/dg/r_CREATE_MODEL.html#r_byom_create_model)をご確認下しさい。


### モデルの学習

SageMaker で。


### モデルのインポート

```SQL
DROP MODEL IF EXISTS demo_ml.abalone_xgboost_multi_predict_age;

CREATE MODEL demo_ml.abalone_xgboost_multi_predict_age
FROM 'redshiftml-20211123144922081536-xgboost'
FUNCTION demo_ml.ml_fn_abalone_xgboost_multi_predict_age_byom(float, float, float, float, float, float, float)
RETURNS int
IAM_ROLE 'arn:aws:iam::815969174475:role/RedshiftML'
SETTINGS (S3_BUCKET '20211117-redshiftml');
```