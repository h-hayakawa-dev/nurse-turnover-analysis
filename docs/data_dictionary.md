📘 Data Dictionary
Nurse Turnover Analysis – Master Dataset Specification
1. Overview

本ドキュメントは、本プロジェクトで使用する master.csv（統合分析データ） の列仕様を定義する。

本データセットは、複数の公的統計を統合し、
都道府県単位で看護師離職構造を分析するための単一データソース（Single Source of Truth） として設計されている。

2. Data Scope

単位：都道府県

行数：47行（固定）

分析基準年：2023年

一部指標は取得可能な直近年データを使用

3. Join Key
Column	Type	Description
prefecture	string	都道府県名（正規化済）
正規化ルール

「都」「道」「府」「県」を付与して統一

JIS X 0401順

全国・未回答は除外

4. Target Variables（目的変数）
turnover_total

定義：常勤看護職員離職率（総数）

単位：%

出所：日本看護協会 病院看護実態調査

計算：

年間退職者数 ÷ 平均職員数 × 100

turnover_new_grad

定義：新卒採用者離職率

単位：%

出所：同上

turnover_experienced

定義：既卒採用者離職率

単位：%

出所：同上

5. Compensation Variables（給与構造）
salary_10yr_total

定義：勤続10年看護師の年収

単位：万円

計算：

基本給 × 12 + 年間賞与


出所：賃金構造基本統計調査

bonus

定義：年間賞与額

単位：万円

備考：給与構造差の分析用

annual_income

定義：看護師平均年収

単位：万円

出所：賃金構造基本統計調査

6. Workforce / Workload Variables（労働環境）
nurse_count

定義：常勤換算看護職員数

出所：衛生行政報告例

nurse_per_100k

定義：人口10万人あたり看護師数

計算：

nurse_count ÷ population × 100000

night_shift_72h_plus

定義：月72時間以上夜勤を行う看護師割合

単位：%

出所：病院看護実態調査

7. Healthcare Facility Structure（医療供給構造）
hospital_count

定義：一般病院数

出所：医療施設調査

注意：
精神病院・診療所は含まない

large_hospital_count

定義：500床以上の一般病院数

出所：医療施設調査

large_hospital_ratio

定義：大規模病院比率

計算：

large_hospital_count ÷ hospital_count

hospital_per_100k

定義：人口10万人あたり病院数

計算：

hospital_count ÷ population × 100000

8. Cost of Living Variables（生活コスト）
rent_private

定義：民営借家平均家賃

単位：円

条件：

共益費を含む

家賃0円物件除外

出所：住宅・土地統計調査

home_ownership_rate

定義：持ち家率

分母：

主世帯


出所：住宅・土地統計調査

commute_time

定義：平均通勤時間

単位：分

変換：

時分表記 → 分へ変換


例：
1時間24分 → 84分

9. Labor Market Variables（労働市場）
job_openings_ratio

定義：看護職有効求人倍率

単位：倍

出所：一般職業紹介状況

10. Demographic Variables（人口構造）
population

定義：都道府県人口

出所：国勢調査

population_density

定義：人口密度

単位：人/km²

11. Regional Dummy Variables（都市構造）
metro_b

定義：大都市圏ダミー変数

値：

1：
東京
神奈川
千葉
埼玉
愛知
大阪
京都
兵庫
福岡

0：
その他地域

12. Data Quality Rules

master.csv は以下条件を満たす必要がある。

行数
47固定

主キー
prefecture 重複禁止

欠損制約

以下はNULL不可：

turnover_total

salary_10yr_total

nurse_count

13. Known Limitations

通勤時間は全産業平均であり、看護職特化データではない

家賃は地域平均であり、病院周辺価格を直接反映しない

求人倍率は公表統計の集計粒度に依存

14. Update Policy

分析基準年は固定（2023）

外部統計更新時は差分検証を実施する