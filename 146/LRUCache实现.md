# LRUCache实现

## 有序字典

使用有序字典的数据结构实现，在Java中可以继承【LinkedHashMap】来实现，综合了哈希表和链表。

```java
class LRUCache extends LinkedHashMap<Integer, Integer> {

    private int remian;

    public LRUCache(int capacity) {
        super(capacity, 0.75F, true);
        this.remian = capacity;
    }

    public int get(int key) {
        return super.getOrDefault(key, -1);
    }

    public void put(int key, int value) {
        super.put(key, value);
    }

    @Override
    protected boolean removeEldestEntry(Map.Entry<Integer, Integer> eldest) {
        return size() > this.remian;
    }
}
```



## 哈希表 + 双向链表

1. 哈希表用于快定位找缓存中的结点
2. 双向链表用于新增/删除

```java
import java.util.Hashtable;

class LRUCache {
    /**
     * 定义结点
     */
    class DLinkNode {
        int key;
        int value;
        DLinkNode prev;
        DLinkNode next;
    }

    /**
     * 1. 链表当前size
     * 2. 链表的capacity
     * 3. 双向链表的头指针和尾指针
     * 4. 哈希表，用于快速定位结点
     */
    private int size;
    private int capacity;
    private DLinkNode head, tail;
    private Hashtable<Integer, DLinkNode> cache;

    /**
     * 向链表中新增结点
     *
     * @param node
     */
    private void addNode(DLinkNode node) {
        node.prev = head;
        node.next = head.next;
        head.next.prev = node;
        head.next = node;
    }

    /**
     * 链表删除结点
     *
     * @param node
     */
    private void removeNode(DLinkNode node) {
        node.prev.next = node.next;
        node.next.prev = node.prev;
    }

    /**
     * 将某个结点直接移动到头指针的位置
     * 1. 先将结点remove
     * 2. 再将此结点重新加入聊表
     *
     * @param node
     */
    private void moveToHead(DLinkNode node) {
        removeNode(node);
        addNode(node);
    }

    /**
     * 删除并返回尾部的结点
     *
     * @return
     */
    private DLinkNode popTail() {
        DLinkNode node = tail.prev;
        removeNode(node);
        return node;
    }

    public LRUCache(int capacity) {
        this.size = 0;
        this.capacity = capacity;
        this.cache = new Hashtable<>();

        head = new DLinkNode();
        tail = new DLinkNode();
        head.next = tail;
        tail.prev = head;
    }

    public int get(int key) {
        DLinkNode node = this.cache.get(key);
        if (node == null) {
            return -1;
        }

        moveToHead(node);
        return node.value;
    }

    /**
     * 1. 新增既存结点：将结点移动到头部
     * 2. 新增新结点，将新结点加入到哈希表中并加入链表。
     *    如果超出链表capacity时。将尾部的结点抛出。
     *
     * @param key
     * @param value
     */
    public void put(int key, int value) {
        DLinkNode node = this.cache.get(key);
        if (node == null) {
            DLinkNode newNode = new DLinkNode();
            newNode.key = key;
            newNode.value = value;

            this.cache.put(key, newNode);
            addNode(newNode);
            size++;

            if (size > capacity) {
                DLinkNode tail = popTail();
                this.cache.remove(tail.key);
                size--;
            }
        } else {
            node.value = value;
            moveToHead(node);
        }
    }

}
```

