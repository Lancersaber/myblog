---
title: TreeMap
time: 2019-11-16
categories: sourceread
tags: TreeMap
---

# TreeMap
---

## TreeMap简介
TreeMap是一个有序的key-value集合，基于红黑树（Red-Black tree）的 NavigableMap实现。该映射根据其键的自然顺序进行排序，或者根据创建映射时提供的 Comparator进行排序，具体取决于使用的构造方法。

## TreeMap中的属性
* private transient Entry<K,V> root:一个Treemap的根节点
* private final Comparator<? super K> comparator;比较器，由于treemap是有序的，所以需要有一个比较的规则
* private transient int size = 0;记录treemap中有多少个结点
* private transient int modCount = 0;记录treemap经过多少次修改
* private static final boolean RED   = false;treemap是基于红黑树这种数据结构的，定义这两个变量用于区分结点是红结点还是黑结点
* private static final boolean BLACK = true;

## TreeMap中比较重要的点

### 1、Entry<K,V>
TreeMap中传入key和value后，内部会将key，value封装为一个Entry,然后将该Entry放入到合适的位置

![2019-11-16 14_01_13-HashMapRead _D__垃圾_HashMapRead_ - C__Program Files_Java_jdk1.8.0_144_src.zip!_ja.png](https://i.loli.net/2019/11/16/1GupYB8kfUQMPX4.png)

可以看到，在Entry中定义了上面几个属性，
* key:存储key值
* value：存储value值
* left：该结点的左结点
* right：该结点的右结点
* parent：该结点的父节点
* color：标识该结点是红结点还是黑结点，默认是黑结点

![2019-11-16 14_08_10-HashMapRead _D__垃圾_HashMapRead_ - C__Program Files_Java_jdk1.8.0_144_src.zip!_ja.png](https://i.loli.net/2019/11/16/r61Dx8Y4iVfkuy9.png)

Entry中需要注意的是它的构造函数，它传入了parent结点，标识当前结点的parent结点，其他的函数没什么。

### 2、使用一个小程序进行debug
```
Map<Integer,String> treemap=new TreeMap<>();
        treemap.put(1,"Mike");
        treemap.put(2,"Jack");
        treemap.put(-99,"hike");
        System.out.println(treemap);
```

在第一行打上断点，进行调试
![2019-11-16 14_23_07-HashMapRead _D__垃圾_HashMapRead_ - C__Program Files_Java_jdk1.8.0_144_src.zip!_ja.png](https://i.loli.net/2019/11/16/xH3sIW16RXiVnDy.png)

可以看到初始化了比较器为null。其余的操作都未进行。下面一张图表示了treemap各个属性的初始值情况
![2019-11-16 14_37_05-HashMapRead _D__垃圾_HashMapRead_ - C__Program Files_Java_jdk1.8.0_144_src.zip!_ja.png](https://i.loli.net/2019/11/16/LFPdWT5VfUtGlD6.png)


继续往下看，进入put方法，在进入put方法之前，我们可以先预测几种情况
1. treemap中root为null，这是第一个插入的值
2. treemap中root不为null，那么需要从root开始根据比较器中的规则进行比较，根据比较结果是否大于0，等于0，小于0来判断该结点的走向
	* 大于0，前往右边
	* 小于0，去往左边‘
	* 等于0，说明这是替换操作
下面我们来验证一下
![2019-11-16 14_30_40-HashMapRead _D__垃圾_HashMapRead_ - C__Program Files_Java_jdk1.8.0_144_src.zip!_ja.png](https://i.loli.net/2019/11/16/GFejvIiQh1roLtU.png)

使用t来接收root值，防止更改root值造成混乱。可以看到，如果是第一次插入值，root为null，那么这个值就是root的初始值，注意高亮的部分调用了一个compare方法

![2019-11-16 14_38_27-HashMapRead _D__垃圾_HashMapRead_ - C__Program Files_Java_jdk1.8.0_144_src.zip!_ja.png](https://i.loli.net/2019/11/16/3f8wIQLJz5XNHKu.png)

注意看画线的部分，是将传入的key转化为Comparable类型然后调用compareTo()方法，这表明如果没有传入可用的Comparator，那么传入的key值就必须实现了Comparable接口。否则会报下面的错误

![2019-11-16 14_42_43-HashMapRead _D__垃圾_HashMapRead_ - ..._src_com_lancer_package1_Test.java _HashMap.png](https://i.loli.net/2019/11/16/7I1MOcjhPUmxrt4.png)

好，到现在我们已经验证了第一种情况，下面我们继续看put函数中后面的代码

![2019-11-16 14_51_27-HashMapRead _D__垃圾_HashMapRead_ - C__Program Files_Java_jdk1.8.0_144_src.zip!_ja.png](https://i.loli.net/2019/11/16/RPX82WSwVZsQyzU.png)

如果root不为null，那么执行这段代码，首先定义了三个值，
1. cmp：用于保存比较的结果
2. parent：用于保存父节点
3. cpr：用于保存比较器

后面根据比较器是否使用默认的分为两种情况，如果使用自定义的比较器，则进入块1，使用默认的比较器就进入块2，显然我们没有显示地传入构造器，因此进入else块中。
首先判断key值是否为null，这也表明key不能为null。将key转成Comparable类型赋值给cpr，根据比较结果选择进入左子树遍历还是右子树遍历，一通循环后，退出循环时待插入的结点的父节点就是parent结点。

来看最后的一点代码
![2019-11-16 15_03_15-HashMapRead _D__垃圾_HashMapRead_ - C__Program Files_Java_jdk1.8.0_144_src.zip!_ja.png](https://i.loli.net/2019/11/16/siRpFocXGfqC2Ij.png)


将待插入结点转换为一个Entry结点，与parent的key值进行比较，判断是插入到parent的左边还是右边。修改size和modCount的值，在这里面调用了fixAfterInsertion()函数，这个函数就是将修改后的tree转换化一棵红黑树，搞懂了这个函数，对treemap源码可以说就理解2/3了。

### 红黑树
除了二叉树的基本要求外，红黑树必须满足以下几点性质。
* 节点必须是红色或者黑色。
* 根节点必须是黑色。
* 叶节点(NIL)是黑色的。（NIL节点无数据，是空节点）
* 红色节点必须有两个黑色儿子节点。
* 从任一节点出发到其每个叶子节点的路径，黑色节点的数量是相等的。

我们先来看一下平衡二叉树的旋转

#### 平衡二叉树的旋转
![20170429150914755.png](https://i.loli.net/2019/11/16/wZYeTvSs8PWIpEa.png)
![20170429151109740.png](https://i.loli.net/2019/11/16/k1Iai9rnNyzuYcM.png)
注意在treemap中，第二种情况会先化为第一种情况
![2019-11-16 16_40_58-未命名表单.drawio - draw.io.png](https://i.loli.net/2019/11/16/YQTbPX4dSKHv1GB.png)
然后再使用第一种情况来进行旋转

我们来看一下再treemap中右旋转的代码
![2019-11-16 16_43_35-HashMapRead _D__垃圾_HashMapRead_ - C__Program Files_Java_jdk1.8.0_144_src.zip!_ja.png](https://i.loli.net/2019/11/16/HhgOPRkQVKE5CDl.png)

这个代码可以分为四部分
其中传入的参数为最上面的结点p(parent)，也就是上面40那个结点。
* 第一部分获得传入结点的左子结点l(left)。
* 第二部分将r的右子结点赋予给p，作为p的左子结点，这是因为r最终的右子结点是p，这会导致原来的右子节点丢失，所以必须将它们转移至p的左子结点，因为它们只有小于p才会位于左侧。
* 第三部分设置r的parent为p的parent，并判断r是p.praent的左子结点还是右子结点。这个判断对应得模型为下面三种模型
![2019-11-16 16_58_16-未命名表单.drawio - draw.io.png](https://i.loli.net/2019/11/16/QF3TYaEqXZdzmny.png)
![2019-11-16 17_01_35-未命名表单.drawio - draw.io.png](https://i.loli.net/2019/11/16/AIt4oev6bMdUqsK.png)
![2019-11-16 18_30_50-未命名表单.drawio - draw.io.png](https://i.loli.net/2019/11/16/y9lACvc6tkbGo3U.png)

* 第四部分将p设为r的右子结点，r设为p的父节点

左旋转代码类似

回到fixAfterInsertinon()函数
![2019-11-16 18_41_28-HashMapRead _D__垃圾_HashMapRead_ - C__Program Files_Java_jdk1.8.0_144_src.zip!_ja.png](https://i.loli.net/2019/11/16/L9nY28csilbtM5a.png)

之所以先设置每个节点颜色的初始值为red，还是因为需要保证从根节点到每个叶子节点经过的黑节点数目必须相等，因此增加红色节点不影响，还有就是旋转时将x的parent设置为black，给x的parent的parent设置为red是为了到达旋转后黑节点下面挂着两个红节点，这样也不会破坏条件。后半部分代码类似，只不过是针对x的parent位于右子节点时的情况。

