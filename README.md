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

##### 链表相加

Add two Numbers

两个链表的相加

```java
//例如两个链表地相加，最后返回结果
/**
 4->6->3 链表1 与 3->4->4 相加
 
 结果应该是 7->0->8 
 就像是 进位也算是 只不过相加是反方向地
*/

public NodeList addNodeList(NodeList n1,NodeList n2){
    if(n1==null){
       return n2;
    }
    if(n2==null){
       return n1;
    }
    //接着我们需要一个哨兵节点来接收完成最后地结果
    ListNode dummy = new ListNode(-1);
    ListNode pre = dummy;
    //保存进位地值
    int carry  = 0;
    while(n1!=null&&n2!=null){
        //获得开始累加地值,加上进位地值
        int number=n1.val+n2.val+carry;
        //计算这个结果会不会造成进位
        carry = number/10;//如果超过10将会是1
        //对结果进行取余或者个位上地值
        ListNode node = new ListNode(number%10);
        //放到哨兵地后面
        pre.next = node;
        //pre地递增
        pre = pre.next;
        //下一次循环
        n1=n1.next;
        n2=n2.next;
    }
    //以上为理想情况，即n1和n2的长度都一样的时候，接下来来处理n1和n2长度不一样地情况
    while(n1!=null){
        int number = n1.val+carry;
        carry = number/10;
        ListNode node = new ListNode(number%10);
        pre.next = node;
        pre = pre.next;
        n1=n1.next;
    }
    while(n2!=null){
        int number = n2.val+carry;
        carry = number/10;
        ListNode node = new ListNode(number%10);
        pre.next = node;
        pre = pre.next;
        n2=n2.next;
    }
    //第三种情况，如果这个时候carry还有值地画，仍然需要进一位
    if(carry!=0){
        ListNode node = new ListNode(carry%10);
        pre.next = node;
    }
    return dummy.next;
}
```

LRU Cache

面试高频，LRU缓存 ，最近最少使用原则

![1608213997144](./img/1608213997144.png)

```java
class LRUCache {
    // 基本思路使用HashMap来弥补链表地查询性能缺乏地问题
    // 使用链表来完成lru的功能，当超过容量，将最后地一个元素踢出链表
    private class CacheNode{
        //要完成踢出操作，就需要找到前节点和后节点，所以这里做成双向
        CacheNode pre;
        CacheNode next;
        int value;
        int key;
        public CacheNode(int key,int value){
            this.key = key;
            this.value = value;
            this.pre = null;
            this.next = null;
        }
    }
    //容量，当缓存超过此值时，进行lru
    private int capacity;
    //来存储数据
    private Map<Integer,CacheNode> cacheMap = new HashMap<Integer,CacheNode>();
    //类似哨兵机智，设置头和尾节点来确定链表得完整性
    private CacheNode head = new CacheNode(-1,-1);
    private CacheNode tail = new CacheNode(-1,-1);
    public LRUCache(int capacity) {
        this.capacity = capacity;
        //将头尾相连
        head.next = tail;
        tail.pre = head;
        // head -> tail
        //      <-
    }
    
    public int get(int key) {
        //首先判断能不能找到
        if(!cacheMap.containsKey(key)){
            //没找到对应地，返回-1
            return -1;
        }
        //如果找到，就取出来
        CacheNode hit=cacheMap.get(key);
        //将命中地节点前后断开，并移动到尾部
        hit.pre.next=hit.next;
        hit.next.pre=hit.pre;
        //移动到尾部
        move2Tail(hit);
        return hit.value;
    }

    private void move2Tail(CacheNode node){
        //首先要断开现在尾部节点地练习
        node.pre = tail.pre;
        tail.pre = node;
        node.pre.next = node;
        node.next = tail;
        
    }
    
    public void put(int key, int value) {
        int result = get(key);
        if(result!=-1){
            //说明是替换
            cacheMap.get(key).value = value;
            return;
        }
        //否则说明是新增
        //判断是否长度已经大于了最大长度
        if(cacheMap.size()==capacity){
            //将链表尾部地节点提出
            CacheNode dropNode = head.next;
            head.next=dropNode.next;
            dropNode.next.pre = head;
            cacheMap.remove(dropNode.key);
            //等待gc
        }
        //增加
        CacheNode addNode = new CacheNode(key,value);
        cacheMap.put(key,addNode);
        move2Tail(addNode);
    }
}

// 使用java api实现
class LRUCache {

        private int cap;
	private Map<Integer, Integer> map = new LinkedHashMap<>();  // 保持插入顺序

	public LRUCache(int capacity) {
		this.cap = capacity;
	}

	public int get(int key) {
		if (map.keySet().contains(key)) {
			int value = map.get(key);
			map.remove(key);
                       // 保证每次查询后，都在末尾
			map.put(key, value);
			return value;
		}
		return -1;
	}

	public void put(int key, int value) {
		if (map.keySet().contains(key)) {
			map.remove(key);
		} else if (map.size() == cap) {
			Iterator<Map.E***y<Integer, Integer>> iterator = map.e***ySet().iterator();
			iterator.next();
			iterator.remove();

			// int firstKey = map.e***ySet().iterator().next().getValue();
			// map.remove(firstKey);
		}
		map.put(key, value);
	}
}
```

 #### 栈

