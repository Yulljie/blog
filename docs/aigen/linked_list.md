

# C++ 链表教学

---

## 一、先搞清楚链表是什么——和数组做对比

**数组**是一排连续的储物柜，编号0、1、2、3……你可以直接跑到第 n 号柜子前打开它（随机访问 O(1)），但要在中间插一个柜子？后面所有柜子都得往后挪（插入 O(n)）。

**链表**是一串散落各处的盒子，每个盒子里除了存你的东西，还多放了一张纸条，写着"下一个盒子的地址"。你不能直接跳到第 n 个（访问 O(n)），但要在中间插/删一个盒子？只需要改两张纸条（插入/删除 O(1)）。

```
数组:  [ A | B | C | D | E ]   ← 内存连续
        0   1   2   3   4

链表:  [A|→] → [B|→] → [C|→] → [D|→] → [E|∅]
       散落在内存各处，靠指针串起来
```

**什么时候用链表？** 频繁在中间位置做插入/删除、不需要随机访问的场景。

---

## 二、从最简单的单向链表开始

### 2.1 节点的定义

一个链表节点只需要两样东西：**数据** 和 **指向下一个节点的指针**。

```cpp
struct Node {
    int data;       // 存的东西
    Node* next;     // 指向下一个节点的指针（纸条上的地址）

    // 构造函数，方便创建节点
    Node(int val) : data(val), next(nullptr) {}
};
```

> 💡 `next` 是一个指向 `Node` 自身类型的指针——这是链表的核心：**结构体内包含指向同类型的指针**，构成递归式的链式结构。

### 2.2 链表类的骨架

```cpp
class LinkedList {
private:
    Node* head;  // 头指针：指向第一个节点

public:
    LinkedList() : head(nullptr) {}  // 初始为空链表
    ~LinkedList();                   // 析构函数：释放所有节点

    void push_front(int val);        // 头插
    void push_back(int val);         // 尾插
    void insert_after(Node* pos, int val); // 在某节点后插入
    void remove(int val);            // 删除第一个值为 val 的节点
    Node* find(int val);             // 查找
    void print();                    // 打印
};
```

---

## 三、逐个实现核心操作

### 3.1 头插 `push_front` — 最简单的插入

```cpp
void LinkedList::push_front(int val) {
    Node* new_node = new Node(val);
    new_node->next = head;  // 新节点指向原来的头
    head = new_node;        // 头指针指向新节点
}
```

图解：
```
插入前:  head → [B|→] → [C|∅]
插入 A:  head → [A|→] → [B|→] → [C|∅]
                 ↑ new_node
```

**时间复杂度：O(1)**——不管链表多长，只改两个指针。

### 3.2 尾插 `push_back`

```cpp
void LinkedList::push_back(int val) {
    Node* new_node = new Node(val);

    if (!head) {                // 链表为空，新节点就是头
        head = new_node;
        return;
    }

    Node* cur = head;
    while (cur->next) {         // 走到最后一个节点
        cur = cur->next;
    }
    cur->next = new_node;       // 最后一个节点指向新节点
}
```

**时间复杂度：O(n)**——必须遍历到尾部。（这也是为什么如果频繁尾插，会维护一个 `tail` 指针优化到 O(1)。）

### 3.3 在指定节点后插入

```cpp
void LinkedList::insert_after(Node* pos, int val) {
    if (!pos) return;

    Node* new_node = new Node(val);
    new_node->next = pos->next;  // 新节点接上后续
    pos->next = new_node;        // pos 指向新节点
}
```

图解：
```
在 B 后插入 X:
前:  [A|→] → [B|→] → [C|∅]
后:  [A|→] → [B|→] → [X|→] → [C|∅]
```

**⚠️ 注意赋值顺序**——必须先让 `new_node->next` 接上后续，再改 `pos->next`。反过来会丢失后面的链。

### 3.4 删除节点

```cpp
void LinkedList::remove(int val) {
    if (!head) return;

    // 特殊情况：要删的是头节点
    if (head->data == val) {
        Node* to_delete = head;
        head = head->next;
        delete to_delete;
        return;
    }

    // 一般情况：找到目标节点的前一个节点
    Node* cur = head;
    while (cur->next && cur->next->data != val) {
        cur = cur->next;
    }

    if (cur->next) {  // 找到了
        Node* to_delete = cur->next;
        cur->next = to_delete->next;  // 跳过要删的节点
        delete to_delete;             // 释放内存
    }
}
```

图解：
```
删除 B:
前:  [A|→] → [B|→] → [C|∅]
后:  [A|→] ————————→ [C|∅]
              [B] 被 delete
```

**关键点：** 删除需要找到**前驱节点**，因为要改前驱的 `next` 指针。这是单向链表的一个固有不便——也是双向链表存在的理由。

