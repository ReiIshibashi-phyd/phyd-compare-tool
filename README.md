# CSV比較ツール - S3連携版

S3バケット内の2つのPrefix間でCSVファイルを比較し、差分レポートをS3に出力するツールです。

## 機能

- S3上の2つのPrefix（Before, After）間でCSVファイルを比較
- パターンマッチング（正規表現）によるCSVファイルの自動検出と比較
- 正規表現パターンで比較ファイルを指定可能（例：`tccontract.*\.csv$`）
- キー列による突合比較（単一・複数キー対応）
- 全項目の差分検出（追加/削除/変更）
- 差分レポートをS3に自動出力

## セットアップ

### 1. 依存関係のインストール

```bash
pip install -r requirements.txt
```

### 2. AWS認証情報の設定

```bash
# AWS CLIで認証情報を設定
aws configure
```

## 使用方法

### Jupyter Notebookで実行

```bash
jupyter notebook
```

`notebooks/csv_compare_s3.ipynb` を開いて実行

### ノートブック内の設定（セル2）

```python
# ファイル別設定
FILE_SETTINGS = [
    {
        'pattern': r'tccontract.*\.csv$',
        'has_header': True,
        'key_columns': ['vin'],
        'ignore_columns': []
    },
    {
        'pattern': r'run-.*-part-r-\d+$',
        'has_header': True,
        'key_columns': ['vin'],
        'ignore_columns': []
    }
]
```

## データの配置

```bash
# 処理日のディレクトリを作成
PROCESS_DATE=$(date +%Y%m%d)

# S3にファイルをアップロード
aws s3 cp your_file.csv s3://k-ishibashi-test/${PROCESS_DATE}/before/
aws s3 cp your_file.csv s3://k-ishibashi-test/${PROCESS_DATE}/after/

# 確認
aws s3 ls s3://k-ishibashi-test/${PROCESS_DATE}/before/
aws s3 ls s3://k-ishibashi-test/${PROCESS_DATE}/after/
```

## 比較用S3バケット構成

```
s3://my-bucket/
├── 20260106/                # 処理日（YYYYMMDD）
│   ├── before/              # Before（比較元）
│   │   ├── sample.csv
│   │   └── tccontract.csv
│   ├── after/               # After（比較先）
│   │   ├── sample.csv
│   │   └── tccontract.csv
│   └── output/              # 差分レポート出力先
│       ├── sample_diff_20250122_120000.csv
│       └── tccontract_diff_20250122_120000.csv
└── 20260107/
    ├── before/
    ├── after/
    └── output/
```

## 出力ファイル

### 差分レポート（CSV）
- S3パス: `s3://{bucket}/{output-prefix}/{ファイル名}_diff_{タイムスタンプ}.csv`
- 内容: key, diff_type, column, before_value, after_value

### 差分タイプ
- `ADDED`: Afterに新規追加された行
- `DELETED`: Beforeから削除された行
- `MODIFIED`: 変更された項目

## 処理フロー

### 1. バイナリチェック（MD5ハッシュ）
- ファイル全体をMD5ハッシュで比較
- 同一の場合は処理終了
- 異なる場合は詳細比較へ進む

### 2. キー突合比較
- ファイルをメモリに読込
- キー列でソート
- 削除・追加・変更を検出

### 3. 差分レポート出力
- 差分をCSVで出力
- S3に自動アップロード
- 詳細レポートを表示

```
