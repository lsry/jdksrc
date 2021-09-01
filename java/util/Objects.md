## Objects  

### 0、概述   
提供了一些有关对象条件判断、基本操作的工具类，避免在代码中增加对 null 的判断。不可对该类创建对象，会抛出 AssertionError("No java.util.Objects instances for you!")；   

### 一、相等判断    
**equals​(a,b):** a,b 均为空或者调用对象的 equals 方法判断相等；   
**deepEquals(a, b)：** 相对来说增加了对参数可能为数组的判断，调用 Arrays.deepEquals0 来判断可能为数组吗？数组则使用 Array.deepEquals 判断,否则调用对象的 equals 方法；

### 二、Hash 值    
**hashCode(o)：** null 时为0， 其他调用对象的 hashCode 方法；   
**hash(Object... values)：** 对可变参数列表求 hash，不适用于单个对象，而是调用上面那个；   

#### 三、toString  
**toString(Object o)：** 调用 String 类的 valueOf 方法，null 返回 "null", 其他返回对象的 toString 方法返回值；   
**toString(Object o, String nullDefault)：** 对 null 对象返回默认值 nullDefault，其他返回 toString 方法值；

### 四、比较    
**compare(T a, T b, Comparator<? super T> c)：** 使用比较器 c 比较 a, b 的大小, 注意要在 c 中实现对 null 的判断，否则会抛 NPE，不过 a, b 均为 null，返回 0；     

### 五、空值处理   
**1. requireNonNull(T obj)：** 不为空直接返回参数对象，否则抛出 NPE；   
**2. requireNonNull(T obj, String message)：** message 为抛出 NPE 时的附带信息，提示出错位置及参数；     
**3. isNull(Object obj)：** 判断对象是否为空，空为 true。用在 lambda 的 filter 中；   
**4. nonNull(Object obj)：** 判断对象是否为空，不为空为 true；    
以下使用方法 2 来对提供的替代对象作空判断：    
**5. requireNonNullElse(T obj, T defaultObj)：** 提供一个对应对象为空时候的默认值，若默认值为空，抛出 NPE；   
**6. requireNonNullElseGet(T obj, Supplier<? extends T> supplier)：** obj 为空的话，使用 supplier 来提供一个，若还为空，抛出 NPE；   
**7. requireNonNull(T obj, Supplier<String> messageSupplier)：** 提供一个 supplier 来对抛出 NPE 时的信息；    

### 六、索引范围检查    
checkIndex 的各种重载方法，对 index 可能为 int 和 long 两种类型判断；    
#### 参数说明：    
+ index：索引位置；   
+ length：列表总长度；  
+ fromIndex：子列表开始索引；
+ toIndex：子列表结束索引；   
+ size：子列表长度；   

#### 索引正确值    
+ 0 <= index < length   
+ 0 <= fromIndex < toIndex < length   
+ 0 <= fromIndex < fromIndex + size <= length, size >= 0 