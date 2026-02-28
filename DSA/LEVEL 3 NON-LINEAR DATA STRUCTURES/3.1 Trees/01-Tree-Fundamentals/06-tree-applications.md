# 1.6 Tree Applications

[← Previous: Types of Trees](05-types-of-trees.md) | [Back to TOC](../README.md) | [Next Unit: Binary Tree Basics →](../02-Binary-Tree-Basics/01-binary-tree-definition.md)

---

## 📖 Real-World Uses

| Domain | Application | Tree Type |
|--------|------------|-----------|
| **Operating Systems** | File system hierarchy | General / N-ary tree |
| **Databases** | Indexing (B+ Trees) | B-Tree, B+ Tree |
| **Compilers** | Abstract Syntax Trees (AST) | Binary / N-ary tree |
| **Networking** | Routing tables | Trie |
| **AI / ML** | Decision trees | Binary tree |
| **Compression** | Huffman coding | Binary tree |
| **UI Frameworks** | DOM tree (HTML/XML) | N-ary tree |
| **Gaming** | Game state trees (minimax) | N-ary tree |
| **Search Engines** | Autocomplete, spell check | Trie |
| **Programming Languages** | Expression evaluation | Expression tree |
| **Memory Management** | Buddy system allocation | Binary tree |

---

## Example: File System as a Tree

```
    /  (root)
    ├── home/
    │   ├── user1/
    │   │   ├── documents/
    │   │   └── downloads/
    │   └── user2/
    │       └── pictures/
    ├── etc/
    │   └── config/
    └── var/
        └── log/
```

---

## Example: HTML DOM Tree

```
    <html>
    ├── <head>
    │   ├── <title>
    │   └── <meta>
    └── <body>
        ├── <div>
        │   ├── <h1>
        │   └── <p>
        └── <footer>
```

---

## Why Trees Matter

```
    Problem: Search in sorted data
    
    Array (linear search):  O(n)     ████████████████████
    Array (binary search):  O(log n) ████
    BST (balanced):         O(log n) ████
    BST (unbalanced):       O(n)     ████████████████████
    Hash table:             O(1)     ██
    
    Trees offer the best balance of:
    ✓ Fast search
    ✓ Fast insertion
    ✓ Fast deletion
    ✓ Ordered data maintained
    ✓ Flexible structure
```

---

[← Previous: Types of Trees](05-types-of-trees.md) | [Back to TOC](../README.md) | [Next Unit: Binary Tree Basics →](../02-Binary-Tree-Basics/01-binary-tree-definition.md)
