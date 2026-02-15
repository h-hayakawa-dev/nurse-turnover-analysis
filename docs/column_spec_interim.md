# カラム定義・技術仕様書 (Column Specification)

## 1. 概要 (Overview)
本ドキュメントは、本プロジェクトにおける各カラムの技術仕様、およびデータ契約（Data Contract）を定義するものである。

### 本ドキュメントの目的
* **ETL再現性の担保**: 処理ロジックを明文化し、属人化を防ぐ。
* **データ品質ルールの明確化**: NULL許容範囲や値域（Range）を定義する。
* **分析解釈の統一**: ビジネス定義（Definition）と言葉の揺らぎをなくす。
* **データ契約の明文化**: ソースデータと出力データの整合性を保証する。

---

## 2. 共通基準 (Global Standards)

### データの粒度 (Canonical Grain)
本プロジェクトにおけるデータの最小粒度（Primary Key相当）は以下とする。
> **prefecture × fiscal_year**

### 単位の統一基準 (Standard Unit Policy)
数値データの単位は、特段の記載がない限り以下の基準に従う。

| Category | Standard Unit | Description |
| :--- | :--- | :--- |
| **Ratio** | % (0–100) | 割合。0.0–1.0ではなく、**0–100のパーセント表記**とする。 |
| **Monetary** | 円 (JPY) | 金額。整数表記を基本とする。 |
| **Population** | 人 | 人口、職員数など。 |
| **Density** | 人/km² | 密度。 |
| **Time** | 分 | 時間。 |
| **Job Ratio** | 倍 | 有効求人倍率など。 |

---

## 3. カラム定義テンプレート (Template)
新規カラムを追加する際は、以下のテンプレートをコピーして使用すること。

