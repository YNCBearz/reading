# Chapter3 透過虛設常式解決依賴問題

### 3.4.5 透過屬性get或set注入假物件

這個方式是為每一個相依物件建立對應的get與set屬性。

> 使用屬性進行依賴注入會比建構函式注更加簡單。

```c#
public class LogAnalyzer
{
    private IExtensionManager manager;
    public LogAnalyzer ()
    {
        manager = new FileExtensionManager();
    }
    public IExtensionManager ExtensionManager
    {
        get { return manager; }
        set { manager = value; }
    }
    public bool IsValidLogFileName(string fileName)
    {
        return manager.IsValid(fileName);
    }
}

[Test]
Public void
IsValidFileName_SupportedExtension_ReturnsTrue()
{
    //set up the stub to use, make sure it returns true
    ...
    //create analyzer and inject stub
    LogAnalyzer log =
        new LogAnalyzer ();
    log.ExtensionManager=someFakeManagerCreatedEarlier;
    //Assert logic assuming extension is supported
    ...
}
```

> 使用屬性注入表達了：**這個相依物件並非是必要的**。

### 3.4.6 在呼叫方法之前才注入假物件

這一節所討論的場景時是：**當你要對某物件進行操作前，才取得執行個體**。

#### 1. 使用工廠類別
在這種情況下，回到最基本的設計。
利用工廠模式來回傳虛設常式物件。

```c#
public class LogAnalyzer
{
    private IExtensionManager manager;
    public LogAnalyzer ()
    {
        manager = ExtensionManagerFactory.Create();
    }
    public bool IsValidLogFileName(string fileName)
    {
        return manager.IsValid(fileName)
        && Path.GetFileNameWithoutExtension(fileName).Length>5;
    }
}
[Test]
public void
IsValidFileName_SupportedExtension_ReturnsTrue()
{
    //set up the stub to use, make sure it returns true
    ...
    ExtensionManagerFactory
        .SetManager(myFakeManager);
    //create analyzer and inject stub
    LogAnalyzer log =
        new LogAnalyzer ();
    //Assert logic assuming extension is supported
    ...
}
class ExtensionManagerFactory
{
    private IExtensionManager customManager=null;

    public IExtensionManager Create()
    {
        If(customManager!=null)
          return customManager;
        Return new FileExtensionManager();
    }
    public void SetManager(IExtensionManager mgr)
    {
        customManager = mgr;
    }
}
```

> 你唯一要確認的是，要在工廠類別加一個接縫，讓它們可以回傳自訂的虛設常式。

#### 2. 在發佈版本中隱藏接縫
如何能在發佈版本中隱藏接縫呢？
我們將在3.6.2節詳細這種處理方式。

#### 3. 不同的中間層深度等級
在每一個不同的深度等級，你可以選擇產生一個假物件（或虛設常式）。

| 被測試程式 | 可以進行的操作 |
| :-----| :---- |
| 層次深度1：針對類別中的FileExtensionManager變數 | 新增一個建構函式參數，以便傳入相依物件。此時只有被測試類別中的一個成員是偽造的，其餘程式碼皆保持不變。 |
| 層次深度2：針對從工廠注入被測試類別的相依物件 | 透過工廠類別的賦值方法設定一個假的相依物件。此時工廠內的成員是偽造的，被測試類別完全不需要調整。 |
| 層次深度3：針對返回相依物件的工廠類別 | 將工廠類別直接替換成一個假工廠，假工廠會回傳假的相依物件。此時測試執行過程中，工廠是假的，回傳的物件也是偽造的，被測試類別完全不需要調整。 |

ʕ •ᴥ•ʔ：天衣無縫、包開、橫刀奪愛 (Hunter x Hunter)

關於中間層的使用，你需要瞭解的是**當控制的中間層深度越深（程式執行堆疊越深），你對被測試程式的控制能力越大。**

然而這也使得測試程式變的更難理解，更不好找到適合插入接縫的位置。

- 第一層（在被測試程式中偽造一個成員）：

你需要增加一個建構函式，以便在建構函式中設定類別相關資訊。
這個方式會改變被測試程式的語意及呼叫，建議避免使用這種方法。

- 第二層（在工廠類別中偽造一個成員）：
在工廠類別中新增一個setter，供測試方法注入一個你想要的假相依物件。
唯一缺點是，你需要瞭解誰會在何時呼叫工廠，但這方式還是比其他方法來的簡單合理。

- 第三層（偽造假工廠類別）：
一個假物件回傳另一個假物件。這會讓測試程式變得不容易理解，最好還是盡量避免。


#### 4. 偽造方法 - 使用一個區域的工廠方法（擷取與覆寫）
這種方式不屬於表3-1中任何一層，它在接近被測試程式的表層建立全新的中介層。

使用這種方法，你透過被測試程式類別中的一個區域**虛擬**方法(virtual method)做為工廠方法。
這就產生了接縫，你可以在測試中新增類別，**繼承**自被測試類別，
透過**覆寫**其虛擬的工廠方法，注入你設定好的假相依物件。