先入后出，想象成盒子，添加数据称为入栈，压栈，进栈，取出数据或者移除数据称为出栈，弹栈

应用的话，想象浏览器的回退功能，想退回上一个浏览的东西可以使用这个。

检测代码中括弧匹配问题

Valid Parenthese 括号验证

输入`()`返回ture，`()[]{}`返回true，`(]`false

```java
public boolean isValid(String s){
    
    if(s==null||s.length()==0){
        return true;
    }
    
    Stack<Character> stack = new Stack<Character>();
    
    char[] stacks = stack.toCharArray();
    for(char c : stacks){
        if(c=='('||c=='{'||c=='['){
        	//是我们需要匹配的内容，压栈
            stack.push(c);
            continue;
        }
        if(c==')'){
            //由于（）是成对出现，所以根据栈的结构，他的上一个应该是和他一样的
            if(stack.isEmpty()||stack.pop()!='('){
                return false;
            }
        }
        if(c=='}'){
            //由于（）是成对出现，所以根据栈的结构，他的上一个应该是和他一样的
            if(stack.isEmpty()||stack.pop()!='{'){
                return false;
            }
        }
        if(c==']'){
            //由于（）是成对出现，所以根据栈的结构，他的上一个应该是和他一样的
            if(stack.isEmpty()||stack.pop()!='['){
                return false;
            }
        }
        //正确的匹配完了。应该一个都不剩，所以如果还有剩余，说明也是false
        return stack.isEmpty();
    }
    
}
```

Min Stack 最小栈

设计一个这样的栈，插入若干数值，可以使用O（1）的时间来快速找到插入若干数值中最小的元素

首先，根据栈的数据结构，想要取出数值必须将一个一个元素弹出来，你必须`翻箱倒柜`的从顶查到尾，这显然不是O（1），而且如果有这样栈能够这样的话，其实也算是一个很完美的数据结构了。

所以我们可以用两个栈来实现，保持和前一个栈的插入行为一致，但插入的元素却有些区别![1608453067792](./img/1608453067792.png)

后者只存储最小的元素，当你压入元素的时候，会比较与栈顶的元素大小，如果比元素大，那么`minstack`

将会保持插入一条和之前最小的元素一样的值，来保持和原栈的层数一致，如果新插入的要销，那么就替换成新的元素。

如果是基本的元素操作，操作原栈的同时，也要移除minstack里的元素，来保持栈深度的一致，如果要取出最小的值，直接操作minstack就可以了，不过也要记住，同步移除掉原栈的元素

```java
class MinStack{
    
    Stack<Integer> stack;
    Stack<Integer> minStack;
    
    public void push(int x){
        stack.push(x);
        if(minStack.isEmpty()||x<minStack.peek()){
            minStack.push(x);
        }else{
            //保持原样
            minStack.push(minStack.peek());
        }
    }
    
    public void pop(){
        minStack.pop();
        stack.pop();
    }
    
    public int top(){
        return stack.peek();
    }

    public int getMin(){
        return minStack.peek();
    }
}
```

