## Unsafe   

### O、概述   
jdk 内部的类，直接操作内存中的值，需要调用者来保证安全；

### 一、Unsafe 对象
jdk 使用了单例模式来构建 Unsafe，保证全局唯一   
1. registerNatives：native 方法做一些初始化；其中调用使用静态代码块的方式提前调用；   
2. 将构造方法设为私有方法；
3. 懒加载模式生成一个 Unsafe 对象；
4. 对外提供一个静态方法  getUnsafe；     

       private static final Unsafe theUnsafe = new Unsafe();
       public static Unsafe getUnsafe() {
        return theUnsafe; }

### 二、功能    
#### 0. 内存地址的表示方法：    
+ 对象 o 作为基地址 + 偏移量 offset；
- 对象 o 为空，则使用绝对地址，即该偏移量就是地址

#### 1. 变量的取值与写值：在 java 堆中的操作     
**${TYPE}**：表示基本类型及引用类型；

偏移量获得：
+ Field 类反射的方法 objectFieldOffset、staticFieldOffset、staticFieldBase
+ 数组对象中 基址 + 索引 * 索引所占内存大小   

方法：      
+ putType(o, offset, newValue), getType(o, offset);    
- put/getAddress: 获得变量的指针  

数据竞争下使用来保证线程安全：  
+ get/put${TYPE}Volatile: 以 volatile 形式从主内存获取或设置值   
以下四种只是直接调用了 volatile 版本：   
+ get${TYPE}Aquire: 以 aquire 形式获取值
+ put${TYPE}Release:   
+ get/put${TYPE}Opaque：    

以下使用上述方法及 CAS 操作来原子获取更新变量：     

获得旧值并自加一个值：   
- getAndAdd${TYPE}：
- getAndAdd${TYPE}Release：
- getAndAdd${TYPE}Acquire：   

获得旧值并设置新值：   
- getAndSet${TYPE}：
- getAndSet${TYPE}tRelease：   
- getAndSet${TYPE}Release：

位操作：   
- getAndBitwise${位操作类型}${针对的数据类型}；
- getAndBitwise${位操作类型}${针对的数据类型}Release；
- getAndBitwise${位操作类型}${针对的数据类型}Aquire；

在特定偏移量处访问对应类型的值：TYPE={Long, int, byte, short, char}    
- get${TYPE}Unaligned；
- put${TYPE}Unaligned；
- make${TYPE}: 根据字节组合成对应类型值；

#### 2. 在本地内存（C heap）中的操作：    
方法：调用了上面的堆中方法，不过对应的对象 o 为空，偏移量成了绝对地址     
+ getType(address)   
+ putType(address, newValue)      

#### 3. 参数值有效性验证：失败抛出 IllegalArgumentException   

#### 4. 内存管理：对 malloc, realloc, free 的封装，作用于本地内存       
内存属性：   
+ ADDRESS_SIZE：地址所占字节，根据机器而定，4 或者 8；
+ PAGE_SIZE：内存页大小；   
+ DATA_CACHE_LINE_FLUSH_SIZE：缓存行刷新大小，功能不启用为 0；
+ BIG_ENDIAN: 表示内存数据是否是大端编址；
+ UNALIGNED_ACCESS：判断是否能在地址未对齐情况下访问；

地址规整化：让分配的空间大小是地址所占字节的整数倍       
  
    (bytes + ADDRESS_SIZE - 1) & ~(ADDRESS_SIZE - 1)

过程：进行参数检查，然后内存管理操作；   
方法：    
+ allocateMemory(bytes)：分配 bytes 大小的内存，失败则OOM； 
+ reallocateMemory(address, bytes)：将内存地址 address 处改为 bytes 大小的内存，  
+ setMemory：内存初始化成为一个固定值；   
+ copyMemory：将内存设置成源内存的值；  
+ copySwapMemory：  
+ freeMemory：释放内存  
+ writebackMemory：保证对应地址范围的内存写到物理内存中；

内存屏障：是CPU或编译器在对内存随机访问的操作中的一个同步点，使得此点之前的所有读写操作都执行后才可以开始执行此点之后的操作，避免代码重排序   
+ loadFence/loadLoadFence：禁止load操作重排序。屏障前的load操作不能被重排序到屏障后，屏障后的load操作不能被重排序到屏障前；   
+ storeFence/storeStoreFence：禁止store操作重排序。屏障前的store操作不能被重排序到屏障后，屏障后的store操作不能被重排序到屏障前；   
+ fullFence：禁止load、store操作重排序；   

#### 5. Class 操作   
方法：   
+ defineClass：定义一个类，此方法会跳过JVM的所有安全检查，默认情况下，ClassLoader（类加载器）和ProtectionDomain（保护域）实例来源于调用者；  
+ allocateInstance：为给予的 Class 实例化一个对象，但不调用构造方法；   
+ objectFieldOffset：内存访问器中的 Cookie；
- objectFieldOffset：从Class 和 字段名来获取地址；
+ staticFieldOffset：静态字段地址获得
+ staticFieldBase：静态字段基地址（指针）；
+ shouldBeInitialized：判断给予的 Class 是否需要初始化；
+ ensureClassInitialized：保证给予的 Class 被初始化；

#### 6. 数组操作
方法：    
+ allocateUninitializedArray：分配一个基本类型数组，但没有使用零值初始化；
+ arrayBaseOffset：获得数组第一个元素的偏移量。  
+ ARRAY_${TYPE}_BASE_OFFSET：针对基本类型及 Object 类定义了静态常量字段来表示；

#### 7. CAS 方式更新变量
针对 char, short, byte, int 及引用类型都有对应方法；不过 char，byte，short 都是转为 int 来设置的；
+ int，long，引用：独特的 native 方法；
+ byte，short：转为 int  
+ char：转为 short；   
+ boolean：转为 byte，
+ float：调用 Float.floatToRawIntBits 转为 int；
+ double：调用 Double.doubleToRawLongBits 转为 long；

#### 8. 线程操作     
方法：   
+ unpark：取消阻塞线程；
+ park：阻塞线程；