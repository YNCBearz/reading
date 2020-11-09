# CASE陳述式的建議

## 語法

```sql
-- 單純CASE陳述式
CASE sex
  WHEN '1' THEN '男'
  WHEN '2' THEN '女'
ELSE '其他' END

-- 搜尋CASE陳述式
CASE WHEN sex = '1' THEN '男'
     WHEN sex = '2' THEN '女'
ELSE '其他' END

```

注意事項：
1. 各分岐條件傳回的資料類型必須一致
2. 別忘了加上END
3. 一定要撰寫ELSE陳述句

---

## 將現有的代碼體系轉換成新體系再彙整

主要是SELECT時搭配AS來讓程式碼易讀，
缺點在於只有**PostgreSQL**和**MySQL**能這樣做。

---

## 以單一SQL完成不同條件的彙整

```sql
-- 男性的人口
SELECT pref_name,
       population,
FROM PopTb12
WHERE sex = '1';

-- 女性的人口
SELECT pref_name,
       population,
FROM PopTb12
WHERE sex = '2';
```

```sql
SELECT pref_name,
    -- 男性的人口
    SUM ( CASE WHEN sex = '1' THEN population ELSE 0 END ) AS cnt_m,
    -- 女性的人口
    SUM ( CASE WHEN sex = 'm' THEN population ELSE 0 END ) AS cnt_f,
FROM PopTb12
GROUP BY pref_name
```

> 以WHERE陳述句撰寫條件分歧是外行人，專家都是以SELECT敘述撰寫條件分歧

---

## 以CHECK限制定義多個欄位的條件關係

MySQL 8.0 未支援CHECK限制。

---

## 讓條件產生分歧的UPDATE

利用下列條件更新薪水：
1. 薪水30萬以上要減薪10%
2. 薪水25萬到28萬的要加薪20%

錯誤示範
```sql
-- 條件1
UPDATE Personnel
SET salary = salary * 0.9
WHERE salary >= 300000;

-- 條件2
UPDATE Personnel
SET salary = salary * 1.2
WHERE salary >= 250000 AND salary < 280000;

```


正確的下法
```sql
UPDATE Personnel
SET sakary = CASE WHEN salary >= 300000
                  THEN salary * 0.9
                  WHEN salary >= 250000 AND salary < 280000
                  THEN salary * 1.2
            ELSE salary END;
```

也可用在換主key的情況（不適用於MySQL）

---

## 資料表的比對

---

## 在CASE陳述式使用彙總函數

根據下列條件查詢：
1. 針對參加單一社團的學生，取得社團ID
2. 針對參加多個社團的學生，取得主要社團ID

```sql
-- 條件一
SELECT std_id, MAX(club_id) AS main_club,
FROM StudentClub
GROUP BY std_id
HAVING COUNT(*) = 1;

-- 條件二
SELECT std_id, club_id AS main_club,
FROM StudentClub
WHERE main_club_flg = 'Y';
```

```sql
SELECT std_id,
CASE WHEN COUNT(*) = 1 -- 學生只參加一個社團的情況
     THEN MAX(club_id)
     ELSE MAX(CASE WHEN main_club_flg = 'Y'
                   THEN club_id
              ELSE NULL END) END AS main_club
FROM StudentClub
GROUP BY std_id;
```

> 利用HAVING撰寫條件分歧的是外行人，利用SELECT陳述句撰寫是專家

---

## 總結

> CASE陳述式通常可寫在欄位名稱或常數的位置

可在任何位置撰寫
- SELECT陳述句
- WHERE陳述句
- GROUP BY陳述句
- HAVING子句
- ORDER BY陳述句
- PARTITION BY陳述句
- CHECK限制之中
- 函數的參數
- 述詞的參數
- 其他算式之中（包含CASE陳述式本身）

### 複習本章重點：
1. 於GROUP BY陳述句使用CASE陳述式可靈活設定作為彙整單位使用的代碼或階層，而這項優點可在非典型的彙整作業發揮極大的優勢。
2. 在彙總函數之中使用CASE陳述式可隨意讓列資料展開成欄資料。
3. 將彙總函數嵌入條件式可不使用HAVING子句整合查詢的部分。
4. 比起規格相依的函數，CASE陳述式的應用範圍與相容性都很高，可說是一石二鳥。
5. 之所以有這些優點，在於CASE陳述式是「算式」而非「陳述句」。
6. CASE陳述式可將多個SQL陳述句整合成單一SQL陳述句，同時提升易讀性與效能。



