JAVA笔记

关键字：transient-被修饰的变量则不能序列化

集合ArrayList<E>和LinkedList<E>的区别(参考:https://mp.weixin.qq.com/s/g1E3GQU1JJzpAxV4zwRKgg和https://mp.weixin.qq.com/s/oA0D1BjzBi7z0Xuvt4O-PQ)：

1、顺序插入速度ArrayList会比较快，因为ArrayList是基于数组实现的，数组是事先new好的，只要往指定位置塞一个数据就好了；LinkedList则不同，每次顺序插入的时候LinkedList将new一个对象出来，如果对象比较大，那么new的时间势必会长一点，再加上一些引用赋值的操作，所以顺序插入LinkedList必然慢于ArrayList



2、基于上一点，因为LinkedList里面不仅维护了待插入的元素，还维护了Entry的前置Entry和后继Entry，如果一个LinkedList中的Entry非常多，那么LinkedList将比ArrayList更耗费一些内存



3、数据遍历的速度，ArrayList<E>使用for循环速度快，LinkedList<E>使用forEach循环速度快



4、有些说法认为LinkedList做插入和删除更快，这种说法其实是不准确的：



（1）LinkedList做插入、删除的时候，慢在寻址，快在只需要改变前后Entry的引用地址



（2）ArrayList做插入、删除的时候，慢在数组元素的批量copy，快在寻址



所以，如果待插入、删除的元素是在数据结构的前半段尤其是非常靠前的位置的时候，LinkedList的效率将大大快过ArrayList，因为ArrayList将批量copy大量的元素；越往后，对于LinkedList来说，因为它是双向链表，所以在第2个元素后面插入一个数据和在倒数第2个元素后面插入一个元素在效率上基本没有差别，但是ArrayList由于要批量copy的元素越来越少，操作速度必然追上乃至超过LinkedList。



**ArrayList的优缺点**



从上面的几个过程总结一下ArrayList的优缺点。ArrayList的优点如下：



1、ArrayList底层以数组实现，是一种随机访问模式，再加上它实现了RandomAccess接口，因此查找也就是get的时候非常快



2、ArrayList在顺序添加一个元素的时候非常方便，只是往数组里面添加了一个元素而已



不过ArrayList的缺点也十分明显：



1、删除元素的时候，涉及到一次元素复制，如果要复制的元素很多，那么就会比较耗费性能



2、插入元素的时候，涉及到一次元素复制，如果要复制的元素很多，那么就会比较耗费性能



因此，ArrayList比较适合顺序添加、随机访问的场景。



**ArrayList和Vector的区别**



ArrayList是线程非安全的，这很明显，因为ArrayList中所有的方法都不是同步的，在并发下一定会出现线程安全问题。那么我们想要使用ArrayList并且让它线程安全怎么办？一个方法是用Collections.synchronizedList方法把你的ArrayList变成一个线程安全的List，比如：



> List<String> synchronizedList = Collections.synchronizedList(list);
>
> synchronizedList.add("aaa");
>
> synchronizedList.add("bbb");
>
> for (int i = 0; i < synchronizedList.size(); i++)
>
> {
>
> ​    System.out.println(synchronizedList.get(i));
>
> }



另一个方法就是Vector，它是ArrayList的线程安全版本，其实现90%和ArrayList都完全一样，区别在于：



1、Vector是线程安全的，ArrayList是线程非安全的



2、Vector可以指定增长因子，如果该增长因子指定了，那么扩容的时候会每次新的数组大小会在原数组的大小基础上加上增长因子；如果不指定增长因子，那么就给原数组大小*2，源代码是这样的：



> int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
>
> ​                                 capacityIncrement : oldCapacity);



**为什么ArrayList的elementData是用transient修饰的？**



最后一个问题，我们看一下ArrayList中的数组，是这么定义的：



> private transient Object[] elementData;



不知道大家有没有想过，为什么elementData是使用transient修饰的呢？关于这个问题，说说我的看法。我们看一下ArrayList的定义：



> public class ArrayList<E> extends AbstractList<E>
>
> ​        implements List<E>, RandomAccess, Cloneable, java.io.Serializable



看到ArrayList实现了Serializable接口，这意味着ArrayList是可以被序列化的，用transient修饰elementData意味着我不希望elementData数组被序列化。这是为什么？因为序列化ArrayList的时候，ArrayList里面的elementData未必是满的，比方说elementData有10的大小，但是我只用了其中的3个，那么是否有必要序列化整个elementData呢？显然没有这个必要，因此ArrayList中重写了writeObject方法：



> private void writeObject(java.io.ObjectOutputStream s)
>
> ​        throws java.io.IOException{
>
> // Write out element count, and any hidden stuff
>
> int expectedModCount = modCount;
>
> s.defaultWriteObject();
>
> ​        // Write out array length
>
> ​       s.writeInt(elementData.length);
>
> ​    // Write out all elements in the proper order.
>
> for (int i=0; i<size; i++)
>
> ​           s.writeObject(elementData[i]);
>
> ​    if (modCount != expectedModCount) {
>
> ​           throw new ConcurrentModificationException();
>
> ​    }
>
> }



每次序列化的时候调用这个方法，先调用defaultWriteObject()方法序列化ArrayList中的非transient元素，elementData不去序列化它，然后遍历elementData，只序列化那些有的元素，这样：



1、加快了序列化的速度



2、减小了序列化之后的文件大小



**HashMap和Hashtable的区别**



HashMap和Hashtable是一组相似的键值对集合，它们的区别也是面试常被问的问题之一，我这里简单总结一下HashMap和Hashtable的区别：



1、Hashtable是线程安全的，Hashtable所有对外提供的方法都使用了synchronized，也就是同步，而HashMap则是线程非安全的



2、Hashtable不允许空的value，空的value将导致空指针异常，而HashMap则无所谓，没有这方面的限制



3、上面两个缺点是最主要的区别，另外一个区别无关紧要，我只是提一下，就是两个的rehash算法不同，Hashtable的是：



> private int hash(Object k) {
>
>   // hashSeed will be zero if alternative hashing is disabled.
>
>   return hashSeed ^ k.hashCode();
>
> }



这个hashSeed是使用sun.misc.Hashing类的randomHashSeed方法产生的。HashMap的rehash算法上面看过了，也就是：



> static int hash(int h) {
>
>   // This function ensures that hashCodes that differ only by
>
>   // constant multiples at each bit position have a bounded
>
>   // number of collisions (approximately 8 at default load factor).
>
>   h ^= (h >>> 20) ^ (h >>> 12);
>
>   return h ^ (h >>> 7) ^ (h >>> 4);
>
> }



LinkedhashMap（双向链表结构）会记录他的下一个元素和前一个插入的元素和后一个插入元素；根据前一个插入元素和后一个插入元素来实现排序

LRU算法缓存原理：本次访问的数据会放到最后面，那么访问次数最少的会在第一位，然后删除第一位即可

