#### 算法

一些算法的总结

##### 复杂度

什么是**时间复杂度**(What is time complexity)

数据量的增加所导致的时间花费的增加是怎么样的

时间是呈线性增加，还是常数，还是指数，指的是一种变化趋势

```java
//嵌套代码
int n = 10;
for(int i =0;i<=n;i++){
    
    for(int j =0;j<=2;j++){
        ...
    }
    //随着当 n 为 10 的时候，这里要计算 10*2 次，当为100 时，计算100*2
    //所以我们可以认为这个循环的时间复杂度为 O(2n)
}

for(int i =0;i<=n;i++){
    
    for(int j =0;j<=n;j++){
        ...
    }
    // 沿用上面的观点，这个为0(n^2)
}
```

常见的算法时间复杂度由小到大依次为

```
          n        n
O(1)<O(log )<O(nlog )<O(n^2)<O(n^3)<...<O(n^2)<O(n!)
          2		   2
```

什么是**空间复杂度**(What is space complexity)

trade off

选择，是空间换时间，还是反之

##### 维度

* 时间维度
  * 是指执行当前算法所消耗的时间，通常用时间复杂度来描述
* 空间维度
  * 是指执行当前算法需要占用多少内存空间，我们通常用**空间复杂度**来描述

##### 空间复杂度定义

是对一个算法运行过程中临时占用存储空间大小的量度

记做S(n)=O(f(n))



#### 链表

* 什么是线性表(What is Linear List)?
  * 线性表就是数据排成一条线一样的结构
    * 数组
    * 栈
    * 队列
    * 链表
  * 非线性
    * 二叉树
    * 图
    * 堆
    * ....

对于`ArrayList`的解析。我们从增删改差几个方法来入手看看`ArrayList`的实现这些`api`的时间复杂度是怎样的。

```java
//ArrayList.java

 public void add(int index, E element) {
     	//检查是否越界 否则做扩容操作
        rangeCheckForAdd(index);
		//如果没有越界，则进行累加
        ensureCapacityInternal(size + 1);  // Increments modCount!!
     	//由于我们是指定位置的插入，所有需要将该 index位置后方的所有元素全部移动一位
     	//有多少便多少位置，所以时间负责度为O(n)
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
     	//最后赋值操作
        elementData[index] = element;
        size++;
    }

public E remove(int index) {
    	//同样检查是否越界
        rangeCheck(index);

        modCount++;
    	//取出该位置的值
        E oldValue = elementData(index);

        int numMoved = size - index - 1;
    	//如果需要删除的这个元素的位置是合法位置，即是在 0~size 之间
        if (numMoved > 0)
            //将指定删除位置的元素后方的所有元素向前移动一位，因此时间复杂度为O(n)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work

        return oldValue;
    }

public E get(int index) {
        rangeCheck(index);
	// 由于ArrayList 当你不指定大小的时候，会默认给你初始化长度为10的Object数组，这里直接用
    // 数据的索引直接获取结果，所以复杂度为O(1),可以直接定位
        return elementData(index);
    }
 E elementData(int index) {
        return (E) elementData[index];
    }

 public E set(int index, E element) {
        rangeCheck(index);
		//修改方法，用于直接替换，查找方法也沿用了get方法O(1),所以设值时间复杂度和get的时间复杂度一致
        E oldValue = elementData(index);
        elementData[index] = element;
        return oldValue;
    }
	//默认构造器的初始化常量数组，未指定大小
	private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
	//实际用于存放List数组的数据
    transient Object[] elementData; // non-private to simplify nested class access、

	public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }

	public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            //当指定容量大小的时候，会为你创建默认大小，这也是比比较推荐的初始化方法
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }

// 剖析ArrayList的无参构造器的具体容量在哪里
	public boolean add(E e) {
    	//进入这个方法
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }
	private void ensureCapacityInternal(int minCapacity) {
        //进入第二个 这里其实会考虑做扩容操作，比较现在的数据大小和增加新元素后的List大小
        ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
    }
	private static int calculateCapacity(Object[] elementData, int minCapacity) {
        //很明显，如果我们使用了无参的构造器的话，会进入这个判断，因为初始化用的就是这个
        //这个使用会返回 DEFAULT_CAPACITY 
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        return minCapacity;
    }
	/**
     * Default initial capacity.
       所以无参构造器在初始化的时候其实还没有指定容量的大小，在真正add方法执行的时候
       会做真正的初始化，默认大小为10，即会默认创建大小为10的Object数组，即使你没用到10个元素
     */
    private static final int DEFAULT_CAPACITY = 10;
```

* `ArrayList`的几个使用建议
  * 能不用默认构造器就不要使用默认构造器
  * 如果事先知道需要填入的大小，可以使用含有参数的构造器，并填入数值进行初始化
  * 如果你不清楚的实际使用大小，也可以预估将会有多少元素
  * `ArrayList`的crud方法的算法时间复杂度维度也可以看出，该List在查和修改性能很出色都为0(1)，删除和新增都为O(n)较差
  * 因为底层是对象数组实现，所以他们在**物理**和**逻辑**存储单元都是连续的。

