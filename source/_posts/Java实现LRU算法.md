---
title: Java实现LRU算法
date: 2018-09-24 16:09:29
updated: 2018-09-24 16:09:29
categories: tech
tags: 
- Java
- OS
description: 操作系统里的LRU算法，Least Recently Used大家都很熟悉，在操作系统里学习过，Java里的LinkedHashMap已经就是LRU的典型应用.
---
## Java里自带LinkedHashMap实现

Java里的LinkedHashMap就是以一个双向链表来实现的，可以制定按查询排序或者是插入排序，使用时继承改集合，初始化时指定好一些参数、重写删除最老元素的方法。

```java
import java.util.Iterator;
import java.util.LinkedHashMap;
import java.util.Map;

/**
 * 使用Java自带的LinkedHashMap实现LRU缓存
 */
public class UseLinkedHashMap {
    public static void main(String[] args) {
        UserLinkedHashMap<Integer, String> LRUCache = new UserLinkedHashMap<>(4);
        LRUCache.put(1, "One");
        LRUCache.put(2, "two");
        LRUCache.put(3, "three");
        LRUCache.put(4, "four");
        LRUCache.put(2, "two");
        LRUCache.put(3, "three");
        Iterator<Map.Entry<Integer, String>> it = LRUCache.entrySet().iterator();
        while (it.hasNext()) {
            Map.Entry<Integer, String> item = it.next();
            System.out.println("key: " + item.getKey() + " / value: " + item.getValue());
        }
    }
}

class UserLinkedHashMap<K, V> extends LinkedHashMap<K, V> {
    private int cacheSize;

    public UserLinkedHashMap(int cacheSize) {
        super(16, 0.75f, true);
        this.cacheSize = cacheSize;
    }

    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return super.removeEldestEntry(eldest);
    }
}
```

## 数组实现

数组实现比较简单，元素用数组存储，有涉及到顺序修改时，将数组整体移位

```java
/**
 * 使用数组实现的LRU算法，刚使用的放在数组最后，主要考察元素是否已经存在和缓存是否已满这两种情况，使用与否只考察了set，没考察get
 */
public class MyArrayLRU {
    /**
     * 计数
     */
    private static int count = 0;
    /**
     * 已有元素个数
     */
    private static int size = 0;
    /**
     * 缓存大小
     */
    private int maxSize;
    /**
     * 使用数组存储元素
     */
    private int[] listArray;

    public MyArrayLRU(int max) {
        listArray = new int[max];
        this.maxSize = max;
    }

    public int getSize() {
        return size;
    }

    public Object get(int index) {
        return listArray[index];
    }

    private void moveArrayElement(int[] arr, int start, int end) {
        for (int i = start; i <= end; i++) {
            arr[i] = arr[i + 1];
        }
    }

    public void insert(int item) {
        //第一步查看是否已存在
        boolean exist = false;
        int existLocation = 0;
        for (int i = 0; i < maxSize; i++) {
            if (item == listArray[i]) {
                exist = true;
                existLocation = i;
            }
        }

        // 分情况查看是否已经满了
        if (size < maxSize) {
            if (exist) {
                if (existLocation < size - 1) {
                    moveArrayElement(listArray, existLocation, size - 2);
                }
                listArray[size - 1] = item;
            } else {
                listArray[size] = item;
                size++;
            }
        } else {
            if (!exist || item == listArray[0]) {
                moveArrayElement(listArray, 0, maxSize - 2);
            } else if (item != listArray[maxSize - 1]) {
                moveArrayElement(listArray, existLocation, maxSize - 2);
            }
            listArray[maxSize - 1] = item;
        }
        count++;
    }

    // 测试
    public static void main(String[] args) {
        int cacheSize = 5;
        MyArrayLRU lru = new MyArrayLRU(cacheSize);
        try {
            lru.insert(1);
            lru.insert(2);
            lru.insert(3);
            lru.insert(4);
            lru.insert(5);
            lru.insert(2);
            lru.insert(3);
            lru.insert(5);
            lru.insert(4);
            lru.insert(4);
            lru.insert(5);
            lru.insert(1);
            for (int i = 0; i < cacheSize; i++) {
                System.out.println(lru.get(i));
            }
            System.out.println("成功插入" + count + "次元素.");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}



```

