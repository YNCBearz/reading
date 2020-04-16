## Introduce Parameter Object (使用參數物件)

```
    function amountInvoiced(startDate, endDate) {...}
    function amountReceived(startDate, endDate) {...}
    function amountOverdue(startDate, endDate) {...}
```

```
    function amountInvoiced(aDateRange) {...}
    function amountReceived(aDateRange) {...}
    function amountOverdue(aDateRange) {...}
```

-------------------------------

### 將資料放在一個結構裡面有很大的價值，因為它可以清楚展示資料項目之間的關係，也可以讓使用新結構的函式參數列變短。

### 但是這種重構最厲害的地方是，它可以讓我更深層地修改程式碼。
### 決定新架構之後，可以修改程式行為來使用這些架構、可以將各個地方使用資料的行為寫成函式，甚至用一個類別將資料結構與函式結合。