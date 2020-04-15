## Encapsulate Variable (封裝變數)

```
    let defaultOwner = {firstName: "Martin", lastName: "Fowler"};
```

```
    let defaultOwnerData = {firstName: "Martin", lastName: "Fowler"};
    export function defaultOwner() {return defaultOwnerData;}
    export function setDefaultOwner(arg) {defaultOwnerData = arg;}
```

> ### 重構就是調整程式的元素，而資料比函式更難調整。

### 如果想移動被廣泛使用的資料，最好的辦法通常是先封裝它，用函式來轉傳它的所有存取動作。
### 作者不喜歡 getter 使用 get 前綴詞 (prefix)，但會保留 setter 的 set 前綴詞 (prefix)

---------------------------------

### 有時候我們不僅控制對變數的變動，也想控制對其內容的變動。
### 最簡單的方法是防止任何針對值的修改，回傳資料的副本。

```
    let defaultOwnerData = {firstName: "Martin", lastName: "Fowler"};
    export function defaultOwner() {return Object.assign({}, defaultOwnerData);}
    export function setDefaultOwner(arg) {defaultOwnerData = arg;}
```
