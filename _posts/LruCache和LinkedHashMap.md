---
title: LruCache和LinkedHashMap
date: 2017-12-05 12:38:42
tags:
categories: Android_基础
---



昨天笔试的时候，有一题是补充LruCache的实现，我想了想，应该不至于要写到链表实现把，于是写了LinkedHashMap进去了，<!--more-->面试的时候面试官问我，这题你是不是用手机查过了......。

于是今天补充下记忆，前提知识：知道LinkedHashMap本身有实现LRU算法，这个会在后面去写

Android下LruCache源码也就300多行，在android.util包下

```Java
//看了源码后可以发现LruCache实际上就是对LinkedHashMap的包装，只是增加了一些统计数据比如命中率，丢失率
//因为几乎每个方法都加锁，是一个线程安全的类
public class LruCache<K, V> {
   private final LinkedHashMap<K, V> map;
   private int size; //数据的条目数
   private int maxSize;  //LruCache的大小
  //构造函数很简单，就是new了一个LinkedHashMap,初始化大小为0，装载因子是0.75，最后一个参数表示
  //使用LRU算法 
  public LruCache(int maxSize) {
        if (maxSize <= 0) {
            throw new IllegalArgumentException("maxSize <= 0");
        }
        this.maxSize = maxSize;
        this.map = new LinkedHashMap<K, V>(0, 0.75f, true);
    }
  
  
    /**
     * Returns the value for {@code key} if it exists in the cache or can be
     * created by {@code #create}. If a value was returned, it is moved to the
     * head of the queue. This returns null if a value is not cached and cannot
     * be created.
     * 这里可以看到get仅仅是调用了LinkedHashMap的map.get(key)方法，加了锁，读取value本身
     * 应该是不用加锁的，猜测加锁是为了同步hitcount这些统计数据
     */
    public final V get(K key) {
        if (key == null) {
            throw new NullPointerException("key == null");
        }

        V mapValue;
        synchronized (this) {
            mapValue = map.get(key);
            if (mapValue != null) {
                hitCount++;
                return mapValue;
            }
            missCount++;
        }

        /*
         * Attempt to create a value. This may take a long time, and the map
         * may be different when create() returns. If a conflicting value was
         * added to the map while create() was working, we leave that value in
         * the map and release the created value.
         */
		//create(key)默认实现就是直接返回null,因此这里会直接return掉
        V createdValue = create(key);
        if (createdValue == null) {
            return null;
        }
		...
    }
  
    /**
     * Caches {@code value} for {@code key}. The value is moved to the head of
     * the queue.
     * 写操作，加锁，map.put(key, value)后返回了key之前对应的value  previous,safeSizeOf
     * 默认实现return 1,entryRemoved默认空实现，
     * 写入后检查如果超出了maxSize，就把最久的条目移出map直到size = maxSize 
     * @return the previous value mapped by {@code key}.
     */
    public final V put(K key, V value) {
        if (key == null || value == null) {
            throw new NullPointerException("key == null || value == null");
        }

        V previous;
        synchronized (this) {
            putCount++;
            size += safeSizeOf(key, value);
            previous = map.put(key, value);
            if (previous != null) {
                size -= safeSizeOf(key, previous);
            }
        }

        if (previous != null) {
            entryRemoved(false, key, previous, value);
        }

        trimToSize(maxSize);
        return previous;
    }
  
  /**
     * Remove the eldest entries until the total of remaining entries is at or
     * below the requested size.
     * 缩小LruCache到maxSize
     * 一个while循环，remove掉map.eldest()
     * @param maxSize the maximum size of the cache before returning. May be -1
     *            to evict even 0-sized elements.
     */
    public void trimToSize(int maxSize) {
        while (true) {
            K key;
            V value;
            synchronized (this) {
                if (size < 0 || (map.isEmpty() && size != 0)) {
                    throw new IllegalStateException(getClass().getName()
                            + ".sizeOf() is reporting inconsistent results!");
                }

                if (size <= maxSize) {
                    break;
                }

                Map.Entry<K, V> toEvict = map.eldest();
                if (toEvict == null) {
                    break;
                }

                key = toEvict.getKey();
                value = toEvict.getValue();
                map.remove(key);
                size -= safeSizeOf(key, value);
                evictionCount++;
            }

            entryRemoved(true, key, value, null);
        }
    }
  
  /**
     * Removes the entry for {@code key} if it exists.
     *	调用map.remove(key);
     * @return the previous value mapped by {@code key}.
     */
    public final V remove(K key) {
        if (key == null) {
            throw new NullPointerException("key == null");
        }

        V previous;
        synchronized (this) {
            previous = map.remove(key);
            if (previous != null) {
                size -= safeSizeOf(key, previous);
            }
        }

        if (previous != null) {
            entryRemoved(false, key, previous, null);
        }

        return previous;
    }
```

