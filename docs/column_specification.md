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