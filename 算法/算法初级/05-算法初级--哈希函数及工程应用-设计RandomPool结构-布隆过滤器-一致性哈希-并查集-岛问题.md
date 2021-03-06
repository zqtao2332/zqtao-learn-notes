**年轻即出发**...

**简书**：https://www.jianshu.com/u/7110a2ba6f9e

**知乎**：https://www.zhihu.com/people/zqtao23/posts

**GitHub源码**：https://github.com/zqtao2332

**个人网站**：http://www.zqtaotao.cn/  （停止维护更新内容，转移简书）

**QQ交流群**：606939954

	    咆哮怪兽一枚...嗷嗷嗷...趁你现在还有时间，尽你自己最大的努力。努力做成你最想做的那件事，成为你最想成为的那种人，过着你最想过的那种生活。也许我们始终都只是一个小人物，但这并不妨碍我们选择用什么样的方式活下去，这个世界永远比你想的要更精彩。



最后：喜欢编程，对生活充满激情

------

------

**本节内容预告**

实例1：理解哈希函数和哈希表

实例2：设计RandomPool结构

实例3：认识布隆过滤器（处理大数据）

实例4：认识一致性哈希。

实例5：

实例6：岛问题

------

------



**实例1**

理解哈希函数和哈希表

使用一个哈希函数随机得到1000个哈希函数，同时需要相对独立

做法：通过一个哈希函数，得到一个16字节哈希码，把结果分为两份h1,h2

前八个字作为h1,后8个字节作为h2.

h1 + i * h2 = hn

如

h1 + 1 * h2  = h3

h1 + 2 * h2 = h3

....

这里就是将一个完全独立的哈希函数切割得到的两个哈希码作为种子。

因为哈希函数得到的哈希码的16位，每一   位都是相对独立的。

哈希函数特点：相同输入，导致相同输出，不同输入，导致输出均匀分布。

哈希表：就是哈希函数的桶，如果桶的效率开始变差，就把整体扩容。其中默认哈希表的增删改查都是O(1)



**实例2**

设计RandomPool结构 

【题目】 设计一种结构，在该结构中有如下三个功能： 

insert(key)：将某个key加入到该结构，做到不重复加入。 

delete(key)：将原本在结构中的某个key移除。

 getRandom()： 等概率随机返回结构中的任何一个key。 

【要求】 Insert、delete和getRandom方法的时间复杂度都是 O(1)

思路：准备两个HashMap

map1

key：要存储的值

value：入结构的顺序

map2

key：入结构的顺序

value：要存储的值



size：记录入RandomPool的数量



insert(key)：每次新增的同时size++

getRandom：通过系统的random方法冲size个数中选择一个，通过map2返回。

delete(key)：例如删除c,则找到2号位，删除，同时将2号位替换为map2中的最后一个999所对应的元素，同时更新map1。


