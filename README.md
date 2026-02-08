
# 🏥 Data-Driven Approach to Nurse Retention in Japan  
**オープンデータ統合による構造的な看護師離職要因の定量整理（都道府県別）**

![Python](https://img.shields.io/badge/Python-3.10+-blue.svg?style=for-the-badge&logo=python&logoColor=white)
![Pandas](https://img.shields.io/badge/pandas-%23150458.svg?style=for-the-badge&logo=pandas&logoColor=white)
![Scikit-Learn](https://img.shields.io/badge/scikit--learn-%23F7931E.svg?style=for-the-badge&logo=scikit-learn&logoColor=white)

---

> 🚧 **Work in Progress**
>
> 本リポジトリは、**公開統計の統合〜分析までを「できるだけ再現可能な形」に近づける**ことを目的に継続的に改善しています。  
> データサイエンス講座の課題として始めた分析をベースに、<b>データ統合プロセス（作り方）</b>を含めて整理する方針で発展させています。
>
> **現在の実装状況**
> - 47都道府県の公開統計を統合した分析用マスタデータを作成し、分析に利用しています。
> - 一部の統計資料については、資料形式（PDF / Excel / CSV）の不統一や表構造の差異により、自動統合の難易度が高いため、現時点では手動統合したCSVを利用しています。
> - ただし再現性を担保するため、Rawデータからマスタデータを再生成するPython統合処理の整備を段階的に進めています（現在リファクタリング中）。
> - データ品質を担保するため、以下の検証を実施しています。
>> - 行数確認
>> - 主キー重複チェック
>> - 欠損値確認
> - マスタデータを用いて以下の分析を実施しています。
>> - 探索的データ分析（EDA）回帰モデルによる構造要因の整理
>> - 回帰モデルによる構造要因の整理
>
> **今後の改善計画**
> - 手動統合している統計資料について、段階的にETL処理へ移行し、Rawデータから再生成できる範囲を拡張します。
> - 統合処理について、可読性・保守性を考慮した構造への整理を進めます。
> - データ定義書および品質チェックリストを拡充し、データ仕様の明文化を進めます。
> - 既卒・新卒別および都市・地方の観点から分析を拡張予定です。


## Development Status

| Component | Status |
|-----------|------------|
| Data Integration | 🔄 Refactoring into reproducible Python ETL pipeline |
| Data Quality Validation | ✅ Implemented |
| Exploratory Analysis | ✅ Implemented |
| Modeling | ✅ Implemented (Baseline structural model completed) |
| Interpretation | ✅ Implemented (Structural hypothesis derived) |
| Subgroup Modeling | 🔄 In Progress |
| Visualization Assets | 🟡 Expanding for presentation |
| Documentation | 🟡 Expanding |




## ⚡ 3-Second Summary
- 47都道府県統計を統合し、看護師離職率の構造要因を分析  
- PDF/CSVなど形式差・定義差がある統計を扱い、作成手順を残すことを重視
- Raw統計から分析データを再生成できるETL設計を目指す
---

---

## 📚 Documentation

- データ定義書：`docs/data_dictionary.md`

---

## 📖 Overview
**「看護師の離職は、本当に“個人の問題”なのか？」**

私は約10年間、看護師として臨床現場に従事し、多くの同僚が離職していく現実を目の当たりにしてきました。  
現場では離職理由が個人の資質や精神的要因として語られることが多い一方で、私は  
**「地域特性や労働市場などの構造的要因も関与しているのではないか」** という仮説を持ちました。

本プロジェクトでは、47都道府県の公開統計データを統合し、探索的データ分析（EDA）および回帰分析を通じて、看護師離職率を規定する要因を定量的に整理しました。

また分析を進める過程で、統計資料ごとに以下の差異が存在することを確認しました。

・CSVの文字コード差異（UTF-8 / Shift_JIS）  
・PDF統計表における列構造の不統一  
・統計年度の不一致  

特に統計年度の差異については、比較可能性を担保するため、可能な範囲で年度を揃えて統合しました。

---

## ✅ Key Findings

- 離職率は「個人要因」だけでなく、**労働負荷・生活コスト・労働市場**などの構造要因と整合的な関連が見られた  
- **都市部 vs 地方**など、地域構造によって離職のパターンが異なる可能性が示唆された  
- データ品質が結果の信頼性を左右することを確認した  

以下の図は、地域構造によって離職要因のパターンが異なる可能性を示した代表的な分析結果です。

![Urban vs Rural Discovery](outputs/figures/final_discovery.png)
※ 地方では持ち家率が既卒看護師の定着と負の関連を示し、
都市部では人口密度が新卒離職と正の関連を示した。

---

## ✅ Model Selection

### モデル性能比較（最終評価）

![モデル性能比較](outputs/figures/model_comparison.png)

最終的に 全体離職率のモデルは**Model D** を採用しました。  
Model D は Model C に含まれていた説明変数のうち、統計的に有意でなかった変数を削除し、説明力を維持したままモデルの簡素化を行ったものです。

採用理由は以下の通りです。

- 調整済み決定係数を維持したままモデルの簡素化に成功した  
- LOOCV性能が改善し、予測の安定性が向上した  
- AIC / BICが改善し、モデルの効率性が向上した  
- 説明変数が整理され、解釈性が向上した  
- 不要な変数を削減することで過学習リスクを低減できた

---

## 🧩 Extension: Subgroup Analysis
メインモデルを新卒・既卒に分割して適用したところ、全国一律の構造では十分な説明力が得られなかった。
この結果を踏まえ、離職構造がキャリア段階と地域特性の組み合わせによって異なる可能性を検証するため、サブグループ分析を実施した。

本分析では、都道府県を人口密度に基づいて2群に分割した。
人口密度の中央値（259.6人/km²）を閾値とし、

Urban（高密度群）：24都道府県

Rural（低密度群）：23都道府県

として分析を行った。

分析の結果、全国一律のモデルでは捉えられなかった離職構造が、都市部と地方部に分割することで明確に異なることが示唆された。

主な示唆
地方 × 既卒
持ち家率が高いほど離職率が低下する傾向が確認された（定着を促進する“アンカー”効果）

都市 × 新卒
人口密度が高いほど離職率が上昇する傾向が確認された（密度ストレス仮説）

※ 本分析は探索的検証であり、因果関係を示すものではない。<br>
※ 行政区分としての都市・地方とは一致しない統計的区分を使用している。<br>
※ 06_subgroup_analysis.ipynb にて再現可能（同一マスターデータ基盤を使用）。

---
## 🛠 How the Data is Structured

本プロジェクトでは、公開統計データを統合し、
Rawデータとコードから分析用データを再生成できる構造を意識して作成しています。

公的統計は、定義や単位、集計粒度が資料ごとに異なることが多いため、
分析結果の解釈がぶれないよう、以下の点を意識してデータを整理しています。

---

### 1) Rawデータの保持
- `data/raw`：取得したデータをそのまま保存  
- `data/processed`：加工後のデータを分けて管理  

---

### 2) 変数定義の整理
- 変数名を統一  
- 出典・単位・定義を整理  

---

### 3) データ品質の確認
- 都道府県キーが重複していないか  
- 行数が47都道府県分そろっているか  
- 欠損値や異常値の確認  

---

### 4) 再現性を意識した構成
- 抽出・整形・分析を Notebook 単位で分離  
- 再実行時にどこで問題が起きたかを確認しやすい構造にしています  


---

## 🔍 Analysis Flow
1. **Data Validation**  
   統計資料ごとの定義差・欠損・分布を確認  
2. **Exploratory Data Analysis（EDA）**  
   地域差の可視化、都市部・地方部の構造差を把握  
3. **Modeling**  
   回帰分析により離職率に関連する主要因子を推定  
4. **Interpretation**  
   統計結果と臨床経験を統合し、構造的背景を仮説として整理<br>
　　回帰モデルで得られた係数を、現場で理解しやすいスケールに変換し、<br>
　　離職に影響する構造要因を整理した。

　　![係数の現場スケール翻訳](outputs/figures/impact_translation.png)

　　統計的な係数をそのまま提示するのではなく、
　　現場の意思決定に活用できる形へ翻訳することを重視した。 
5. **Subgroup Analysis**  
   新卒/既卒など、集団差で構造が変わるかを検証  

---

## 📦 Master Dataset
統合マスタは `prefecture` をキーとして以下の主要変数を保持します（例）。

### Primary Key
-`prefecture`

### Outcome
- `turnover_total`
- `turnover_new_grad`
- `turnover_experienced`

### Workforce / Working Conditions
- `night_shift_72h_plus`
- `overtime_hours`
- `average_age`
- `annual_income`
- `nurse_per_100k`

### Regional Controls
- `population`
- `population_density`

---

## 💻 Quickstart

### 1) リポジトリ取得
```bash
git clone https://github.com/<your-username>/nurse-turnover-analysis.git
cd nurse-turnover-analysis

2) 依存関係インストール
pip install -r requirements.txt

3) Notebookを上から順に実行すると、rawデータから master データが再生成されます。

jupyter notebook

推奨実行順：
notebooks/00_extract_from_pdf.ipynb

notebooks/01_create_master_dataset.ipynb

notebooks/02_data_validation.ipynb

notebooks/03_eda_overview.ipynb

notebooks/04_modeling.ipynb

notebooks/05_interpretation.ipynb

notebooks/06_subgroup_analysis.ipynb

生成物：

data/processed/master_nurse_turnover.csv

## 📁 Project Structure

nurse-turnover-analysis/
├── data/
│   ├── raw/
│   └── processed/
│
├── notebooks/
│   ├── 00_extract_from_pdf.ipynb
│   ├── 01_create_master_dataset.ipynb
│   ├── 02_data_validation.ipynb
│   ├── 03_eda_overview.ipynb
│   ├── 04_modeling.ipynb
│   ├── 05_interpretation.ipynb
│   └── 06_subgroup_analysis.ipynb
│
├── src/
│   └── ETL処理およびデータ変換ロジックのモジュール化を想定（現在リファクタリング中）
│
├── outputs/
│   └── figures/
│ 
│
├── requirements.txt
└── README.md

***

## ⚠️ Limitations & Scope

本分析は相関構造の整理を目的としており、因果関係を証明するものではありません

解釈には臨床経験に基づく仮説が含まれます（政策提言は示唆の範囲）

都道府県単位の集計データのため、施設・個人単位の違いを直接説明するものではありません

***

👤 Author  
Hideomi Hayakawa
