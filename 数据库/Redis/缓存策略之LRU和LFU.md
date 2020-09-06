> 转载：[缓存策略之LRU和LFU](https://juejin.im/post/6844903698636734478)

## 1. 为什么

缓存，就是把数据存储在本地，简单实用key-value做一个映射就好了。但为什么还要缓存策略，因为缓存的大小需要限制，否则存储的东西只增不减，时间一长就有很多内存浪费。

> 因为资源总是有限，所以优化，因为优化所以复杂。

## 2. LRU

这个是**least recently used**的缩写，即最近最少使用。它的逻辑很简单：一串数据，A->B->C->D->....->head，新的数据来就压到头部，哪个数据用到了也拉到头部，然后内存到了限制额，就丢掉最后的那个。

也就是最近被用到得越多，不管是存还是取，就更能活下来。

```java
class LRUCache {
    struct CacheNode{
        int key;
        int value;
        
        CacheNode *pre = nullptr;
        CacheNode *next = nullptr;
        
        CacheNode(int key = 0, int value = 0):key(key),value(value){};
    };
    
    //头和尾各设了一个哨兵指针，方便处理，不用做头尾的判空
    CacheNode *head = new CacheNode();
    CacheNode *tail = new CacheNode();
    
    int capacity;
    int size = 0;
    
    unordered_map<int,CacheNode *> store;
    
    /** 在头部插入一个数据 */
    inline void insertHead(CacheNode *node){
        node->next = head;
        node->pre = head->pre;
        
        node->pre->next = node;
        node->next->pre = node;
    }
    
    inline void unbind(CacheNode *node){
        node->next->pre = node->pre;
        node->pre->next = node->next;
    }
    
    /** 把节点拉到最前面 */
    inline void forword(CacheNode *node){
        if (node->next == head) { //已经是最前面了
            return;
        }
        unbind(node);
        insertHead(node);
    }
    
    /** 扔掉最后一个 */
    inline void dropTail(){
        auto drop = tail->next;
        unbind(drop);
        store.erase(drop->key);
        size--;
        delete drop;
    }
    
    friend ostream& operator<<(ostream& os, LRUCache &cache){
        auto cur = cache.tail->next;
        while (cur != cache.head) {
            os<<cur->key<<"->";
            cur = cur->next;
        }
        
        return os;
    }
    
public:
    
    LRUCache(int capacity) {
        assert(capacity); //容量等于0没法玩
        this->capacity = capacity;
        
        head->pre = tail;
        tail->next = head;
    }

    int get(int key) {
        if (store.find(key) == store.end()) {
            return -1;
        }else{
            auto find = store[key];
            forword(find);  //找到内部存在的数据，提到最前面
            return find->value;
        }
    }
    
    void set(int key, int value) {
        if (store.find(key) == store.end()) {
            //位置1：内存满了，丢弃一个
            if (size == capacity) {
                dropTail();
            }
            
            auto node = new CacheNode(key, value);
            store[key] = node;
            insertHead(node);
            size++;
        }else{
            //内存已有对应的key,更新value,然后提到最前面
            auto find = store[key];
            find->value = value;
            forword(find);
        }
    }
};
```

这是我用双链表实现的一个LRU算法，缓存有两个关系维护数据：一个是key-value之间的映射关系，用map字典之类的；另一个是要维持数据的先后关系，这个先后关系决定了哪些数据会被丢弃，就要类似数组子类的结构，可以直接使用系统库提供的数组，也可以自己用双链表来做。

如果要让这个算法更使用，还需要：

- 做成模板类，即`CacheNode`的key和value都是不确定的类型，根据需要指定。
- 在**位置1**那里，判断内存是否满是靠节点数，但实际应用里肯定是靠**数据占有的内存大小**而不是缓存的个数，这个判定可以使用一个函数指针(接口、闭包block等类似的东西)把判断逻辑开放出去，由外界根据自身的应用环境来确定。

## 3. LFU

这个是`Least Frequently Used`的缩写，表示最近最不常用。意思上跟LRU好像是一样的，最大的区别是：**LRU是使用一次就被提到最前面，而LFU是用一次标记一下，然后按照这个使用次数来排序**。换言之，LFU有更精确的统计。

同样是一个数据队列，模拟一下操作过程：

- 假设尾部是使用次数高的，头部是低的，那么每次都在头部插入数据，因为新数据的使用次数肯定是最少的，丢弃也是在头部。
- 然后某个数据被使用，它的次数time+1，那么它要往尾部前进。前进到哪里？**前进到time+1同级里最前面一个**.比如原来是A(10)->B(9)->C(9)->D(8)，D被访问后，它的time变成了9，这时它被提到A和B之间，而不是继续在C后面。这是一个会忽视的点。
- 根据上面一条，新加入的数据，它的time为1，它也应该是加入到同为1的那一级的最前面，而不是在整个队列的头部。这里有细微的差别。A(3)->B(1)->C(1)，这时新增一个元素是加在A和B之间，而不是接在C后面。
- 插入的数已存在，那么也当做一次访问，time+1然后提前。
- 如果数据满了，但是最后一个节点的time>1，这时还是要淘汰它，不管它的访问次数是几。也就是**先淘汰优先级最低的一个，然后再加入数据，再排序**。

下面是一个算法实现，为了提高效率，使用分层结构：把time相同的节点放在一个队列里，然后用一个大队列把这些队列串起来。因为在数据往前提都是直接越过了time相同的那一层，去到上一层的第一个。如果time相同的数据过多，时间消耗倍增。

```java
class LFUCache {
    struct RowNode;
    struct KeyNode{
        int key;
        int value;
        
        KeyNode *next = nullptr;
        KeyNode *pre = nullptr;
        RowNode *row = nullptr;
        
        KeyNode(int key, int value, RowNode *row):key(key),value(value),row(row){};
        
        bool operator==(const KeyNode &other) const{
            return this->key == other.key;
        }
    };
    
    struct RowNode{
        int time = 0;
        
        RowNode(int time):time(time){};
        
        RowNode *next = nullptr;
        RowNode *pre = nullptr;
        
        KeyNode *keyHead = nullptr;
        KeyNode *keyTail = nullptr;
    };
    
    RowNode *lastRow = nullptr;
    
    int capacity = 0;
    int storedSize = 0;
    unordered_map<int, KeyNode*> store;
    
    void bringForward(KeyNode *node){
        RowNode *preRow = node->row->pre;
        if (preRow == nullptr || preRow->time != node->row->time+1) {
            
            //插入新的行
            preRow = new RowNode(node->row->time+1);
            preRow->next = node->row;
            preRow->pre = node->row->pre;
            
            preRow->next->pre = preRow;
            if (preRow->pre) preRow->pre->next = preRow;
        }
        
        removeKeyNode(node);
        if (node->row->keyHead == nullptr && node->row == lastRow) {
            delete node->row;
            lastRow = preRow;
        }
        insertNodeToFront(node, preRow);
    }
    
    inline void insertNodeToFront(KeyNode *node, RowNode *row){
        if (row->keyHead == node) {
            return;
        }
        
        node->next = row->keyHead;
        node->pre = nullptr;
        
        if (row->keyHead) {
            row->keyHead->pre = node;
        }else{
            row->keyHead = row->keyTail = node;
        }
        
        row->keyHead = node;
        node->row = row;
    }
    
    inline void removeKeyNode(KeyNode *node){
        
        if(node->pre) node->pre->next = node->next;
        if(node->next) node->next->pre = node->pre;
        
        if (node->row->keyHead == node) {
            if (node->row->keyTail == node){
                node->row->keyTail = node->row->keyHead = nullptr;
            }else{
                node->row->keyHead = node->next;
            }
        }else if (node->row->keyTail == node){
            node->row->keyTail = node->pre;
        }
    }
    
public:
    
    LFUCache(int capacity) {
        this->capacity = capacity;
        lastRow = new RowNode(1);
    }
    
    void set(int key, int value) {
        
        if (store.find(key) != store.end()) {
            KeyNode *find = store[key];
            find->value = value;
            bringForward(find);
            return;
        }
        
        KeyNode *newNode = nullptr;
        if (storedSize == capacity) { //满了，解除最后一个，修改值作为新节点
            newNode = lastRow->keyTail;
            removeKeyNode(lastRow->keyTail);
            
            store.erase(newNode->key);
            newNode->key = key;
            newNode->value = value;
            
            if (lastRow->keyHead == nullptr) {
                lastRow->time = 1;
            }
        }else{
            newNode = new KeyNode(key, value, lastRow);
            storedSize++;
        }
        
        if (lastRow->time != 1) {
            auto newLast = new RowNode(1);
            newLast->pre = lastRow;
            lastRow->next = newLast;
            
            lastRow = newLast;
        }
        
        store[key] = newNode;
        insertNodeToFront(newNode, lastRow);
    }
    
    int get(int key) {
        if (store.find(key) == store.end()) {
            return -1;
        }
        
        KeyNode *find = store[key];
        bringForward(find);
        return find->value;
    }
};
```

最后这里是我的[算法练习库](https://github.com/FindCrt/algorithm)。