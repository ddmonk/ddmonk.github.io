title: JAVA Integer
date: 2019-04-02 01:11:46
tags: Java Foundation
---

# Java Integer小实验 #
## 背景 ##
在项目过程中，有同事在Integer之间使用的`==`进行比较，导致现网上面一直无法匹配，但是在测试过程中并没有发现类似的问题，所以有必要对Integer之间的比较进行一个调研。
## 实验 ##
在比较过程中我书写了如下代码进行比较：
```java 
public class Main {

    public static void main(String[] args) {
        Integer integer127 = 127;
        Integer integer128 = 128;
        Integer newInteger127 = new Integer(127);
        int int127 = 127;
        int int128 = 128;
        Integer integer128copy = 128;
        Integer integer127copy = 127;
        boolean compareA = integer127 == newInteger127;
        boolean compareB = integer127 == int127;
        boolean compareC = integer128 == int128;
        boolean compareD = integer128 == integer128copy;
        boolean compareE = integer127 == integer127copy;
        System.out.println("integer127 == newInteger127: " + compareA);
        System.out.println("integer127 == int127: " + compareB);
        System.out.println("integer128 == int128: " + compareC);
        System.out.println("integer128 == integer128copy: " + compareD);
        System.out.println("integer127 == integer127copy: " + compareE);
    }
}
```  
输出结果如下：
```JAVA
integer127 == newInteger127: false
integer127 == int127: true
integer128 == int128: true
integer128 == integer128copy: false
integer127 == integer127copy: true
```
## 分析 ##
### Compile代码 ###
首先使用`javac -p Main.claas`查看其编译过程。部分内容截图如下：  
```java
public class Main {
  public Main();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
       0: bipush        127
       2: invokestatic  #2                  // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
       5: astore_1
       6: sipush        128
       9: invokestatic  #2                  // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
      12: astore_2
      13: new           #3                  // class java/lang/Integer
      16: dup
      17: bipush        127
      19: invokespecial #4                  // Method java/lang/Integer."<init>":(I)V
      22: astore_3
      23: bipush        127
      25: istore        4
      27: sipush        128
      30: istore        5
      32: sipush        128
      35: invokestatic  #2                  // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
      38: astore        6
      40: bipush        127
      42: invokestatic  #2                  // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
      45: astore        7
      47: aload_1
      48: aload_3
      49: if_acmpne     56
      52: iconst_1
      53: goto          57
      56: iconst_0
      57: istore        8
      59: aload_1
      60: invokevirtual #5                  // Method java/lang/Integer.intValue:()I
      63: iload         4
      65: if_icmpne     72
      68: iconst_1
      69: goto          73
      72: iconst_0
      73: istore        9
      75: aload_2
      76: invokevirtual #5                  // Method java/lang/Integer.intValue:()I
      79: iload         5
      81: if_icmpne     88
      84: iconst_1
      85: goto          89
      88: iconst_0
      89: istore        10
      91: aload_2
      92: aload         6
      94: if_acmpne     101
      97: iconst_1
      98: goto          102
     101: iconst_0
     102: istore        11
     104: aload_1
     105: aload         7
     107: if_acmpne     114
     110: iconst_1
     111: goto          115
     114: iconst_0
     115: istore        12
```
查看编译初始化情况：  
```Java
    Integer integer127 = 127;
    Integer integer128 = 128;
```
其均是调用`java/lang/Integer.valueOf`方法构建，`valueOf`代码如下：  
```java
    public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
```
`IntegerCache`是什么东东，直接贴代码：  
```java
private static class IntegerCache {
        static final int low = -128;
        static final int high;
        static final Integer cache[];

        static {
            // high value may be configured by property
            int h = 127;
            String integerCacheHighPropValue =
                sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
            if (integerCacheHighPropValue != null) {
                try {
                    int i = parseInt(integerCacheHighPropValue);
                    i = Math.max(i, 127);
                    // Maximum array size is Integer.MAX_VALUE
                    h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
                } catch( NumberFormatException nfe) {
                    // If the property cannot be parsed into an int, ignore it.
                }
            }
            high = h;

            cache = new Integer[(high - low) + 1];
            int j = low;
            for(int k = 0; k < cache.length; k++)
                cache[k] = new Integer(j++);

            // range [-128, 127] must be interned (JLS7 5.1.7)
            assert IntegerCache.high >= 127;
        }

        private IntegerCache() {}
    }
```
概括来说就是在JVM初始化的时候会默认生成一个value值在[-128，127]之间的一个Integer对象Cache。  
在调用`valueOf`的时候会先去`Cache`中查找，若存在，则直接返回对应对象，若不存在则重新new一个对象。  
而`new Integer(127)`则是调用了Integer的构造函数进行初始化。   
### 分析Compare值 ###
1、`compareA`查看比那一过程，其主要是通过使用`if_acmpne`进行比较，该指令主要是比较地址块是否相同。由于两个是不同的地址快，所以返回为`false`。   
2、`cmopareB`，在比较之前`integer127`首先调用了` java/lang/Integer.valueOf`来获取对应int值，因为int是基本数据类型，所以在地址块比较中，两者是相同的。   
3、`compareC`与 序号2相同   
4、`compareD`由于初始化时，128超过了cache范围，所以使用`Integer xx = 128`均会生成一个新的Integer对象。比较他们的地址块，不相同，所以范围为`false`   
5、`compareE`初始化值为127时，每次都是会返回cache中的同一个对象，所以在比较地址时会返回`true`.   
## 总结 ##
1. 在对象比较中还是最好使用equal。  
2. Integer会初始化[-128,127]中的所有Integer对象，用于今后调用。  