#### 什么是Linked List 什么是链表

链表是一种物理存储单元上非连续的，非顺序的存储结构，数据元素的逻辑顺序是通过链表中的指针连接次序实现的。

这里不做源码分析，由于链表的形式

```java
class ListNode{
    Object data;//当前节点保存的值
    ListNode next;//指向下一个节点
}
```

![1607785612003](./img/1607785612003.png)

对于链表而言，删除和新增对于他来说是简单的只要断开原本的引用指针，指向新的即可，对于删除的，只需要断开被删除的next指针，将被删除的节点的前后节点next重新指向即可，所以删除和新增的操作的时间复杂度都是0(1)

## 约瑟夫问题

约瑟夫问题是个著名的问题：N个人围成一圈，第一个人从1开始报数，报M的将被杀掉，下一个人接着从1开始报。如此反复，最后剩下一个，求最后的胜利者。 
例如只有三个人，把他们叫做A、B、C，他们围成一圈，从A开始报数，假设报2的人被杀掉。

- 首先A开始报数，他报1。侥幸逃过一劫。
- 然后轮到B报数，他报2。非常惨，他被杀了
- C接着从1开始报数
- 接着轮到A报数，他报2。也被杀死了。
- 最终胜利者是C

但是链表只是逻辑上连续，他并不能做到像是数组那样，使用位置即可定位，他的依据是从头节点开始遍历，不断的寻找下一个节点。所以他的时间复杂度是O(n)



##### 链表和数组的区别

在内存分配上，数组一定要保证内存一定要有数组申请的那么大的空间才会创建成功，而链表不会。

![1607786105106](./img/1607786105106.png)

#### 循环链表

即尾巴节点直接和头部相连接，组成一个环

#### 双向链表

what is double linked list，双向链表在单链表的每个结点中，再设置一个指向前驱节点的指针。

```java
public class Node{
    //数据本身
    private Object data;
    //前一个节点
    private Node pre;
    //后一个节点
    private Node next;
    
}
```

也是从增删改查这些问题入手。

对于增加和删除和之前的链表没有什么区别，都是O(1)，查询其实会比普通链表稍微高一点，因为他可以记住前节点的应用，他可以根据找到的值大小来决定是否接着向后遍历还是向前遍历。单向链表如果想要知道他的前一个节点是什么，还需要重新再遍历一次。

即便双向链表在空间上花费比单向要高些，这也是空间换时间的例子。

```java
// 反转链表 1->2->3->4->5
public ListNode reverseList(ListNode head){
    if(head==null){
        return null;
    }
    ListNode prev = head;
    ListNode current = head.next;
    //断开下一个节点
    prev.next = null;
    // 1 2->3->4->5
    while(current!=null){
        ListNode next = current.next;
        // // 1<-2->3->4->5
        current.next = prev;//将断开的节点反方向接上
        prev = current;//切换到下一个
        current=next;
    }
    return prev;
}

// 1->2->3->4->5 
// 1->4->3->2->5
// 1<=m<=n<=length
public ListNode reversedBetween(ListNode head,int m,int n){
    if(head==null||m>=n){
        return head;
    }
    //为了保证头节点不丢失，创建哨兵节点dummy
    ListNode dummy = new ListNode(-1);
    //将头节点地引用保留
    dummy.next = head;
    //将指针变为头节点
    head=dummy;
    //接着以m为界限，找到反转地起始点地前一个节点
    for(int i=0;i<m;i++){
        head=head.next;
    }
    //接着，定义几个变量
    //保留目前起始反转节点的前一个节点
    // -1->1->2->3->4->5
    ListNode prevN = head;
    ListNode mNode = head.next;//找到m位置地节点
    ListNode nNode = mNode;//保存这个节点地位置
    ListNode postN = nNode.next;//m节点地下一个位置
    //现在把他当做是普通地反转链表来处理就行
    for(int i =m;i<n;i++){
        /*
        ListNode next = current.next;
        current.next = prev;//将断开的节点反方向接上
        prev = current;//切换到下一个
        current=next;
        
        */
        ListNode next=postN.next;
        //同样，反向接上
        postN.next = nNode;
        nNode = postN;
        postN = next;
    }
    prevN.next=nNode;
    mNode.next=postN;
    return head;
}
```

![1607873146857](./img/1607873146857.png)

 ##### 深度拷贝带随机指针地链表

Copy List With Random Pointer

例如，A对象含有引用类型得变量B，对A对象进行深度拷贝后得到C，这个时候C中得引用应该与母体地变量B不一样，即深度拷贝后，引用类型也必须是完全不一样地值。如果一样就是只是简单地值拷贝，引用还是一样地。

现在有一个链表，节点地结构是这样的

``` java
class Node{
    //含有下一个节点
     Node next;
    //某一个随机指向这条链表某个节点地指针，可能没有指向，为null
     Node random;
    //含有地值
     int val;
    public Node(int val){
        this.val = val;
    }
}
```

![1608038123954](./img/1608038123954.png)

