📘 Data Dictionary  
Nurse Turnover Analysis – Master Dataset Specification  

---

## 1. Overview

本ドキュメントは、本プロジェクトで使用する master.csv（統合分析データ）の列仕様を定義する。

本データセットは、複数の公的統計を統合し、  
都道府県単位で看護師離職構造を分析するための単一データソース（Single Source of Truth）として設計されている。

本データセットの1レコードは、  
**「都道府県 × 年度（Fiscal Year）」の集約値**を表す。

---

## Change Log

- 2026-02-13:
  - データ粒度を「都道府県 × 年度（Fiscal Year）」に統一
  - Join Key を `prefecture` → `prefecture + fiscal_year` に変更
  - fiscal_year を int 型で管理（FY2023 = 2023/04〜2024/03）
  - 求人倍率を FY平均（利用可能月のみ）に統一
  - 月数・最終月を品質補助指標として追加
  - 年度不一致統計に対し reference_year / as_of_date 管理方針を追加

---

## 2. Data Scope

単位：都道府県  

粒度：  
都道府県 × 年度（Fiscal Year）

主分析対象：  
FY2023（2023年4月〜2024年3月）

行数：  
主分析年度では 47 行（都道府県数）

備考：  
一部統計指標は取得可能な直近年データを使用する。  
年度不一致データは reference_year または as_of_date により管理する。

---

## 3. Join Key

| Column | Type | Description |
|--------|-----------|----------------|
| prefecture | string | 都道府県名（正規化済） |
| fiscal_year | int | 年度（FY）。例：FY2023 = 2023/04〜2024/03 |

### 正規化ルール
- 「都」「道」「府」「県」を付与して統一  
- JIS X 0401順  
- 全国・未回答は除外  

---

## 4. Target Variables（目的変数）

### turnover_total  
定義：常勤看護職員離職率（総数）  
単位：%  
出所：日本看護協会 病院看護実態調査  

計算：  
年間退職者数 ÷ 平均職員数 × 100  

---

### turnover_new_grad  
定義：新卒採用者離職率  
単位：%  

---

### turnover_experienced  
定義：既卒採用者離職率  
単位：%  

---

## 5. Compensation Variables（給与構造）

### salary_10yr_total  
定義：勤続10年看護師の年収  
単位：万円  

計算：  
基本給 × 12 + 年間賞与  

出所：賃金構造基本統計調査  

---

### bonus  
定義：年間賞与額  
単位：万円  

---

### annual_income  
定義：看護師平均年収  
単位：万円  

---

## 6. Workforce / Workload Variables（労働環境）

### nurse_count  
定義：常勤換算看護職員数  
出所：衛生行政報告例  

---

### nurse_per_100k  
定義：人口10万人あたり看護師数  

計算：  
nurse_count ÷ population × 100000  

---

### night_shift_72h_plus  
定義：月72時間以上夜勤を行う看護師割合  
単位：%  

---

## 7. Healthcare Facility Structure（医療供給構造）

### hospital_count  
定義：一般病院数  
注意：精神病院・診療所は含まない  

---

### large_hospital_count  
定義：500床以上の一般病院数  

---

### large_hospital_ratio  
計算：  
large_hospital_count ÷ hospital_count  

---

### hospital_per_100k  
計算：  
hospital_count ÷ population × 100000  

---

## 8. Cost of Living Variables（生活コスト）

### ent_private  
定義：民営借家の月額住居費  
単位：円  

算出式：  
rent_private = avg_rent_excl0 + avg_fee_excl0  

---

### home_ownership_rate  
定義：持ち家率  

---

### commute_time  
定義：平均通勤時間  
単位：分  

変換：  
時分表記 → 分  

例：  
1時間24分 → 84分  

---

## 9. Labor Market Variables（労働市場）

### job_openings_ratio_overall_fy_avg  
定義：  
全体有効求人倍率（月次）を FY 内で単純平均した値  

対象期間：  
FY2023（2023年4月〜2024年3月）

集約ルール：  
利用可能月のみで平均  

単位：  
倍  

出所：  
一般職業紹介状況  

---

### job_openings_ratio_months_n  
定義：  
FY平均算出に使用した月数（1〜12）  

---

### job_openings_ratio_as_of_date  
定義：  
FY平均算出に使用した最終月  

---

## 10. Demographic Variables（人口構造）

### population  
定義：都道府県人口  

---

### population_density  
定義：人口密度  
単位：人/km²  

---

## 11. Regional Dummy Variables（都市構造）

### metro_b  
定義：大都市圏ダミー変数  

対象：
- 東京  
- 神奈川  
- 千葉  
- 埼玉  
- 愛知  
- 大阪  
- 京都  
- 兵庫  
- 福岡  

---

## 12. Data Quality Rules

master.csv は以下条件を満たす必要がある。

行数：  
主分析年度では 47 行  

主キー：  
prefecture × fiscal_year  
重複禁止  

欠損制約：  

以下は NULL 不可  

- turnover_total  
- salary_10yr_total  
- nurse_count  

---

## 13. Known Limitations

- 通勤時間は全産業平均であり看護職特化ではない  
- 家賃は地域平均であり病院周辺価格を直接反映しない  
- 求人倍率は公表統計の集計粒度に依存  

---

## 14. Update Policy

分析基準年度は固定（FY2023）  
外部統計更新時は差分検証を実施する  

---

## 15. As-of / Reference Policy

年度（Fiscal Year）と一致しない統計データについては、  
時点差を明示するため以下の補助情報を保持する。

### reference_year
統計が取得された基準年  

### as_of_date
統計値が有効な最新時点  

目的：  
データ解釈の透明性および再現性を担保するため。

---

### Hospital Structure Variables (Interpretive Design)

| 変数名 | 定義 | 解釈ラベル |
|---|---|---|
| hospital_per_100k | 人口10万人あたり病院数 | 医療供給密度 |
| large_hospital_500p_per_100k | 500床以上病院 / 人口 | 地域中核医療集約 |
| large_hospital_700p_per_100k | 700床以上病院 / 人口 | 高度医療集約 |
| mega_hospital_900p_per_100k | 900床以上病院 / 人口 | 構造的異質性 |
| psychiatric_ratio | 精神科病院 / 全病院 | 診療構成 |

※ 病床規模は制度上の区分ではないが、医療機能や組織複雑性を示す代理指標として用いている。  
※ 主分析では500床以上病院指標を使用し、700床以上・900床以上の指標は感度分析用に位置付ける。
