# SQL的集合運算

## 前言

聯集 (UNION)是早於SQL-86採用的運算子。
交集 (INTERSECT)與差集 (EXCEPT)則是等到SQL-92才正式採用，
除法 (DIVIDE BY)也尚未標準化。

不過，現在的標準SQL總算備有齊全的集合運算子。

ʕ •ᴥ•ʔ：MySQL沒有交集 (INTERSECT)與差集 (EXCEPT)、也沒有除法 (DIVIDE BY)運算子。

---

## 採用 - 關於集合運算的幾個注意事項
顧名思義，SQL的集合運算子就是以「集合」為輸入值的運算。

**[注意事項1]：**

集合論的集合通常不允許重複元素的存在，不過關聯式資料庫的資料表是允許重複列出現的多重集合 (multiset, bag)。

若依習慣使用UNION或INTERSECT，就會從**結果排除重複列**。
想**保留重複列，就必須加上ALL選項**，寫成UNION ALL的語法。

這與SELECT陳述句的DISTINCT選項剛好背道而馳，
但令人不解的是不能寫成「UNION DISTINCT」。

兩種寫法除結果外，不允許重複列的集合運算子會自動排序。
但在**加上ALL選項後，不會自動排序**，所以有些許提昇效能。

ʕ •ᴥ•ʔ：尼馬提昇效能，上禮拜我很開心的在GUI用UNION ALL，然後直接slow query..(6分鐘)

同事表示：只有菜雞才會選擇用UNION。

**[注意事項2]：**

以標準SQL而言，INTERSECT的順序優於UNION與EXCEPT，
所以需要同時使用UNION與INTERSECT，卻又讓UNION優先時，
必須以括號指定運算順序。

**[注意事項3]：**

SQL Server從2005版之後，開始支援INTERSECT與EXCEPT。

MySQL直到2018年仍未支援這兩種運算子，
也有Oralce這種將EXCEPT稱為MINUS的DBMS。

**[注意事項4]：**

四則運算的聯集 (UNION)、差集 (EXCEPT)、乘積 (CROSS JOIN)都已成為標準規格。

只有商 (DIVIDE BY)仍因各種緣故，遲遲未標準化。

ʕ •ᴥ•ʔ：不好意思，MySQL還是沒有差集 (EXCEPT)。

---

## 資料表之間的比對 - 確認集合的相等性 [基本篇]

這裡的「相等」指的是列數、欄數、資料表內容都相等，也就是完全相同的兩個集合。

```sql
select count(*) as row_cnt from (Select * from tbl_A union select * from tbl_B) TMP;
```

UNION會排除重複列。

> S UNION S = S

UNION在數學上稱為「冪等性」(idempotency)的性質。
這是群論抽象代數所使用的概念，定義也非常多種。
與這次有關的意義為**二項運算子＊的任意輸入值S符合S＊S = S的定義**。

在程式設計的領域，這個定義被稍微擴張成「即使重複執行相同處理，其結果與只執行一次處理的結果相同」。

> S UNION S UNION S ... UNION S = S

UNION ALL每執行一次，結果就會改變，所以不具備冪等性。

如此美麗又強力的性質，僅止於集合的世界。
在多重集合的世界是無法成立的。

**主key是非常重要的。**

---

## 資料表之間的比對 - 確認集合的相等性 [應用篇]

集合論有兩個著名的公式，可以用來驗證集合的相等性。

1. (A ⊆ B) 且 (B ⊇ A) ↔ (A = B)
2. (A ∪ B) = (A ∩ B) ↔ (A = B)

除了UNION之外，INTERSECT也具有冪等性的性質。

思考 (A UNION B) EXCEPT (A INTERSECT B) 的結果，
是否為空集合，來判斷A與B是否相等。

```sql
SELECT CASE WHEN COUNT(*) = 0
            THEN '相等'
            ELSE '不相等' END AS result
    FROM ((SELECT * from tbl_A
           UNION
           SELECT * from tbl_B)
           EXCEPT
           (SELECT * from tbl_A
           INTERSECT
           SELECT * FROM tbl_B)) TMP;
```