```yaml
### <カラム名_英語>
- **Type**: <データ型 (例: float64, int, object)>
- **Unit**: <単位>
- **Grain**: prefecture × fiscal_year
- **Definition**: <業務的な定義・意味>
- **Calculation**: <算出ロジック・計算式>
- **Source**: <データソース元>
- **Reference Year Policy**: <年度のマッピングルール>
- **Transformation**:
    - <ETL処理内容>
    - <単位変換>
    - <正規化処理>
- **Null Policy**: <NULLの可否 (例: 不可)>
- **Value Range**: <値の許容範囲 (例: 0-100)>
- **Quality Checks**:
    - <検証ルール>
- **Dependencies**: <依存するカラム>
- **Lineage**: raw → interim → master
- **Notes**: <補足事項>
- **Last Verified**: <YYYY-MM-DD>

4. カラム詳細仕様 (Column Specifications)
目的変数 (Target Variables)

### prefecture

**Type**: string[python]  
**Unit**: -  
**Grain**: prefecture × fiscal_year  

**Definition**  
都道府県名（結合キー）

**Calculation**  
元データの都道府県列を使用（「全国」は除外）。

**Source**  
厚生労働省（衛生行政報告例）

**Reference Year Policy**  
適用なし（キー列）。

**Transformation**
- header行調整（メタ行除外）
- 都道府県列→prefecture（strip）
- 「全国」行除外

**Null Policy**  
NULL不可

**Value Range**  
都道府県名（47都道府県）

**Quality Checks**
- prefecture.nunique() = 47
- NULL=0

**Dependencies**  
なし

**Lineage**  
厚労省_衛生行政報告例_看護師数_2023.csv → interim_nurse_staff_2022.parquet

**Notes**  
CSVは多段ヘッダのため、処理途中でMultiIndex列になり得る。保存前に列をflattenしている。

**Last Verified**  
2026-02-14

---

### fiscal_year

**Type**: int64  
**Unit**: 年  
**Grain**: prefecture × fiscal_year  

**Definition**  
master結合のために付与した分析年度（結合キー）

**Calculation**  
固定値 2023。

**Source**  
（ETL付与）

**Reference Year Policy**  
観測年とは独立。結合キーとしての年度。

**Transformation**
- fiscal_year = 2023 を付与

**Null Policy**  
NULL不可

**Value Range**  
= 2023

**Quality Checks**
- fiscal_year.nunique() = 1
- 値が2023のみ
- NULL=0

**Dependencies**  
なし

**Lineage**  
厚労省_衛生行政報告例_看護師数_2023.csv → interim_nurse_staff_2022.parquet

**Notes**  
本指標の時点は reference_year を参照。

**Last Verified**  
2026-02-14

---

### reference_year

**Type**: int64  
**Unit**: 年  
**Grain**: prefecture × fiscal_year  

**Definition**  
指標の観測時点（年）

**Calculation**  
固定値 2022（令和4年度/2022年末現在）。

**Source**  
厚生労働省（衛生行政報告例）

**Reference Year Policy**  
本データの実際の時点は reference_year で管理し、master結合は fiscal_year で行う。

**Transformation**
- reference_year = 2022 を付与

**Null Policy**  
NULL不可

**Value Range**  
= 2022

**Quality Checks**
- reference_year.nunique() = 1
- 値が2022のみ
- NULL=0

**Dependencies**  
なし

**Lineage**  
厚労省_衛生行政報告例_看護師数_2023.csv → interim_nurse_staff_2022.parquet

**Notes**  
ファイル名に「_2023」が含まれるが、表の時点は2022年末。

**Last Verified**  
2026-02-14

---

### 保健師数【人】

**Type**: int64  
**Unit**: 人  
**Grain**: prefecture × fiscal_year  

**Definition**  
就業保健師数（都道府県別、実数）

**Calculation**  
統計定義に従う（実数）。

**Source**  
厚生労働省（衛生行政報告例）

**Reference Year Policy**  
観測時点は reference_year=2022。

**Transformation**
- header行調整（多段ヘッダ処理）
- 「全国」除外（47都道府県）
- to_numeric（errors="coerce"）
- fiscal_year=2023付与
- reference_year=2022付与
- 必要列射影

**Null Policy**  
NULL不可

**Value Range**  
>= 0

**Quality Checks**
- rows=47
- dup_keys(prefecture,fiscal_year)=0
- NULL=0
- negative=0

**Lineage**  
厚労省_衛生行政報告例_看護師数_2023.csv → interim_nurse_staff_2022.parquet

**Last Verified**  
2026-02-14

---

### 助産師数【人】

**Type**: int64  
**Unit**: 人  
**Grain**: prefecture × fiscal_year  

**Definition**  
就業助産師数（都道府県別、実数）

**Calculation**  
統計定義に従う（実数）。

**Source**  
厚生労働省（衛生行政報告例）

**Reference Year Policy**  
観測時点は reference_year=2022。

**Transformation**
- 多段ヘッダ処理
- 「全国」除外
- to_numeric
- fiscal_year=2023付与
- reference_year=2022付与
- 必要列射影

**Null Policy**  
NULL不可

**Value Range**  
>= 0

**Quality Checks**
- rows=47
- dup_keys=0
- NULL=0
- negative=0

**Lineage**  
厚労省_衛生行政報告例_看護師数_2023.csv → interim_nurse_staff_2022.parquet

**Last Verified**  
2026-02-14

---

### 看護師数【人】

**Type**: int64  
**Unit**: 人  
**Grain**: prefecture × fiscal_year  

**Definition**  
就業看護師数（都道府県別、実数）

**Calculation**  
統計定義に従う（実数）。

**Source**  
厚生労働省（衛生行政報告例）

**Reference Year Policy**  
観測時点は reference_year=2022。

**Transformation**
- 多段ヘッダ処理
- 「全国」除外
- to_numeric
- fiscal_year=2023付与
- reference_year=2022付与
- 必要列射影

**Null Policy**  
NULL不可

**Value Range**  
>= 0

**Quality Checks**
- rows=47
- dup_keys=0
- NULL=0
- negative=0

**Lineage**  
厚労省_衛生行政報告例_看護師数_2023.csv → interim_nurse_staff_2022.parquet

**Last Verified**  
2026-02-14

---

### 准看護師数【人】

**Type**: int64  
**Unit**: 人  
**Grain**: prefecture × fiscal_year  

**Definition**  
就業准看護師数（都道府県別、実数）

**Calculation**  
統計定義に従う（実数）。

**Source**  
厚生労働省（衛生行政報告例）

**Reference Year Policy**  
観測時点は reference_year=2022。

**Transformation**
- 多段ヘッダ処理
- 「全国」除外
- to_numeric
- fiscal_year=2023付与
- reference_year=2022付与
- 必要列射影

**Null Policy**  
NULL不可

**Value Range**  
>= 0

**Quality Checks**
- rows=47
- dup_keys=0
- NULL=0
- negative=0

**Lineage**  
厚労省_衛生行政報告例_看護師数_2023.csv → interim_nurse_staff_2022.parquet

**Last Verified**  
2026-02-14

---

### 保健師数【人口10万対】

**Type**: float64  
**Unit**: 人/人口10万  
**Grain**: prefecture × fiscal_year  

**Definition**  
就業保健師数（人口10万対、都道府県別）

**Calculation**  
原典に記載の「率（人口10万対）」をそのまま保持（ETLで再計算はしない）。

**Source**  
厚生労働省（衛生行政報告例）

**Reference Year Policy**  
観測時点は reference_year=2022。

**Transformation**
- 多段ヘッダ処理
- 「全国」除外（47都道府県）
- to_numeric
- fiscal_year=2023付与
- reference_year=2022付与
- 必要列射影
- 保存前に列をflatten（MultiIndex対策）

**Null Policy**  
NULL不可

**Value Range**  
>= 0

**Quality Checks**
- rows=47
- dup_keys(prefecture,fiscal_year)=0
- NULL=0
- negative=0

**Lineage**  
厚労省_衛生行政報告例_看護師数_2023.csv → interim_nurse_staff_2022.parquet

**Notes**  
人口10万対は原典の算出方法・分母人口（時点）に依存するため、人口データからの再計算とは一致しない場合がある。

**Last Verified**  
2026-02-14

---

### 助産師数【人口10万対】

**Type**: float64  
**Unit**: 人/人口10万  
**Grain**: prefecture × fiscal_year  

**Definition**  
就業助産師数（人口10万対、都道府県別）

**Calculation**  
原典に記載の「率（人口10万対）」をそのまま保持。

**Source**  
厚生労働省（衛生行政報告例）

**Reference Year Policy**  
観測時点は reference_year=2022。

**Transformation**
- 多段ヘッダ処理
- 「全国」除外
- to_numeric
- fiscal_year=2023付与
- reference_year=2022付与
- 必要列射影
- 保存前に列をflatten

**Null Policy**  
NULL不可

**Value Range**  
>= 0

**Quality Checks**
- rows=47
- dup_keys=0
- NULL=0
- negative=0

**Lineage**  
厚労省_衛生行政報告例_看護師数_2023.csv → interim_nurse_staff_2022.parquet

**Notes**  
原典の人口10万対を保持。再計算は行わない。

**Last Verified**  
2026-02-14

---
### 看護師数【人口10万対】

**Type**: float64  
**Unit**: 人/人口10万  
**Grain**: prefecture × fiscal_year  

**Definition**  
就業看護師数（人口10万対、都道府県別）

**Calculation**  
原典に記載の「率（人口10万対）」をそのまま保持。

**Source**  
厚生労働省（衛生行政報告例）

**Reference Year Policy**  
観測時点は reference_year=2022。

**Transformation**
- 多段ヘッダ処理
- 「全国」除外
- to_numeric
- fiscal_year=2023付与
- reference_year=2022付与
- 必要列射影
- 保存前に列をflatten

**Null Policy**  
NULL不可

**Value Range**  
>= 0

**Quality Checks**
- rows=47
- dup_keys=0
- NULL=0
- negative=0

**Lineage**  
厚労省_衛生行政報告例_看護師数_2023.csv → interim_nurse_staff_2022.parquet

**Notes**  
原典の人口10万対を保持。人口データと組み合わせて自前算出する場合は時点差に注意。

**Last Verified**  
2026-02-14

---

### 准看護師数【人口10万対】

**Type**: float64  
**Unit**: 人/人口10万  
**Grain**: prefecture × fiscal_year  

**Definition**  
就業准看護師数（人口10万対、都道府県別）

**Calculation**  
原典に記載の「率（人口10万対）」をそのまま保持。

**Source**  
厚生労働省（衛生行政報告例）

**Reference Year Policy**  
観測時点は reference_year=2022。

**Transformation**
- 多段ヘッダ処理
- 「全国」除外
- to_numeric
- fiscal_year=2023付与
- reference_year=2022付与
- 必要列射影
- 保存前に列をflatten

**Null Policy**  
NULL不可

**Value Range**  
>= 0

**Quality Checks**
- rows=47
- dup_keys=0
- NULL=0
- negative=0

**Lineage**  
厚労省_衛生行政報告例_看護師数_2023.csv → interim_nurse_staff_2022.parquet

**Notes**  
原典の人口10万対を保持。再計算は行わない。

**Last Verified**  
2026-02-14


### turnover_total

Type:
float64

Unit:
%

Grain:
prefecture × fiscal_year

Definition:
常勤看護職員の年間離職率（総数）

Calculation:
年間退職者数 ÷ 平均職員数 × 100

Source:
日本看護協会 病院看護実態調査（都道府県別データ）

Reference Year Policy:
2023データを FY2023 に割当（fiscal_year=2023 を付与）

Transformation:
- prefecture 表記統一（CSV上で47都道府県に整形済み。念のため strip 等）
- %単位（0–100）として保持（CSVで整形済み）
- 全国・計行は含まれない（CSVで除外済み）

Null Policy:
NULL 不可（欠損があればエラーとして扱い、データ源を確認）

Value Range:
0 <= turnover_total <= 100

Quality Checks:
- 値域検証（0–100%）
- 主キー（prefecture, fiscal_year）単位で一意
- 行数が47であること（FY2023想定）

Dependencies:
なし

Lineage:
日本看護協会_離職率_都道府県別_2023.csv → interim_turnover.parquet → master

Notes:
新卒・既卒を含む総数ベース

Last Verified:
2026-02-13


### turnover_new_grad

Type:
float64

Unit:
%

Grain:
prefecture × fiscal_year

Definition:
新卒採用者の年間離職率

Calculation:
年間退職者数 ÷ 平均職員数 × 100（新卒区分）

Source:
日本看護協会 病院看護実態調査（都道府県別データ）

Reference Year Policy:
2023データを FY2023 に割当（fiscal_year=2023 を付与）

Transformation:
- prefecture 表記統一（CSV上で47都道府県に整形済み。念のため strip 等）
- %単位（0–100）として保持（CSVで整形済み）
- 全国・計行は含まれない（CSVで除外済み）

Null Policy:
NULL 不可（欠損があればエラーとして扱い、データ源を確認）

Value Range:
0 <= turnover_new_grad <= 100

Quality Checks:
- 値域検証（0–100%）
- 主キー（prefecture, fiscal_year）単位で一意
- 行数が47であること（FY2023想定）

Dependencies:
なし

Lineage:
日本看護協会_離職率_都道府県別_2023.csv → interim_turnover.parquet → master

Notes:
新卒採用区分

Last Verified:
2026-02-13


### turnover_experienced

Type:
float64

Unit:
%

Grain:
prefecture × fiscal_year

Definition:
既卒採用者の年間離職率

Calculation:
年間退職者数 ÷ 平均職員数 × 100（既卒区分）

Source:
日本看護協会 病院看護実態調査（都道府県別データ）

Reference Year Policy:
2023データを FY2023 に割当（fiscal_year=2023 を付与）

Transformation:
- prefecture 表記統一（CSV上で47都道府県に整形済み。念のため strip 等）
- %単位（0–100）として保持（CSVで整形済み）
- 全国・計行は含まれない（CSVで除外済み）

Null Policy:
NULL 不可（欠損があればエラーとして扱い、データ源を確認）

Value Range:
0 <= turnover_experienced <= 100

Quality Checks:
- 値域検証（0–100%）
- 主キー（prefecture, fiscal_year）単位で一意
- 行数が47であること（FY2023想定）

Dependencies:
なし

Lineage:
日本看護協会_離職率_都道府県別_2023.csv → interim_turnover.parquet → master

Notes:
既卒採用区分

Last Verified:
2026-02-13



## 労働環境（Workload / Night Shift）

### night_shift_72h_plus

Type:
float64

Unit:
%

Grain:
prefecture × fiscal_year

Definition:
月72時間以上の夜勤を行う看護師割合（都道府県別）

Calculation:
統計定義に従う（72時間以上夜勤者の割合）

Source:
日本看護協会（夜勤72h超過率：都道府県別）

Reference Year Policy:
調査結果の参照年を reference_year に保持する（本データは 2024）

Transformation:
- prefecture 表記の統一（CSVで47都道府県に整形済み。念のため strip）
- 数値型へ変換（to_numeric）
- fiscal_year=2023 を付与（master結合キー）
- reference_year=2024 を付与（指標の参照年）

Null Policy:
NULL 不可（欠損があればエラーとして扱い、データ源を確認）

Value Range:
0 <= night_shift_72h_plus <= 100

Quality Checks:
- 値域検証（0–100%）
- 主キー（prefecture, fiscal_year）単位で一意
- 行数が47であること（FY2023想定）

Dependencies:
なし

Lineage:
日本看護協会_夜勤72h超過率_都道府県別_2024.csv → interim_night_shift.parquet → master

Notes:
fiscal_year は master データセットへの結合キーとして付与している。
本指標の時点は reference_year（2024年の調査結果）を参照する。

Last Verified:
2026-02-13

---

### night_shifts_per_month_three_shift

Type:
float64

Unit:
回/月

Grain:
prefecture × fiscal_year

Definition:
3交代制における月あたり夜勤回数（都道府県別）

Calculation:
統計定義に従う（3交代制の夜勤回数）

Source:
日本看護協会（夜勤72h超過率：都道府県別）

Reference Year Policy:
調査結果の参照年を reference_year に保持する（本データは 2024）

Transformation:
- prefecture 表記の統一（CSVで47都道府県に整形済み。念のため strip）
- 数値型へ変換（to_numeric）
- fiscal_year=2023 を付与（master結合キー）
- reference_year=2024 を付与（指標の参照年）

Null Policy:
NULL 不可（欠損があればエラーとして扱い、データ源を確認）

Value Range:
night_shifts_per_month_three_shift >= 0

Quality Checks:
- 非負制約（>=0）
- 主キー（prefecture, fiscal_year）単位で一意
- 行数が47であること（FY2023想定）

Dependencies:
なし

Lineage:
日本看護協会_夜勤72h超過率_都道府県別_2024.csv → interim_night_shift.parquet → master

Notes:
fiscal_year は master データセットへの結合キーとして付与している。
本指標の時点は reference_year（2024年の調査結果）を参照する。

Last Verified:
2026-02-13

---

### night_shifts_per_month_two_shift

Type:
float64

Unit:
回/月

Grain:
prefecture × fiscal_year

Definition:
2交代制における月あたり夜勤回数（都道府県別）

Calculation:
統計定義に従う（2交代制の夜勤回数）

Source:
日本看護協会（夜勤72h超過率：都道府県別）

Reference Year Policy:
調査結果の参照年を reference_year に保持する（本データは 2024）

Transformation:
- prefecture 表記の統一（CSVで47都道府県に整形済み。念のため strip）
- 数値型へ変換（to_numeric）
- fiscal_year=2023 を付与（master結合キー）
- reference_year=2024 を付与（指標の参照年）

Null Policy:
NULL 不可（欠損があればエラーとして扱い、データ源を確認）

Value Range:
night_shifts_per_month_two_shift >= 0

Quality Checks:
- 非負制約（>=0）
- 主キー（prefecture, fiscal_year）単位で一意
- 行数が47であること（FY2023想定）

Dependencies:
なし

Lineage:
日本看護協会_夜勤72h超過率_都道府県別_2024.csv → interim_night_shift.parquet → master

Notes:
fiscal_year は master データセットへの結合キーとして付与している。
本指標の時点は reference_year（2024年の調査結果）を参照する。

Last Verified:
2026-02-13

---

### starting_salary_nurse_diploma_monthly_yen

Type:
float64

Unit:
円/月

Grain:
prefecture × fiscal_year

Definition:
本年度採用の新卒看護師（高卒＋3年課程新卒）の月額「税込給与総額」（都道府県別）

Calculation:
統計定義に従う（新卒・高卒＋3年課程の税込給与総額）。  
※ 税込給与総額は、通勤手当・住宅手当・家族手当・夜勤手当・当直手当・看護職員処遇改善に係る手当等を含み、時間外勤務手当は除く（元資料注記に従う）

Source:
日本看護協会（看護職員の給与：都道府県別）

Reference Year Policy:
調査結果の参照年を reference_year に保持する（本データは 2024）

Transformation:
- prefecture 表記の統一（47都道府県。念のため strip）
- 数値型へ変換（to_numeric）
- fiscal_year=2023 を付与（master結合キー）
- reference_year=2024 を付与（指標の参照年）
- 必要列のみ射影（Projection）

Null Policy:
NULL 不可（欠損があればエラーとして扱い、データ源を確認）

Value Range:
starting_salary_nurse_diploma_monthly_yen >= 0

Quality Checks:
- 非負制約（>=0）
- 主キー（prefecture, fiscal_year）単位で一意
- 行数が47であること（FY2023想定）
- NULL が0であること（キー＋指標）

Dependencies:
なし

Lineage:
日本看護協会_給与_都道府県別_2024.csv → interim_salary.parquet → master

Notes:
fiscal_year は master データセットへの結合キーとして付与している。  
本指標の時点は reference_year（2024年度実績）を参照する。

Last Verified:
2026-02-13

---

### starting_salary_nurse_bachelor_monthly_yen

Type:
float64

Unit:
円/月

Grain:
prefecture × fiscal_year

Definition:
本年度採用の新卒看護師（大卒）の月額「税込給与総額」（都道府県別）

Calculation:
統計定義に従う（新卒・大卒の税込給与総額）。  
※ 税込給与総額は、通勤手当・住宅手当・家族手当・夜勤手当・当直手当・看護職員処遇改善に係る手当等を含み、時間外勤務手当は除く（元資料注記に従う）

Source:
日本看護協会（看護職員の給与：都道府県別）

Reference Year Policy:
調査結果の参照年を reference_year に保持する（本データは 2024）

Transformation:
- prefecture 表記の統一（47都道府県。念のため strip）
- 数値型へ変換（to_numeric）
- fiscal_year=2023 を付与（master結合キー）
- reference_year=2024 を付与（指標の参照年）
- 必要列のみ射影（Projection）

Null Policy:
NULL 不可（欠損があればエラーとして扱い、データ源を確認）

Value Range:
starting_salary_nurse_bachelor_monthly_yen >= 0

Quality Checks:
- 非負制約（>=0）
- 主キー（prefecture, fiscal_year）単位で一意
- 行数が47であること（FY2023想定）
- NULL が0であること（キー＋指標）

Dependencies:
なし

Lineage:
日本看護協会_給与_都道府県別_2024.csv → interim_salary.parquet → master

Notes:
fiscal_year は master データセットへの結合キーとして付与している。  
本指標の時点は reference_year（2024年度実績）を参照する。

Last Verified:
2026-02-13

---

### salary_nurse_10yr_non_manager_monthly_yen

Type:
float64

Unit:
円/月

Grain:
prefecture × fiscal_year

Definition:
勤続10年（31～32歳想定）・非管理職看護師の月額「税込給与総額」（都道府県別）

Calculation:
統計定義に従う（勤続10年・非管理職看護師の税込給与総額）。  
※ 税込給与総額は、通勤手当・住宅手当・家族手当・夜勤手当・当直手当・看護職員処遇改善に係る手当等を含み、時間外勤務手当は除く（元資料注記に従う）

Source:
日本看護協会（看護職員の給与：都道府県別）

Reference Year Policy:
調査結果の参照年を reference_year に保持する（本データは 2024）

Transformation:
- prefecture 表記の統一（47都道府県。念のため strip）
- 数値型へ変換（to_numeric）
- fiscal_year=2023 を付与（master結合キー）
- reference_year=2024 を付与（指標の参照年）
- 必要列のみ射影（Projection）

Null Policy:
NULL 不可（欠損があればエラーとして扱い、データ源を確認）

Value Range:
salary_nurse_10yr_non_manager_monthly_yen >= 0

Quality Checks:
- 非負制約（>=0）
- 主キー（prefecture, fiscal_year）単位で一意
- 行数が47であること（FY2023想定）
- NULL が0であること（キー＋指標）

Dependencies:
なし

Lineage:
日本看護協会_給与_都道府県別_2024.csv → interim_salary.parquet → master

Notes:
fiscal_year は master データセットへの結合キーとして付与している。  
本指標の時点は reference_year（2024年度実績）を参照する。

Last Verified:
2026-02-13

---

### #A011000_総人口【人】

**Type**: int64  
**Unit**: 人  
**Grain**: prefecture × fiscal_year  

**Definition**  
総人口（都道府県別）

**Calculation**  
元データ「総人口【万人】」を万人 × 10,000 で人へ変換。

**Source**  
総務省（社会生活統計指標）

**Reference Year Policy**  
観測年は fiscal_year と同一（2023）。reference_year は保持しない。

**Transformation**
- header行調整（メタ行除外）
- 「全国」除外（47都道府県）
- prefecture作成（地域列→strip）
- カンマ除去
- to_numeric
- 万人 × 10,000
- fiscal_year=2023付与
- 必要列射影

**Null Policy**  
NULL不可

**Value Range**  
>= 0

**Quality Checks**
- rows=47
- dup_keys(prefecture,fiscal_year)=0
- NULL=0
- negative=0

**Lineage**  
総務省_社会生活統計指標_人口密度_2023.csv → interim_population.parquet

**Last Verified**  
2026-02-14
---

### #A01201_総面積１km2当たり人口密度【人】

**Type**: float64  
**Unit**: 人/km^2  
**Grain**: prefecture × fiscal_year  

**Definition**  
総面積1km^2当たり人口密度（都道府県別）

**Calculation**  
統計定義に従う（総面積1km^2当たり人口密度）。

**Source**  
総務省（社会生活統計指標）

**Reference Year Policy**  
観測年は fiscal_year と同一（2023）。reference_year は保持しない。

**Transformation**
- header行調整（メタ行除外）
- 「全国」除外（47都道府県）
- prefecture作成（地域列→strip）
- カンマ除去
- to_numeric
- fiscal_year=2023付与
- 必要列射影

**Null Policy**  
NULL不可

**Value Range**  
>= 0

**Quality Checks**
- rows=47
- dup_keys(prefecture,fiscal_year)=0
- NULL=0
- negative=0

**Dependencies**  
なし

**Lineage**  
総務省_社会生活統計指標_人口密度_2023.csv → interim_population.parquet

**Notes**  
密度は元から【人】単位。単位変換なし。

**Last Verified**  
2026-02-14

---

### 年齢【歳】

**Type**: float64  
**Unit**: 歳  
**Grain**: prefecture × fiscal_year  

**Definition**  
看護師の平均年齢（都道府県別）

**Calculation**  
統計定義に従う。

**Source**  
厚生労働省（賃金構造基本統計調査）

**Reference Year Policy**  
調査年は2024年。  
master結合のため fiscal_year=2023 を付与。reference_year=2024 を保持。

**Transformation**
- header行調整（メタ行除外）
- 「全国」除外（47都道府県）
- prefecture作成（地域列→strip）
- to_numeric
- fiscal_year=2023付与
- reference_year=2024付与
- 必要列射影

**Null Policy**  
NULL不可

**Value Range**  
>= 0

**Quality Checks**
- rows=47
- dup_keys(prefecture,fiscal_year)=0
- NULL=0
- negative=0

**Dependencies**  
なし

**Lineage**  
厚労省_賃金構造基本統計調査_看護師年収_2024.xlsx → interim_nurse_wage_2024.parquet

**Last Verified**  
2026-02-14

---

### 勤続年数【年】

**Type**: float64  
**Unit**: 年  
**Grain**: prefecture × fiscal_year  

**Definition**  
看護師の平均勤続年数（都道府県別）

**Calculation**  
統計定義に従う。

**Source**  
厚生労働省（賃金構造基本統計調査）

**Reference Year Policy**  
調査年は2024年。fiscal_year=2023、reference_year=2024。

**Transformation**
- to_numeric
- fiscal_year=2023付与
- reference_year=2024付与
- 必要列射影

**Null Policy**  
NULL不可

**Value Range**  
>= 0

**Quality Checks**
- rows=47
- dup_keys(prefecture,fiscal_year)=0
- NULL=0
- negative=0

**Lineage**  
厚労省_賃金構造基本統計調査_看護師年収_2024.xlsx → interim_nurse_wage_2024.parquet

**Last Verified**  
2026-02-14

---

### 超過実労働時間数【時間】

**Type**: int64  
**Unit**: 時間  
**Grain**: prefecture × fiscal_year  

**Definition**  
看護師の月間平均超過実労働時間数（都道府県別）

**Calculation**  
統計定義に従う。

**Source**  
厚生労働省（賃金構造基本統計調査）

**Reference Year Policy**  
調査年は2024年。fiscal_year=2023、reference_year=2024。

**Transformation**
- to_numeric
- fiscal_year=2023付与
- reference_year=2024付与
- 必要列射影

**Null Policy**  
NULL不可

**Value Range**  
>= 0

**Quality Checks**
- rows=47
- dup_keys(prefecture,fiscal_year)=0
- NULL=0
- negative=0

**Lineage**  
厚労省_賃金構造基本統計調査_看護師年収_2024.xlsx → interim_nurse_wage_2024.parquet

**Last Verified**  
2026-02-14

---

### きまって支給する現金給与額【円】

**Type**: float64  
**Unit**: 円  
**Grain**: prefecture × fiscal_year  

**Definition**  
看護師の月額現金給与額（都道府県別）

**Calculation**  
元データは【千円】。ETLで ×1,000 し円に変換。

**Source**  
厚生労働省（賃金構造基本統計調査）

**Reference Year Policy**  
調査年は2024年。fiscal_year=2023、reference_year=2024。

**Transformation**
- カンマ除去
- to_numeric
- 千円 × 1,000 → 円へ変換
- fiscal_year=2023付与
- reference_year=2024付与
- 必要列射影

**Null Policy**  
NULL不可

**Value Range**  
>= 0

**Quality Checks**
- rows=47
- dup_keys(prefecture,fiscal_year)=0
- NULL=0
- negative=0

**Lineage**  
厚労省_賃金構造基本統計調査_看護師年収_2024.xlsx → interim_nurse_wage_2024.parquet

**Last Verified**  
2026-02-14

---

### 年間賞与その他特別給与額【円】

**Type**: float64  
**Unit**: 円  
**Grain**: prefecture × fiscal_year  

**Definition**  
看護師の年間賞与その他特別給与額（都道府県別）

**Calculation**  
元データは【千円】。ETLで ×1,000 し円に変換。

**Source**  
厚生労働省（賃金構造基本統計調査）

**Reference Year Policy**  
調査年は2024年。fiscal_year=2023、reference_year=2024。

**Transformation**
- カンマ除去
- to_numeric
- 千円 × 1,000 → 円へ変換
- fiscal_year=2023付与
- reference_year=2024付与
- 必要列射影

**Null Policy**  
NULL不可

**Value Range**  
>= 0

**Quality Checks**
- rows=47
- dup_keys(prefecture,fiscal_year)=0
- NULL=0
- negative=0

**Lineage**  
厚労省_賃金構造基本統計調査_看護師年収_2024.xlsx → interim_nurse_wage_2024.parquet

**Last Verified**  
2026-02-14

---

### 家賃平均【円】

**Type**: Int64  
**Unit**: 円/月  
**Grain**: prefecture × fiscal_year  

**Definition**  
民営借家（専用住宅）の1か月当たり家賃平均（家賃0円を含まない）

**Calculation**  
統計定義に従う（住宅・土地統計調査の平均家賃）。  
元表の「家賃平均（0円を含まない）」列をそのまま使用。

**Source**  
総務省（住宅・土地統計調査）

**Reference Year Policy**  
2023年10月1日現在の調査結果として扱い、reference_year=2023。

**Transformation**
- header行を調整して読み込み
- 地域コードから都道府県名を抽出
- 地域コード末尾000のみ抽出（都道府県限定）
- 「全国」行を除外
- 住宅種類「13_民営借家」のみ抽出
- 数値文字列を数値へ変換（to_numeric）
- fiscal_year=2023 を付与
- reference_year=2023 を付与
- 必要列のみ射影

**Null Policy**  
NULL不可

**Value Range**  
>= 0

**Quality Checks**
- rows=47
- dup_keys(prefecture,fiscal_year)=0
- NULL=0
- negative=0

**Dependencies**  
なし

**Lineage**  
総務省_住宅土地統計調査_家賃_2024.xlsx → interim_rent_2024.parquet

**Notes**  
民営借家に限定することで、居住コストの地域差を比較可能にしている。  
家賃0円を除外した平均を採用し、住宅コスト実態を反映。

**Last Verified**  
2026-02-14

---

### 共益費平均【円】

**Type**: Int64  
**Unit**: 円/月  
**Grain**: prefecture × fiscal_year  

**Definition**  
民営借家（専用住宅）の1か月当たり共益費・管理費平均（0円を含まない）

**Calculation**  
統計定義に従う（住宅・土地統計調査の平均共益費）。  
元表の「共益費平均（0円を含まない）」列をそのまま使用。

**Source**  
総務省（住宅・土地統計調査）

**Reference Year Policy**  
2023年10月1日現在の調査結果として扱い、reference_year=2023。

**Transformation**
- header行を調整して読み込み
- 地域コードから都道府県名を抽出
- 地域コード末尾000のみ抽出（都道府県限定）
- 「全国」行を除外
- 住宅種類「13_民営借家」のみ抽出
- 数値文字列を数値へ変換（to_numeric）
- fiscal_year=2023 を付与
- reference_year=2023 を付与
- 必要列のみ射影

**Null Policy**  
NULL不可

**Value Range**  
>= 0

**Quality Checks**
- rows=47
- dup_keys(prefecture,fiscal_year)=0
- NULL=0
- negative=0

**Dependencies**  
なし

**Lineage**  
総務省_住宅土地統計調査_家賃_2024.xlsx → interim_rent_2024.parquet

**Notes**  
共益費は建物要因の影響を受けやすく、家賃より説明力が弱い可能性がある。  
補助変数として扱うことを想定。

**Last Verified**  
2026-02-14

---

### 主世帯総数

**Type**: Int64  
**Unit**: 世帯  
**Grain**: prefecture × fiscal_year  

**Definition**  
主世帯の総数（都道府県別）

**Calculation**  
原典の「世帯数（1_主世帯）」から、住宅の所有の関係が「0_総数」の行を抽出して保持（再計算なし）。

**Source**  
総務省（住宅・土地統計調査）

**Reference Year Policy**  
2023年10月1日現在の調査結果として扱い、reference_year=2023。

**Transformation**
- header行を調整して読み込み
- 地域区分から都道府県名を抽出（`^\d+_` を除去）
- 地域コード末尾000のみ抽出（都道府県限定）
- 「全国」行を除外
- 住宅の所有の関係「0_総数」を抽出
- 世帯数（1_主世帯）列を数値化（to_numeric）
- fiscal_year=2023 を付与
- reference_year=2023 を付与
- 必要列のみ射影
- 保存前に列をflatten（MultiIndex対策が必要な場合）

**Null Policy**  
NULL不可

**Value Range**  
>= 0

**Quality Checks**
- rows=47
- dup_keys(prefecture,fiscal_year)=0
- NULL=0
- negative=0

**Lineage**  
総務省_住宅土地統計調査_持ち家率_2024.xlsx → interim_home_2023.parquet

**Notes**  
「住宅数」ではなく「世帯数（主世帯）」を採用し、居住実態（生活主体）に近い母集団を表す。

**Last Verified**  
2026-02-14

---

### 持ち家主世帯数

**Type**: Int64  
**Unit**: 世帯  
**Grain**: prefecture × fiscal_year  

**Definition**  
持ち家の主世帯数（都道府県別）

**Calculation**  
原典の「世帯数（1_主世帯）」から、住宅の所有の関係が「1_持ち家」の行を抽出して保持（再計算なし）。

**Source**  
総務省（住宅・土地統計調査）

**Reference Year Policy**  
2023年10月1日現在の調査結果として扱い、reference_year=2023。

**Transformation**
- header行を調整して読み込み
- 地域区分から都道府県名を抽出（`^\d+_` を除去）
- 地域コード末尾000のみ抽出（都道府県限定）
- 「全国」行を除外
- 住宅の所有の関係「1_持ち家」を抽出
- 世帯数（1_主世帯）列を数値化（to_numeric）
- fiscal_year=2023 を付与
- reference_year=2023 を付与
- 必要列のみ射影
- 保存前に列をflatten（MultiIndex対策が必要な場合）

**Null Policy**  
NULL不可

**Value Range**  
>= 0

**Quality Checks**
- rows=47
- dup_keys(prefecture,fiscal_year)=0
- NULL=0
- negative=0
- owned > total が0（論理チェック）

**Lineage**  
総務省_住宅土地統計調査_持ち家率_2024.xlsx → interim_home_2023.parquet

**Notes**  
持ち家率は master/EDA で `持ち家主世帯数 / 主世帯総数` として算出する（interimでは分子分母を保持）。

**Last Verified**  
2026-02-14

---

## commute_time_min

**Type**: Int64  
**Unit**: 分  
**Grain**: prefecture × fiscal_year  

**Definition**  
1日の平均通勤・通学時間（都道府県別）

**Calculation**  
原典の「時間.分」表記を分単位へ変換して保持（再計算なし）。  
小数1桁表記は10分単位として扱う（例：1.4 → 1時間40分 → 100分）。

**Source**  
総務省「社会生活基本調査（令和3年）」都道府県ランキング表

**Reference Year Policy**  
令和3年調査（2021年実施）の結果として扱い、  
reference_year=2022 を保持し、結合キーは fiscal_year=2023 に統一。

**Transformation**
- 多段ヘッダExcelから対象指標列のみ抽出（通勤・通学時間）
- 都道府県名列を抽出し前後空白を除去
- 「全国平均」行を除外
- ヘッダ混入行（「都道府県名」）を除外
- 「時間.分」形式を分へ変換
- 小数1桁は10分単位として補正
- 数値列を to_numeric で変換
- fiscal_year=2023 を付与
- reference_year=2022 を付与
- 必要列のみ射影

**Null Policy**  
NULL不可

**Value Range**  
>= 0

**Quality Checks**
- rows=47
- dup_keys(prefecture,fiscal_year)=0
- NULL=0
- negative=0

**Lineage**  
総務省_社会生活基本調査_通勤時間_2022.xlsx → interim_commute_time.parquet

**Notes**  
ランキング形式のため並び順は指標値順であり、都道府県順ではない。  
master結合時は順序に依存しないこと。

**Last Verified**  
2026-02-14

---

## job_openings_ratio

**Type**: Float64  
**Unit**: 倍  
**Grain**: prefecture × fiscal_year  

**Definition**  
有効求人倍率（都道府県別）

**Calculation**  
原典「有効求人倍率（実数）」の **2023年度** 行を抽出し、都道府県列を縦持ちに変換して保持（再計算なし）。

**Source**  
厚生労働省「一般職業紹介状況」  
シート：第１８表ー４　有効求人倍率（実数）

**Reference Year Policy**  
2023年度（2023/4〜2024/3）の年度値として扱い、reference_year=2023。  
結合キーは fiscal_year=2023 に統一。

**Transformation**
- Excel読み込み時に `header=1`（2行目をヘッダとして採用）
- 単位行（各都道府県列が「倍」）を除外（`西暦` が NaN の行を除外）
- `西暦` 列から「年度」行のみ抽出（"年度" を含む）
- 「2023年度」のみ抽出し、都道府県列（都/道/府/県で終わる列）を対象化
- 都道府県列を `to_numeric(errors="coerce")` で数値化
- 横持ち（1行×47列）→ 縦持ち（47行×1列）へ変換
- fiscal_year=2023 を付与
- reference_year=2023 を付与
- 必要列のみ射影

**Null Policy**  
NULL不可

**Value Range**  
>= 0

**Quality Checks**
- rows=47
- dup_keys(prefecture,fiscal_year)=0
- NULL=0
- negative=0

**Lineage**  
厚労省_一般職業紹介状況_有効求人倍率_2025.xlsx → interim_job_openings_ratio.parquet

**Notes**
- 本シートには年次（〜年）と年度（〜年度）が混在するため、FY目的の場合は「年度」行を使用する。  
- 「倍率」は単位が「倍」のため、他の件数系データと混同しない。

**Last Verified**  
2026-02-14

---

## 病院数【実数】

**Type**: Int64  
**Unit**: 施設  
**Grain**: prefecture × fiscal_year  

**Definition**  
病院数（都道府県別）

**Calculation**  
原典「病院数」から、2023年（令和5年）の実数列を抽出して保持（再計算なし）。

**Source**  
厚生労働省「医療施設調査（静態・動態）」  
第1表 病院数，年次・都道府県別

**Reference Year Policy**  
2023年10月1日現在の静態調査値として扱い、reference_year=2023。  
結合キーは fiscal_year=2023 に統一。

**Transformation**
- 表内メタ行を除外してデータ開始行から抽出
- 都道府県列（都道府県_005）を使用
- 「全国」行を除外
- 都道府県名の全角スペースを除去
- 都道府県接尾辞（県など）を補完
- 数値列のカンマを除去し `to_numeric`
- fiscal_year=2023 を付与
- reference_year=2023 を付与
- 必要列のみ射影

**Null Policy**  
NULL不可

**Value Range**  
>= 0

**Quality Checks**
- rows=47
- dup_keys(prefecture,fiscal_year)=0
- NULL=0
- negative=0

**Lineage**  
医療施設調査_T1_病院数.csv → interim_hospital_count_2023.parquet

**Notes**
- 「病院」は医療法上の定義（20床以上の入院施設）に基づく。  
- 診療所数とは別指標。  
- 時系列列が複数存在するため、対象年を明示して抽出する。

**Last Verified**  
2026-02-14

---

## 病院数【人口10万対】

**Type**: Float64  
**Unit**: 施設／人口10万  
**Grain**: prefecture × fiscal_year  

**Definition**  
人口10万人当たりの病院数（都道府県別）

**Calculation**  
原典の「令和5年(2023年)（人口10万対）」列をそのまま保持（再計算なし）。

**Source**  
厚生労働省「医療施設調査（静態・動態）」  
第1表 病院数，年次・都道府県別

**Reference Year Policy**  
2023年静態調査値として扱い、reference_year=2023。  
結合キーは fiscal_year=2023 に統一。

**Transformation**
- 実数列と同じ都道府県抽出ロジックを使用
- 「全国」行を除外
- 都道府県名正規化（スペース除去＋接尾辞補完）
- 数値列を `to_numeric`
- fiscal_year=2023 を付与
- reference_year=2023 を付与
- 必要列のみ射影

**Null Policy**  
NULL不可

**Value Range**  
>= 0

**Quality Checks**
- rows=47
- dup_keys(prefecture,fiscal_year)=0
- NULL=0
- negative=0

**Lineage**  
医療施設調査_T1_病院数.csv → interim_hospital_count_2023.parquet

**Notes**
- 分母人口は調査側で使用された推計人口に基づく。  
- masterでは再計算せず、この値をそのまま使用する。

**Last Verified**  
2026-02-14

---

## ### total_hospitals_all

**Type**: Int64  
**Unit**: 病院（施設数）  
**Grain**: prefecture × fiscal_year  

**Definition**  
都道府県別の病院数（病床規模「総数」= bed_scale_code=1 に対応する施設数）。  
※「病床数」ではなく「病床規模区分における病院数」。

**Calculation**  
医療施設調査T8の  
- bed_scale_code=1（総数）  
- 列「総数」  
をそのまま保持（再計算なし）。

**Source**  
厚生労働省「医療施設調査（静態・動態）」  
第8表「病院数，病院－病床の種類・病床の規模・都道府県－指定都市・特別区・中核市（再掲）別」  
（令和5年=2023年）

**Reference Year Policy**  
2023年静態調査値として扱い、reference_year=2023。  
結合キーは fiscal_year=2023 に統一。

**Transformation**
- 表頭行（"表章項目 コード"）を起点に skiprows で読み込み（メタ行回避）
- 必要列のみ射影（地域列、病床規模コード、総数、一般病院（総数））
- 「全国」行を除外
- 「（再掲）」を含む行を除外（指定都市・特別区・中核市混入防止）
- prefecture_raw を保持し、prefecture を prefecture_raw からのみ生成
- 空白除去 → 東京/大阪/京都例外処理 → 接尾辞補完
- 病床規模コードを数値化し 1〜15 に制限
- 数値列サニタイズ（カンマ除去、"-" は該当なしとして 0 に変換、Int化）
- bed_scale_code=1 を抽出し、列「総数」を total_hospitals_all にリネーム
- fiscal_year=2023 / reference_year=2023 を付与

**Null Policy**  
NULL不可（原典の "-" は該当なしとして 0 に変換）

**Value Range**  
>= 0

**Quality Checks**
- rows=47
- dup_keys(prefecture,fiscal_year)=0
- NULL=0
- negative=0
- prefecture が47都道府県固定リストと一致

**Dependencies**
- prefecture 正規化ロジック
- bed_scale_code 整形処理

**Lineage**  
厚労省_医療施設調査_病床規模_2024.csv → interim_hospitals_summary_2023.parquet

**Notes**
- この指標は「病床数」ではなく「病院数」。
- 再掲行を落とさないと 47 を超えて master 結合事故が起きる。

**Last Verified**  
2026-02-14

---

## ### general_hospitals_all

**Type**: Int64  
**Unit**: 病院（施設数）  
**Grain**: prefecture × fiscal_year  

**Definition**  
都道府県別の一般病院数（病床規模「総数」= bed_scale_code=1 における一般病院施設数）。

**Calculation**  
医療施設調査T8の  
- bed_scale_code=1（総数）  
- 列「一般病院（総数）」  
をそのまま保持（再計算なし）。

**Source**  
厚生労働省「医療施設調査（静態・動態）」  
第8表（令和5年=2023年）

**Reference Year Policy**  
2023年静態調査値として扱い、reference_year=2023。  
結合キーは fiscal_year=2023 に統一。

**Transformation**
- total_hospitals_all と同一の抽出・正規化ロジックを使用
- 数値列サニタイズ（カンマ除去、"-" は 0 に変換、Int化）
- bed_scale_code=1 を抽出し、列「一般病院（総数）」を general_hospitals_all にリネーム
- fiscal_year=2023 / reference_year=2023 を付与

**Null Policy**  
NULL不可（該当なしは 0）

**Value Range**  
>= 0  
<= total_hospitals_all

**Quality Checks**
- rows=47
- dup_keys(prefecture,fiscal_year)=0
- NULL=0
- negative=0
- general_hospitals_all <= total_hospitals_all

**Dependencies**
- prefecture 正規化ロジック
- bed_scale_code 整形処理

**Lineage**  
厚労省_医療施設調査_病床規模_2024.csv → interim_hospitals_summary_2023.parquet

**Notes**
- 一般病院以外（精神科単科など）は含まれない。
- masterでは total に対する割合指標として利用可能。

**Last Verified**  
2026-02-14

---

## ### hospitals_500plus

**Type**: Int64  
**Unit**: 病院（施設数）  
**Grain**: prefecture × fiscal_year  

**Definition**  
都道府県別の「500床以上」の病院数。  
病床規模コード 11〜15 を合算した施設数。

**Calculation**  
医療施設調査T8の  
- bed_scale_code ∈ [11,12,13,14,15]  
- 列「総数」  
を都道府県単位で合算。

**Source**  
厚生労働省「医療施設調査（静態・動態）」  
第8表（令和5年=2023年）

**Reference Year Policy**  
2023年静態調査値として扱い、reference_year=2023。  
結合キーは fiscal_year=2023 に統一。

**Transformation**
- prefecture 正規化ロジックを適用
- bed_scale_code を数値化し 1〜15 に制限
- 数値列サニタイズ（カンマ除去、"-" は該当なしとして 0 に変換、Int化）
- bed_scale_code 11〜15 を抽出
- 都道府県単位で「総数」を合算
- fiscal_year=2023 / reference_year=2023 を付与

**Null Policy**  
NULL不可（該当なしは 0）

**Value Range**  
>= 0  
<= total_hospitals_all

**Quality Checks**
- rows=47
- dup_keys(prefecture,fiscal_year)=0
- NULL=0
- negative=0
- hospitals_500plus <= total_hospitals_all

**Dependencies**
- prefecture 正規化ロジック
- bed_scale_code 整形処理

**Lineage**  
厚労省_医療施設調査_病床規模_2024.csv → interim_hospitals_large_scale_2023.parquet

**Notes**
- 法的な区分ではなく分析用の閾値。
- masterでは「大規模病院割合」などの派生指標に使用。

**Last Verified**  
2026-02-14

---

## ### hospitals_700plus

**Type**: Int64  
**Unit**: 病院（施設数）  
**Grain**: prefecture × fiscal_year  

**Definition**  
都道府県別の「700床以上」の病院数。  
病床規模コード 13〜15 を合算した施設数。

**Calculation**  
医療施設調査T8の  
- bed_scale_code ∈ [13,14,15]  
- 列「総数」  
を都道府県単位で合算。

**Source**  
厚生労働省「医療施設調査（静態・動態）」  
第8表（令和5年=2023年）

**Reference Year Policy**  
2023年静態調査値として扱い、reference_year=2023。  
結合キーは fiscal_year=2023 に統一。

**Transformation**
- hospitals_500plus と同一の整形ロジックを使用
- bed_scale_code 13〜15 を抽出
- 都道府県単位で「総数」を合算
- fiscal_year=2023 / reference_year=2023 を付与

**Null Policy**  
NULL不可（該当なしは 0）

**Value Range**  
>= 0  
<= hospitals_500plus

**Quality Checks**
- rows=47
- dup_keys(prefecture,fiscal_year)=0
- NULL=0
- negative=0
- hospitals_700plus <= hospitals_500plus

**Dependencies**
- prefecture 正規化ロジック
- bed_scale_code 整形処理

**Lineage**  
厚労省_医療施設調査_病床規模_2024.csv → interim_hospitals_large_scale_2023.parquet

**Notes**
- 500床以上の部分集合であるため、必ず単調性を満たす。
- 高度急性期医療機能の近似指標として利用可能。

**Last Verified**  
2026-02-14

---

## ### hospitals_900plus

**Type**: Int64  
**Unit**: 病院（施設数）  
**Grain**: prefecture × fiscal_year  

**Definition**  
都道府県別の「900床以上」の病院数。  
病床規模コード 15 の施設数。

**Calculation**  
医療施設調査T8の  
- bed_scale_code = 15  
- 列「総数」  
をそのまま保持（都道府県単位）。

**Source**  
厚生労働省「医療施設調査（静態・動態）」  
第8表（令和5年=2023年）

**Reference Year Policy**  
2023年静態調査値として扱い、reference_year=2023。  
結合キーは fiscal_year=2023 に統一。

**Transformation**
- hospitals_500plus と同一の整形ロジックを使用
- bed_scale_code=15 を抽出
- fiscal_year=2023 / reference_year=2023 を付与

**Null Policy**  
NULL不可（該当なしは 0）

**Value Range**  
>= 0  
<= hospitals_700plus

**Quality Checks**
- rows=47
- dup_keys(prefecture,fiscal_year)=0
- NULL=0
- negative=0
- hospitals_900plus <= hospitals_700plus

**Dependencies**
- prefecture 正規化ロジック
- bed_scale_code 整形処理

**Lineage**  
厚労省_医療施設調査_病床規模_2024.csv → interim_hospitals_large_scale_2023.parquet

**Notes**
- 超大規模病院の集中度を示す指標。
- masterでは「>=700内の比率」などに利用可能。

**Last Verified**  
2026-02-14

---





5. 運用ルール (Operational Rules)
新しい列を追加する際の手順
テンプレートのコピー
上記「3. カラム定義テンプレート」のコードブロックをコピーする。

該当セクションへの配置
カテゴリ（目的変数、特徴量など）に合わせて配置する。

仕様の記述
空欄を埋め、ETL実装前に仕様を確定させる。

バージョン管理
ドキュメントの更新
カラムの定義変更があった場合は、必ず本ドキュメントを更新する。

コミットログ
Gitのコミットログに変更理由を明記すること。

例: docs: update definition of turnover_total