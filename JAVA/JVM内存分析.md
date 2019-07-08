#  jmap

> JMap可查看目前JVM中各个代的内存状况、JVM中对象的内存的占用状况，以及导出整个JVM中的内存信息。

1.  jmap -heap 612644

```
-bash-4.1# jmap -heap 612644
Attaching to process ID 612644, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 20.0-b11

using thread-local object allocation.
Parallel GC with 53 thread(s)

Heap Configuration:
   MinHeapFreeRatio = 40
   MaxHeapFreeRatio = 70
   MaxHeapSize      = 2147483648 (2048.0MB)
   NewSize          = 1310720 (1.25MB)
   MaxNewSize       = 17592186044415 MB
   OldSize          = 5439488 (5.1875MB)
   NewRatio         = 2
   SurvivorRatio    = 8
   PermSize         = 21757952 (20.75MB)
   MaxPermSize      = 268435456 (256.0MB)
   
Heap Usage:
PS Young Generation
Eden Space:
   capacity = 567345152 (541.0625MB)
   used     = 154716528 (147.54917907714844MB)
   free     = 412628624 (393.51332092285156MB)
   27.27026527936208% used
From Space:
   capacity = 41025536 (39.125MB)
   used     = 40990960 (39.09202575683594MB)
   free     = 34576 (0.0329742431640625MB)
   99.91572078424521% used
To Space:
   capacity = 71958528 (68.625MB)
   used     = 0 (0.0MB)
   free     = 71958528 (68.625MB)
   0.0% used
PS Old Generation
   capacity = 715849728 (682.6875MB)
   used     = 100480688 (95.82585144042969MB)
   free     = 615369040 (586.8616485595703MB)
   14.036561595229104% used
PS Perm Generation
   capacity = 154271744 (147.125MB)
   used     = 154186352 (147.04356384277344MB)
   free     = 85392 (0.0814361572265625MB)
   99.94464832134133% used
```

2. jmap  -histo -F 566230 > histo.txt  # jvm堆中对象的详细占用情况

```
 num     #instances         #bytes  class name
----------------------------------------------
   1:       2909317      116372680  java.util.HashMap$KeyIterator
   2:        610265       59261512  [C
   3:        203541       43778584  [I
   4:        157950       36559608  [B
   5:        251617       34325120  <constMethodKlass>
   6:        251617       34230888  <methodKlass>
   7:         20187       22653880  <constantPoolKlass>
   8:        552356       17675392  java.lang.String
   9:         20187       17446024  <instanceKlassKlass>
  10:         17068       12956448  <constantPoolCacheKlass>
  11:        136035       11971080  java.lang.reflect.Method
  12:        184388       11905216  <symbolKlass>
  13:        222016        7104512  java.util.concurrent.locks.AbstractQueuedSynchronizer$Node
```

