## TimeUnit 

### 零、簡介   
該類是對時間單位的抽象，通過枚舉定義了一下七個時間單位，通過這些枚舉常量可以方便的進行時間單位閒的轉換。 
1. 納秒：NANOSECONDS；
2. 微秒：MICROSECONDS；
3. 毫秒：MILLISECONDS；
4. 秒：SECONDS；
5. 分鐘：MINUTES；
6. 小時：HOURS；
7. 天：DAYS；

### 壹、使用    
#### 一、將當前單位表示的時間轉換為其他單位  
以秒和毫秒之間轉換爲例：    
```java
long second = 300L;
// 將秒轉爲毫秒
TimeUnit.SECONDS.toMillis(second); // 輸出：300000  
long milisecond = 4500L;
// 將毫秒轉爲秒，結果取下整
TimeUnit.MILLISECONDS.toSeconds(4500); // 輸出：4 
```

#### 二、將其他單位時間轉換爲當前單位對象   
```java
TimeUnit.SECONDS.convert(4500, TimeUnit.MILLISECONDS); // 輸出：4
```

#### 三、綫程操作
1. join：設置需要 join 綫程的時間長度
2. sleep：設置需要 sleep 綫程的時間長度   
```java
Thread t1 = new Thread(()-> {
    try {
        TimeUnit.MILLISECONDS.sleep(4000);
        System.out.println("in thread");
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
});
t1.start();;
TimeUnit.SECONDS.timedJoin(t1, 1);
System.out.println("main thread");
```
常規的join，sleep 只能指定毫秒數，通過該工具類方法可以指定其他時間單位代表的時間長度。

#### 四、和 ChronoUnit 互換   
之間是相互對應的。     
| TimeUnit | ChronoUnit |
|-|-|  
TimeUnit.NANOSECONDS|ChronoUnit.NANOS;   
TimeUnit.MICROSECONDS|ChronoUnit.MICROS;   
TimeUnit.MILLISECONDS|ChronoUnit.MILLIS;   
TimeUnit.SECONDS|ChronoUnit.SECONDS;   
TimeUnit.MINUTES|ChronoUnit.MINUTES;   
TimeUnit.HOURS|ChronoUnit.HOURS;   
TimeUnit.DAYS|ChronoUnit.DAYS;   

### 叁、源碼分析    
#### 一、屬性：以毫秒爲例       
1. long scale：和納秒之間的轉換關係，1 毫秒 = scale 納秒；
2. long maxNanos：若轉換爲納秒可以表示的最大毫秒數，值為 Long.MAX_VALUE / 1000000;   
3. long maxMicros: 若轉換爲微秒可以表示的最大毫秒數，值為 Long.MAX_VALUE / 1000;   
4. long maxMillis: 最大毫秒數，值為 Long.MAX_VALUE；
5. long maxSecs: 可轉換的最大秒數，值爲 Long.MAX_VALUE / 1000
6. long microRatio: 和微妙閒的換算關係，值爲 1000
7. int milliRatio：和毫秒的換算關係，值爲 1
6. int secRatio：和秒的換算關係，值爲 1000   
構造方法即是計算出上述屬性值，通過每個單位的 scale 進行比較，屬性含義有一定變化。

#### 二、單位轉換：以 toMillis 爲例   
```java
public long toMillis(long duration) {
    long s, m;
    // this 代表毫秒，則直接返回原值
    // this 代表小於毫秒的單位，應該除以毫秒比例
    if ((s = scale) <= MILLI_SCALE)
        return (s == MILLI_SCALE) ? duration : duration / milliRatio;
    // 以下兩個分支判定是否超過了可以轉換爲最大毫秒數（Long.MAX_VALUE)的對應單位時間的邊界，超過則截斷，改為 Long 可表示的最值
    else if (duration > (m = maxMillis))
        return Long.MAX_VALUE;
    else if (duration < -m)
        return Long.MIN_VALUE;
    else
    // 未超出邊界，直接乘以毫秒比
        return duration * milliRatio;
}
```   

由上述代碼可知：
1. 對於時間轉換操作，考慮其數值用 long 表示，注意不能超過 long 的最值，否則截斷為 long 的最值。 
2. 當前對象小於需要轉換的單位，則除以對應比例，否則乘上比例；

超過秒的單位轉換使用以下的輔助函數，同樣遵守上述兩條規則。    
```java
public long toMinutes(long duration) {
    return cvt(duration, MINUTE_SCALE, scale);
}
private static long cvt(long d, long dst, long src) {
    long r, m;
    if (src == dst) // 代表單位相同
        return d;
    else if (src < dst) // 目的單位比原單位大，則除以對應比例
        return d / (dst / src);
    // 以下代碼目的單位比原單位小
    // 超過邊界
    else if (d > (m = Long.MAX_VALUE / (r = src / dst)))
        return Long.MAX_VALUE;
    else if (d < -m)
        return Long.MIN_VALUE;
    else
        return d * r; // 未超過，乘以比例
}
```

#### 三、綫程操作
基本上將時間轉換爲毫秒和納秒組合的形式，然後調用綫程相關方法執行；    
```java
public void sleep(long timeout) throws InterruptedException {
    if (timeout > 0) {
        long ms = toMillis(timeout); // 將當前時間轉爲毫秒表示
        int ns = excessNanos(timeout, ms); // 但是可能有未計算出的納秒數，則分開計算
        Thread.sleep(ms, ns); // 調用毫秒和納秒為參數的方法
    }
}
private int excessNanos(long d, long m) {
    long s;
    if ((s = scale) == NANO_SCALE) // 本身為納秒表示，則去除 m 毫秒后的納秒數目
        return (int)(d - (m * MILLI_SCALE));
    else if (s == MICRO_SCALE) // 為微秒，則轉換為納秒然後去除 m 毫秒后的納秒
        return (int)((d * 1000L) - (m * MILLI_SCALE));
    else
        return 0; // 其他單位不可能有多餘的納秒數目
}
```