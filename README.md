# suf_20172Q

## Amazon Athenaについて

### 概要
- 標準的なSQLを使ってS3のデータを直接分析
- サーバレス: インフラ管理の必要なし
- 複雑なETL処理の必要なし
- 大きなデータセットに対してもインタラクティブなパフォーマンス
- JDBCドライバを使って、BIツールからもアクセス可能
- US East (Virginia), US-West2 (Oregon) リージョンで利用可能

### 技術
- 分散SQLエンジンPestro, Apache Hiveが利用可能
- クエリを実行する時にスキーマを定義(schema-on-read)
- Apache Hive方式のデータパーティション
 - 月、週、日、時間、または顧客ID、もしくはそれらすべてを使って、パーティションすることができる

### 課金
- クエリ毎にスキャンしたデータ量に応じて課金
 - 1TBあたり5USD
- スキャン以外のステートメント、正常に実行されなかったクエリに対しては課金されない
- ストレージ、リクエスト、データ転送に対して S3 の標準料金が発生
- クエリごとのスキャンされたデータ量は、Athenaコンソールで確認可

### コスト削減
- データのパーティションをすることで、Athenaでクエリを投げた際のスキャンするデータ量を減らし、コストを抑えられる
- Apache Parquetなどの列志向フォーマットに変換するとコストを抑えられる

### 作業
- テーブルの作成（### これだけでデータをクエリすることが可能 ###）
 - 成功すると、テーブルとスキーマがデータカタログに現れる
 - Athenaは内部にデータカタログを持っていて、テーブル、データベース、パーティションに関する情報を保存するために使われている
- [Option] Athenaで使われる形式でデータをパーティション
 - スキーマ定義をパーティションを含む様に変更
 - パーティションのメタデータをAthenaにロード
- [Option] Apache Parquetなどの列志向フォーマットに変換
 - https://github.com/awslabs/aws-big-data-blog/tree/master/aws-blog-spark-parquet-conversion
- クエリを流す

### 機能
- 任意の正規表現を使って、Athenaにテキスト内の各行をどのように解釈するかを指定することができる
- CSV, TSV, JSON形式を処理する場合には、正規表現は必要無い

---

## S3のアクセスコントロールの基本

### アクセスコントロールの種類

|アクセスコントロールタイプ|AWSアカウントレベルの制御|IAMユーザレベルの制御|形式|
|:--|:--:|:--:|:--:|
|ACL|○|☓|XML|
|Bucket Policy|○|○|JSON|
|IAM Policy|☓|○|JSON|

- Access Control List (ACL)
 - Bucket/Object単位でアクセス権を管理
 - AWSマネコンから制御対象を選択して、Properties > Permissionsで設定

- Bucket Policy
 - Bucket/Object単位でアクセス権を管理
 - AWSマネコンから制御対象を選択して、Properties > Permissions > Add(Edit) Bucket Policyで設定
 - AWSアカウントやバケットにアクセス権を設定したい場合に利用

- IAM Policy
 - S3を含むAWSリソースへのアクセス権を、IAMリソース(IAMユーザ, IAMグループ, IAMロール)単位で管理
 - [誰が] [どのAWSサービスの] [どのリソースに対して] [どんな操作を] [許可する(許可しない)]などを記述
 - AWSマネコンからIAMのホームディレクトリでユーザ/グループ/ロールのポリシーを設定
 - ユーザにアクセス権を設定したい場合に利用

### リソースの指定

