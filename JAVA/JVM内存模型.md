 JDK 8 中永久代向元空间的转换

1. 字符串存在永久代中，容易出现性能问题和内存溢出。
2. 类及方法的信息等比较难确定其大小，因此对于永久代的大小指定比较困难，太小容易出现永久代溢出，太大则容易导致老年代溢出。
3. 永久代会为 GC 带来不必要的复杂度，并且回收效率偏低。
4. Oracle 可能会将HotSpot 与 JRockit 合二为一。

https://blog.csdn.net/java1993666/article/details/55805037

https://www.cnblogs.com/aspirant/p/8662690.html

https://www.jianshu.com/p/30e8ff0f7dd9