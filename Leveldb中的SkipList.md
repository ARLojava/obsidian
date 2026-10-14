
void Insert
bool Contains
Iterator

```c++
template <typename Key, class Comparator>
bool SkipList<Key, Comparator>::KeyIsAfterNode(const Key& key, Node* n) const {
  return (n != nullptr) && (compare_(n->key, key) < 0);  //如果next->key比key前则返回true，意味着key在next->key后面
}
```
+ as usually, compare(a,b) < 0 becouse a < b beyond its value
+ but in this situation ,compare on lexicographic,such as in dictionary,compare(“a","c") and return "a"<"c" but don't beyond its ASII value

```c++
SkipList<Key, Comparator>::FindGreaterOrEqual(const Key& key,
                                              Node** prev) const {
  Node* x = head_;    // 设置头指针
  int level = GetMaxHeight() - 1; // 从最高层往下找
  while (true) {
    Node* next = x->Next(level);   // 这一层中关于x的后继节点
    if (KeyIsAfterNode(key, next)) { // 若next->key < key (目标还在next的右侧)则在本层中继续往右移动
      x = next;
    } else {   // next为空或next->key >= key 则需要记录前驱并记录下一层
      if (prev != nullptr) prev[level] = x;    // 记录该层前驱节点
      if (level == 0) {  // 已到最底层了，
        return next;
      } else {
        level--;   // 往下一层找（更深一层)
      }
    }
  }
}
```


```c++
template <typename Key, class Comparator>

void SkipList<Key, Comparator>::Insert(const Key& key) {
  Node* prev[kMaxHeight];    // 前驱节点数组
  Node* x = FindGreaterOrEqual(key, prev);  // 返回比key大的或相等的节点指针
  assert(x == nullptr || !Equal(key, x->key));
  int height = RandomHeight();   //创建节点时随机的高度
  if (height > GetMaxHeight()) {
    for (int i = GetMaxHeight(); i < height; i++) {
      prev[i] = head_;
    }
    max_height_.store(height, std::memory_order_relaxed);
  }
  x = NewNode(key, height); // 创建一个Node节点
  for (int i = 0; i < height; i++) {
    x->NoBarrier_SetNext(i, prev[i]->NoBarrier_Next(i)); // 相当于x->next = prev->next， 将新节点x的next指向前驱节点的next
    prev[i]->SetNext(i, x); // 前驱节点next指向x
  }
}
```
std::atomic<Node*> next_[1]; 一种占位技巧
迭代器
