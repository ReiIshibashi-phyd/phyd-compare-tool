# CSV比較ツール

## ディレクトリ構成
```
phyd-compare-tool/
├── data/
│   ├── before/           # 比較元データ（旧）
│   │   └── *.csv
│   └── after/            # 比較先データ（新）
│       └── *.csv
├── notebooks/
│   └── csv_compare.ipynb # メイン比較ノートブック
├── output/
│   └── (差分レポート出力先)
├── requirements.txt
└── README.md
```

## 使用方法
1. `pip install -r requirements.txt`
2. `jupyter notebook`
3. `notebooks/csv_compare.ipynb` を開いて実行