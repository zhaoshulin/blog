#What Programmers Can Do

----------
这是我的《What Every Programmer Should Know About Memory》中的第六章“What Programmer Can Do”的读书笔记。
##绕过Cache
∵一些大的数据结构，比如矩阵，一个写操作后会很快再写，在这种情况下，使用Cache反而会使性能降低。
∴处理器提供了非暂存性写操作。

非暂存性写操作：处理器直接写到内存，而不是先把内存读到cache中然后在cache中修改。

write-combining（写融合）策略：只有当一个完整的cache line被多个写填满之后，才flush掉该cache line，避免了频繁的flush。