接下来，我们来看LinkeHashMap是怎么实现LRU算法的

```Java
//LinkedHashMap继承自HashMap
public class LinkedHashMap<K,V> extends HashMap<K,V> implements Map<K,V>
{
	//继承了HashMap的内部类，增加了before和after引用指向每一个<K,V>对的前后<K,V>对，
	//增加为双向链表表示
   static class LinkedHashMapEntry<K,V> extends HashMap.Node<K,V> {
        LinkedHashMapEntry<K,V> before, after;
        LinkedHashMapEntry(int hash, K key, V value, Node<K,V> next) {
            super(hash, key, value, next);
        }
    }
       /**
     * The head (eldest) of the doubly linked list.
     * 双链表的头结点
     */
    transient LinkedHashMapEntry<K,V> head;

    /**
     * The tail (youngest) of the doubly linked list.
     * 双链表的尾节点
     */
    transient LinkedHashMapEntry<K,V> tail;
    
     /**
     * The iteration ordering method for this linked hash map: <tt>true</tt>
     * for access-order, <tt>false</tt> for insertion-order.
     * 新增的构造函数参数，false表示按插入的顺序获取，true表示按接触的顺序遍历，此时就是
     * LRU实现
     * @serial
     */
    final boolean accessOrder;
}
```

看一下HashMap.Node是什么

```Java
        /**
         *  内部类Node实现了map.Entry接口，当我们存一个<K,V>对的时候，就会保存成Node类存
         *  到HashMap里面去
         *  Node类重写了Equals方法和hashCode方法，不过看代码中好像没有用到，比较的时候依旧
         *  是拿node1.key == node2.key2 和 node1.key.equals(node2.key)去比较的
         *
         */
 static class Node<K,V> implements Map.Entry<K,V> {
            final int hash;  //保存的是key的hash值
            final K key;
            V value;
            Node<K,V> next;

            Node(int hash, K key, V value, Node<K,V> next) {
                this.hash = hash;
                this.key = key;
                this.value = value;
                this.next = next;
            }

            public final K getKey()        { return key; }
            public final V getValue()      { return value; }
            public final String toString() { return key + "=" + value; }

            public final int hashCode() {
                return Objects.hashCode(key) ^ Objects.hashCode(value);
            }

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

然后是linkedhashMap的一些操作

```Java
//重写了HashMap的get方法，只是增加了 if (accessOrder) afterNodeAccess(e);
//修改访问顺序
 public V get(Object key) {
        Node<K,V> e;
        if ((e = getNode(hash(key), key)) == null)
            return null;
        if (accessOrder)
            afterNodeAccess(e);
        return e.value;
    }

public Map.Entry<K, V> eldest() {
 	 	return head;
	}			
//重写newNode方法，新增节点时链接到双链表的末尾
Node<K,V> newNode(int hash, K key, V value, Node<K,V> e) {
        LinkedHashMapEntry<K,V> p =
            new LinkedHashMapEntry<K,V>(hash, key, value, e);
        linkNodeLast(p);
        return p;
 }

//在get（K key）之后调用，把这个节点移到双链表的末尾
 void afterNodeAccess(Node<K,V> e) { // move node to last
        LinkedHashMapEntry<K,V> last;
        if (accessOrder && (last = tail) != e) {
            LinkedHashMapEntry<K,V> p =
                (LinkedHashMapEntry<K,V>)e, b = p.before, a = p.after;
            p.after = null;
            if (b == null)
                head = a;
            else
                b.after = a;
            if (a != null)
                a.before = b;
            else
                last = b;
            if (last == null)
                head = p;
            else {
                p.before = last;
                last.after = p;
            }
            tail = p;
            ++modCount;
        }
    }
```

可以看到，LinkedHashmap的LRU实现过程就是，新增的<K,V>对会链接到双链表的末尾，get操作后的<K,V>对也会把操作的节点双链表断开然后移动到双链表的末尾，每次取最久未使用的节点就是双链表的头结点，在LRUCache里面，每次要移除也就是移除的这个节点