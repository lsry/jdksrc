## Optional&lt;T&gt;

### 零、概述     
Optional是一個容器類型，它提供了一種選項，用於表示一些所有 T 類型值或者值不存在（null）。強制調用者判斷是否存在T類型值，這樣可以避免空指針異常的產生。

例如，一個方法返回的結果可能是一個字符串，但是這個字符串可能為空。如果直接返回null，那麼調用這個方法的代碼就可能忘記 null 存在，造成空指針異常。如果使用Optional類型，則代碼就可以更加優雅地處理這種情況。

### 一、構造方式：   
**of(T):** 根據 T 類型值來創建，不可傳 null，否則抛出異常；  
**ofNullable(T):** 傳 null 時返回 Optional.EMPTY 常量，代表值不存在；   

### 二、判斷：   
#### Ⅰ、純判斷   
**isPresent():** 值存在時返回 true   
**isEmpty():** 值不存在返回 true 

#### Ⅱ、根據值存在進一步處理：  
**ifPresent(consumer):** 值存在執行 consumer 對應的方法    
**ifPresentOrElse(consumer, throw):** 值存在執行 consumer 對應的方法，否則執行 throw 對應的方法    
**filter(test):** 當值不存在或者 test 檢驗失敗返回 Optional.EMPTY 常量，否則原樣返回。    

#### Ⅲ、map, flatMap    
假設二者參數都爲 function，兩者均是當值存在時將對應的值通過 function 轉換為另外一個類型 U 所對應的值。但是 map 的 function 是純轉換為另外的類型 U 的值，而 flatMap 的 function 返回結果需要是 Optional&lt;U&gt;，對應代碼如下：   
```java
// map   
return Optional.ofNullable(mapper.apply(value));
// flatMap   
Optional<U> r = (Optional<U>) mapper.apply(value);
return Objects.requireNonNull(r);

// 使用如下：   
var strOpt = Optional.of("foo");
Optional<Integer> iOpt = strOpt.map(s -> s.hashCode());
System.out.println("iopt: " + (iOpt.isPresent()) + ", value: " + iOpt.get());
iOpt = strOpt.flatMap(s -> Optional.of(s.hashCode()));
System.out.println("iopt2: " + (iOpt.isPresent()) + ", value: " + iOpt.get());
```

### 三、獲取值    
**empty():** 返回一個值不存在的 Optional 對象
**get():** 值存在則返回，否則抛出異常    
**or(supplier):** 值存在則返回當前 Optional 對象，否則返回 supplier 結果對應的 Optional 對象；    
**orElse(T):** 值存在則返回原值，否則返回新值 T    
**orElseGet(supplier):** 值存在返回舊值，否則返回 supplier 對應的新值   
**orElseThrow():** 值存在返回原值，否則抛出 NoSuchElementException 異常   
**orElseThrow(throw):** 值存在返回原值，否則抛出 throw 返回的異常   

### 四、一些技巧    
#### Ⅰ、免除連續的 if 調用：      
假設有以下的代碼：     
```java
if (x.getY() != null) {
    Y y = x.getY();
    if (y.getZ() != null) {
        z = y.getZ();
        ...
    }
}
// 或者：   
if (x != null && x.getY() != null && x.getY().getZ() != null && ...) {}
```   
那麽可以使用 Optional 優化成如下的形式     
```java   
var t = Optional.ofNullable(x)
        .map(X::getY)
        .map(Y::getZ)
        .orElse(new Z());
```

#### Ⅱ、函數返回值時的 Optional，禁止返回 null 
```java
Optional<T> fun() {
    // bad  
    return null;
    // good   
    return Optional.empty();
}
```