ʕ •ᴥ•ʔ：MySQL不支援INTERSECT與EXCEPT語法，這裡用想像的。

這個查詢不需要取得欄位名稱與數量，也能用來處理包含NULL的資料表，
更棒的是，還不需要事先取得資料表的列數。
但是會執行三次集合運算的排序，導致效能不彰。

```sql
(SELECT * FROM tbl_A
 EXCEPT
 SELECT * FROM tbl_B)
UNION ALL
(SELECT * FROM tbl_B
 EXCEPT
 SELECT * FROM tbl_A);
```

由於A-B與B-A沒有相同的部分，所以用UNION ALL也無妨。

---

## 以差集呈現關聯式除法

SQL尚未有除法專用的運算子，較具代表性的做法有：

1. 將NOT EXISTS寫成巢狀結構
2. 使用HAVING陳述句打造一對一關聯性
3. 以減法代替除法

這邊介紹第三種方法。

```sql
SELECT DISTINCT emp
FROM emp_skills ES1
WHERE NOT EXISTS
        (SELECT skill
         FROM skills
         EXCEPT
         SELECT skill
         FROM emp_skills ES2
         WHERE es1.emp = es2.emp);
```

「原來如此，若只需要除法 (division) ， 就可以利用分割 (divide) 來解題」

---

解法二 (Zero 提供)

```sql
SELECT
	emp
FROM
	emp_skills
INNER JOIN skills ON
	skills.skill = emp_skills.skill
GROUP BY
	emp
HAVING
	count(*) >= (select count(*) from skills);
```

---

## 找出相等的部分集合

```sql
SELECT SP1.sup AS s1, SP2.sup AS s2
FROM sup_parts SP1, sup_parts SP2
WHERE SP1.sup < SP2.sup
GROUP BY SP1.sup, SP2.sup;
```

- 條件1: 兩邊業者都銷售種類相同的零件
- 條件2: 銷售的零件數量相同（意即對射的關係存在）

```sql
SELECT SP1.sup AS s1, SP2.sup AS s2
From sup_parts SP1, sup_parts SP2
WHERE SP1.sup < SP2.sup
AND SP1.part = SP2.part
GROUP BY SP1.sup, SP2.sup
HAVING COUNT(*) = (SELECT COUNT(*)
                   FROM sup_parts SP3
                   WHERE SP3.sup = SP1.sup)
AND COUNT(*) = (SELECT COUNT(*)
                FROM sup_parts SP4
                WHERE SP4.sup = SP2.sup)
```

Having陳述句的兩個條件若想成除法，或許比較容易想像。

傳說的述詞 CONTAINS

```sql
SELECT 'A' CONTAINS 'B'
FROM sup_parts
WHERE (SELECT part
       FROM sup_parts
       WHERE sup = 'A')
        CONTAINS
         (SELECT part
          FROM sup_parts
          WHERE sup = 'B')
```

---

## 刪除重複列的高速查詢

先前的做法

```sql
DELETE FROM products
WHERE rowid < (SELECT MAX(P2.rowid)
               FROM products P2
               WHERE products.name = P2.name
               AND products.price = P2.price);
```

這個程式不差，但關聯子查詢還是會有效能上的問題。

```sql
DELETE FROM products
WHERE rowid IN (SELECT rowid
                FROM products
                EXCEPT
                SELECT MAX(rowid)
                FROM products
                GROUP BY name, price);
```

```sql
DELETE FROM products
WHERE rowid NOT IN (SELECT MAX(rowid)
                    FROM products
                    GROUP BY name, price);
```

---

## 總結
1. SQL的集合運算功能太晚建立，導致各家DB的支援程度不一，使用時須多加注意。
2. 未附加ALL選項的集合運算子會排除重複資料，此時也會進行排序，所以有損效能。
3. UNION與INTERSECT都具備冪等性這項重要性質，EXCEPT則沒有這項性質。
4. 由於目前沒有專為除法設計的運算子，所以必須自行撰寫。
5. 要知道集合是否相等，可透過冪等性或對射這兩種方法。
6. EXCEPT可輕鬆呈現補集合。


