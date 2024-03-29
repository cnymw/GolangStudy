# 概念

LRU 是 Least Recently Used 的缩写，即最近最少使用，是一种常用的页面置换算法，选择最近最久未使用的页面予以淘汰。

该算法赋予每个页面一个访问字段，用来记录一个页面自上次被访问以来所经历的时间 t，当须淘汰一个页面时，选择现有页面中其 t 值最大的，即最近最少使用的页面予以淘汰。

## 哈希链表

- 哈希表是由 key-value 键值对组成的，在逻辑上，这些键值对没有联系，没有前后顺序

- 哈希链表是哈希表和双向链表的组合，双向链表的结点就是键值对，这也就意味着每一个键值对都有前驱和后继

![算法-LRU-哈希链表.png](https://cnymw.github.io/GolangStudy/docs/img/算法-LRU-哈希链表.png)

## LRU 算法

依靠哈希链表的的有序性，可以把键值对按照最后的使用时间来排序。

1. 假设我们使用哈希链表来缓存用户信息，目前缓存了4个用户，这4个用户是按照时间顺序依次从链表右端插入的。

![算法-LRU-哈希链表步骤1.png](https://cnymw.github.io/GolangStudy/docs/img/算法-LRU-哈希链表步骤1.png)

2. 此时，业务方访问用户5，由于哈希链表中没有用户5的数据，我们从数据库中读取出来，插入到缓存当中。这时候，链表中最右端是最新访问到的用户5，最左端是最近最少访问的用户1。

![算法-LRU-哈希链表步骤2.png](https://cnymw.github.io/GolangStudy/docs/img/算法-LRU-哈希链表步骤2.png)

3.我们把用户2从它的前驱节点和后继节点之间移除，重新插入到链表最右端。 这时候，链表中最右端变成了最新访问到的用户2，最左端仍然是最近最少访问的用户1。

![算法-LRU-哈希链表步骤3.png](https://cnymw.github.io/GolangStudy/docs/img/算法-LRU-哈希链表步骤3.png)

4. 接下来，业务方请求修改用户4的信息。 同样道理，我们把用户4从原来的位置移动到链表最右侧，并把用户信息的值更新。 这时候，链表中最右端是最新访问到的用户4，最左端仍然是最近最少访问的用户1。

![算法-LRU-哈希链表步骤4.png](https://cnymw.github.io/GolangStudy/docs/img/算法-LRU-哈希链表步骤4.png)

5 业务方访问用户6，用户6在缓存里没有，需要插入到哈希链表。 假设这时候缓存容量已经达到上限，必须先删除最近最少访问的数据，那么位于哈希链表最左端的用户1就会被删除掉，然后再把用户6插入到最右端。

![算法-LRU-哈希链表步骤5.png](https://cnymw.github.io/GolangStudy/docs/img/算法-LRU-哈希链表步骤5.png)

