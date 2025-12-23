# CSV比較ツール - S3連携版

S3バケット内の2つのPrefix間でCSVファイルを比較し、差分レポートをS3に出力するツールです。

## 機能

- S3上の2つのPrefix（PrefixA, PrefixB）間でCSVファイルを比較
- 同じファイル名のCSVを自動検出して比較
- キー列による突合比較（単一・複数キー対応）
- 全項目の差分検出（追加/削除/変更）
- 大容量ファイル対応（自動チャンク処理）
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
# S3設定
S3_BUCKET = 'k-ishibashi-test'  # バケット名
S3_PREFIX_A = 'data/before/'    # 比較元Prefix
S3_PREFIX_B = 'data/after/'     # 比較先Prefix
S3_OUTPUT_PREFIX = 'output/'    # 出力先Prefix

# 処理設定
CHUNK_SIZE = 10000              # チャンクサイズ（行数）
MEMORY_THRESHOLD_MB = 100       # メモリ閾値（MB）

# ファイル別設定
FILE_SETTINGS = {
    'sample.csv': {
        'key_columns': ['id'],
        'ignore_columns': ['updated_at']
    }
}
```

## テストデータの配置

```bash
# S3にテストファイルをアップロード
aws s3 cp your_file.csv s3://k-ishibashi-test/data/before/
aws s3 cp your_file.csv s3://k-ishibashi-test/data/after/

# 確認
aws s3 ls s3://k-ishibashi-test/data/before/
aws s3 ls s3://k-ishibashi-test/data/after/
```

## S3バケット構成例

```
s3://my-bucket/
├── data/
│   ├── before/              # PrefixA（比較元）
│   │   ├── sample.csv
│   │   └── tccontract.csv
│   └── after/               # PrefixB（比較先）
│       ├── sample.csv
│       └── tccontract.csv
└── output/                  # 差分レポート出力先
    ├── sample_diff_20250122_120000.csv
    └── tccontract_diff_20250122_120000.csv
```

## 出力ファイル

### 差分レポート（CSV）
- S3パス: `s3://{bucket}/{output-prefix}/{ファイル名}_diff_{タイムスタンプ}.csv`
- 内容: key, diff_type, column, baseline_value, candidate_value

### 差分タイプ
- `ADDED`: PrefixBに新規追加された行
- `DELETED`: PrefixAから削除された行
- `MODIFIED`: 変更された項目

## IAM権限要件

S3操作に必要な最小限のIAM権限：

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::your-bucket-name",
        "arn:aws:s3:::your-bucket-name/data/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject"
      ],
      "Resource": [
        "arn:aws:s3:::your-bucket-name/output/*"
      ]
    }
  ]
}
```

## 課題と制約事項

### 現在の課題

1. **大容量ファイル対応** ✅ 実装済
   - `compare_s3_large.py`でチャンク処理に対応
   - メモリ閉値を超えるファイルは自動的にチャンク処理
   - デフォルト: 100MB超でチャンク処理、チャンクサイズ10,000行

2. **並列処理未対応**
   - 複数ファイルを順次処理するため、ファイル数が多い場合は時間がかかる
   - 対策: マルチスレッド/マルチプロセス処理の実装

3. **エラーハンドリング**
   - ネットワークエラー時のリトライ機能なし
   - 対策: boto3のリトライ設定やエラー時の継続処理

4. **進捗表示**
   - 大容量ファイル処理時の進捗が不明
   - 対策: tqdmなどの進捗バーの追加

5. **差分レポートの詳細度**
   - 統計情報やサマリーレポートが未実装
   - 対策: 統計レポート機能の追加

### 制約事項

- S3のリージョンは環境変数またはAWS設定に依存
- CSVファイルのエンコーディングはUTF-8を想定
- ファイル名が完全一致するもののみ比較対象
- Prefix末尾のスラッシュ（/）は自動調整されるが、明示的な指定を推奨

## トラブルシューティング

### 認証エラーが発生する場合
```bash
# AWS認証情報を確認
aws sts get-caller-identity
```

### ファイルが見つからない場合
```bash
# S3のファイル一覧を確認
aws s3 ls s3://my-bucket/data/before/

# Prefixが正しいか確認（末尾のスラッシュに注意）
```

### メモリエラーが発生する場合
- ファイルサイズを確認し、大容量の場合は分割処理を検討
- EC2インスタンスのメモリを増強

## ライセンス

MIT License
