# Snowflake & Databricks 学習ロードマップ

キャリア向上のためのデータエンジニアリング・データ分析プラットフォーム学習計画。

---

## 目次

1. [前提知識](#前提知識)
2. [Snowflake 学習ロードマップ](#snowflake-学習ロードマップ)
3. [Databricks 学習ロードマップ](#databricks-学習ロードマップ)
4. [共通スキル](#共通スキル)
5. [実践プロジェクト](#実践プロジェクト)
6. [資格・認定試験](#資格認定試験)
7. [学習リソース](#学習リソース)
8. [キャリアパス](#キャリアパス)

---

## 前提知識

学習を始める前に以下の基礎を固める。

### 必須スキル

| スキル | 習得レベル | 目安期間 |
|--------|-----------|---------|
| SQL（SELECT, JOIN, GROUP BY, ウィンドウ関数） | 中級以上 | 2〜4週間 |
| Python（pandas, 基本的なデータ操作） | 入門〜初級 | 2〜4週間 |
| クラウド基礎（AWS/Azure/GCP のいずれか） | 入門 | 1〜2週間 |

### あると望ましいスキル

- データウェアハウスの概念（スタースキーマ、ディメンション/ファクトテーブル）
- ETL/ELT の基本概念
- Git の基本操作

---

## Snowflake 学習ロードマップ

### フェーズ 1：基礎理解（2〜3週間）

**目標：** Snowflakeのアーキテクチャと基本操作を理解する

- [ ] Snowflakeのアーキテクチャ理解
  - マルチクラスタ・シェアードデータアーキテクチャ
  - ストレージ・コンピュート分離の概念
  - 仮想ウェアハウス（Virtual Warehouse）の仕組み
- [ ] Snowflake無料トライアルアカウントの作成
- [ ] 基本SQL操作
  - データベース・スキーマ・テーブルの作成
  - データのロード（COPY INTO コマンド）
  - クエリの実行と最適化
- [ ] Snowflake Web UI（Snowsight）の操作に慣れる

**練習課題：**
```sql
-- サンプルデータのロード
CREATE DATABASE learning_db;
CREATE SCHEMA learning_db.sales;

CREATE TABLE learning_db.sales.orders (
    order_id    INT,
    customer_id INT,
    order_date  DATE,
    amount      DECIMAL(10,2)
);

-- ウィンドウ関数を使った分析
SELECT
    customer_id,
    order_date,
    amount,
    SUM(amount) OVER (PARTITION BY customer_id ORDER BY order_date) AS running_total
FROM learning_db.sales.orders;
```

### フェーズ 2：中級機能（3〜4週間）

**目標：** Snowflake固有の高度な機能を使いこなす

- [ ] Time Travel とデータ復元
- [ ] クローン機能（Zero-Copy Clone）
- [ ] データ共有（Secure Data Sharing）
- [ ] Snowpipe による継続的データロード
- [ ] ストリーム（Streams）とタスク（Tasks）による自動化
- [ ] ロールとアクセス制御（RBAC）
- [ ] マテリアライズドビュー

**練習課題：**
```sql
-- Time Travel でのデータ復元
SELECT * FROM orders AT (OFFSET => -60*5); -- 5分前のデータ

-- ゼロコピークローン
CREATE TABLE orders_dev CLONE orders;

-- Streamの作成
CREATE STREAM orders_stream ON TABLE orders;
```

### フェーズ 3：データエンジニアリング応用（4〜6週間）

**目標：** 本番環境を意識したデータパイプライン構築

- [ ] dbt（data build tool）との連携
- [ ] Snowflakeでのデータモデリング
- [ ] パフォーマンスチューニング
  - クラスタリングキーの設定
  - クエリプロファイルの読み方
  - マイクロパーティションの理解
- [ ] Snowpark（Python/Java/Scala での処理）
- [ ] Streamlitアプリのデプロイ（Streamlit in Snowflake）

---

## Databricks 学習ロードマップ

### フェーズ 1：基礎理解（2〜3週間）

**目標：** Databricksの環境とApache Sparkの基礎を理解する

- [ ] Databricksの概要とアーキテクチャ
  - Lakehouse アーキテクチャの概念
  - Delta Lake の役割
  - Cluster の種類（All-Purpose / Job Cluster）
- [ ] 無料トライアル環境（Databricks Community Edition）のセットアップ
- [ ] Notebook の基本操作
  - PySpark の基礎
  - SQL クエリの実行
  - ビジュアライゼーション
- [ ] Delta Lake の基礎
  - ACID トランザクション
  - スキーマ進化

**練習課題（PySpark）：**
```python
# Databricks Notebookでの基本操作

# DataFrameの作成
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, sum as spark_sum

df = spark.read.format("csv").option("header", True).load("/databricks-datasets/retail-org/sales_orders/")

# 集計
result = df.groupBy("customer_id") \
           .agg(spark_sum("total_price").alias("total_spent")) \
           .orderBy("total_spent", ascending=False)

display(result)
```

### フェーズ 2：Delta Lake & データエンジニアリング（3〜4週間）

**目標：** Delta Lakeを活用したデータパイプライン構築

- [ ] Delta Lake 高度な機能
  - Time Travel
  - MERGE INTO（Upsert）
  - Change Data Feed（CDF）
  - Vacuum と Optimize
- [ ] Databricks Workflows（ジョブのスケジュール実行）
- [ ] Auto Loader による継続的データ取り込み
- [ ] Unity Catalog によるデータガバナンス
- [ ] Structured Streaming

**練習課題（Delta Lake）：**
```python
# Delta テーブルへの書き込み
df.write.format("delta").saveAsTable("sales.orders")

# Time Travel
df_historical = spark.read.format("delta") \
    .option("versionAsOf", 0) \
    .load("/path/to/delta/table")

# MERGE（Upsert）
from delta.tables import DeltaTable

target = DeltaTable.forName(spark, "sales.orders")
target.alias("t").merge(
    source=updates.alias("s"),
    condition="t.order_id = s.order_id"
).whenMatchedUpdateAll() \
 .whenNotMatchedInsertAll() \
 .execute()
```

### フェーズ 3：機械学習 & MLflow（4〜6週間）

**目標：** MLOpsパイプラインの構築と管理

- [ ] MLflow によるモデル管理
  - Experiment Tracking
  - Model Registry
  - モデルのデプロイ
- [ ] Feature Store の活用
- [ ] AutoML の利用
- [ ] Model Serving（リアルタイム推論）
- [ ] LLM / AI との連携（Databricks Model Serving）

**練習課題（MLflow）：**
```python
import mlflow
import mlflow.sklearn
from sklearn.ensemble import RandomForestClassifier

with mlflow.start_run():
    model = RandomForestClassifier(n_estimators=100)
    model.fit(X_train, y_train)

    accuracy = model.score(X_test, y_test)
    mlflow.log_metric("accuracy", accuracy)
    mlflow.log_param("n_estimators", 100)
    mlflow.sklearn.log_model(model, "model")
```

---

## 共通スキル

SnowflakeとDatabricks両方で重要なスキル。

### データモデリング

- スタースキーマ・スノーフレークスキーマ
- OBT（One Big Table）vs 正規化
- データメッシュ設計原則

### オーケストレーション

- Apache Airflow（dbt との連携）
- Dagster / Prefect

### データ品質

- Great Expectations
- dbt Test

---

## 実践プロジェクト

### 初級プロジェクト：eコマース分析ダッシュボード

**期間：** 2〜3週間

1. 公開データセット（例：TPC-H, Kaggle）をSnowflake/Databricksにロード
2. 売上・顧客分析SQLを作成
3. dbtでデータモデルを構築
4. Tableau/Power BI/Streamlit でダッシュボード作成

### 中級プロジェクト：リアルタイムデータパイプライン

**期間：** 4〜6週間

1. Kafka / Kinesis からのストリーミングデータ取り込み
2. Snowpipe または Auto Loader でリアルタイム処理
3. Delta Live Tables でのパイプライン構築
4. モニタリング・アラート設定

### 上級プロジェクト：ML パイプライン統合

**期間：** 6〜8週間

1. Databricks Feature Store でのフィーチャーエンジニアリング
2. MLflow での実験管理
3. モデルのデプロイとA/Bテスト
4. SnowflakeのSnowpark MLとの比較評価

---

## 資格・認定試験

### Snowflake 認定資格

| 資格名 | レベル | 対象者 |
|--------|--------|--------|
| SnowPro Core Certification | 基礎 | 全職種 |
| SnowPro Advanced: Architect | 上級 | データアーキテクト |
| SnowPro Advanced: Data Engineer | 上級 | データエンジニア |
| SnowPro Advanced: Administrator | 上級 | DBA/管理者 |

**推奨順序：** SnowPro Core → SnowPro Advanced: Data Engineer

### Databricks 認定資格

| 資格名 | レベル | 対象者 |
|--------|--------|--------|
| Databricks Certified Data Engineer Associate | 基礎 | データエンジニア |
| Databricks Certified Data Engineer Professional | 上級 | シニアエンジニア |
| Databricks Certified Associate Developer for Apache Spark | 基礎 | 開発者 |
| Databricks Certified Machine Learning Associate | 基礎 | MLエンジニア |

**推奨順序：** Data Engineer Associate → Professional

---

## 学習リソース

### Snowflake

- **公式ドキュメント：** docs.snowflake.com
- **Snowflake University：** training.snowflake.com（無料コース多数）
- **Quickstart Tutorials：** quickstarts.snowflake.com
- **書籍：** "The Data Cloud" (Snowflake Inc.)

### Databricks

- **公式ドキュメント：** docs.databricks.com
- **Databricks Academy：** academy.databricks.com（無料ラーニングパス）
- **Delta Lake：** delta.io
- **書籍：** "Delta Lake: The Definitive Guide" (O'Reilly)

### 共通

- **dbt Learn：** courses.getdbt.com（無料）
- **Data Engineering Zoomcamp：** github.com/DataTalksClub/data-engineering-zoomcamp（無料）
- **Mode Analytics SQL Tutorial：** mode.com/sql-tutorial

---

## キャリアパス

### ロール別推奨優先度

| ロール | Snowflake優先度 | Databricks優先度 |
|--------|----------------|-----------------|
| データエンジニア | ★★★★★ | ★★★★★ |
| データアナリスト | ★★★★★ | ★★★☆☆ |
| MLエンジニア | ★★★☆☆ | ★★★★★ |
| データアーキテクト | ★★★★★ | ★★★★★ |
| データサイエンティスト | ★★★☆☆ | ★★★★★ |

### 市場価値・求人動向（2025年）

- Snowflake: BIツール・SQLベースの分析基盤として広く採用。金融・小売・ヘルスケア業界で特に需要が高い
- Databricks: ビッグデータ処理・機械学習パイプラインで主流。テック系・スタートアップで需要が急増中
- 両方習得: データプラットフォームエンジニアとして高い市場価値。年収レンジが大幅に広がる

### 学習スケジュール目安

```
Month 1-2:  前提知識 + Snowflake フェーズ1
Month 3-4:  Snowflake フェーズ2-3 + SnowPro Core 受験
Month 5-6:  Databricks フェーズ1-2
Month 7-8:  Databricks フェーズ3 + Data Engineer Associate 受験
Month 9-10: 実践プロジェクト（中級）
Month 11-12: 上級資格対策 + ポートフォリオ整備
```

---

## 進捗チェックリスト

### Snowflake
- [ ] アカウント作成・初回ログイン完了
- [ ] 基本SQL操作（ロード・クエリ）
- [ ] Virtual Warehouseの理解
- [ ] Time Travel 実習
- [ ] SnowPro Core 受験

### Databricks
- [ ] Community Edition セットアップ
- [ ] Notebook基本操作
- [ ] Delta Lake CRUD操作
- [ ] Auto Loader でのデータ取り込み
- [ ] MLflow での実験記録
- [ ] Data Engineer Associate 受験

---

*最終更新：2026-05-28*
