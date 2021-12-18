---

title: 基本数据结构
categories:
- 算法
---

***基于《算法导论》中的伪代码 以及部分网络资源***

# Stack

## Class

```c++
class Stack
{
private:
    int *m_stack;
    int max_size;
    int m_top;

public:
    Stack(int size)
    {
        this->max_size = size;
        this->m_top = -1;
        this->m_stack = new int[this->max_size];
    };
    bool STACK_EMPTY();
    void PUSH(int x);
    int POP();
    void PRINTER();
    ~Stack() { delete[] this->m_stack; };
};
```

## Empty


```c++
bool Stack::STACK_EMPTY()
{
    if (this->m_top == 0)
    {
        return true;
    }
    else
    {
        return false;
    }
}
```

## Push


```c++
void Stack::PUSH(int x)
{
    if (this->m_top <= this->max_size - 1)
    {
        this->m_top++;
        this->m_stack[this->m_top] = x;
    }
    else
    {
        //Full
        this->max_size++;
        this->m_top++;
        int *temp = new int[this->max_size];
        for (int i = 0; i < this->max_size - 1; i++)
        {
            temp[i] = this->m_stack[i];
        }
        temp[this->max_size - 1] = x;

        delete[] this->m_stack;
        this->m_stack = temp;
    }
}
```

## Pop

```c++
int Stack::POP()
{
    if (this->STACK_EMPTY())
    {
        throw "Underflow";
    }
    else
    {
        this->m_top--;
        return this->m_stack[this->m_top + 1];
    }
}
```
## Printer

```c++
void Stack::PRINTER()
{
    for (int i = 0; i <= this->m_top; i++)
    {
        cout << this->m_stack[i] << " ";
    }
    cout << endl;
}
```

# Queue

## Class

```c++
class Queue
{
private:
    int *m_queue;
    int m_head;
    int m_tail;
    int max_size;
    int temp_size;

public:
    Queue(int size)
    {
        this->m_queue = new int[size];
        this->max_size = size;
        this->m_head = 0;
        this->m_tail = 0;
        //current num of elements in queue
        this->temp_size = 0;
    };
    ~Queue() { delete[] this->m_queue; };
    void ENQUEUE(int x);
    void PRINTER();
    int DEQUEUE();
};
```

## Enqueue

```c++
void Queue::ENQUEUE(int x)
{
    if (this->m_head == this->m_tail && this->temp_size != 0)
    {
        throw "Queue if full";
    }

    this->m_queue[this->m_tail] = x;
    this->temp_size++;
    if (this->m_tail == this->max_size - 1)
    {
        this->m_tail = 0;
    }
    else
    {
        this->m_tail++;
    }
}
```

## Dequeue

```c++
int Queue::DEQUEUE()
{
    if (this->temp_size == 0)
    {
        throw "Queue is empty";
    }

    int x = this->m_queue[this->m_head];
    if (this->m_head == this->max_size - 1)
    {
        this->m_head = 0;
    }
    else
    {
        this->m_head++;
    }

    this->temp_size--;
    return x;
}
```

## Printer

```c++
//print the queue in order
void Queue::PRINTER()
{
    if (this->m_head < this->m_tail)
    {
        for (int i = this->m_head; i < this->m_tail; i++)
        {
            cout << this->m_queue[i] << " ";
        }
        cout << endl;
    }
    else if (this->m_head > this->m_tail)
    {
        for (int i = this->m_head; i < this->max_size; i++)
        {
            cout << this->m_queue[i] << " ";
        }
        for (int i = 0; i < this->m_tail; i++)
        {
            cout << this->m_queue[i] << " ";
        }
        cout << endl;
    }
    else if (this->temp_size == 0)
    {
        cout << "Queue is empty" << endl;
    }
    else if (this->m_head == this->m_tail && this->temp_size != 0)
    {
        cout << "Queue is full" << endl;
        for (int i = this->m_head; i < this->max_size; i++)
        {
            cout << this->m_queue[i] << " ";
        }
        for (int i = 0; i < this->m_tail; i++)
        {
            cout << this->m_queue[i] << " ";
        }
        cout << endl;
    }
}
```



# LinkedList

## LinkedNode

```c++
struct LinkedNode
{
    LinkedNode(int x)
    {
        key = x;
        prev = nullptr;
        next = nullptr;
    }
    int key;
    LinkedNode *prev;
    LinkedNode *next;
};
```



## Class

