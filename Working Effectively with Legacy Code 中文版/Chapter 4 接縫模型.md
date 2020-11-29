## Chapter 4
# The Seam Model（接縫模型）

欲得到**易於測試**的程式碼，普遍只有兩種方式：
1.  邊開發邊編寫測試
2.  將易測試性納入整體的設計考量

人們對第一種方式期望甚高。

而以業界現存的大量程式碼來說，第二種方式並沒有取得成功。


## 4.1 一大段文字

> 程式對當時的我來說，就是一張清單。每隔一兩個小時的我都要從實驗室走到影印室，將程式列印出來並仔細檢查，試圖找出哪些地方對了，哪些地方錯了。 [name=Michael C.Feathers]

---

## 4.2 接縫

當你試圖將單一類別從系統中抽離，以便進行單元測試時，

常常需要做一系列的**解依賴**工作。

```c++
bool CAsyncSslRec::Init()
{
    if (m_bSslInitialized) {
        return true;
    }

    m_smutex.Unlock();
    m_nSslRefCount++;

    m_bSslInitialized = true;

    FreeLibrary(m_hSslDll1);
    m_hSslDll1 = 0;
    FreeLibrary(m_hSslDll2);
    m_hSslDll2 = 0;

    if (m_bFailureSent) {
        m_bFailureSent = TRUE;
        PostReceiverError(SOCKETCALLBACK, SSL_FAILURE);
    }

    CreateLibrary(m_hSslDll1, "syncesel1.dll");
    CreateLibrary(m_hSslDll2, "syncesel2.dll");

    m_hSslDll1->Init();
    m_hSslDll2->Init();

    return true;
}
```

假設我們想要執行除了以下這行的所有程式碼：

```c++
PostReceiverError(SOCKETCALLBACK, SSL_FAILURE);
```

關鍵在於，要在測試中避免呼叫到這個方法，

又不能在最終產品程式碼呼叫不到它。

> **Seam**

> A seam is a place where you can alter behavior in your program without editing in that place.

在PostReceiveError的呼叫點存在接縫嗎？答案是肯定的。

我們有多種方式來去除該處的行爲。

PostReceiveError是一個全域函數，並非CAsyncSslRec類別的一部分。

所以如果在CAsyncSslRec類別中使用完全相同的簽章來添加一個方法呢？

```c++
class CAsyncSslRec
{
    ...
    virtual void PostReceiveError(UINT type, UINT errorcode);
    ...
}
```

其實作如下：

```c++
void CAsyncSslRec::PostReceiveError(UINT type, UINT errorcode)
{
    ::PostReceiveError(type, errorcode);
}
```

這個更動只引入了小小的**間接層**，最終還是呼叫到相同的函數。

接下來我們作一個CAsyncSslRec的子類別，並覆寫PostReceiveError。

```c++
class TestingAsyncSslRec : public CAsyncSslRec
{
    virtual void PostReceiveError(UINT type, UINT errorcode)
    {
    }
};
```

> 補充：這個技巧稱作「**擷取與覆寫**」(Extract and Override)

這下子我們就可以避開PostReceiverError的副作用了！

上面這類接縫，我們稱為物件接縫 (object seam)。

為遺留程式碼編寫測試的最大挑戰：**解依賴**。

---

## 4.3 接縫類型

### 4.3.1 Preprocessing Seams （預處理期接縫）

有些語言在編譯前，會先由巨集預處理器進行預處理。
這讓程式中，帶來了更多的接縫。

> **Seam**

> A seam is a place where you can alter behavior in your program without editing in that place.

```c++
#include <DFHLItem.h>
#include <DHLSRecord.h>

extern int db_update(int, struct DFHLItem *);

#include "localdefs.h" //為了利用預處理所引入的標頭檔

void account_update(int account_no, struct DHLSRecord *record, int activated)
{
    if (activated) {
        if (record->dateStamped && record->quantity > MAX_ITEMS) {
            db_update(account_no, record->item);
        } else {
            db_update(account_no, record->backup_item);
        }
    }
    db_update(MASTER_ACCOUNT, record->item);
}
```

在該標頭檔中，我們可以提供另一個db_update的定義：

```c++
#ifdef TESTING
...
struct DFHLItem *last_item = NULL;
int last_account_no = -1;
#define db_update(account_no,item)\
{
    last_item = (item);
    last_account_no = (account_no);
}
...
#endif
```

db_update原先的實作被替換之後，

我們就可以編寫測試來驗證db_update被呼叫時的時候，

是否接受到正確的參數。

當遇到一個接縫時，即意味著我們可以改變其所在之處的行為。這裡有一個**致能點**的概念。

> **Enabling Point**

> Every seam has an enabling point, a place where you can make the decision to use one behavior or another.

**解依賴流程：找出接縫，然後替換致能點。**

### 4.3.2 Link Seams（連接期接縫）

在許多語言系統中，編譯並非建構過程的最後一步。
會透過連結器進行呼叫，以便得到可執行的完整程式。

```java
package fitnesse;

import fit.Parse;
import fit.Fixture;

import java.io.*;
import java.util.Date;


import java.io.*;
import java.util.*;

public class FitFilter {
    public String input;
    public Parse tables;
    public Fixture fixture = new Fixture();
    public PrintWriter output;

    public static void main (String argv[]) {
        new FitFilter().run(argv);
    }

    public void run (String argv[]) {
        args(argv);
        process();
        exit();
    }

    public void process() {
        try {
            tables = new Parse(input);
            fixture.doTables(tables);
        } catch (Exception e) {
            exception(e);
        }
        tables.print(output);
    }
    ...
}
```

該檔案**import**了fit.Parse和fit.Fixture。