Maximun Range

区间最大值

意思为给定一组数字，计算出这组数据的最大区域值是多少，区间最大值地计算方式为，这个区间里最小得值，和这个区间所有数字的和，最小值和数字和相乘，所得地结果为最大地区域值

例如 `[5,2,3]`，组合的方式有`[5],[2],[3],[5,2],[5,3],[5,2,3]`

所以要计算这几种组合地最大区域值是这样地

```
[5] 5*5 = 25
[2] 2*2 = 4
[3] 3*3 = 3
[5,2] 2*(5+2)=20
....
```

在此之前，先介绍一种算法，叫做`前缀和数`，即可以很快得算出这个区间的和

```
这个代表原来地数组 orgin=[5,2,3,1]
这个代表索引地位置 index=[0,1,2,3]
这个代表得是前缀和 total=[5,7,12,24] //对前面地数组地累加
如果你想要获得索引 0 - 3 中间地区间和，那么你可以用第三个数组所以0和3得值进行相减，就可以获得他们地区间和
total[3]-total[0]=24-5=19
验证一下，total[1]+total[2]=19是符合他们地区间和地，这样我们就用O(1)的时间获得了他们得区间和
```

所以，我们需要拿到`最小数`，和`区间和`

需要理解的一点地是，未必数字个数越多地组合，他的区间最大值就最大，例如`[5,2]`的组合，是20，也未必数字组合越少地区间最大值就最小，你看`[5]`的组合，是25。

因为结果会因为`区间最小数`而发生变化，越小，乘积地结果也就越小。也意味着区间和也就越小。

所以第一个任务就是需要找到`区间最小值`，然后用这个最小值以及使用`前缀和数`得用法，用O(n)的时间算出所有组合的最大值，并选出他们地最大值作为返回结果，得到区间最小数地概念可以用Stack来实现。

```java
public int max(int[] numbers){
    if(numbers==null||numbers.length==0){
        return 0;
    }
    Stack<Integer> stack = new Stack<Integer>();
    int max=0;//区间最大值
    //初始化地比传入地要多一位
    int[] sum = new int[numbers,length+1];
    //计算前缀和数
    for(int i=1,len=sum.length;i<=len;i++){
        sum[i]=sum[i-1]+numbers[i-1];
    }
    for(int i=0,len=numbers.length;i<len;i++){
        //这里我们需要记录地是数组地下标
        while(!stack.isEmpty()&&numbers[i]<numbers[stack.peek()]){
            //有比前一个小地数就放进去计算
            int index = stack.pop();//弹出当前numbers数组中最小数地索引位
            int left = i;
            int right = i;//左右都记录，来计算前缀和数
            if(stack.isEmpty()){
                //表示弹出了这一次，就已经空了,意味着当前地值已经是在当前循环历史中最小地值，对需要对前面地值进行前缀数地求和
                left = 0;
            }else{
                //还有比他更小地没有弹出，还需要继续比较
                left = index;
            }
            max = Math.max(max,numbers[index]*(sum[right]-sum[left]));
        }
        stack.push(i);
    }
    //为了避免最后插入地数字也可能比前面地插入数字要小，再做一次遍历
    if(!stack.isEmpty()){
        int index = stack.pop();//弹出当前numbers数组中最小数地索引位
        int left = numbers.length;
        int right = numbers.length;//左右都记录，来计算前缀和数
        if(stack.isEmpty()){
                //表示弹出了这一次，就已经空了
           left = 0;
        }else{
                //还有比他更小地没有弹出，还需要继续比较
           left = index;
        }
        max = Math.max(max,numbers[index]*(sum[right]-sum[left]));
    }
    return max;
}
```

这里做一个总结，以上stack用法在于存储目标数组最小值得任意组合。

以`[6,2,3,4,1]`为例子，这个取名为`目标数组`

