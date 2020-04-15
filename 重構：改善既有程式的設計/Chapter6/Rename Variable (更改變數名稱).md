## Rename Variable (更改變數名稱)

```
    let a = height * width;
```

```
    let area = height * width;
```

> ### 名稱的重要性取決於它被使用的廣泛程度。

### 存留期間超過一次函式呼叫的持久性欄位必須取個好名字。

---------------------------------

### 如果變數被廣泛使用，考慮採取 *Encapsulate Variable* (封裝變數)
### 如果更改的是常數名稱，則可以考慮製作副本，逐個修改參考點，最後移除副本。


```
    const cpyNm = "Acme Gooseberries";
```

```
    const companyName = "Acme Gooseberries";
    const cpyNm = companyName;
```