### PL shared node Js

- lo space (large object )
- old space ()
- map space







- GC

  > 根据统计学原理根据生成的空间大小而分配内存

  - Copy

    > 把new空间的对象copy到Schemedule空间，之后再copy回新空间，如果反复存在则copy至old-space

    mark-sweep(清除)

    mark-compact(标记压缩)

    

  - GC root

    > 会有引用的一颗内存树，如果old-space发现没有在root上有引用的对象，则清除

    - 如果有一个大对象放入不进去old-space则对原有对象进行compact操作，后排序，再挪入新的大对象