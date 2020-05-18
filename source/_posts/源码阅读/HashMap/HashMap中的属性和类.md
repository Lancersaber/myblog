---
title: HashMap源码阅读
time: 2019-11-14
categories: sourceread
tags: HashMap
---

# HashMap源码阅读
---
在阅读源码前我们应该知道Hashmap的基本数据结构是数组加链表，可以使用下面的图解表示
![2019-11-15 09_00_53-未命名表单.drawio - draw.io.png](https://i.loli.net/2019/11/15/fGaU7TZo4vNhi1C.png)
所以我们可以预测在Hashmap中应该定义了一个数组，

## HashMap中的属性值
* DEFAULT_INITIAL_CAPACITY：默认的容量大小
* MAXIMUM_CAPACITY：最大的容量大小，为2的30次方
* DEFAULT_LOAD_FACTOR：默认的加载因子，大小为0.75
* threshold(临界值)：哈希表内元素数量的阈值，当哈希表内元素数量超过阈值时，会发生扩容resize()。
* transient Node<K,V>[] table：哈希桶，存放链表。 长度是2的N次方，或者初始化时为0.
* TREEIFY_THRESHOLD:当每个节点挂载数超过了该变量时，就将链表这种数据结构该为使用红黑树
* UNTREEIFY_THRESHOLD：当每个节点挂载数少于该数时，就改为使用链表的形式进行存储

## 链表节点Node
在开始之前，我们先看一下挂载在哈希表上的元素，链表的结构：
Map.Entry是在Map接口中内部定义的Entry接口
```
    static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;//哈希值
        final K key;//key
        V value;//value
        Node<K,V> next;//链表后置节点

        Node(int hash, K key, V value, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }

        public final K getKey()        { return key; }
        public final V getValue()      { return value; }
        public final String toString() { return key + "=" + value; }

        //每一个节点的hash值，是将key的hashCode 和 value的hashCode 亦或得到的。
        public final int hashCode() {
            return Objects.hashCode(key) ^ Objects.hashCode(value);
        }
        //设置新的value 同时返回旧value
        public final V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }

        public final boolean equals(Object o) {
            if (o == this)
                return true;
            if (o instanceof Map.Entry) {
                Map.Entry<?,?> e = (Map.Entry<?,?>)o;
                if (Objects.equals(key, e.getKey()) &&
                    Objects.equals(value, e.getValue()))
                    return true;
            }
            return false;
        }
    }

```

## 从头开始
我们先写一个简单的使用hashmap的例子
```
Map<String,String> map=new HashMap<>();
        map.put("s1","v1");
        map.put(null,null);
        map.put("name","mike");
        System.out.println(map);
```

将断点设置第一行，进行debug，看看内部创建map对象的过程
![2019-11-15 12_12_49-HashMapRead _D__垃圾_HashMapRead_ - C__Program Files_Java_jdk1.8.0_144_src.zip!_ja.png](https://i.loli.net/2019/11/15/h81AraY6MbiB5dO.png)
可以看到调用了一个无参构造器，设置了负载因子为默认定义的负载因子

接下来进入put函数

首先会进入下面这个函数
![2019-11-15 12_22_30-HashMapRead _D__垃圾_HashMapRead_ - C__Program Files_Java_jdk1.8.0_144_src.zip!_ja.png](https://i.loli.net/2019/11/15/fGkD4HpxwCSqVaY.png)

这个函数调用了另外的一个函数putVal
我们先来看这个函数的前几行
![2019-11-15 12_27_04-HashMapRead _D__垃圾_HashMapRead_ - C__Program Files_Java_jdk1.8.0_144_src.zip!_ja.png](https://i.loli.net/2019/11/15/5Umz8bHY7ygAC4Q.png)

可以看到，tab是用于接收table的临时变量，变量n是用来记录tab的长度的
由于刚开始table为null，所以需要进入resize()函数。可以确定，resize()函数中有初始化table数组的功能。

下面进入resize()函数中看看
![2019-11-15 12_42_18-HashMapRead _D__垃圾_HashMapRead_ - C__Program Files_Java_jdk1.8.0_144_src.zip!_ja.png](https://i.loli.net/2019/11/15/KbgLeQZF3GE7HC1.png)

定义了下面三个变量
* oldTab：接收原来的table
* oldCap:原来table的容量大小,指数组的大小
* oldThr：原来table的阈值

根据这几个判断条件我们可以总结出以下三种情况
1. oldCap大于0
2. oldCap=0但是oldThr大于0
3. oldCap和oldThr都等于0
显然我们处于第三种情况，此时容量和阈值都使用了默认值。
在5号标记中对数组进行初始化，注意这里使用的是newCap，在6中将其赋值给table。这就完成了初始化操作。

继续putVal()函数中查看
![2019-11-15 13_17_54-HashMapRead _D__垃圾_HashMapRead_ - C__Program Files_Java_jdk1.8.0_144_src.zip!_ja.png](https://i.loli.net/2019/11/15/wgVuxsjk7Yb8NZ9.png)

使用p来接收table[(n-1)&hash]
现在我们来研究一下(n-1)&hash的值
n默认为16，n-1为15，转换为二进制的话为0000 0000 0000 0000 0000 0000 0000 1111
&操作是对应位置都为1结果才为1，所以前28位必定为0，所得结果值在0-15之间，正好符合数组下标的范围。

这个判断表面可能有两种情况
1. 根据hash值得到的下标没有值
2. 根据hash值得到的下标有值

第一种情况直接将传入的结点放入即可。
第二种情况又需要分以下几种情况
1. table[(n-1)&hash]有值且该值的hash值和要插入的结点hash值相同
2. table[(n-1)&hash]值是树的结点，表明待插入的结点需要作为树结点插入
3. 待插入结点以链表形式插入

分别对应2，3，4 框中的代码

现在来看后几行代码
![2019-11-15 15_08_56-HashMapRead _D__垃圾_HashMapRead_ - C__Program Files_Java_jdk1.8.0_144_src.zip!_ja.png](https://i.loli.net/2019/11/15/KhFMqYaT2n6OQVD.png)

if(e!=null)说明插入的位置原本是有值，现在只不过是将值进行了替换，，onlyIfAbsent这个参数的作用在于,如果我们传入的key已存在我们是否去替换，true:不替换，false：替换。
每对hashmap进行修改都会增加modcount的值，每插入一个值size都加1，判断hashmap中值的个数是否到达阈值，如果超过阈值则进行扩容。
至于高亮的那两行代码，并没有具体实现，
![2019-11-15 15_20_48-HashMapRead _D__垃圾_HashMapRead_ - C__Program Files_Java_jdk1.8.0_144_src.zip!_ja.png](https://i.loli.net/2019/11/15/YoxwaZ613umkUB4.png)


putVal()函数就算看完了，下面我们来研究resize()函数剩下的部分
![2019-11-15 15_25_57-HashMapRead _D__垃圾_HashMapRead_ - C__Program Files_Java_jdk1.8.0_144_src.zip!_ja.png](https://i.loli.net/2019/11/15/Sbgoj4WqhXid9CY.png)

如果oldTab不为null，那么我们就必须将oldTab中的Node都转移到新的newTab中，使用for循环来遍历oldTab数组，下面的1，2，3分别代表三种情况，（前提是e不为null，也就是table[j]!=null)
1. 该table[j]就只有一个值，它后面不挂着其它结点
2. 该table[j]后面有结点，且挂的结点是以红黑树的形式存在
3. 该table[j]后面有结点，且挂的结点是以链表的形式存在

我们重点来看第三种形式
将table分为两部分，大小都为原本oldTab的大小，循环遍历该table[j]后面挂的结点，判断该结点是放在前0-oldCap-1的位置还是oldCap-newCap-1的位置。
判断的方式是(e.hash & oldCap) == 0
以oldCap为16为例
16的二进制表示为0000 0000 0000 0000 0000 0000 0001 0000
只要e.hash值的二进制表示的第五位大于16，表示它一定处于oldCap-newCap-1的位置，因为处于0-oldCap-1位置的hash值第五位的值一定是0

说明一下lohead和loTail的作用，lohead表示链表的头部，从代码中也可以看到，刚开始head和tail都指向e，后面由于tail!=null，所以不会修改head，最后设置tail.next=null也符合链表的形式。

至此，HashMap的resize()和putVal()函数就分析完了。