首先准备一个stack，从6开始，即索引为0开始寻找，6是到索引0为止最小地数，那么压栈，注意这个时候压入地是`索引`，是在目标数组地索引，而不是6本身。到了2，我们将stack中存入地目前为止在目标数组中已知最小数的索引与下一个进行比较，我们发现2比6要小，所以我们将6弹出，将2的索引压栈，这个时候我们可以对6进行区间最大值地运算，接着，我们比较3，和4与2的大小，结果发现都没有2大，所以2，3，4数得索引被同样被压栈，被保留，因为在同为最小数是2地情况下，这中以2为最小值地组合，后面地成员数越多，区间最大值也必然是越大的，所以还未找到目标数组最小值的时候，我们就可以认为这是`一组以2为最小值地组合`，到最后可以一起计算，但是最后找到了元素1，所以，1比之前压栈得数都小，那么我们认为之前地stack中地值都可以拿出来做计算了，因为从找到1开始，这会变成`一组以1为最小值地组合`。至于最后还要进行一次运算，也是怕最后一个元素会比之前地元素还要小，做得兼容操作。

#### 队列

what is queue

用栈实现队列

![1608560705193](./img/1608560705193.png)

首先，一个栈肯定是实现不了队列的效果，我们这里可以使用两个栈来模拟这个效果。

当栈1插入了若干个节点后，如果你希望取出头节点地话，符合队列地先入先出原则，可以将栈1中地全部节点取出在塞入到栈2，这个时候，栈2里地弹出顺序就符合了队列地效果，如果后续地元素进行添加，那么只需要往栈1里面添加就行，只要栈2里面地数据不为空，取值就可以一直往栈2中弹出，当栈2里地元素空地时候，再进行一次对栈1得全量获取

```java
class MyQueue{
    //负责存值
    Stack<Integer> stackA;
    //负责取值
    Stack<Integer> stackA;
    
    public MyQueue(){
        stackA = new Stack<Integer>();
        stackB = new Stack<Integer>();
    }
    
    public void push(int x){
        stackA.push(x);
    }
    
    public int pop(){
        if(empty()){
            return -1;
        }
        fill();
        return stackB.pop();
    }
    
    private void fill(){
        if(stackB.isEmpty()){
            //需要重新填充数据到B中
            while(!stackA.isEmpty()){
                stackB.push(stackA.pop());
            }
        }
    }
    
    public int peek(){
        if(empty()){
            return -1;
        }
        fill();
        return stackB.peek();
    }
    
    public boolean empty(){
        return stackB.isEmpty()&&stackA,isEmpty();
    }
    
}
```

推荐结果打散

快手面试题，快手作为一个短视频平台，有时候会推荐图片和广告，给你一组随机地图片和广告，要让他们穿插的交替地进行排列展示，这里用v来比作视频，p来比作图片。给你这样一组数据

```
v1 v2 v3 v4 v5 v6 v7 p1 p2 p3 p4
```

当然实际上，在终端上展示你不可能这样一下子一排地视频，视频完了后面是一堆图片，这看起来不友好。

``` java
public List<String> getRecommendResult(List<String> picAndVideo,int maxInterval){
     //存储结果
    List<String> result = new ArrayList<String>();
    if(picAndVideo==null&&picAndVideo.isEmpty()){
        return result;
    }
   	Queue<String> picQueue = new LinkedList<String>();
    Queue<String> videoQueue = new LinkedList<String>();
    boolean firstPics = false;
    //获得总共地数据长度
    int index = 0;
    int picAndVideoLength = picAndVideo.size();
    //这个方法用于找到第一个图片
    while(!firstPics&&index<picAndVideoLength){
        String item = picAndVideo.get(index);
        if(isVideo(item)){
            result.add(item);
            index++;
        }else{
            //是图片
            firstPics = true;
        }
    }
    //在找到第一个图片地时候，将会到这个
    while(index<picAndVideoLength){
        String item = picAndVideo.get(item);
        if(isVideo(item)){
            videoQueue.add(item);
        }else{
            picQueue.add(item);
        }
        index++;
    }
    //以上，对找到第一张图面后地内容进行归类
    int currentSize = result.size();
    while(!videQueue.isEmpty()&&!picQueue.isEmpty()){
        if(currentSize>=maxInterval){
            //当前视频内容如果已经超过了最大间隔，那么我们将插入一个图片
            result.add(picQueue.poll());
            curentSize=0;//设置回0
        }else{
            //否则我们可以接着添加视频
            result.add(videoQueue.poll())
                currentSize++;
        }
    }
    //以上地内容，当视频 videQueue 或者 picQueue为空地时候，都会弹出
    //所以，如果这个时候videoQueue，也就是视频队列里如果还有东西，我们是允许接着插入地
    if(!videoQueue.isEmpty()){
        result.add(videoQueue.poll());
    }
    //如果以上currentSize在最后一个视频累加完了后，正好超过间隔，也尝试加上图片
    while(currentSize>=maxInterval&&!picQueue.isEmpty()){
        result.add(picQueue.poll());
    }
    return result;
}

private boolean isVideo(String slice){
    if(slice.indexOf("v"!=-1){
        return true;
    }
    return false;
}
```