```c++
class LinkedList
{
private:
    LinkedNode *head;
    LinkedNode *tail;
    int size;

public:
    LinkedList(int x);
    ~LinkedList();

    //print all nodes in linkedlist
    void PRINTER();

    //find the node whose key==k
    LinkedNode *LIST_SEARCH(int k);

    //insert a node as the head
    void PUSHFRONT(int x);

    //insert a node as the tail
    void PUSHBACK(int x);

    //delete the first node
    void POPFRONT();

    //delete the last node
    void POPBACK();

    //delete the node whose key==x
    void LIST_DELETE(int x);

    //insert a node whose key==x after the node whose key==x
    void LIST_INSERT(int x, int key);
};
```



## Initialize and Destroy

```c++
LinkedList::LinkedList(int x)
{
    LinkedNode *temp = new LinkedNode(x);
    this->head = temp;
    this->tail = temp;
    this->size = 1;
}

LinkedList::~LinkedList()
{
    LinkedNode *temp = this->head;
    while (temp)
    {
        this->head = this->head->next;
        delete temp;
        temp = this->head;
    }
    this->head = nullptr;
    this->tail = nullptr;
    this->size = 0;
}
```



## PopFront and PopBack

```c++
void LinkedList::POPFRONT()
{
    if (this->head == nullptr)
    {
        cout << "Empty linkedlist" << endl;
    }

    LinkedNode *temp = this->head;

    //assign class_head to second node in original linked list
    this->head = this->head->next;
    this->head->prev = nullptr;
    delete temp;
    this->size--;
}

void LinkedList::POPBACK()
{
    //similar to POPFRONT()
    if (this->tail == nullptr)
    {
        cout << "Empty linkedlist" << endl;
    }

    LinkedNode *temp = this->tail;

    this->tail = this->tail->prev;
    this->tail->next = nullptr;
    delete temp;
    this->size--;
}
```



## PushFront and PushBack

```c++
void LinkedList::PUSHFRONT(int x)
{
    LinkedNode *temp = new LinkedNode(x);

    //construct new_head-->old_head
    temp->next = this->head;

    if (this->head != nullptr)
    {
        //construct old_head-->new_head
        this->head->prev = temp;
    }
    //assign class_head to new_head
    this->head = temp;

    //if original linked list is empty, we also should assign tail to new head
    if (this->tail == nullptr)
    {
        this->tail = this->head;
    }

    temp->prev = nullptr;
    this->size++;
}

void LinkedList::PUSHBACK(int x)
{
    //similar to PUSHFRONT()
    LinkedNode *temp = new LinkedNode(x);

    temp->prev = this->tail;

    if (this->tail != nullptr)
    {
        this->tail->next = temp;
    }

    this->tail = temp;

    if (this->head == nullptr)
    {
        this->head = this->tail;
    }

    temp->next = nullptr;
    this->size++;
}
```



## Delete

```c++
void LinkedList::LIST_DELETE(int x)
{
    //first locate the node will be deleted
    LinkedNode *target = this->LIST_SEARCH(x);
    if (target == nullptr)
    {
        return;
    }
    else if (target == this->head)
    {
        this->POPFRONT();
    }
    else if (target == this->tail)
    {
        this->POPBACK();
    }
    else
    {
        //node_a --- target_node --- node_b

        //construct node_a --> node_b
        target->next->prev = target->prev;

        //construct node_a <-- node_b
        target->prev->next = target->next;

        //now: node_a --- node_b
        delete target;
        this->size--;
    }
}
```



## Insert

```c++
void LinkedList::LIST_INSERT(int x, int key)
{
    //first locate the node will be inserted
    LinkedNode *location = this->LIST_SEARCH(x);
    if (location == this->tail)
    {
        this->PUSHBACK(key);
    }
    else if (location)
    {
        LinkedNode *temp = new LinkedNode(key);
        //node_a(location) --- node_b

        //construct new_node --> node_b
        temp->next = location->next;

        //construct new_node <-- node_b
        location->next->prev = temp;

        //construct node_a --> new_node
        location->next = temp;

        //construct node_a <-- new_node
        temp->prev = location;

        //now: node_a --- new_node --- node_b
        this->size++;
    }
}
```



## Search

```c++
LinkedNode *LinkedList::LIST_SEARCH(int k)
{
    //traverse linked list
    LinkedNode *x = this->head;
    while (x != nullptr && x->key != k)
    {
        x = x->next;
    }

    if (x == nullptr)
    {
        cout << "No such element in this linkedlist" << endl;
    }

    return x;
}
```



## Printer

```c++
void LinkedList::PRINTER()
{
    LinkedNode *temp = this->head;
    if (this->head == nullptr && this->tail == nullptr)
    {
        cout << "Empty linkedlist" << endl;
        return;
    }

    cout << "Node number: " << this->size << endl;
    while (temp)
    {
        cout << temp->key << " --- ";
        temp = temp->next;
    }
    cout << endl;
}
```