![2_1_randompool.png](https://upload-images.jianshu.io/upload_images/18567339-c4fe2c870134b166.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


```
import java.util.HashMap;

/**
 * @description:
 *
 * @version: 1.0
 */
public class Code_32_RandomPool {
    public static class RandomPool<K>{
        private HashMap<K, Integer> key_index;
        private HashMap<Integer, K> index_key;
        private int size;

        public RandomPool(){
            this.key_index = new HashMap<>();
            this.index_key = new HashMap<>();
            size = 0;
        }

        public void insert(K key){
            if (!this.key_index.containsKey(key))
            this.key_index.put(key, this.size);
            this.index_key.put(this.size++, key);
        }

        public K getRandom(){
            if (this.size == 0){
                return null;
            }
            int key = (int) (Math.random() * this.size);
            return this.index_key.get(key);
        }

        public void delete(K key) {
            if (this.key_index.containsKey(key)){
                int deleteIndex = this.key_index.get(key);
                int lastIndex = --this.size;
                K lastKey = this.index_key.get(lastIndex);
                this.key_index.put(lastKey, deleteIndex);
                this.index_key.put(deleteIndex, lastKey);
                this.key_index.remove(key);
                this.index_key.remove(lastIndex);
            }
        }
    }

    public static void main(String[] args) {
        RandomPool<String> pool = new RandomPool<>();
        pool.insert("test");
        pool.insert("fuck");
        pool.insert("code");
        System.out.println(pool.getRandom());
        System.out.println(pool.getRandom());
        System.out.println(pool.getRandom());
        System.out.println(pool.getRandom());
        System.out.println(pool.getRandom());
        System.out.println(pool.getRandom());

    }
}

```

**实例3**

认识布隆过滤器（处理大数据）

**先看一个经典的问题**

100亿的URL黑名单，当你搜索这些黑名单的时候，不想给你显示这个URL的内容。
假设每一个黑名单（URL）都是64字节，当用户搜索某一个URL在黑名单的时候，对它进行过滤。

简单说了就是给你一个URL，在黑名单，返回true，不在返回false。



**分析存储**

如果使用一个哈希表存储，需要多大的空间？
这里哈希表只需要存key,可以使用 HashSet

不算索引空间等其他空间占用，就单单100亿URL，最少需要64 * 100亿 个字节才能把所有的数据存储下，
差不多需要640G内存，这个设计需要花费巨大的代价。



**换一种存储结构？**

这里考虑能不能使用另一种结构来存储数据，它能够将使用空间极度的减小，
让我们用一台机器就足以支撑？



再次之前先了解一下布隆过滤器的概念。

**布隆过滤器**

布隆过滤器就是某种类型的集合，概率型数据结构，但是它是有**失误率**的。
布隆过滤器失误率：如URL**不在里面，但是返回在里面**，这种失误不是另一种情况：URL在，返回false；

布隆过滤器可以理解为：比特类型的Map。



**面试过程**

先了解一下面试过程：

面试官：。。。上诉题。。100亿的黑名单URL。。。
应聘者：首先想到的是哈希的分流，分给不同的机器，每一台机器处理一部分，让整个集群来支撑处理这100亿的数据；
面试官：这样做是不是太浪费空间了，你能不能在想想办法？
应聘者：（这时候你可以问一下面试官）你们这个系统允许有很低的失误率吗？
面试官：（这时候一般都会微笑一下，小伙子很上道）可以！
应聘者：可以使用布隆过滤器。



这里根据面试过程对话，我们了解到，面试时，先和面试官讨论的是一般情况的处理方式：**利用哈希函数分流**

但是这样做代价很高！所以很自然的引到了**布隆过滤器**。



**举例子**：怎么做出长度为0~m-1的比特类型的数组？？？

假设一个数组（0，m-1），每一个位置上不是整数、字符串，这个数组的每一个位置上，
它是**比特**，即每一个位置上只有两种状态**0和1**。

- int[]  int 类型4个字节，32个比特
- 一个整数可以表示32个比特
- 如果声明一个长度1000的数组，它其实可以表示1000 * 32 个比特位



**下面是一个长度为1000的数组，30000需要进入那个比特位？**的存储过程。

![3_1_(0,m-1)范围的比特数组.png](https://upload-images.jianshu.io/upload_images/18567339-cb9d738ff42b16f8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)





```
int[]  int 类型4个字节，32个比特
一个整数可以表示32个比特
如果声明一个长度1000的数组，它其实可以表示1000 * 32 个比特位


int[] arr = new int[1000]; // 可以表示32000个比特位，每一个arr[i] 表示一个大桶,一个大桶表示有32个比特位

int index = 30000; // 想要把30000位描黑

int intIndex = index / 32; // 计算30000在哪个大桶;intIndex = 

int bitIndex = index % 32; // 计算30000具体在某个大桶里面哪一个比特位

System.out.println(intIndex); // 937
System.out.println(bitIndex); //16
这里总结就是：30000比特位在937号桶的16号比特位，这个位置需要被描黑
 1 << 16  0000 0000 0000 0001 0000 0000 0000 0000 // 表示将第16号位置置为1，其他为0
arr[intIndex] = (arr[intIndex] | (1 << bitIndex));
```



**做一个比特数组**

```
做比特类型的数组
（拿基础类型拼，这里用int，32位，当然如果你想要省空间可以使用long类型，64位）
如果还嫌空间大，可以使用矩阵


long[][] map = new long[1000][1000]
map 可以表示的比特位64 * 1000 * 1000
```



前面了解了如何去做一个比特数组。问题来了，有了比特数组，**黑名单问题该怎么设计呢？**

**插入100亿URL过程：**
	

```
准备一个 (0 , m-1) 的比特数组
进入一个URL，处理过程
URL 
--> 哈希函数 
--> code  
--> code % m 
-->  得到 (0, m-1) 中的一个比特位置 
--> 描黑

准备k 个哈希函数  
--->  重复URL算比特位置   
---> 描黑（有可能两个哈希函数会达到同一个位置，都描黑）  
--> 到这里，这个URL算是进入到了比特数组了
```

图解URL插入过程



![3_2_100亿URL布隆过滤器设计数据插入过程.png](https://upload-images.jianshu.io/upload_images/18567339-e62774fbc7cba065.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




**查URL是否是黑名单过程：**
	

```
URL 
-->  依然经过 k 个哈希函数  
-->  哈希出k 个位置  
-->  如果k 个位置都是描黑过的，则URL在黑名单里。

注意：这个比特数组要大一些，不然100亿个URL进入到比特数组，数组都满了，还查个屁，都黑了，不是说明任意一个URL进来都是黑名单了吗？
```



**问题：比特数组开多大跟哪些因素有关？**

两个因素：样本量和预期失误率

**1、样本量有关**
	和具体的单个样本字节无关，有关的是样本种类。这里本题指的就是URL的种类
**2、预期的失误率有关**
	 如果100亿数据，样本的失误率只要求做到1/100，那么数组的长度就无需太长。
	 哈希函数比特类型的数组长度  m = - ((N*lnp) / ((ln2)^2))

- 	 N：样本量
- 	 p：失误率

	 实际用计算器计算一下：N=100亿，p=0.0001, m=(131,571,428,572)（比特）
	 实际需要的内存空间:   m / 8（字节）=22.3G

 所以原来需要640G内存的空间，现在只需要22.3G

**布隆过滤器的大小m公式**

![3_3_布隆过滤器的大小m公式.png](https://upload-images.jianshu.io/upload_images/18567339-9e6db275c90cf19b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


**哈希函数的个数k公式**

![3_4_哈希函数的个数k公式.png](https://upload-images.jianshu.io/upload_images/18567339-7df3da3d4f425ce6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



**布隆过滤器真实失误率p公式**

![3_5_布隆过滤器真实失误率p公式.png](https://upload-images.jianshu.io/upload_images/18567339-940445539e053f5f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




**这里可以根据实际情况来进行优化，较低失误率！！！**

为什么用 k 个哈希函数？
	实际过程中需要k 函数个数：
	k = ln2*(m/n) = 0.7*(m/n) = 12.7       k 向上取整 **约13**
	当k 向上取整时，布隆过滤器的真实失误率p会和预期值不一样
	一个结论：如果想要做到万分之一的失误率

- 		m 内存22.3 ---> 24G 
- 		k   12.7 ---> 13
- 		算的真实p:约 6/10万  十万分之六



**优化总结**：基本求值做完后，在面试官允许的范围内可以适当的增大 m（比如这题，他给你32G内存，你算完了，只需要22.3G，可以提升到24） ,带入k 计算公式，计算出k 后向上取整（比如这道题原来计算 m = 24k= 12,7，向上取整后13），带入第三个公式求真实的p计算出来的值会变小（比如这道题，m=24，k=13,p=约十万分之六，远小于原来的万分之一）



**实例4**

认识一致性哈希。

一致性哈希在集群上使用的特别多，经典哈希服务器相对存在的问题，一致性哈希可以较为有效的解决。

先说一下经典服务器存在的问题在哪里

新增新机器，删除新机器代价非常大。



**举例：经典服务器的抗压结构**

服务操作：查"test" 的 value 是什么？

前端集群：无差别接收request
后端集群组：3机器编号： 0 1 2

**插入数据和查询数据**都是一样的。因为哈希函数的性质：同入同出，异入均匀分布。

前端各自带相同的一个哈希函数  --->  code    ---> code % 3   ---> 得到哪一台机器

**经典负载均衡如图**

![4_1_经典负载均衡.png](https://upload-images.jianshu.io/upload_images/18567339-c89c9216da23e98c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




问题来了！！！

想要**增加机器**或者是**去除机器**的时候就完蛋了！！！
原来有3台机器，哈希码%3 ，现在因为需求增加，需要5台机器，哈希码%5
原来("zqt",88) 这个消息 %3 比如在机器2上，现在必须重新 %5来确定数据需要往哪台机器上迁移。这个代价是非常大的。

这就引出了**一致性哈希结构**，它能够把**数据迁移的代价降到很低**，同时实现**负载均衡**。



**一致性哈希结构实现负载均衡的实现**

假设hash 函数h1,  h1 哈希出来的哈希码 的范围是0 ~ 2^64 ，把这个范围想象成均匀分布的环。

将3台机器的IP利用哈希函数 h1 计算出来的哈希码对应到环。code0 code1 code2

**消息处理**

一个request消息
--->  前端集群（使用同一个哈希函数 h1）
--->  哈希码code（不需要像经典抗压集群一样 模3）
--->  查看在哈希范围环中code 距离3台机器的哈希值（code0、code1、code2）哪一台最近（顺时针），如code2
--->  将消息发送到code2对应机器2

![4_2_哈希范围和机器分布.png](https://upload-images.jianshu.io/upload_images/18567339-41a97599ee8f2365.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


注意：如图
code1~code2 范围，如图红色区域，消息发送到---> 2号机器
code2~code0 范围，如图黄色区域，消息发送到---> 0号机器
code0~code1 范围，如图绿色区域，消息发送到---> 1号机器



**具体实现查找哪一台机器过程？**

1、首先IP（也可以是其他的标识机器的属性）进行哈希，将哈希值排序存进数组如：[c2,c0,c1]，
将数组发送到每一台前端集群上的机器。
2、前端集群接收到消息，也进行哈希得到code
3、采用二分法等算法查找数组。这里二分法找到第一个>=code 的哈希位置，如c0

例如 5台机器哈希排序数组[m0, m1, m2, m3, m4]，这里为了直观看出效果，哈希值使用整数
[22, 65, 132,387,643]

消息("zqt") 计算的哈希码  code = 99

**二分法**找到第一个大于等于39 的哈希码  65  它对应的机器就是访问机器

**新增机器**

新增一个机器 3
如图：只需要把code3~code2之间的数据从0机器迁移给3机器就行了，不需要动其他机器的数据
所以一致性哈希集群新增和删除机器代价都是非常小的。

![4_3_一致性哈希集群新增机器.png](https://upload-images.jianshu.io/upload_images/18567339-2d1ed0aa0069b1fa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




**一致性哈希存在的问题**：在机器数量比较少的时候，不能保证机器映射到环是均匀分布的！
这个问题跟哈希函数的性质有关。哈希函数保证的均分是在**样本量多**的情况下。

**解决方式**：虚拟节点技术

3台机器
m-0:给m-0分配1000个虚拟节点
m-1:  ...
m-2:  ...

增加一张路由表映射 虚拟节点和真实机器
利用这3000 个虚拟节点来消费消息，可以说3个机器消费数据是差不多的。

**再看增加新机器3**
也是增加了1000个虚拟节点，均匀分布在哈希环上。有人说新增虚拟节点覆盖在老机器的虚拟节点怎么办？
答案是：给新机器（其实这个概率**非常低**，哈希取值范围非常大，4000个节点相遇概率实在太低了，所以你就算让新机器和老机器共用也是在容错率范围内的允许的）。
同理，删除一台机器是一样的。

**用虚拟节点的数量，以比例来化解一致性哈希集群无法负载均衡的问题。**

**实例5**

并查集：并查集类似一棵多枝树，有一个集合代表节点的pre指向自己，其他节点的pre指向上一个节点。

**并查集结构**

![5_1_并查集结构.png](https://upload-images.jianshu.io/upload_images/18567339-f949dfd9864df65d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


并查集，并查集两个重要作用
	1、两个元素是否属于同一个集合。
	2、两个元素各自所在的集合，合并他们。

**并查集注意事项**

并查集在初始化的时候必须先把**所有的数据样本**给它，不能动态的加载。

**查**
isSameSet(A,B)
查A，B是否属于同一个集合，只需要向上查，查到代表节点止，比较A，B指向的代表节点是否为同一个。
例如：isSameSet(3,4)，如图
3 向上查 3 -->2 -->1 
4 向上查 4 -->1
代表节点相同，A，B属于同一个并查 集。

**合并**

union(A,B)

合并两个元素所在的集合，合并时查看两个集合长度，短的挂载到长的身上。如图set1 size=4；set2 size=3；
set2 挂载到set1的代表节点上

**合并过程如图**

![5_2_合并.png](https://upload-images.jianshu.io/upload_images/18567339-fbcf0f7aaea7993c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)






**优化**（并查集唯一可以优化的地方）：扁平化
每一次涉及到查，都将长链打散成短链，挂载到代表节点上：如查7

![5_3_并查集结构优化.png](https://upload-images.jianshu.io/upload_images/18567339-8383c2352e76de5f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




查询和合并次数，平均时间复杂度O(1)。

```
import java.util.HashMap;
import java.util.List;

/**
 * @description: 并查集
 * @version: 1.0
 */
public class Code_34_UnionFind {

    public static class Node {
        // whatever you like
    }

    // 并查集结构
    public static class UnionFindSet {
        public HashMap<Node, Node> fatherMap; // 存储节点的父节点
        public HashMap<Node, Integer> sizeMap; // 存储节点所在的集合的节点数size

        public UnionFindSet(List<Node> nodes) { // 并查集在初始化的时候需要一次性给定所有的数据，不能后期动态加载
            this.fatherMap = new HashMap<>();
            this.sizeMap = new HashMap<>();
            this.initSet(nodes);
        }

        private void initSet(List<Node> nodes) {
            // 每次初始化先清空原集合
            this.fatherMap.clear();
            this.sizeMap.clear();

            for (Node node : nodes) {
                fatherMap.put(node, node);
                sizeMap.put(node, 1);// 初始每一个元素，单成一个集合结构
            }
        }

        // 递归找父节点，同时扁平化
        private Node findHead(Node node) {
            Node father = fatherMap.get(node);
            if (father != node) { // 不是代表节点
                father = findHead(father);
            }
            fatherMap.put(node, father);// 扁平化，将递归沿途找到的节点都指向代表节点
            return father;
        }

        // 两个元素是否属于同一个集合
        public boolean isSameSet(Node a, Node b) {
            return findHead(a) == findHead(b);
        }

        // 合并a b 所在的集合
        public void union(Node a, Node b) {
            if (a == null || b == null) return;

            Node headA = findHead(a);
            Node headB = findHead(b);

            if (headA != headB) {
                int sizeA = sizeMap.get(headA);
                int sizeB = sizeMap.get(headB);
                if (sizeA <= sizeB) {
                    fatherMap.put(headA, headB);
                    sizeMap.put(headB, sizeA + sizeB);
                } else {
                    fatherMap.put(headB, headA);
                    sizeMap.put(headA, sizeA + sizeB);
                }
            }
        }
    }
}
```



**实例6**

岛问题
一个矩阵中只有0和1两种值，每个位置都可以和自己的上、下、左、右 四个位置相连，如果有一片1连在一起，这个部分叫做一个岛，求一个 矩阵中有多少个岛？
举例：

0 0 1 0 1 0 

1 1 1 0 1 0 

1 0 0 1 0 0

 0 0 0 0 0 0

这个矩阵中有三个岛。

![6_1_岛问题圈岛方案.png](https://upload-images.jianshu.io/upload_images/18567339-0c832195d8a0b527.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


思路：病毒感染原理，将 1 上下左右四个区域的 1 都感染为2 ，每感染一次，num++；

如图思路分解：

以下是一个岛分布图

![6_2_原问题.png](https://upload-images.jianshu.io/upload_images/18567339-9daa4b616fe577c1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


利用感染函数对连续区域的1 进行感染，扩散形式如图



![6_3_感染.png](https://upload-images.jianshu.io/upload_images/18567339-9886848ae89bdb08.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


具体实现

```
/**
 * @description: 岛问题
 * 一个矩阵中只有0和1两种值，
 * 每个位置都可以和自己的上、下、左、右 四个位置相连，
 * 如果有一片1连在一起，这个部分叫做一个岛，
 *
 * 求一个 矩阵中有多少个岛？
 * 举例：
 *
 * 0 0 1 0 1 0
 *
 * 1 1 1 0 1 0
 *
 * 1 0 0 1 0 0
 *
 * 0 0 0 0 0 0
 *
 * 这个矩阵中有三个岛。
 * @version: 1.0
 */
public class Code_35_Islands {

    public static int countIslands(int[][] m){
        if (m == null || m[0] == null) {
            return 0;
        }

        int r = m.length; // 矩阵行数
        int c = m[0].length; // 矩阵列数

        int res = 0;
        for (int i = 0; i < r; i++) {
            for (int j = 0; j < c; j++) {
                if (m[i][j] == 1){
                    res++;
                    infect(m, i, j, r, c);// 感染函数，将每一个感染点置为2
                }
            }
        }
        return res;
    }

    /**
     * 感染函数，递归感染连续的 1 区域
     *
     * 按照当前点的上下左右进行扩散感染
     *
     * 终止条件
     * 1、不要越界
     * 2、当前点已经被感染 为 2
     * 3、当前点为 0
     *
     * @param m 矩阵
     * @param i 点横坐标
     * @param j 点纵坐标
     * @param r 矩阵行数
     * @param c 矩阵列数
     */
    public static void infect(int[][] m, int i, int j, int r, int c){
        if (i < 0 || i >= r || j < 0 || j >= c // 坐标不越界
                || m[i][j] != 1 // 当前点0或2，不是1
        ){
            return; // 不需要感染
        }

        m[i][j] = 2; // 感染当前点

        infect(m, i + 1, j, r, c);
        infect(m, i - 1, j, r, c);
        infect(m, i, j + 1, r, c);
        infect(m, i, j - 1, r, c);
    }

    public static void main(String[] args) {
        int[][] m1 = {  { 0, 0, 0, 0, 0, 0, 0, 0, 0 },
                { 0, 1, 1, 1, 0, 1, 1, 1, 0 },
                { 0, 1, 1, 1, 0, 0, 0, 1, 0 },
                { 0, 1, 1, 0, 0, 0, 0, 0, 0 },
                { 0, 0, 0, 0, 0, 1, 1, 0, 0 },
                { 0, 0, 0, 0, 1, 1, 1, 0, 0 },
                { 0, 0, 0, 0, 0, 0, 0, 0, 0 }, };
        System.out.println(countIslands(m1));

        int[][] m2 = {  { 0, 0, 0, 0, 0, 0, 0, 0, 0 },
                { 0, 1, 1, 1, 1, 1, 1, 1, 0 },
                { 0, 1, 1, 1, 0, 0, 0, 1, 0 },
                { 0, 1, 1, 0, 0, 0, 1, 1, 0 },
                { 0, 0, 0, 0, 0, 1, 1, 0, 0 },
                { 0, 0, 0, 0, 1, 1, 1, 0, 0 },
                { 0, 0, 0, 0, 0, 0, 0, 0, 0 }, };
        System.out.println(countIslands(m2));
    }
}

```



**思考**：将上述岛问题上升到面试常常考察的大数据问题。当矩形的数据量非常大，单靠一台机器，一个cup已经无法短时间内解决这个求解任务怎么办？



**多任务解决岛问题**
一个特别大的矩阵，单台机器，单CPU就显得不上道了。

多任务并行计算

拆分，合并。

先看拆分，将原问题分解成子问题，分别在不同机器上求解子问题的解，求解完后再进行结果合并。

![6_4_多任务并行计算png.png](https://upload-images.jianshu.io/upload_images/18567339-16d4b65463a32566.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


最重要的是合并过程，容易出现岛的数量计算错误。对于到的合并问题，要处理的就是岛边界。根据岛边界计算两个岛合并时真实存在的岛数量。例如下图一个C形岛，分为两个子问题求解时，两个子问题求得共有3座岛，合并边界不处理好，返回的岛数量将不是1座。

![6_5_合并问题.png](https://upload-images.jianshu.io/upload_images/18567339-86c4441a20a4cf54.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


**边界问题**

合并过程需要收集边界信息



![6_6_边界处理.png](https://upload-images.jianshu.io/upload_images/18567339-8042cdb226e4165d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


由同一个扩散点（感染中心）开始扩散感染的点都进行统一的标识， 记住感染中心。所以每一个子问题中都只需要记录岛数量和边界信息（边界信息记录由哪个感染中心扩散的）。

在使用并查集进行合并不同的子集。



如图：

左集合岛数量2，右集合岛数量2，一共4座岛 mergeNum=4

从边界开始分析

![6_7_边界两点合并.png](https://upload-images.jianshu.io/upload_images/18567339-85fea0de6497c45e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


查A所在的集合和C所在的集合是不是一个，如过不是一个说明之前没有合并过，现在要合并了，岛数量减一1, merge=3，A，C合并（A，C）。 

![6_8_第二对边界点合并.png](https://upload-images.jianshu.io/upload_images/18567339-631e4ba9e4f853eb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


重复上述过程，发现A，C已经合并过了,岛数量不变。

继续查看其他边界点。重复上述过程

![6_9_所有边界.png](https://upload-images.jianshu.io/upload_images/18567339-8437baf78dd890c1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


最终mergeNum=2。





---

---

以上为学习总结