#### 二分法

Binary Search

###### 在旋转有序的数组中搜数

Search in Rotated Sorted Array

先写一个二分法的模板，**请牢记**

``` java
public static void main(String[] args){
    //给定一组数据，这里必须是有序地
    int[] num = {1,4,7,9,10,14,16,20,56,89};
    System.out.println(getIndex(num,4) );
}

public static int getIndex(int[] num,int target){
 	//校验查找容器是否合法
    if(num==null||num.length==0){
        return -1;
    }
    int start = 0;
    int end = num.length-1;
    //以上定义开头和结尾
    //这里定义二分法得灵魂，中间地索引位
    int mid;
    for(start+1<end){
        mid = start+(end-start)/2;
        if(num[mid]==target){
            //找到了直接返回
            return mid;
        }else if(num[mid]>target){
            //说明在左边
            end = mid;
        }else{
            //比他大，说明在右边
            start = mid;
        }
    }
    //在这个以上返回都找到了
    if(num[start]==target){
        return start;
    }
    if(num[end]==target){
        return end;
    }
    return -1;
}
```

回到之前地旋转有序数组中，这是一组有顺序地数组，例如 `{1,4,7,9,10,14,16,20,56,89}`这样的数组，但是这个数组会从某个位置，开始旋转，意味着他可能会变成这样`{14,16,20,56,89,1,4,7,9,10}`，意味着从某个点开始，会进行某段地不同位置得升序。

![1608650422561](./img/1608650422561.png)

可以把反转后地数组，看做是这样地两段线条，A和B，目标target可能在A或者B上，同时，当你取mid也就是中间值地时候，也可能是在A上也可能在B上，所以我们需要考虑几种情况。

第一种，当mid落在了A上，且target确实也在A上，且start<=target<=mid地时候，可以完全确定target一定是在A的上面。反之，那么mid后面，也就是B线段可以完全舍弃。否则target>mid

![1608650811314](./img/1608650811314.png)

第二种情况，目标是比start要小的，那么就从B线条上寻找，如果符合mid<=target<=end这样地条件，那么一定就是在B线条上，抛弃A线条上地所有条件

![1608650973977](./img/1608650973977.png)

``` java
public void search(int[] num,int target){
    if(num==null&&num.length==0){
        return -1;
    }
    int start = 0;
    int end = num.length-1;
    int mid;
    while(start+1<end){
   		//确定可能是在A线条还是B线条
        mid = start+(end-start)/2;
        if(num[mid]==target){
            return mid;
        }
        //说明在A端
        if(num[mid]>num[start]){
            if(target<=num[mid]&&target>=num[start]){
                //完美落到A上，对后面地内容进行剔除
                end = mid;
            }else{
                //否则往前找
                start=mid;
            }
        }else{
            //说明在B段
            if(target<=num[end]&&target>=num[mid]){
                start=mid;
            }else{
                end=mid;
            }
        }
    }
   
    //补充
    if(num[start]==target){
        return start;
    }
    if(num[end]==target){
        return end;
    }
    return -1;
    
}
```

在旋转有序数组中找最小

Find Minimum in Rotated Sorted Array

```java
//沿用之前地例子
public int search(int[] num){
    if(num==null||num.length==0){
        return -1; 
    }
    int start=0;
    int end = num.length-1;
    int mid;
    while(start+1<end){
        mid = start+(end-start)/2;
        //看具体落到哪里
        if(num[mid]>=num[start]){
            //说明在A段
            if(num[mid<=num[end]]){
                end = mid;
            }else{
                start=mid;
            }
            
        }else{
            end = mid;
        }
    }
    return Math.min(num[start],num[end]);
}
```

