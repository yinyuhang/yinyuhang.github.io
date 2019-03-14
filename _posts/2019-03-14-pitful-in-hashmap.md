---
layout: post
title: "HashMap的\"bug\""
subtitle: "随记"
date: 2019-03-14 10:42:05
tag: 
    - java
---

如果面试官问你，如何修改集合中的值，你会怎么回答？ 直接entrySet,keySet遍历然后Map::remove？那你可能要和眼前的工作擦肩而过了。不过相信大家都踩过这个坑，这样遇到老朋友`ConcurrentModificationException`。emmm，那么筛选删除两步走、Java8的Stream？恭喜你，这是正确答案。

那么，我想问的是，Iterator是支持遍历时删除元素的，那么，`HashMap.keySet().iterator()`能否用于处理集合呢？

今天我要分享的就是，在使用过程中遇到的一个"bug"。

---------------------------------------

现有代码
```
1  Map<Map<String, String>, String> map = new HashMap<>();
2  map.put(new HashMap<>(), "value");
3  System.out.println("Before:" + map.size());
4  Iterator<Map.Entry<Map<String, String>, String>> iterator = map.entrySet().iterator();
5  while (iterator.hasNext()) {
6      iterator.next().getKey().put("123", "1a23");
7      //  iterator.next().getKey().put("123", "123");
8      iterator.remove();
9  }
10 System.out.println("After:" + map.size());
```

输出如下
```
Before:1
After:1
```

但如果把第6行注释掉，然后解开第7行注释，那么输出会变为
```
Before:1
After:0
```

嗯？？？发生了什么，什么时候JDK也变成了朝三暮四的"小人"了。讲道理，我的引用又没变，凭什么同样是赋值，多个A就棍一些？ 

不过我们先冷静，按往常的惯例来说，自己和编译器二选一，那一定是自己错了。

那我们先来整理一下思路。
* 第一点：`Iterator.remove`会影响到`HashMap`的值
* 第二点：不同的值`Iterator.remove`对`HashMap`的影响不一致

已知HashMap是根据equals和hashCode来进行元素判同，大胆猜测一下，是否与这个有关？

那么，我们先来跟到源码里看一下(笔者这里用的是JDK1.8u191)，现在针对示例代码进行调试，在第8行打上断点。通过反射我们找到了对应的remove方法代码在`HashMap$HashIterator`中，代码如下：
```
public final void remove() {
    Node<K,V> p = current;
    if (p == null)
        throw new IllegalStateException();
    if (modCount != expectedModCount)
        throw new ConcurrentModificationException();
    current = null;
    K key = p.key;
    removeNode(hash(key), key, null, false, false);
    expectedModCount = modCount;
}
```

这里的`removeNode`调用的是`HashMap.removeNode`，对应的关键代码如下:
```
// 这里对代码进行了排版，并加上了自己的理解注释
final Node<K,V> removeNode(int hash, Object key, Object value, boolean matchValue, boolean movable) {
    Node<K,V>[] tab; Node<K,V> p; int n, index;
    // 当前Map不为空，且根据hash来看，Map中可能会有该元素
    if ((tab = table) != null && (n = tab.length) > 0 && (p = tab[index = (n - 1) & hash]) != null) {
        Node<K,V> node = null, e; K k; V v;
        // 根据hashcode与equals来进行元素判同
        // 蛛丝1号
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            ......
        else if ((e = p.next) != null) {
            ......
        }
        if (node != null && (!matchValue || (v = node.value) == value || (value != null && value.equals(v)))) {
            ......
        }
    }
    return null;
}
```

跟到这里，读者发现再往下就到了HashMap如何去组织数据了，那么现在尝试使用调试器进行分析。这里我发现，在"1号蛛丝"位置，当`put("123", "123")`时，p.hash与hash为0，当`put("123", "123a")`时，p.hash为0，hash不为0。那么我们再返回上一层函数，找到计算hash的代码如下:
```
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

发现又绕回了`HashMap.hashCode()`，然后找到具体的代码实现在`AbstractMap`中如下
```
public int hashCode() {
    int h = 0;
    Iterator<Entry<K,V>> i = entrySet().iterator();
    while (i.hasNext())
        h += i.next().hashCode();
    return h;
}
```

原来是把HashMap中所有的Entry的HashCode累加起来。然后找到HashMap.Entry并找到对应代码:
```
public final int hashCode() {
    return Objects.hashCode(key) ^ Objects.hashCode(value);
}
```

呼，万恶之源，就是他了。本质上，这是因为hashCode碰撞导致的一个大乌龙。简单来说：
```
Map<String, String> map = new HashMap<>();
System.out.println("empty map:" + map.hashCode());
// empty map:0
map.put("1", "1");
System.out.println("碰撞了:" + map.hashCode());
// hashCode 碰撞了:0
map.clear();
map.put("12", "1");
// hashCode:1552
System.out.println("hashCode:" + map.hashCode());
```

#### 总结
**下面所有提到的Iterator都是通过`HashMap.entrySet().iterator()`获取的`HashIterator`**

`Iterator.remove`方法，底层会调用`HashMap.removeNode`方法。而作为`HashMap`，它是以`hashCode`与`equals`进行判同。所以当我们在遍历`Iterator`时，如果修改了entry中key的值，再把它从迭代器中移除，会分以下两种情况
* 修改前后的`key.hashCode`一致，则成功移除
* 修改前后的`key.hashCode`不一致，则无法正常移除

今天就先写到这，其实这里还有几个问题
* HashCode的碰撞，在该场景下，会污染Map中的元素么？
* Entry.hashCode为什么要键值异或？看不懂啊混蛋~~

如有不对，希望大家不吝赐教
