#### 双向链表+hashmap实现

```
import java.util.HashMap;
class Entry<K, V> {
    public Entry pre;
    public Entry next;
    public K key;
    public V value;
    Entry(){

    }
    Entry(K key,V value){
        this.key = key;
        this.value = value;
    }
}
public class LRUCache<K, V> {
    private int capacity;
    private Entry head = new Entry();
    private Entry tail = new Entry();

    private HashMap<K, Entry<K, V>> map;

    public LRUCache(int cacheSize) {
        capacity = cacheSize;
        map = new HashMap<K, Entry<K, V>>();
        head.next = tail;
        tail.pre = head;
    }

    public V get(K key){
        Entry<K, V> node = map.get(key);
        if (node == null){
            return null;
        }
        moveToHead(node);
        return node.value;

    }
    public void put(K key, V value) {
        Entry node = map.get(key);
        if(node == null){
            Entry newNode = new Entry(key,value);
            map.put(key,newNode);
            addNode(newNode);
            if(map.size() > capacity){
                // pop the tail
                Entry tail = popTail();
                 map.remove(tail.key);
            }
        }else{
            node.value = value;
            moveToHead(node);
        }
    }

    private void moveToHead(Entry node){
        removeNode(node);
        addNode(node);
    }
    //head——0——1——2——3——tail
    private void addNode(Entry node){
		//插入到head和0中间
        node.pre = head;
        node.next = head.next;     
        head.next.pre = node;
        head.next = node;
    }
    //head——0——1——2——3——tail
    private void removeNode(Entry node){
        Entry pre = node.pre;//1
        Entry next = node.next;//3
        next.pre = pre;
        pre.next = next;
    }
    private Entry popTail(){
        Entry res = tail.pre;
        this.removeNode(res);
        return res;
    }
    @Override
    public String toString() {
        StringBuilder sb = new StringBuilder();
        Entry entry = head;
        while (entry != null) {
            sb.append(String.format("%s:%s ", entry.key, entry.value));
            entry = entry.next;
        }
        return sb.toString();
    }
}
```

#### LinkedHashMap

```
public class LRUCache<K, V> extends LinkedHashMap<K, V> {
    private int capacity;
    public LRUCache(int cacheSize) {
        super((int) Math.ceil(cacheSize / 0.75) + 1, 0.75f, true);
        capacity = cacheSize;
    }
    @Override
    protected boolean removeEldestEntry(Map.Entry eldest) {
        return size() > capacity;
    }

    @Override
    public String toString() {
        StringBuilder sb = new StringBuilder();
        for (Map.Entry<K, V> entry : entrySet()) {
            sb.append(String.format("%s:%s ", entry.getKey(), entry.getValue()));
        }
        return sb.toString();
    }
}
```



#### 测试

```
    public static void main(String[] args) {
        LRUCache<Integer, String> lru = new LRUCache(5);
        lru.put(1, "11");
        lru.put(2, "11");
        lru.put(3, "11");
        lru.put(4, "11");
        lru.put(5, "11");
        System.out.println(lru.toString());
        lru.put(6, "66");
        lru.get(2);
        lru.put(7, "77");
        lru.get(4);
        System.out.println(lru.toString());

    }
```

> 输出：
>
> **list+hashmap**
>
> null:null 5:11 4:11 3:11 2:11 1:11 null:null 
> null:null 4:11 7:77 2:11 6:66 5:11 null:null 
>
> **linkedhashmap**
>
> 1:11 2:11 3:11 4:11 5:11 
> 5:11 6:66 2:11 7:77 4:11 