如图，例如这是一个五个节点地链表，其中2节点地random参数，随机指向了5节点，其它都没有特殊地指向，对这样地一个链表进行深度拷贝。

解题思路，要对这样链表进行深度拷贝，首先要创建和他一样多地节点，并且按照顺序将他们连接起来，这样就完成了拷贝。首先如果要按照原链表地顺序地话，就要用关系将他们对应起来。

```java
public Node copyRandomList(Node head){
    if(head==null){
        return null;
    }
    //保存映射关系的map
    Map<Node,Node> map = new HashMap<Node,Node>();
    //首先将头保留一下
   	Node newHead = head;
    //循环，开始复制
    while(newHead!=null){
        // 创建映射
        if(!map.containsKey(newHead)){
            Node copyNode = new Node(newHead.val);
            map.put(newHead,copyHead);
            //这个循环走下去，其实已经完成了所有节点地拷贝，连顺序也对应好了
        }
        //再判断一下是否有random地存在
        if(newHead.random!=null){
            Node random = newHead.random;
                if(!map.containsKey(random)){
					  //对随机指针对象进行拷贝
            		Node copyRandom = new Node(random.val)；    
                    map.put(random,copyRandom);
                }
            //根据原链表地random地映射关系，建立关系
            map.get(newHead).random = map.get(random);
        }
     	//走下一次循环
        newHead = newHead.next;
    }
    //以上代码，结构变成了这样
}
```

![1608039812382](./img/1608039812382.png)

```java
//接着，将他们地关系根据map对应起来
newHead = head;
while(newHead!=null){
    map.get(newHead).next = map.get(newHead.next);
    newHead = newHead.next;
}
return map.get(head);
```

```java 
//完整代码
public Node copyRandomList(Node head){
    if(head==null){
        return null;
    }
    //保存映射关系的map
    Map<Node,Node> map = new HashMap<Node,Node>();
    //首先将头保留一下
   	Node newHead = head;
    //循环，开始复制
    while(newHead!=null){
        // 创建映射
        if(!map.containsKey(newHead)){
            Node copyNode = new Node(newHead.val);
            map.put(newHead,copyHead);
            //这个循环走下去，其实已经完成了所有节点地拷贝，连顺序也对应好了
        }
        //再判断一下是否有random地存在
        if(newHead.random!=null){
            Node random = newHead.random;
                if(!map.containsKey(random)){
					  //对随机指针对象进行拷贝
            		Node copyRandom = new Node(random.val)；    
                    map.put(random,copyRandom);
                }
            //根据原链表地random地映射关系，建立关系
            map.get(newHead).random = map.get(random);
        }
     	//走下一次循环
        newHead = newHead.next;
    }
    //接着，将他们地关系根据map对应起来
	newHead = head;
	while(newHead!=null){
    	map.get(newHead).next = map.get(newHead.next);
    	newHead = newHead.next;
	}
	return map.get(head);
}
```

另一种优化方案，由于在上面算法上，使用了map，空间复杂度增加，能不能节省掉这个map，我们使用map地目的在于建立映射，如果我们可以用另一种方式映射地话，是不是就可以省略到map地建立。

![1608040751280](./img/1608040751280.png)

我们无非就是希望，在找到原表地1节点地时候能够快速找到另一个拷贝地1节点，这也是我们使用map地初衷，那么我们可以建立这样一种关系，将原1节点地后面直接接上拷贝的1节点，这样当我们获得原先地1节点地时候，他的next节点就是我们要找地拷贝地节点

```java
 public Node copyRandomList(Node head) {
        if(head==null){
            return null;
        }
        //第一步连接节点
        connectNode(head);
        //拷贝指针
        copyRandom(head);
        //第二部断开节点
        return split(head);
        
    }

    public void copyRandom(Node head){
        Node node = head;
        while(node!=null&&node.next!=null){
            if(node.random!=null){
                node.next.random = node.random.next;
            }
            node = node.next.next;
        }
    }
    
    public void connectNode(Node head){
        //老规矩，保存头节点
        Node newHead = head;
        while(newHead!=null){
            Node copyNode = new Node(newHead.val);
            //接下来，我们要断开原来的链表关系，在其中插入这个新地copy节点
            copyNode.next = newHead.next;
            newHead.next = copyNode; 
            //以上成功插入，进入下一次循环
            //newHead = newHead.next;
            //记住这里不能这样写了，因为现在地newHead.next已经是拷贝地1节点了
            newHead = copyNode.next;
            
        }
    }

    public Node split(Node head){
        //从已经拼凑好地节点切开
        //取得拷贝后地拷贝头节点
        Node result  = head.next;
        Node move = head.next;
        //完整关联关系
        while(head!=null&&head.next!=null){
            //将copy后地节点，重新组成关联关系
             //如果还有random
            //这个是还原原来地链表关系
            head.next = head.next.next;
            //组成拷贝链表地对应关系
            //下一次循环地准备
            head = head.next;
            if(move!=null&&move.next!=null){
                //找到拷贝地下一个节点，进行连接
                move.next = move.next.next;
                move = move.next;//然后替换
            }
            
        }
        return result;
        
    }
```

