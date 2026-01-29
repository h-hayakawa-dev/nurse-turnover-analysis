# 🏥 Data-Driven Approach to Nurse Retention in Japan
### データ分析に基づく、看護師離職防止のための地域別・最適化戦略

![Python](https://img.shields.io/badge/Python-3.10+-blue.svg?style=for-the-badge&logo=python&logoColor=white)
![Pandas](https://img.shields.io/badge/pandas-%23150458.svg?style=for-the-badge&logo=pandas&logoColor=white)
![Scikit-Learn](https://img.shields.io/badge/scikit--learn-%23F7931E.svg?style=for-the-badge&logo=scikit-learn&logoColor=white)

## 📖 Overview (概要)
「看護師の離職は、本当に“個人の問題”なのか？」

私は約10年間、看護師として臨床現場に従事し、多くの同僚が離職していく現実を目の当たりにしてきました。
現場では離職理由が個人の資質や精神的要因として語られることが多い一方で、
私は 「地域特性や労働市場などの構造的要因も大きく関与しているのではないか」 という問題意識を持ちました。

本プロジェクトでは、47都道府県の公開統計データを用い、
探索的データ分析（EDA）、回帰分析、クラスタリングを通じて、
看護師離職率を規定する構造的要因を定量的に整理・可視化することを目的としています。

また、分析結果そのものだけでなく、
再現性を意識したデータ処理・ディレクトリ設計にも重点を置いています。

## 🛠 Engineering Focus (データエンジニアリング的側面)
本リポジトリは、分析を「一度きりのNotebook」で終わらせないことを意識し、
以下の点を重視して構成しています。

Data Quality

欠損値・外れ値の確認

分析前提条件の明示

Separation of Concerns

データ確認・EDA・モデリング・解釈をNotebook単位で分離

分析ロジックと考察の混在を避ける構成

Reproducibility

raw / processed データの分離

相対パスによるデータ参照

使用ライブラリの明示

※ 現在、再現性向上のため Notebook整理および src/ への処理切り出しを段階的に進行中です。

---

## 🔍 Key Findings （分析の流れ）

本プロジェクトでは、以下の流れで分析を行っています。

Data Validation

データ構造・欠損・分布の確認

Exploratory Data Analysis（EDA）

離職率の地域差

都市部・地方部の構造的違いの可視化

Modeling

回帰分析による要因整理

クラスタリングによる地域分類

Interpretation

統計結果を踏まえた構造的解釈

地域特性に応じた示唆整理

---

### Directory Structure
## 📁 Directory Structure

```text
nurse-turnover-analysis/
├── data/
│   ├── raw/                 # 取得したままの元データ
│   └── processed/           # 前処理・整形後の分析用データ
│
├── notebooks/
│   ├── 01_data_validation.ipynb   # データ確認・前提条件チェック
│   ├── 02_eda_overview.ipynb      # 探索的データ分析・可視化
│   ├── 03_modeling.ipynb          # 回帰分析・クラスタリング
│   └── 04_interpretation.ipynb    # 結果の解釈・考察
│
├── src/                     # 再利用可能な処理ロジック（今後整理予定）
│
├── outputs/
│   ├── figures/             # 作成した図表
│   └── tables/              # 集計結果・モデル出力
│
├── requirements.txt         # 依存ライブラリ一覧
└── README.md
'''

---
📌 Notes

本分析は 因果関係の証明ではなく、相関・構造の整理を目的としています。

結果の解釈は、現場経験と統計結果を踏まえた示唆として位置づけています。
---
Author: Hideomi Hayakawa
