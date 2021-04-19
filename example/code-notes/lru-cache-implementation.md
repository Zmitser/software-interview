---
title: Реализация LRUCache на Java
tags:
 - tutorials 
created: 2021-04-13
---

**LRU** (_least recently used_) — это алгоритм, при котором вытесняются значения, которые дольше всего не запрашивались. 
Соответственно, необходимо хранить время последнего запроса к значению. 
И как только число закэшированных значений превосходит N необходимо вытеснить из кеша значение, которое дольше всего не запрашивалось.

Исходя из определения, реализация LRUCache должна обладать следующими свойствами и атрибутами:

* **Граничный размер** `bounded size`: Кэш должен обладать ограниченным размером в соответствии с ограничениями по объему памяти
* **Быстрый доступ** `fast access`: При проектировании кэша мы должны обеспечить быстрый доступ и обновление данных
* **Удаление наименее часто используемых данных** `evict least recently used entry`: при достижении предела емкости кеша вытеснить данные, которые дольше всего не запрашивались

Для создания LRU-кеша мы можем использовать `LinkedHashMap`. В конструкторе можно задать порядок доступа `accessOrder`, который указывает, каким образом будет осуществляться доступ к элементам при использовании итератора. 
При значении true — по порядку последнего доступа (об этом в конце статьи). При значении false доступ осуществляется в том порядке, в котором элементы были вставлены. 

```java
/**
 * Constructs an empty <tt>LinkedHashMap</tt> instance with the
 * specified initial capacity, load factor and ordering mode.
 *
 * @param  initialCapacity the initial capacity
 * @param  loadFactor      the load factor
 * @param  accessOrder     the ordering mode - <tt>true</tt> for
 *         access-order, <tt>false</tt> for insertion-order
 * @throws IllegalArgumentException if the initial capacity is negative
 *         or the load factor is nonpositive
 */
public LinkedHashMap(int initialCapacity,
        float loadFactor,
        boolean accessOrder) {
        super(initialCapacity, loadFactor);
        this.accessOrder = accessOrder;
}
```

В итоге LRUCache будет выглядеть следующим образом:

```java
package org.arpit.java2blog;

import java.util.LinkedHashMap;
import java.util.Map;

class LRUCache {
    private LinkedHashMap<Integer, Integer> cacheMap;

    public LRUCache(int capacity) {
        cacheMap = new LinkedHashMap<>(capacity, 0.75f, true) {
            protected boolean removeEldestEntry(Map.Entry eldest) {
                return size() > capacity;
            }
        };
    }

    // This method works in O(1) 
    public int get(int key) {
        return cacheMap.getOrDefault(key, -1);
    }

    // This method works in O(1) 
    public void set(int key, int value) {
        cacheMap.put(key, value);
    }
}

public class LRUUsingLinkedHashMapMain {

    public static void main(String[] args) {
        LRUCache lrucache = new LRUCache(4);
        lrucache.set(1, 100);
        lrucache.set(10, 99);
        lrucache.set(15, 98);
        lrucache.set(10, 97);
        lrucache.set(12, 96);
        lrucache.set(18, 95);
        lrucache.set(1, 94);

        System.out.println(lrucache.get(1));
        System.out.println(lrucache.get(10));
        System.out.println(lrucache.get(15));

    }
}
```