```c#
public class LogAnalyzerUsingFactoryMethod
{
    public bool IsValidLogFileName(string fileName)
    {
        return GetManager().IsValid(fileName);
    }
    protected virtual IExtensionManager GetManager()
    {
        return new FileExtensionManager();
    }
}

[TestFixture]
public class LogAnalyzerTests
{
   [Test]
    public void overrideTest()
    {
        FakeExtensionManager stub = new FakeExtensionManager();
        stub.WillBeValid = true;

        TestableLogAnalyzer logan =
             new TestableLogAnalyzer(stub);

        bool result = logan.IsValidLogFileName("file.ext");
        Assert.True(result);
    }
}

class  TestableLogAnalyzer
               :LogAnalyzerUsingFactoryMethod
{
    public TestableLogAnalyzer(IExtensionManager mgr)
    {
        Manager = mgr;
    }

    public IExtensionManager Manager;

    protected override IExtensionManager GetManager()
    {
        return Manager;
    }
}

internal class FakeExtensionManager : IExtensionManager
{
    //no change from the previous samples
    ...
}

```

這個技巧稱為「**擷取與覆寫**」(Extract and Override)，
試過幾次後，你會發現這方法的強大之處。因為它讓你無須進入到過深的層次（改變呼叫堆疊的深度依賴）就可以直接替換相依物件。

### 5. 何時該使用這種方式
「**擷取與覆寫**」適合用來模擬提供給被測試類別的**輸入(input)**，
但如果要拿來驗證被測試程式對相依物件的呼叫，卻十分不方便。

當需要模擬被測試類別由外部程式取得或回傳的輸入值時，使用此手法，
可以讓程式碼語意不會為了可測試性而改變（新介面、新的建構函式等）。

除非程式碼已明顯具備可測試性的接縫：一個可偽造的介面或一個可以注入接縫的位置。否則推薦優先使用此技巧。

---

## 3.5 重構技術的變形

想避免使用基底類別而改使用介面的其中一個原因，
可能是因為在產品程式碼中，該基底類別已經內建（或可能會有）了某個相依物件的依賴，這讓實作衍生子類別的難度更高。

### 3.5.1 透過擷取與覆寫直接模擬假結果

```c#
public class LogAnalyzerUsingFactoryMethod
{
    public bool IsValidLogFileName(string fileName)
    {
        return this.IsValid(fileName);
    }

     protected virtual bool IsValid(string fileName)
    {
       FileExtensionManager mgr = new FileExtensionManager();
       return mgr.IsValid(fileName);
    }
}

[Test]
public void overrideTestWithoutStub()
{
    TestableLogAnalyzer logan = new TestableLogAnalyzer();
    logan.IsSupported = true;

    bool result = logan.IsValidLogFileName("file.ext");
    Assert.True(result,"...");
}

class TestableLogAnalyzer: LogAnalyzerUsingFactoryMethod
{
    public bool IsSupported;
    protected override bool IsValid(string fileName)
    {
        return IsSupported;
    }
}

```

---

## 3.6 克服封裝問題

> 請將測試程式視為另一個最終使用者！

### 3.6.1 使用internal和[InternalsVisibleTo]

如果你不喜歡在類別中加入人人可見的公開建構函式，
可以將它標記為internal，而不是public。

```c#
public class LogAnalyzer
{
    ...
    internal LogAnalyzer (IExtensionManager extentionMgr)
        {
            manager = extentionMgr;
        }
    ...
}
using System.Runtime.CompilerServices;
[assembly:
    InternalsVisibleTo("AOUT.CH3.Logan.Tests")]
```

### 3.6.2 使用[Conditional]特性

透過System.Diagnostics.ConditionalAttribute特性進行標記也是種很直覺的方式。

(DEBUG 和 RELEASE 是最常見的兩個，
Visual Studio在預設情況下依據你的編譯種類使用這兩個值。)

在編譯時，如果這個編譯標記**不**存在，帶標記方法的**呼叫端**就不會包含在這個編譯版本中。

例如，在編譯release版本時，下面這個方法所有的呼叫行為都會被移除，
而這個方法內容仍會保留下來。

```c#
[Conditional ("DEBUG")]
public void DoSomething()
{
}
```

如果有些方法只在某些偵錯模式下使用，就可以使用這個特性進行標記（建構函式除外）。

> 這些帶著標記的方法在產品程式碼中並不會被隱藏。

注意：在產品程式碼中使用條件編譯，會降低程式碼的可讀性。

### 3.6.3 使用#if和#endif進行條件編譯

把方法和專門提供給測試呼叫的建構函式，放在#if與#endif結構裡，
可以確保它們只在對應的編譯參數設定下編譯。

```c#
#if DEBUG
        public LogAnalyzer (IExtensionManager extensionMgr)
        {
            manager = extensionMgr;
        }
#endif
...
#if DEBUG
        [Test]
        public void
IsValidFileName_SupportedExtension_True()
        {
...
            //create analyzer and inject stub
            LogAnalyzer log =
                new LogAnalyzer (myFakeManager);
            ...
        }
#endif
```

這個方式被廣泛地使用，但也會讓程式看起來很亂。
為了保持程式的清晰，在合適的時候建議使用[InternalsVisibleTo]特性。

---

## 3.7 小結

在程式中注入虛設常式物件的方法有很多。
關鍵在於找到合適的中間層，或建立出這個中間層，把它拿來作**接縫**。

因為假物件並不一定為虛設常式(stub)或模擬物件(mock)，
因此我們在命名時習慣使用**fake**這個詞。