### 3.5 查找

```cpp
Node* LinkedList::find(int val) {
    Node* cur = head;
    while (cur) {
        if (cur->data == val) return cur;
        cur = cur->next;
    }
    return nullptr;  // 没找到
}
```

### 3.6 打印

```cpp
void LinkedList::print() {
    Node* cur = head;
    while (cur) {
        std::cout << cur->data;
        if (cur->next) std::cout << " → ";
        cur = cur->next;
    }
    std::cout << " → ∅\n";
}
```

### 3.7 析构函数——别忘了释放内存

```cpp
LinkedList::~LinkedList() {
    Node* cur = head;
    while (cur) {
        Node* next = cur->next;  // 先保存下一个
        delete cur;              // 再释放当前
        cur = next;
    }
}
```

> ⚠️ **这是 C++ 手动管理内存的经典场景**：`new` 出来的每个节点，必须有对应的 `delete`，否则内存泄漏。

---

## 四、完整可运行示例

```cpp
#include <iostream>

struct Node {
    int data;
    Node* next;
    Node(int val) : data(val), next(nullptr) {}
};

class LinkedList {
private:
    Node* head;
public:
    LinkedList() : head(nullptr) {}

    ~LinkedList() {
        Node* cur = head;
        while (cur) {
            Node* next = cur->next;
            delete cur;
            cur = next;
        }
    }

    void push_front(int val) {
        Node* n = new Node(val);
        n->next = head;
        head = n;
    }

    void push_back(int val) {
        Node* n = new Node(val);
        if (!head) { head = n; return; }
        Node* cur = head;
        while (cur->next) cur = cur->next;
        cur->next = n;
    }

    void remove(int val) {
        if (!head) return;
        if (head->data == val) {
            Node* d = head;
            head = head->next;
            delete d;
            return;
        }
        Node* cur = head;
        while (cur->next && cur->next->data != val)
            cur = cur->next;
        if (cur->next) {
            Node* d = cur->next;
            cur->next = d->next;
            delete d;
        }
    }

    void print() {
        Node* cur = head;
        while (cur) {
            std::cout << cur->data;
            if (cur->next) std::cout << " -> ";
            cur = cur->next;
        }
        std::cout << " -> null\n";
    }
};

int main() {
    LinkedList list;

    list.push_back(1);
    list.push_back(2);
    list.push_back(3);
    list.push_front(0);

    std::cout << "构建后: ";
    list.print();  // 0 -> 1 -> 2 -> 3 -> null

    list.remove(2);
    std::cout << "删除2: ";
    list.print();  // 0 -> 1 -> 3 -> null

    return 0;
}
```

---

## 五、进阶：双向链表

单向链表只能往前走，删除时需要找前驱。**双向链表**每个节点多一个 `prev` 指针，可以双向遍历：

```cpp
struct DNode {
    int data;
    DNode* prev;
    DNode* next;
    DNode(int val) : data(val), prev(nullptr), next(nullptr) {}
};
```

```
∅ ← [A|↔] ⇄ [B|↔] ⇄ [C|↔] → ∅
```

双向链表的优势：
- 删除某个节点时**不需要额外找前驱**，因为 `node->prev` 直接可用
- 可以从尾部反向遍历
- `std::list` 就是双向链表的实现

代价：每个节点多一个指针的内存开销，插入/删除时需要维护两个方向的指针。

---

## 六、现实中：`std::list` 和 `std::forward_list`

实际工程中你很少需要手写链表，标准库已经提供了：

| STL 容器 | 底层结构 | 特点 |
|----------|---------|------|
| `std::forward_list` | **单向链表** | 最小内存开销，只能单向遍历 |
| `std::list` | **双向链表** | 双向遍历，O(1) 任意位置插删 |

```cpp
#include <list>

std::list<int> lst = {1, 2, 3, 4};
auto it = std::find(lst.begin(), lst.end(), 2);
lst.insert(it, 99);  // 在 2 前面插入 99 → {1, 99, 2, 3, 4}
lst.erase(it);        // 删除 2 → {1, 99, 3, 4}
```

**但手写链表依然是必修课**——它训练的是对指针操作和内存管理的直觉，这是理解 C++ 底层机制的基本功。

---

## 七、常见踩坑点总结

1. **指针赋值顺序错误** → 链断了，节点丢失（尤其插入操作）
2. **忘记处理空链表** → `head` 为 `nullptr` 时解引用，段错误
3. **忘记处理头节点特殊情况** → 删除头节点时没更新 `head`
4. **内存泄漏** → `delete` 了节点但忘记先保存 `next`，或者根本没 `delete`
5. **悬垂指针** → `delete` 后继续使用原指针

---



