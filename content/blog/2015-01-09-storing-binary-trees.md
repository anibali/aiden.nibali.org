+++
title = "Storing binary trees"
date = "2015-01-09T00:00:00.000Z"
categories = ["misc"]
tags = ["optimisation"]
comments = false
draft = false
showpagemeta = true
slug = ""
+++

### Humble beginnings

Let's say that have a binary tree, such as the following:

![A binary tree](/img/binary_tree.svg)

The traditional text-book way of representing such a structure in code is
something like this:

```d
struct Node {
  Node* leftChild;
  Node* rightChild;
  char value;
  bool isLeaf;
}

Node cNode = { value: 'C', isLeaf: true };
Node cParentNode = { rightChild: &cNode };
Node bNode = { value: 'B', isLeaf: true };
Node bParentNode = { leftChild: &bNode, rightChild: &cParentNode };
Node aNode = { value: 'A', isLeaf: true };
Node rootNode = { leftChild: &aNode, rightChild: &bParentNode };

writeln(rootNode.rightChild.leftChild.value); //=> B
```

The storage space required for a single node on a 64-bit machine is
`8 + 8 + 2 + (6) = 24` bytes. The last 6 bytes is padding automatically added
by the compiler to ensure proper alignment of the struct in arrays.

### Cache locality

We can ensure that nodes are located near each other in memory by allocating
all of the nodes we need in an array ahead of time. This improves the likelihood
of hitting the cache during tree traversal.

```d
Node[6] nodes;

// cNode
nodes[5].value = 'C';
nodes[5].isLeaf = true;
// cParentNode
nodes[4].rightChild = &nodes[5];
// ...
```

This leads to another optimisation - we can ensure that each branching node
has its left child stored immediately after it in memory as we construct the
tree. As a result we only need to explicitly store one pointer to refer to
the right child.

```d
struct Node {
  Node* rightChild;
  char value;
  bool isLeaf;

  // Can calculate location of left child rather than store it.
  @property Node* leftChild() {
    return &this + 1;
  }
}

Node[6] nodes;

// cNode
nodes[5].value = 'C';
nodes[5].isLeaf = true;
// cParentNode
nodes[4].rightChild = &nodes[5];
// bNode
nodes[3].value = 'B';
nodes[3].isLeaf = true;
// bParentNode
// Implicit left child is nodes[3]
nodes[2].rightChild = &nodes[4];
// aNode
nodes[1].value = 'A';
nodes[1].isLeaf = true;
// rootNode
// Implicit left child is nodes[1]
nodes[0].rightChild = &nodes[2];

writeln(nodes[0].rightChild.leftChild.value); //=> B
```

### Unions

A node is only ever a branch or a leaf - these states are mutually exclusive.
This means that we only need ever store a node's value OR child references.
Whilst this is not particularly relevant for `char` values (the space savings
become padding for struct alignment), this technique is useful when the data is
larger, such as a pointer.

```d
struct Node {
  union {
    Node* rightChild;
    char value;
  }
  bool isLeaf;

  @property Node* leftChild() {
    return &this + 1;
  }
}
```

### The ultimate

If you think more carefully about things, the idea of a union can be taken to
the extreme. In fact, we can cut down the size of a node to a SINGLE BYTE!

This is achieved by using the sign bit of the byte as an indication of whether
the node is a leaf or not (negative means a branch node). If the node is a
leaf, the remaining 7 bits represent the value of the node. If the node is
a branch, the remaining 7 bits contain a pointer offset to the right child
of the node.

Disclaimer: this approach does have some limitations to tree size and
halves the resolution of our value type.

```d
struct Node {
  byte contents;

  @property bool isLeaf() {
    return contents >= 0;
  }

  @property Node* leftChild() {
    assert(!isLeaf);
    return &this + 1;
  }

  @property Node* rightChild() {
    assert(!isLeaf);
    return &this - contents;
  }

  @property char value() {
    assert(isLeaf);
    return cast(char)contents;
  }
}

enum InvalidNode = -1;

byte[] treeData = [
  -2, 'A', -2, 'B', InvalidNode, 'C'
];
auto rootNode = cast(Node*)treeData.ptr;

writeln(rootNode.rightChild.leftChild.value); //=> B
```

As it turns out the restrictions of this binary tree representation are
actually not that bad, and the benefits are very good. For example,
it's possible to store a Huffman tree for all ASCII characters within
four cache lines on most CPUs.