那麼編譯器和JVM是如何找到這些類別的呢？

在Java中你可以使用名為**classpath**的環境變數，

來告訴Java系統到哪裡去尋找這些類別。

> Suppose we wanted to supply a different version of the Parse class for testing. Where would the **seam** be?

> The seam is the **new Parse** call in the process method.

> Where is the **enabling point**?

> The enabling point is the **classpath**.

這招常見於對第三方函式庫的依賴。

```c++
void CrossPlaneFigure::rerender()
{
    // draw the label
    drawText(m_nX, m_nY, m_pchLabel, getClipLen());
    drawLine(m_nX, m_nY, m_nX + getClipLen(), m_nY);
    drawLine(m_nX, m_nY, m_nX, m_nY + getDropLen());
    if (!m_bShadowBox) {
        drawLine(m_nX + getClipLen(), m_nY,
            m_nX + getClipLen(), m_nY + getDropLen());
        drawLine(m_nX, m_nY + getDropLen(),
            m_nX + getClipLen(), m_nY + getDropLen());
    }

    // draw the figure
    for (int n = 0; n < edges.size(); n++) {
        ...
    }
    ...
}
```

以上程式碼對圖形函式庫進行了許多直接呼叫。

可以透過建立該函式庫的stub版本，用它去連結應用程式的其他剩餘部分，
來解除依賴。

```c++
void drawText(int x, int y, char *text, int textLength) {

}
void drawLine(int firstX, int firstY, int secondX, int secondY)
{
}
```

遇到帶有返回值的函數，只需在stub函數裡也返回某些東西。

```c++
int getStatus()
{
    return FLAG_OKAY;
}
```

使用圖形函式庫舉例，是因為帶有帶有回傳值的情況時，

你所預設的回傳值通常都不是正確的。

使用連結期接縫的目的之一是分離。你也可以實作感測。
透過引入額外的資料結構，來記錄這些呼叫。

```c++
std::queue<GraphicsAction> actions;

void drawLine(int firstX, int firstY, int secondX, int secondY)
{
    actions.push_back(GraphicsAction(LINE_DRAW, firstX, firstY, secondX, secondY);
}
```

藉助於這些資料結構，就可以在測試中感測某個函數的作用：

```c++
TEST(simpleRender,Figure)
{
    std::string text = "simple";
    Figure figure(text, 0, 0);

    figure.rerender();
    LONGS_EQUAL(5, actions.size());
    GraphicsAction action;
    action = actions.pop_front();
    LONGS_EQUAL(LABEL_DRAW, action.type);

    action = actions.pop_front();
    LONGS_EQUAL(0, action.firstX);
    LONGS_EQUAL(0, action.firstY);
    LONGS_EQUAL(text.size(), action.secondX);
}

```

雖然可以用來進行感測方案，但一般來說都會逐漸變得複雜。

連結期接縫的致能點，始終是位於程式碼之外。

有時在建構或部署腳本中，這使得連接期接縫的使用，顯得不那麼醒目。

> **Usage Tip.**

> If you use link seams, make sure that the difference between test and production envi- ronments is obvious.

---

### 4.3.3 Object Seams（物件接縫）

如果你觀察物件導向程式的呼叫，會發現沒有定義哪一個方法會被執行。

```java
cell.Recalculate();
```

這裡的cell到底會呼叫哪個方法呢？
要看實際程式碼才知道。

在物件導向語言中，並非所有方法呼叫都是接縫。

以下就是一個反例：

```c++
public class CustomSpreadsheet extends Spreadsheet {
    public Spreadsheet buildMartSheet() {
        ...
        Cell cell = new FormulaCell(this, "A1", "=A2+A3");
        ...
        cell.Recalculate();
        ...
    }
    ...
}
```

上述的程式碼建立了一個cell，並緊接著在同一個方法中使用了它。

請問上面的程式碼對Recalculate()的呼叫，會是一個物件接縫嗎？

不是。因為它並沒有對應的致能點。

我們無法控制哪一個Recalculate方法被呼叫，因為這取決於cell所指向的物件類型。

讓我們稍作修改。

```c++
public class CustomSpreadsheet extends Spreadsheet
{
    public Spreadsheet buildMartSheet(Cell cell) {
        ...
        cell.Recalculate();
        ...
    }
    ...
}
```

請問上述的程式碼中對cell.Recalculate的呼叫是不是一個接縫呢？

是的。我們可以在測試案例中使用任何類型的Cell物件為參數來呼叫這個builderMartSheet方法。

那麼請問它的致能點在哪？

答案是builderMartSheet參數列表。

> **Seam**

> A seam is a place where you can alter behavior in your program without editing in that place.

> **Enabling Point**

> Every seam has an enabling point, a place where you can make the decision to use one behavior or another.

---

### 補充：Dependency Injection （依賴注入）

1. Constructor Injection （建構式注入)
2. Method Injection （方法注入）
3. Property Injection （屬性注入）：即setter

---

然而，如果程式碼像下面這樣呢？

```c++
public class CustomSpreadsheet extends Spreadsheet {
    public Spreadsheet buildMartSheet(Cell cell) {
        ...
        Recalculate(cell);
        ...
    }

    private static void Recalculate(Cell cell) {
        ...
    }

    ...
}
```

我們可以透過修改Recalculate()的宣告，透過「**擷取與覆寫**」(Extract and Override)來達成目的。

```c++
public class CustomSpreadsheet extends Spreadsheet {

    ...

    protected Recalculate(Cell cell) {
        ...
    }

    ...
}

public class TestingCustomSpreadsheet extends CustomSpreadsheet {
    protected void Recalculate(Cell cell) {
        ...
    }
}
```
