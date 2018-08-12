---
title: Android消息队列和链表
date: 2017-11-19 23:07:06
tags:
categories: 数据结构和算法
---



链表是很常用的数据结构，JDK已经提供了LinkedList，也就是双向链表。今天写了个单链表，主要是想写一下单链表的逆置：

<!--more-->

```java
public class MyList<E> implements MyListInterface<E> {
    private Node<E> first;
    private Node<E> last;  //用于添加到尾节点
    private int size;

    @Override
    public void add(E e) {
        linkInLast(new Node<>(e, null));
    }

    @Override
    public void add(int index, E e) {
        Node temp = first;
        Node prev = null;
        for (int i = 0; i < index; i++) {
            prev = temp;
            temp = temp.next;
        }

        Node<E> node = new Node<>(e, null);
        link(prev, node);
    }

    private void link(Node<E> prev, Node<E> current) {
        if (prev == null) {
            linkInFirst(current);
        } else if (current.next == null) {
            linkInLast(current);
        } else {
            current.next = prev.next;
            prev.next = current;
            size++;
        }
    }

    private void linkInFirst(Node<E> node) {
        if (first == null) {
            first = node;
            last = node;
        } else {
            node.next = first;
            first = node;
        }
        size++;
    }

    private void linkInLast(Node<E> node) {
        if (last == null) {
            first = node;
            last = node;
        } else {
            last.next = node;
            last = node;
        }
        size++;
    }

    @Override
    public void remove(E e) {
        Node temp = first;
        Node prev = null;
        while ((temp != null)) {
            if (temp.data.equals(e)) {
                unlink(prev, temp);
            }
            prev = temp;
            temp = temp.next;

        }
    }

    private void unlink(Node<E> prev, Node<E> current) {
        if (prev == null) {  //说明要删掉的节点是头结点
            first = current.next;
        } else if (current.next == null) {  //说明要删掉的节点是尾节点
            prev.next = null;
            last = prev;
        } else {
            prev.next = current.next;
        }
        current.data = null;
        size--;

        if (size == 0) {
            first = last = null;
        }
    }

    @Override
    public boolean contains(E e) {
        return (indexOf(e) != -1);
    }

    public int indexOf(E e) {
        int index = 0;
        Node temp = first;
        while ((temp != null)) {
            if (temp.data.equals(e)) {
                return index;
            }
            temp = temp.next;
            index++;
        }
        return -1;
    }

    @Override
    public int size() {
        return size;
    }

    public void print() {
        Node temp = first;
        if (first != null) {
            System.out.print("first.data : " + first.data);
        } else {
            System.out.print("first : null ");
        }
        if (last != null) {
            System.out.print(",last.data : " + last.data);
        } else {
            System.out.print(",last : null ");
        }
        System.out.print(", size = " + size());
        System.out.print(", list: ");
        while ((temp != null)) {
            System.out.print(temp.data);
            System.out.print(" ");
            temp = temp.next;
        }
        System.out.println();
    }

    public void reverse() {
        if (size == 0 || size == 1) {
            return;
        }
        if (size == 2) {
            last.next = first;
            first.next = null;
            first = last;
            last = first.next;
        } else {
            last = first;
            Node current = first;
            Node prev = null;
            Node next = current.next;
            while (next != null) {
              //实际的操作就这一步，然后就是不停的循环赋值了
                current.next = prev;
                prev = current;
                current = next;
                next = next.next;
            }
            current.next = prev;
            first = current;
        }
    }

    private static class Node<E> {
        E data;
        Node next;

        public Node(E e, Node next) {
            this.data = e;
            this.next = next;
        }
    }

}
```

不过在网上看了下，好像别人实现的都是一个Node类不停next就完了，我这里还写了个List类感觉有点格格不入。。Android里面在MessageQueue和Message类里面都有用到链表:



MessageQueue::mMessages

```Java
public final class MessageQueue {
  	...
    //链表的头节点
    Message mMessages;
    private final ArrayList<IdleHandler> mIdleHandlers = new ArrayList<IdleHandler>();
 	...
    Message next() {
     		...
            synchronized (this) {
			   ...
                Message msg = mMessages;
                ...
                if (msg != null) {
                  		...
                          //链表取节点操作
                        mMessages = msg.next;
                        msg.next = null;
                        if (DEBUG) Log.v(TAG, "Returning message: " + msg);
                        msg.markInUse();
                        return msg;
                    }
                } 
			   ...
        }
    }
    ...
    boolean enqueueMessage(Message msg, long when) {
        ...
        synchronized (this) {
           ...
            msg.markInUse();
            msg.when = when;
            Message p = mMessages;
            ...
            if (p == null || when == 0 || when < p.when) {
                // New head, wake up the event queue if blocked.
              //如果mMessages为空，那么新发送的这个消息就成为头结点
                msg.next = p;
                mMessages = msg;
                needWake = mBlocked;
            } else {
                for (;;) {
                    prev = p;
                    p = p.next;
                    if (p == null || when < p.when) {
                        break;
                    }
                    if (needWake && p.isAsynchronous()) {
                        needWake = false;
                    }
                }
              //否则根据时间找个合适的位置插入到链表中
                msg.next = p; // invariant: p == prev.next
                prev.next = msg;
            }

            // We can assume mPtr != 0 because mQuitting is false.
            if (needWake) {
                nativeWake(mPtr);
            }
        }
        return true;
    }
```



Message类里面也有类似的成员变量 Message::sPool

```Java
public final class Message implements Parcelable {
   ...
    // sometimes we store linked lists of these things
     //链表节点定义
    /*package*/ Message next;

    private static final Object sPoolSync = new Object();
    //这是一个静态成员变量，用于指示消息池里面的消息链表的头指针
    private static Message sPool;
    private static int sPoolSize = 0;

    private static final int MAX_POOL_SIZE = 50;

    private static boolean gCheckRecycle = true;

    /**
     * Return a new Message instance from the global pool. Allows us to
     * avoid allocating new objects in many cases.
       取出消息时，取出sPool所指向的那个消息
     */
    public static Message obtain() {
        synchronized (sPoolSync) {
            if (sPool != null) {
                Message m = sPool;
                sPool = m.next;
                m.next = null;
                m.flags = 0; // clear in-use flag
                sPoolSize--;
                return m;
            }
        }
        return new Message();
    }
  
  //由MessageQueue调用，回收消息加入到消息池
   void recycleUnchecked() {
        // Mark the message as in use while it remains in the recycled object pool.
        // Clear out all other details.
        flags = FLAG_IN_USE;
        what = 0;
        arg1 = 0;
        arg2 = 0;
        obj = null;
        replyTo = null;
        sendingUid = -1;
        when = 0;
        target = null;
        callback = null;
        data = null;
		//sPool是一个头节点
        //回收时，设置自己的next节点为sPool所指向的节点，然后把自己成为新的sPool
        synchronized (sPoolSync) {
            if (sPoolSize < MAX_POOL_SIZE) {
                next = sPool;
                sPool = this;
                sPoolSize++;
            }
        }
    }
```

我到底想说啥......