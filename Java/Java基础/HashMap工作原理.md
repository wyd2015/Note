# HashMap实现原理
## 一、概述
HashMap基于Map接口实现；  
- 允许键、值为Null；  
- 非同步，线程不安全；  
- 不保证有序，也不能保证顺序不随时间变化（扩容时会重新计算buckets的索引值）；
- 以键值对的形式存储数据，使用put(key, value)方法添加，使用get(key)取值。
## 二、两个重要参数
- capacity: 容量，就是buckets的数目，默认16；
- load factor: 负载因子，buckets填满程度的最大比例，默认为0.75。
>如果对迭代性能要求很高，不能把capacity设置过大，也不能把load factor设置过小。当buckets填充的数目（hashMap中元素的个数）大于`capacity * load factory`的乘积时，就需要调整buckets的数目为当前capacity值的2倍。
## 三、put方法实现
大致思路：  
1. 对key的hashCode()做hash，再计算槽的index：`(table.length-1) & hash`，以此获得buckets的位置; 
2. 如果没碰撞直接放到bucket里；
3. 如果碰撞了，以链表的形式存放在buckets后；
4. 如果碰撞导致链表过长(大于等于TREEIFY_THRESHOLD)，就把`链表`转换成`红黑树`；
5. 如果节点已经存在就替换old value(保证key的唯一性)
6. 如果bucket满了(超过load factor*current capacity)，就要resize。
## 四、get方法实现
1. bucket里的第一个节点，直接命中；
2. 如果有冲突，则通过key.equals(k)去查找对应的entry：
若为树，则在树中通过key.equals(k)查找，O(logn)；
若为链表，则在链表中通过key.equals(k)查找，O(n)。
>在Java 8之前的实现中是用链表解决冲突的，在产生碰撞的情况下，进行get时，两步的时间复杂度是O(1)+O(n)。因此，当碰撞很厉害的时候n很大，O(n)的速度显然是影响速度的。
因此在Java 8中，利用红黑树替换链表，这样复杂度就变成了O(1)+O(logn)了，这样在n很大的时候，能够比较理想的解决这个问题