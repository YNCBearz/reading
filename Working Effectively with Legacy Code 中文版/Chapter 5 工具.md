## Chapter 5
# Tools（工具）

## 5.1 自動化重構工具

>  **refactoring** (n.).
>
> A change made to the internal structure of software to make it easier to understand and cheaper to modify without changing its existing behavior.[name=Martin Fowler]

唯有當你的修改不會改變行為時，才能算是重構。
重構工具應當檢驗修改是否改變了行為。

例如當你試圖提取一個方法，並將它命名為與它所在類別中，某個既有方法同名時，該重構工具會不會將它標示為一個錯誤？

而若是與該類別的基底類別中的方法同名時，工具是否能檢測出來？

---

> 測試與自動化重構

```java
public class A {
    private int alpha = 0;
    private int getValue() {
        alpha++;
        return 12;
    }
    public void doSomething() {
        int v = getValue();
        int total = 0;
        for (int n = 0; n < 10; n++) {
            total += v;
        }
    }
}
```

利用至少兩款Java重構工具下，可以消除變數v

```java
public class A {
    private int alpha = 0;
    private int getValue() {
        alpha++;
        return 12;
    }
    public void doSomething() {
        int total = 0;
        for (int n = 0; n < 10; n++) {
            total += getValue();
        }
    }
}
```

看到問題了嗎？雖然變數移除了，alpha的值卻被加了10次！！

---

## 5.2 Mock Objects （仿物件）

面對遺留程式碼，最大的問題就是解依賴。
而這項工作從來不那麼簡單，如果將它依賴的程式碼拿走的話，就得有其他東西來填補空缺。

---

## 5.3 單元測試工具

人們常被「只需透過程式的GUI或Web介面即可對其進行測試，而無需進行任何特別動作」的念頭所誘惑。
但這樣的工作量往往超過團隊所想像。

xUnit測試框架，由Kent Beck用Smalltalk編寫。接著由Kent Beck和Erich Gamma移植到Java上。

其特點有：
- 它允許使用開發語言來編寫測試
- 所有測試互不干擾而獨立執行
- 一組測試可以集合成一個測試套件 (suite)，根據需要不斷執行

### 5.3.1 JUnit

```java
public class FormulaTest extends TestCase {
    public void testEmpty() {
        assertEquals(0, new Formula("").value());
    }
    public void testDigit() {
        assertEquals(1, new Formula("1").value());
    }
}
```

void testXXX() 常見的取名測試的方法。
每個測試都包含程式碼與斷言。

關鍵是，每一個測試方法的執行都是由**完全獨立**的物件來完成。
所以不可能會互相影響。

```java
public class EmployeeTest extends TestCase {
    private Employee employee;

    protected void setUp() {
        employee = new Employee("Fred", 0, 10);
        TDate cardDate = new TDate(10, 10, 2000);
        employee.addTimeCard(new TimeCard(cardDate,40));
    }

    public void testOvertime() {
        TDate newCardDate = new TDate(11, 10, 2000);
        employee.addTimeCard(new TimeCard(newCardDate, 50));
        assertTrue(employee.hasOvertimeFor(newCardDate));
    }

    public void testNormalPay() {
        assertEquals(400, employee.getPay());
    }
}
```

setUp為測試的特殊方法，會在測試方法執行前執行。
tearDown也是測試的特殊方法，會在測試方法執行後執行。

> 補充來源：單元測試的藝術（第二版）
>
> 1. 我並**不**使用setup來初始化被測試類別的物件執行個體，而是採用工廠方法 (factory method)。
> 2. 在單元測試的專案中，幾乎永遠不會有tearDown方法。

---

### 5.3.2 CppUnitLite

CppUnit是作者寫的，但因為C++特性（缺少反射機制），導致實作變得複雜。得在測試類別上編寫自己的suite函數。

```c++
Test *EmployeeTest::suite()
{
    TestSuite *suite = new TestSuite;
    suite.addTest(new TestCaller<EmployeeTest>("testNormalPay",testNormalPay));
    suite.addTest(new TestCaller<EmployeeTest>("testOvertime",testOvertime));
    return suite;
}
```

後來採用TEST巨集的方式簡化，重新寫成CppUnitLite。
在發布前消除C++後期的特性，以便提升程式碼清晰度。

```c++
#include "testharness.h"
#include "employee.h"
#include <memory>

using namespace std;

TEST(testNormalPay,Employee)
{
    auto_ptr<Employee> employee(new Employee("Fred", 0, 10));
    LONGS_EQUALS(400, employee->getPay());
}
```

本書C++範例採用CppUnitLite。

### 5.3.3 NUnit

NUnit是.NET語言的測試框架。與JUnit很接近。
顯著的特點是使用特性 (attribute)來標示測試方法與測試類別。

```vb
Imports NUnit.Framework

<TestFixture()> Public Class LogOnTest
    Inherits Assertion

    <Test()> Public Sub TestRunValid()
        Dim display As New MockDisplay()
        Dim reader As New MockATMReader()
        Dim logon As New LogOn(display, reader)
        logon.Run()
        AssertEquals("Please Enter Card", display.LastDisplayedText)
        AssertEquals("MainMenu",logon.GetNextTransaction().GetType.Name)
    End Sub
End Class
```

<TestFixture()>和<Test()>兩個特性分別標出。

LogonTest是個測試類別、TestRunValid是個測試方法。

---

### 5.3.4 其他xUnit框架

前往www.xprogramming.com看看你所用的平台或語言，
有沒有xUnit家族所公認的知識庫。

---

## 5.4 一般測試控制工具

前面的xUnit框架，主要是為單元測試所設計的。

---

### 5.4.1 整合測試框架 (FIT)

FIT是個簡練而優雅的整合測試框架，由Ward Cunningham所開發。

其背後的概念是簡單而強大的。

如果你可以為你的系統編寫檔案，並在其中嵌入系統輸入和輸出的表格，
並且這些檔案可以儲存為HTML的話，FIT就可以將它們視為測試來執行。

FIT接受HTML的資料，執行其中HTML表格所定義的測試，然後以HTML的形式輸出結果。表格中的儲存格會以綠色代表資料通過測試，紅色代表測試失敗。

FIT其中一個很強大的地方是，它能夠**鼓勵軟體編寫者與指定軟體功能者溝通**。

---

### 5.4.2 Fitnesse

Fitnesse本質上是以Wiki為宿主的FIT，大部分由Robert Martin和Micah Martin所開發。

（作者也曾參與其中，但因撰寫本書而退出。）

Fitnesse支援用於定義FIT測試的分級網頁。

測試表格的頁面可單獨，或放在測試套件中執行，
眾多的選擇使團隊合作變得更加容易。