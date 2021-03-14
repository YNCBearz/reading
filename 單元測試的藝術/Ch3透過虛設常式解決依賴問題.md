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
