# suf_20172Q

## 概要
- 標準的なSQLを使ってS3のデータを直接分析
- サーバレス: インフラ管理の必要なし
- 複雑なETL処理の必要なし
- 大きなデータセットに対してもインタラクティブなパフォーマンス
- JDBCドライバを使って、BIツールからもアクセス可能
- US East (Virginia), US-West2 (Oregon) リージョンで利用可能

## 技術
- 分散SQLエンジンPestro, Apache Hiveが利用可能
- クエリを実行する時にスキーマを定義(schema-on-read)
- Apache Hive方式のデータパーティション
 - 月、週、日、時間、または顧客ID、もしくはそれらすべてを使って、パーティションすることができる

## 課金
- クエリ毎にスキャンしたデータ量に応じて課金
 - 1TBあたり5USD
- スキャン以外のステートメント、正常に実行されなかったクエリに対しては課金されない
- ストレージ、リクエスト、データ転送に対して S3 の標準料金が発生
- クエリごとのスキャンされたデータ量は、Athenaコンソールで確認可

## コスト削減
- データのパーティションをすることで、Athenaでクエリを投げた際のスキャンするデータ量を減らし、コストを抑えられる
- Apache Parquetなどの列志向フォーマットに変換するとコストを抑えられる

## 作業
- テーブルの作成（### これだけでデータをクエリすることが可能 ###）
 - 成功すると、テーブルとスキーマがデータカタログに現れる
 - Athenaは内部にデータカタログを持っていて、テーブル、データベース、パーティションに関する情報を保存するために使われている
- [Option] Athenaで使われる形式でデータをパーティション
 - スキーマ定義をパーティションを含む様に変更
 - パーティションのメタデータをAthenaにロード
- [Option] Apache Parquetなどの列志向フォーマットに変換
 - https://github.com/awslabs/aws-big-data-blog/tree/master/aws-blog-spark-parquet-conversion
- クエリを流す

## 機能
- 任意の正規表現を使って、Athenaにテキスト内の各行をどのように解釈するかを指定することができる
- CSV, TSV, JSON形式を処理する場合には、正規表現は必要無い

# 稟議申請内容

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

※100円台は切り上げて、16,000円で申請する。
