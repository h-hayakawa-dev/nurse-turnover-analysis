# 🏥 Data-Driven Approach to Nurse Retention in Japan
### データ分析に基づく、看護師離職防止のための地域別・最適化戦略

![Python](https://img.shields.io/badge/Python-3.10+-blue.svg?style=for-the-badge&logo=python&logoColor=white)
![Pandas](https://img.shields.io/badge/pandas-%23150458.svg?style=for-the-badge&logo=pandas&logoColor=white)
![Scikit-Learn](https://img.shields.io/badge/scikit--learn-%23F7931E.svg?style=for-the-badge&logo=scikit-learn&logoColor=white)

## 📖 Overview (概要)
**「看護師の離職は、本当に“個人の問題”なのか？」**

私は約10年間、看護師として臨床現場に従事し、多くの同僚が離職していく現実を目の当たりにしてきました。
現場では離職理由が個人の資質や精神的要因として語られることが多い一方で、私は **「地域特性や労働市場などの構造的要因も大きく関与しているのではないか」** という仮説を持ちました。

本プロジェクトでは、47都道府県の公開統計データを用い、探索的データ分析（EDA）・回帰分析を通じて、看護師離職率を規定する構造的要因を定量的に可視化しました。

## 🛠 Engineering Focus (データエンジニアリング的側面)
本リポジトリは、一過性の分析に留まらず、**「再現性」と「保守性」**を重視したデータパイプラインとして設計しています。

* **Data Quality**: 欠損値・外れ値の処理プロセスを明確化し、分析の前提条件を担保。
* **Separation of Concerns**: データ確認・EDA・モデリング・解釈をNotebook単位で分離し、ロジックと考察の混在を防止。
* **Reproducibility**: `raw`（生データ）と `processed`（加工済みデータ）を厳格に分離。相対パスによる参照と `requirements.txt` による環境定義で、誰でも再現可能な状態を維持。

## 🔍 Key Findings （分析の流れ）
1.  **Data Validation**: データ構造・欠損・分布の統計的確認
2.  **EDA**: 地域ごとの離職率ヒートマップ作成、都市部・地方部の構造的差異の可視化
3.  **Modeling**: 回帰分析による主要因子の特定
4.  **Interpretation**: ドメイン知識（臨床経験）と統計結果を統合した構造的解釈

## 💻 Usage (環境構築・実行)
```bash
# リポジトリのクローン
git clone [https://github.com/your-username/nurse-turnover-analysis.git](https://github.com/your-username/nurse-turnover-analysis.git)

# ディレクトリ移動
cd nurse-turnover-analysis

# 依存ライブラリのインストール
pip install -r requirements.txt

nurse-turnover-analysis/
├── data/
│   ├── raw/             # 取得したままの不変な元データ
│   └── processed/       # 前処理・整形後の分析用データ
│
├── notebooks/
│   ├── 01_data_validation.ipynb   # データ確認・前提条件チェック
│   ├── 02_eda_overview.ipynb      # 探索的データ分析・可視化
│   ├── 03_modeling.ipynb          # 回帰分析・クラスタリング
│   └── 04_interpretation.ipynb    # 結果の解釈・考察
│
├── src/                 # 共通化された処理ロジック（.py）
├── outputs/
│   ├── figures/         # 作成した図表（png/jpg）
│   └── tables/          # 集計結果・モデル出力（csv）
│
├── requirements.txt     # 依存ライブラリ一覧
└── README.md

---
## ⚠️ Limitations & Scope (分析の前提と限界)
* 本分析は相関・構造の整理を目的としており、因果関係を証明するものではありません。
* 結果の解釈には、筆者の臨床経験に基づく仮説が含まれます。

---
**Author:** Hideomi Hayakawa
