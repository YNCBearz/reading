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



#### 4. 偽造方法 - 使用一個區域的工廠方法（擷取與覆寫）

---

## 3.5 重構技術的變形

### 3.5.1 透過擷取與覆寫直接模擬假結果

---

## 3.6 克服封裝問題

### 3.6.1 使用internal和[InternalsVisibleTo]

### 3.6.2 使用[Conditional]特性

### 3.6.3 使用#if和#endif進行條件編譯

---

## 3.7 小結
