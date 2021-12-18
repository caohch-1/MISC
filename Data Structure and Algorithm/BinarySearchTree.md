---
title: BinarySearchTree
categories:
- 算法
---

***基于《算法导论》中的伪代码 以及部分网络资源***

# BinarySearchNode

```c++
struct BinarySearchNode {
    BinarySearchNode(int x) {
        key = x;
        parent = nullptr;
        left = nullptr;
        right = nullptr;
    }

    int key;
    BinarySearchNode *parent;
    BinarySearchNode *left;
    BinarySearchNode *right;
};
```

# Class

```c++
class BinarySearchTree {
private:
    BinarySearchNode *root;

public:
    //构造和析构
    BinarySearchTree(int data);
    ~BinarySearchTree();
	//获取根节点 获取自己 （用于外部调用）
    BinarySearchNode *GETROOT() { return this->root; };
    BinarySearchTree *GETTREE() { return this; };
	//返回树中最大or最小节点
    BinarySearchNode *TREE_MINIMUM(BinarySearchNode *node);
    BinarySearchNode *TREE_MAXIMUM(BinarySearchNode *node);
    //插入
    void TREE_INSERT(BinarySearchTree *tree, int data);
	//删除
    void TRANSPLANT(BinarySearchTree *tree, BinarySearchNode *u, BinarySearchNode *v);
    void TREE_DELETE(BinarySearchTree *tree, int data);
	//某节点的后继
    BinarySearchNode *TREE_SUCCESSOR(BinarySearchNode *node);
	//中序遍历打印
    void INORDER_TREE_WALK(BinarySearchNode *node);
	//搜索某节点
    BinarySearchNode *TREE_SEARCH(BinarySearchNode *node, int data);
    BinarySearchNode *ITERATIVE_TREE_SEARCH(BinarySearchNode *node, int data);
};
```

# Initialize

```c++
BinarySearchTree::BinarySearchTree(int data) {
    this->root = new BinarySearchNode(data);
}
```

# MinimumNode and MaximumNode

```c++
BinarySearchNode *BinarySearchTree::TREE_MINIMUM(BinarySearchNode *node) {
    while (node->left != nullptr) {
        node = node->left;
    }
    return node;
}

BinarySearchNode *BinarySearchTree::TREE_MAXIMUM(BinarySearchNode *node) {
    while (node->right != nullptr) {
        node = node->right;
    }
    return node;
}
```

# Insert

```c++
void BinarySearchTree::TREE_INSERT(BinarySearchTree *tree, int data) {
    BinarySearchNode *node = new BinarySearchNode(data);
    BinarySearchNode *y = nullptr;
    BinarySearchNode *x = tree->root;
    while (x != nullptr) {
        y = x;
        if (node->key < x->key) {
            x = x->left;
        } else {
            x = x->right;
        }
    }
    node->parent = y;
    if (y == nullptr) {
        tree->root = node;
    } else if (node->key < y->key) {
        y->left = node;
    } else {
        y->right = node;
    }
}
```

# Delete(???)

```c++
//辅助函数 子树替换
void BinarySearchTree::TRANSPLANT(BinarySearchTree *tree, BinarySearchNode *u, BinarySearchNode *v) {
    if (u->parent == nullptr) {
        tree->root = v;
    } else if (u == u->parent->left) {
        u->parent->left = v;
    } else {
        u->parent->right = v;
    }
    if (v != nullptr) {
        v->parent = u->parent;
    }
}

void BinarySearchTree::TREE_DELETE(BinarySearchTree *tree, int data) {
    BinarySearchNode *z = tree->TREE_SEARCH(tree->GETROOT(), data);
    if (z == nullptr) {
        return;
    }

    if (z->left == nullptr) {
        tree->TRANSPLANT(tree, z, z->right);
    } else if (z->right == nullptr) {
        tree->TRANSPLANT(tree, z, z->left);
    } else {
        BinarySearchNode *y = tree->TREE_MINIMUM(z->right);
        if (y->parent != z) {
            tree->TRANSPLANT(tree, y, y->right);
            y->right = z->right;
            y->right->parent = y;
        }
        tree->TRANSPLANT(tree, z, y);
        y->left = z->left;
        y->left = z->left;
    }
}
```

# Successor

```c++
BinarySearchNode *BinarySearchTree::TREE_SUCCESSOR(BinarySearchNode *node) {
    if (node->right != nullptr) {
        return this->TREE_MINIMUM(node->right);
    }
    BinarySearchNode *successor = node->parent;
    while (successor != nullptr && node == successor->right) {
        node = successor;
        successor = successor->parent;
    }
    return successor;
}
```

# Search

```c++
//递归
BinarySearchNode *BinarySearchTree::TREE_SEARCH(BinarySearchNode *node, int data) {
    if (node == nullptr || data == node->key) {
        if (node == nullptr) {
            cout << "No such node in tree" << endl;
        }

        return node;
    }

    if (data < node->key) {
        return this->TREE_SEARCH(node->left, data);
    } else {
        return this->TREE_SEARCH(node->right, data);
    }
}
//一般用这个
BinarySearchNode *BinarySearchTree::ITERATIVE_TREE_SEARCH(BinarySearchNode *node, int data) {
    while (node != nullptr && data != node->key) {
        if (data < node->key) {
            node = node->left;
        } else {
            node = node->right;
        }
    }
    return node;
}
```

# Printer

```c++
void BinarySearchTree::INORDER_TREE_WALK(BinarySearchNode *node) {
    if (node != nullptr) {
        this->INORDER_TREE_WALK(node->left);
        cout << node->key << " <= ";
        this->INORDER_TREE_WALK(node->right);
    }
}
```



