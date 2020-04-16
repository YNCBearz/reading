## Combine Functions into Transform (將函式組成轉換函式)

```
    function base(aReading) {...}
    function taxableCharge(aReading) {...}
```

```
    function enrichReading(argReading) {
        const aReading = _.cloneDeep(argReading);
        aReading.baseCharge = base(aReading);
        aReading.taxableCharge = taxableCharge(aReading);
        return aReading;
    }
```
-------------------------------

### 我們經常將資料傳給各種函式，讓那些程式用資料來計算各種衍生資訊。這些衍生的值可能會被許多地方使用，而使用衍生資料的計算邏輯往往是重複的。

### 作者喜歡把所有計算邏輯放在一起，以便在固定地方尋找與修改它們，並避免任何邏輯重複。

-------------------------------
### 命名風格
#### *enrich* : 產生相同東西但有額外資訊時
#### *transform* : 產生不同東西時