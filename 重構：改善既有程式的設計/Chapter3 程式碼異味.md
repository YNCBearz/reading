# **Chapter3 程式碼異味**

### 接下來會討論的主題：

- Mysterious Name （神秘的名稱）
- Long Parameter List （冗長的參數列）
- Lazy Element （冗員元素）
- Speculative Generality （畫大餅）
- Temporary Field （暫時欄位）
- Insider Trading （內幕交易）
- Large Class （龐大的類別）
- Alternative Classes with Different Interfaces （異曲同工的類別）
- Data Class （呆板的資料類別）
- Refused Bequest （被拒絕的遺產）
- Comments （過多的註解）

----------------------------------------------

## **Mysterious Name （神秘的名稱）**

> **命名是程式設計的兩大難事之一（另一個是緩存失效）。**

  1. *Change Function Declaration*
  2. *Rename Variable*
  3. *Rename Field*
      - 資料欄位是理解來龍去脈的關鍵，效果比流程圖更好。

----------------------------------------------

## **Long Parameter List （冗長的參數列）**

> **全域資料是邪惡的東西，但是冗長的參數列往往也會讓人一頭霧水。**

  1. *Replace Parameter with Query*
     - 若可以用一個參數來決定另一個參數，則不須同時傳入這兩個參數。
     - 要留意**引用透明性 (referential transparency)**，即用同樣的參數值呼叫時，應具有相同的行為。
  2. *Preserve Whole Object*
  3. *Introduce Parameter Object*
  4. *Remove Flag Argument*
        ```
        function delivery(anOrder, isRush) {
            if (isRush) {
                ...
            } else {
                ...
            }
        }
        ```
        法ㄧ：先將程式碼抽出、改寫呼叫方，再把原方法刪除
        ```
        function delivery(anOrder, isRush) {
            if (isRush) {
                rushDelivery(anOrder);
            } else {
                normalDelivery(anOrder);
            }
        }
        ```
        法二：創造新的接口去呼叫
        ```
        function rushDelivery(anOrder) {
            delivery(anOrder, true);
        }

        function normalDelivery(anOrder) {
            delivery(anOrder, false);
        }
        ```
  5. *Combine Functions into Class*
----------------------------------------------

## **Lazy Element （冗員元素）**

> **為了將來可以進行變動與重複使用程式，我們常使用程式元素來加入結構，它可能是個名字看起來與內文程式碼一樣的函式。**

  1. *Inline Function*
  2. *Inline Class*
  3. *Collapse Hierarchy*

----------------------------------------------

## Speculative Generality （畫大餅）

> **噢，我認為我們總有一天需要做這件事。**

  1. *Collapse Hierarchy*
  2. *Inline Function*
  3. *Inline Class*
  4. *Change Function Declaration*
  5. *Remove Dead Code*

----------------------------------------------

## Temporary Field （暫時欄位）

> **一般認為，物件的每個欄位在任何情況下都會用到，不該存在看起來沒有作用的欄位。**

  1. *Extract Class*
  2. *Move Function*
  3. *Introduce Special Case*

----------------------------------------------

## Insider Trading （內幕交易）

> **如果模組有共同興趣，試著建立第三個模組，減少它們對談的需求。**

  1. *Move Function*
  2. *Move Field*
  3. *Hide Delegate*
  4. *Replace Subclass with Delegate*
        - 如果可以使用物件組合，就不要使用繼承
        - 即策略模式 (Strategy Pattern)
  5. *Replace Superclass with Delegate*

----------------------------------------------

## Large Class （龐大的類別）

> **當類別有太多欄位時，重複的程式也會隨之不斷發生。**

  1. *Extract Class*
  2. *Extract Superclass*
  3. *Replace Type Code with Subclasses* （即工廠模式）

----------------------------------------------

## Alternative Classes with Different Interfaces （異曲同工的類別）

> **類別最大的好處之一，就是你可以在必要時改用另一個類別，但前提是它們必須有相同的介面。**

  1. *Change Function Declaration*
  2. *Move Function*
  3. *Extract Subclass*

----------------------------------------------

## Data Class （呆板的資料類別）

> **只有欄位跟getter的類別。**

  1. *Encapsulate Record*
  2. *Remove Setting Method*
  3. *Move Function*
  4. *Extract Function*
  5. *Split Phase*

----------------------------------------------

## Refused Bequest （被拒絕的遺產）

> **子類別可以繼承其父類別的方法與資料，但如果它們不想要或不需要呢？**

  1. *Push Down Method*
  2. *Push Down Field*
  3. *Replace Subclass with Delegate*
  4. *Replace SuperClass with Delegate*

----------------------------------------------

## Comments （過多的註解）

> **註解不是種臭味，而是甜美的味道。但卻經常被當成除臭劑使用。**

  1. *Extract Function*
  2. *Change Function Declaration*
  3. *Introduce Assertion*