找峰值元素

![1608734378351](./img/1608734378351.png)

这道题地关键在于，只要找出峰值就行，每个凸起都可以认为是山峰，这就关键在于mid落在哪里，如果落在前面，即mid+1>mid>mid-1我们认为，这是上坡路，于是我们将start移动到mid地位置，反之mid-1>mid>mid+1这就是下坡路，将end移动到mid节点

```java
public int search(int[] nums){
    if(nums==null||nums.length==0){
        return -1;
    }
    int start = 0;
    int end = nums.length-1;
    int mid;
    while(start+1<end){
        mid = start+(end-start)/2;
        //进行位置地判断
        /*if(nums[mid]>nums[mid-1]){
            
            if(num[mid]<num[mid+1]){
                //说明是在上坡
                start = mid;
            }else{
                //说明已经到点了，直接返回
                return num[mid]
            }
        }else{
            if(num[mid]>num[mid+1]){
                //下坡
                end = mid;
            }else{
                //说明是谷底其实那边过来都一样，这边选start吧
                start=mid;
                
            }
        }*/
        if(nums[mid]<num[mid-1]){
            //可能是下坡
            end = mid;
        }else if(num[mid]>num[mid-1]){
            //可能是上坡
            start=mid;
        }else{
            return nums[mid];
        }
        return nums[start]>nums[end]?start:end;
    }
    
}
```

砍木头

给你几块木头(一组含有数字地数组)，给你一个要将这几块木头砍成地块数，例如

```
[232,124,456] 这些木头，要砍成7块，取他长度最大地块数，你可以砍成7块，或者8块，但是你必须保证最后砍成地块数长度是其他几种可能砍法长度最大的。
比如，232的木头，你可以认为是一根232米地木头，你自然可以把他砍成1米长度的，这样你可以砍232块，这是符合条件地，同样你可以把他砍成2米一块地，这样你可以砍116块，同样符合条件，但是2米地木头比1米地木头要长，所以我们会优先选择2米一块得砍法，以此类推。
所以这道题用二分法去理解，可以把他看成一组有序升序的数组，1，2，3，4，5...都是可以砍成不同米数地木块，可以砍成1米地，2米得，3米的，不断尝试，最后选出最佳地可能性
```

写出代码

```java
public int woodCut(int[] woods,int cutNum){
    
    if(woods==null||woods.length==0){
        return 0;
    }
    int start = 1;
    //获得众多木头中，长度最长地
    int end = getMax(woods);
    int mid;
    while(start+1<end){
        mid = start+(end-start)/2;
        //每块木头都砍 mid长度，看看传入地木头可以砍几块，其中会一直缩小范围，直到最后确定合适地长度
        int pices = getPrices(woods,mid);
        //首先要满足，切地块数一定要大于cutNum
        if(pices>=cutNum){
            start = mid;
        }else{
            end = mid;
        }
    }
    //随着事件地推移，mid地值会越来越大
    if(getPrices(nums,end)>=cutNum){
        return end;
    }
    if(getPrices(nums,start)>=cutNum){
        return start;
    }
    return 0;
}

public int getPrices(int[] woods,int woodLength){
    int pices = 0;
    for(int wood:woods){
        //每个都指定地长度切这么多块，累积多少块
        pices+=wood/woodLength;
    }
    return pices;
}

public int getMax(int[] woods){
 	//找出最长地木头
    int max = woods[0];
    for(int i=1,len=woods.length;i<len;i++){
        if(max<woods[i]){
            max=woods[i];
        }
    }
    return max;
}
```



#### 双指针 

想要说明地是，双指针并不是说next和pre这样地指针，而说地是一个链表中存在两个指针，具体这两个指针地用法是干什么，要根据功能而定。

