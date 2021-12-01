# Amazon Redshift ML サンプルコード
詳細は[こちら](https://docs.aws.amazon.com/ja_jp/redshift/latest/dg/getting-started-machine-learning.html)をご確認下さい。


## 準備
- Amazon S3 バケットの準備
- IAM ロールの作成
- Redshift のクラスタの準備

### Amazon S3 バケットの作成
AWS コンソールから本サンプルコードで活用するバケットを `YYYYMMDD-redshift-ml` というバケット名で作成して下さい。

### IAM ロールの作成
- `RedshiftML` ロールを[ブログ記事](https://aws.amazon.com/jp/blogs/big-data/create-train-and-deploy-machine-learning-models-in-amazon-redshift-using-sql-with-amazon-redshift-ml/) に従い、下記の設定で作成します
    - `AmazonS3ReadOnlyAccess` と `AmazonSageMakerFullAccess`、 `AmazonRedshiftFullAccess` を付与
    - 下記を `redshiftml-incline` として作成して付与
    - `Trust relationsips` に下記を追加

redshiftml-incline ポリシー

``` redshiftml-incline
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "cloudwatch:PutMetricData",
                "ecr:BatchCheckLayerAvailability",
                "ecr:BatchGetImage",
                "ecr:GetAuthorizationToken",
                "ecr:GetDownloadUrlForLayer",
                "logs:CreateLogGroup",
                "logs:CreateLogStream",
                "logs:DescribeLogStreams",
                "logs:PutLogEvents",
                "sagemaker:*Job*"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "iam:PassRole",
                "s3:AbortMultipartUpload",
                "s3:GetObject",
                "s3:DeleteObject",
                "s3:PutObject"
            ],
            "Resource": [
                "arn:aws:iam::<your-account-id>:role/RedshiftML",
                "arn:aws:s3:::redshiftml-<your-account-id>/*",
                "arn:aws:s3:::redshift-downloads/*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetBucketLocation",
                "s3:ListBucket"
            ],
            "Resource": [
                "arn:aws:s3:::redshiftml-<your-account-id>",
                "arn:aws:s3:::redshift-downloads"
            
            ]
        }
    ]
} 
```

Trust relationship
``` 
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": [
          "redshift.amazonaws.com",
          "sagemaker.amazonaws.com"
        ]
      },
      "Action": "sts:AssumeRole"
    }
  ]
}

```

### Redshift クラスタの構築
- `Create cluster` ボタンからクラスタ作成画面へ行き、 `Free trial` でクラスタを作成します
- Admion user name を `awsuser` とし、パスワードを設定します
- `RedshiftML` ロールを付与します。