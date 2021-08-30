## 1、Redis Zset
我们都知道```redis```的```zset```是由 skip-list 实现的，列表的每个节点分为两个部分```element```和```score```。
```
typedef struct zskiplistNode {
  node* ele;
  double score;
} zskiplistNode;
```

在调用 ```zrangebysocre```、```zrembysocre```等函数时，由于提前按照```socre```对链表节点进行排序，在 skip-list 中对```score```的查找时间复杂度达成了类似树形数据结构的```log(n)```.

## 2、Why the time complexity of zrem is log(n)
所有能受益于 skip-list 数据结构的算法，都需要保存```score```信息，而类似```zrem```、```zrank```这样的函数，是不具备```score```信息的，因此算法复杂度就和普通链表一样（我这样认为）。在实践过程中，为了提升业务效率，我们曾尝试寻找一个更高效的方式来完成```zrem```操作，但查看官方API时，却发现```zrem```的时间复杂度是```O(log(n))```，这令人匪夷所思。```redis```是怎么实现在不使用```score```的情况下，达成```O(log(n))```的算法复杂度的呢？
除了 skip-list，```zset``` 还维护了一个```element```和```socre```映射的字典：
```
typedef struct zset {
    zskiplist *zsl;
    dict *dict;
} zset;
```
因此```zrem```的执行过程如下：
1. 通过 Hash 运算，从```dict```中获取元素的```score```，平均复杂度为```O(1)```
2. 通过```socre```搜索skip-list，获取元素匹配的元素列表，再与当前要搜索的元素匹配，复杂度```O(log(n))```