例如，你想要在一个链表中寻找中间地位置，那么你可以规定这样地两个指针，一个指针用于一次移动一个位置，一个用于一次性移动两个位置，当后一个指针移动两个位置并且已经没有指向，即指向为空地时候，说明已经到尾，遍历完了，这个时候那个移动一个地，就有可能是中间值，当然如果这个指针地长度是奇数会更加明显。

#### 2sum and 3sum

2sum，给定一个已经排好序地数组，里面有一些数字，然后给你一个数字，这个数组是之前给的数组，给你地数字将会是数组里某两个数相加地和，例如`[2,7,11,21]`，然后给你一个数字9，那么我们可以知道是2和7相加地和，那么返回2和7的数组索引[1,2]。

最笨地方法就是双循环，直到算出结果。

 ![1609078891265](./img/1609078891265.png)

这里得双指针一个指向了数组地头，一个指向了尾巴，因为这是一个升序的数组，所以头尾相加获得地值就是一个平衡点，如果`num[left]+num[right]>target`，说明结果过大，可以通过减少尾巴的索引，也就是通过减少right的索引来获得更小得结果，同理，如果结果是小于的，那么我们就需要移动left的索引，通过增大来获得更大的结果，最终来获得合适地值

```java
public int[] getIndex(int[] nums,int target){
    if(num==null||num.length==0){
        return null;
    }
    int left = 0;
    int right = num.length-1;
    while(left<right){
        if(nums[left]+nums[right]==target){
            return new int[]{left,right};
        }else if(nums[left]+nums[right]<target){
            left++;
        }else{
            right++;
        }
    }
    return null;
}
// 老师地版本
public int[] getIndex(int[] nums,int target){
    if(num==null||num.length==0){
        return null;
    }
    int[] result = new result[2]{-1,-1};
    int left = 0;
    int right = num.length-1;
    while(left<right){
        if(nums[left]+nums[right]==target){
            result[0]=left;
            result[1]=right;
            break;
        }else if(nums[left]+nums[right]<target){
            left++;
        }else{
            right++;
        }
    }
    return result;
}
```

如果改变一下题目，要是目标数组不止一对可以获得目标数值地索引怎么办

![1609079532880](./img/1609079532880.png)

当我们查找到的时候，同时移动左右就行了

```java
public List<int[]> getIndex(int[] nums,int target){
    if(num==null||num.length==0){
        return null;
    }
    List<int[]> result = new ArrayList<>();
    int left = 0;
    int right = num.length-1;
    while(left<right){
        if(nums[left]+nums[right]==target){
            result.add(new int[]{left,right});
            left++;
            right++;
        }else if(nums[left]+nums[right]<target){
            left++;
        }else{
            right++;
        }
    }
    return results;
}
```

这样，我们就把O(n2)->O(n)，变成了这样的时间复杂度。

那么三个数呢？同理，我们可以使用三个指针来确定他们之间地关系。

![1609079812449](./img/1609079812449.png)

我们固定一个，然后后面沿用2sum的逻辑。这样我们地时间复杂度也从O(n3)变成了O(n2)

```java
public List<List<Integer>> getIndex(int[] nums,int target){
    if(nums==null||nums.length==0){
        return null;
    }
    Arrays.sort(nums);
    List<List<Integer>> result = new ArrayList<List<Integer>>();
    //这里为什么是len-2.因为后面两个位置要留给left 和 right，如果循环真的走到那一步了
    for(int i=0,len<nums.length;i<len-2;i++){
        int left=i+1;
        int right=len-1;
        //和2sum一样
        while(left<right){
            //原题目是要所有累加地数字等于0
            if(nums[i]+nums[left]+nums[right]==0){
                List<Integer> s = Arrays.asList(nums[i],nums[left],nums[right]);
                result.add(s);
                //全部累加推进
                left++;
                right++;
                //去从, 害怕出现left方向有相同数字地值,或者right方向有相同地，就直接忽略
                while(left<right&&nums[left]==nums[left+1]){
                    left++;
                }
                while(left<right&&nums[right]==nums[right-1]){
                    right--;
                }
            }else if(nums[i]+nums[left]+nums[right]>0){
                right--;
            }else{
                left--;
            }
        }
    }
    return result;
}
```

##### 验证三角形

Valid Triangle Number，两边之和大于第三边，两边之差小于第三边。