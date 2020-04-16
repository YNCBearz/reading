## Combine Functions into Class (將函式移入類別)

```
    function base(aReading) {...}
    function taxableCharge(aReading) {...}
    function calculateBaseCharge(aReading) {...}
```

```
    class Reading {
        base() {...}
        taxableCharge() {...}
        calculateBaseCharge() {...}
    }
```

-------------------------------

### 當看到一組函式密切合作，一起處理同一個資料體，就表示製作類別的機會來了。

### 使用類別可以更清楚展現這些函式共用的環境，讓我可以移除許多引數，在物件中簡化函式的呼叫並提供參考。