## 链表实现

上述的数组实现在需要改变顺序的时候需要数组整体复制移位，资源消耗较大，链表在元素的增删上效率较高，对于存储的key-value节点的链表，链表的外部还套了一层key与节点一一对应，这个key可以用来判断元素是否已经存在，用于获取节点，获取链表大小等操作，弥补了链表一定功能的缺失。

```java
import java.util.HashMap;

/**
 * 链表的实现思路，Node形成的链表外部再加一层，用于弥补链表的随机查找，链表头部为最近使用的节点
 */
public class MyLinkedLRU {
    /**
     * 缓存容量
     */
    private int maxSize;
    /**
     * 记录头尾
     */
    private Node head, tail;
    /**
     * 外层套一层HashMap，内部使用以Node为节点的链表
     */
    private HashMap<Integer, Node> keyNodeMap;

    public MyLinkedLRU(int maxSize) {
        this.maxSize = maxSize;
        head = new Node(-1, -1);
        tail = new Node(0, 0);
        head.next = tail;
        tail.pre = head;
        this.keyNodeMap = new HashMap<Integer, Node>();
    }

    public int get(int key) {
        Node node = keyNodeMap.get(key);
        if (node != null) {
            moveToHead(node);
            return node.value;
        }
        return -1;
    }

    public void set(int key, int value) {
        Node node = null;
        if (keyNodeMap.containsKey(key)) {
            node = keyNodeMap.get(key);
            node.value = value;
        } else {
            node = new Node(key, value);
            if (keyNodeMap.size() == maxSize) {
                keyNodeMap.remove(removeTail());
            }
            keyNodeMap.put(key, node);
        }
        moveToHead(node);
    }

    /**
     * 将该节点移动到头部
     * @param node
     */
    private void moveToHead(Node node) {
        if (node.pre != null || node.next != null) {
            node.next.pre = node.pre;
            node.pre.next = node.next;
        }
        node.next = head.next;
        head.next.pre = node;
        node.pre = head;
        head.next = node;
    }

    /**
     * 找到尾部节点（最没被使用）的key值
     * @return
     */
    private int removeTail() {
        int lastKey = -1;
        if (tail.pre != head) {
            Node lastNode = tail.pre;
            lastKey = lastNode.key;
            lastNode.pre.next = tail;
            tail.pre = lastNode.pre;
            lastNode = null;
        }
        return lastKey;
    }

    /**
     * 重写toString()，按链表顺序打印
     * @return
     */
    @Override
    public String toString() {
        String res = "";
        Node item = head;
        while (item != tail.pre) {
            res += "[" + item.next.key + ", " + item.next.value + "] ";
            item = item.next;
        }
        return res;
    }

    public static void main(String[] args) {
        MyLinkedLRU lru = new MyLinkedLRU(5);
        lru.set(1, 1);
        lru.set(2, 2);
        lru.set(3, 3);
        lru.set(4, 4);
        lru.set(5, 5);
        lru.set(6, 6);
        lru.set(7, 7);
        lru.set(1, 1);
        lru.set(2, 2);
        lru.set(5, 5);
        lru.set(8, 8);
        lru.get(5);
        System.out.println(lru.toString());
    }
}

/**
 * 链表的节点
 */
class Node {
    int key;
    int value;
    Node pre;
    Node next;

    public Node(int k, int v) {
        key = k;
        value = v;
    }
}
```

**有任何疑问欢迎随时讨论，关于LRU的算法实现思路肯定不止这么几种，应该还会有效率更高的思路**