- バケットポリシーやIAMポリシーで、ある Action を許可/拒否する場合、対象 Resource を ARN で指定する。
- バケットに対する Action は Resource としてバケットの ARN(arn:aws:s3:::bucket) を指定する。 
- オブジェクトに対する Action は Resource としてオブジェクトの ARN(arn:aws:s3:::bucket/*) を指定する。

### アクセス許可の種類

- READ
- WRITE
- READ_ACP
- WRITE_ACP
- FULL_CONTROL

※ ACP: ACLへのアクセス権

---

## IAM基本

### ポリシーの分類

- ユーザーベースのポリシー
  - AWS管理ポリシー
  - カスタマー管理ポリシー
  - インラインポリシー
- リソースベースのポリシー
- IAM ロールの信頼ポリシー

#### ユーザーベースのポリシー

- IAMユーザー、IAMグループ、IAMロールに関連づけるポリシー。
- [どのAWSサービスの] [どのリソースに対して] [どんな操作を] [許可する(許可しない)]を記述する。
- 関連付けられた対象(IAMユーザー、IAMグループ、IAMロール)が[誰が]に該当する。
- 関連付けられた対象は、操作主体であり `Principal` と呼ばれる。

#### ソースベースのポリシー

- 操作する主体(≒ユーザー)ではなく、操作を行われるモノ(AWSリソース)に関連づけるポリシー。
- 「誰が」の部分にあたる `Principal` の記載が必要。
- リソースベースのポリシーはS3等の一部のAWSサービスのみに対応。

#### IAM ロールの信頼ポリシー(信頼関係)

- IAMロールの権限移譲操作に特化したポリシーで、関連づける先(対象)はIAMロール。

---

### Athenaの運用に関する考察

#### 運用ポリシー

IAM Policyを利用して、一般ユーザ(Athenaを利用しないユーザ)とAthena利用ユーザのアクセスコントロールを行う。

|[誰が]|[どのAWSサービスの]|[どのリソースに対して]|[どんな操作を]|[許可する(許可しない)]|
|:--|:--:|:--:|:--:|:--:|
|一般ユーザ|Athena|-|すべての操作|許可しない|
|Athena利用ユーザ|Athena, S3|対象S3 Bucket|Athenaのすべての操作, 対象S3 bucketのデータ取得|許可する|

#### 具体的な施策

Athena利用ユーザに対して、Athenaのすべての操作と対象S3 bucketのデータ取得を許可するIAMロールを付与する。なお、一般ユーザに対しては、このIAMロールを付与しない。

- S3に関するIAM Policy

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetObject"
            ],
            "Resource": [
                "arn:aws:s3:::target-bucket/*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetBucketLocation",
                "s3:ListBucket"
            ],
            "Resource": [
                "arn:aws:s3:::target-bucket"
            ]
        }
    ]
}
```

- Athenaに関するIAM Policy

`AmazonAthenaFullAccess Managed Policy`(公式)を利用する。

---

## Amazon AthenaとAmazon QuickSightの連携

### QuickSight

様々なデータを簡単な操作でグラフとして表示できるBIツール。

### ユースケース
S3にとにかく何も考えずにデータを投げ込んでおいて、Athenaで吸い上げたデータをデータソースとしてQuickSightで表示する。
<img src="https://github.com/czsuganuma-koji/suf_20172Q/blob/master/athena_quicksight_usecase.jpg">
<img src="https://github.com/czsuganuma-koji/suf_20172Q/blob/master/quicksight_overview.jpg">

---

## 参考サイト

- [Amazon Athena – サーバーレスのインタラクティブなクエリサービス – AWS](https://aws.amazon.com/jp/athena/)
- [AWS IAMポリシーを理解する](http://dev.classmethod.jp/cloud/aws-iam-policy/)
- [【新機能】Amazon QuickSightがAmazon Athenaをサポート！](http://dev.classmethod.jp/cloud/aws/amazon-quicksight-supports-amazon-athena/)

---

## 稟議申請内容

aws Athenaの検証のために、スキルアップフライデーにてAthena, S3を利用する。

--------------------------------------------------------+

【請求されるアカウント名】
　fox-dev

--------------------------------------------------------+

【利用期間】
　2/1 ~ 2/28 (1ヶ月間)

--------------------------------------------------------+

【料金体系】
- Athena
　[スキャン] $5 / TB
- S3
　[ストレージ] $0.023 / GB (最初の50TB)
　[データ転送] $0.090 / GB (10 TBまで)

--------------------------------------------------------+

【想定利用量】
合計: 15,120円 ($132, 114.54円/$)
- Athena: $100
　[スキャン] 200GB / query, 100query / month -> 20TB
- S3:  $32
　[ストレージ] 1TB -> $23
　[データ転送] 100GB -> $9

--------------------------------------------------------+

