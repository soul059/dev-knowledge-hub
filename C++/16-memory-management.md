# 16. Memory Management

## 📋 Overview
Memory management in C++ is a critical aspect that provides developers with direct control over memory allocation and deallocation. Understanding proper memory management is essential for writing efficient, safe, and robust C++ applications.

## 🎯 Dynamic Memory Allocation

### 1. **new and delete Operators**
```cpp
#include <iostream>
#include <string>
using namespace std;

void demonstrateBasicDynamicAllocation() {
    cout << "=== Basic Dynamic Memory Allocation ===" << endl;
    
    // Allocating single variables
    int* ptr = new int(42);
    cout << "Dynamically allocated integer: " << *ptr << endl;
    
    double* dptr = new double(3.14159);
    cout << "Dynamically allocated double: " << *dptr << endl;
    
    string* sptr = new string("Hello, Dynamic Memory!");
    cout << "Dynamically allocated string: " << *sptr << endl;
    
    // Always delete what you new
    delete ptr;
    delete dptr;
    delete sptr;
    
    // Set pointers to nullptr after deletion (good practice)
    ptr = nullptr;
    dptr = nullptr;
    sptr = nullptr;
    
    cout << "Memory successfully deallocated" << endl;
}

// Demonstrating arrays
void demonstrateDynamicArrays() {
    cout << "\n=== Dynamic Array Allocation ===" << endl;
    
    size_t size;
    cout << "Enter array size: ";
    cin >> size;
    
    // Allocate array
    int* arr = new int[size];
    
    // Initialize array
    for (size_t i = 0; i < size; i++) {
        arr[i] = (i + 1) * 10;
    }
    
    // Display array
    cout << "Dynamic array contents: ";
    for (size_t i = 0; i < size; i++) {
        cout << arr[i] << " ";
    }
    cout << endl;
    
    // Delete array (note the [] syntax)
    delete[] arr;
    arr = nullptr;
    
    cout << "Array memory successfully deallocated" << endl;
}

// Memory allocation with error handling
class SafeMemoryAllocator {
public:
    template<typename T>
    static T* allocate(size_t count = 1) {
        T* ptr = nullptr;
        
        try {
            if (count == 1) {
                ptr = new T();
            } else {
                ptr = new T[count];
            }
            
            cout << "Successfully allocated memory for " << count 
                 << " object(s) of type " << typeid(T).name() << endl;
            
        } catch (const bad_alloc& e) {
            cout << "Memory allocation failed: " << e.what() << endl;
            throw;
        }
        
        return ptr;
    }
    
    template<typename T>
    static void deallocate(T* ptr, bool isArray = false) {
        if (ptr != nullptr) {
            if (isArray) {
                delete[] ptr;
            } else {
                delete ptr;
            }
            
            cout << "Memory successfully deallocated" << endl;
        }
    }
};

void demonstrateSafeAllocation() {
    cout << "\n=== Safe Memory Allocation ===" << endl;
    
    try {
        // Allocate single object
        int* singleInt = SafeMemoryAllocator::allocate<int>();
        *singleInt = 100;
        cout << "Single integer value: " << *singleInt << endl;
        SafeMemoryAllocator::deallocate(singleInt);
        
        // Allocate array
        double* doubleArray = SafeMemoryAllocator::allocate<double>(5);
        for (int i = 0; i < 5; i++) {
            doubleArray[i] = i * 1.5;
        }
        
        cout << "Double array: ";
        for (int i = 0; i < 5; i++) {
            cout << doubleArray[i] << " ";
        }
        cout << endl;
        
        SafeMemoryAllocator::deallocate(doubleArray, true);
        
    } catch (const bad_alloc& e) {
        cout << "Allocation error: " << e.what() << endl;
    }
}

int main() {
    demonstrateBasicDynamicAllocation();
    demonstrateDynamicArrays();
    demonstrateSafeAllocation();
    return 0;
}
```

### 2. **malloc, calloc, realloc, and free (C-style)**
```cpp
#include <iostream>
#include <cstdlib>
#include <cstring>
using namespace std;

void demonstrateCStyleAllocation() {
    cout << "=== C-Style Memory Allocation ===" << endl;
    
    // malloc - allocates uninitialized memory
    size_t size = 10;
    int* ptr1 = static_cast<int*>(malloc(size * sizeof(int)));
    
    if (ptr1 == nullptr) {
        cout << "malloc failed!" << endl;
        return;
    }
    
    // Initialize manually
    for (size_t i = 0; i < size; i++) {
        ptr1[i] = i * i;
    }
    
    cout << "malloc array: ";
    for (size_t i = 0; i < size; i++) {
        cout << ptr1[i] << " ";
    }
    cout << endl;
    
    // calloc - allocates zero-initialized memory
    int* ptr2 = static_cast<int*>(calloc(size, sizeof(int)));
    
    if (ptr2 == nullptr) {
        cout << "calloc failed!" << endl;
        free(ptr1);
        return;
    }
    
    cout << "calloc array (zero-initialized): ";
    for (size_t i = 0; i < size; i++) {
        cout << ptr2[i] << " ";
    }
    cout << endl;
    
    // Set some values
    for (size_t i = 0; i < size; i++) {
        ptr2[i] = (i + 1) * 5;
    }
    
    // realloc - resize memory block
    size_t newSize = 15;
    ptr2 = static_cast<int*>(realloc(ptr2, newSize * sizeof(int)));
    
    if (ptr2 == nullptr) {
        cout << "realloc failed!" << endl;
        free(ptr1);
        return;
    }
    
    // Initialize new elements
    for (size_t i = size; i < newSize; i++) {
        ptr2[i] = i * 10;
    }
    
    cout << "realloc array (resized): ";
    for (size_t i = 0; i < newSize; i++) {
        cout << ptr2[i] << " ";
    }
    cout << endl;
    
    // Free memory
    free(ptr1);
    free(ptr2);
    
    cout << "C-style memory successfully freed" << endl;
}

// Comparison function for memory allocation methods
void compareAllocationMethods() {
    cout << "\n=== Allocation Methods Comparison ===" << endl;
    
    const size_t count = 1000000;  // 1 million integers
    
    // Timing C++ new/delete
    auto start = chrono::high_resolution_clock::now();
    
    int* cppArray = new int[count];
    for (size_t i = 0; i < count; i++) {
        cppArray[i] = i;
    }
    delete[] cppArray;
    
    auto end = chrono::high_resolution_clock::now();
    auto cppDuration = chrono::duration_cast<chrono::microseconds>(end - start);
    
    // Timing C malloc/free
    start = chrono::high_resolution_clock::now();
    
    int* cArray = static_cast<int*>(malloc(count * sizeof(int)));
    for (size_t i = 0; i < count; i++) {
        cArray[i] = i;
    }
    free(cArray);
    
    end = chrono::high_resolution_clock::now();
    auto cDuration = chrono::duration_cast<chrono::microseconds>(end - start);
    
    cout << "C++ new/delete time: " << cppDuration.count() << " microseconds" << endl;
    cout << "C malloc/free time: " << cDuration.count() << " microseconds" << endl;
    
    cout << "\nKey Differences:" << endl;
    cout << "- new/delete: Type-safe, calls constructors/destructors" << endl;
    cout << "- malloc/free: Raw memory, no constructor/destructor calls" << endl;
    cout << "- new/delete: Throws exception on failure" << endl;
    cout << "- malloc/free: Returns nullptr on failure" << endl;
}

int main() {
    demonstrateCStyleAllocation();
    compareAllocationMethods();
    return 0;
}
```

## 🔧 Smart Pointers

### 1. **unique_ptr - Exclusive Ownership**
```cpp
#include <iostream>
#include <memory>
#include <vector>
#include <string>
using namespace std;

class Resource {
private:
    string name;
    int id;

public:
    Resource(const string& n, int i) : name(n), id(i) {
        cout << "Resource '" << name << "' (ID: " << id << ") created" << endl;
    }
    
    ~Resource() {
        cout << "Resource '" << name << "' (ID: " << id << ") destroyed" << endl;
    }
    
    void use() {
        cout << "Using resource '" << name << "' (ID: " << id << ")" << endl;
    }
    
    const string& getName() const { return name; }
    int getId() const { return id; }
};

void demonstrateUniquePtr() {
    cout << "=== unique_ptr Demonstration ===" << endl;
    
    // Creating unique_ptr
    {
        cout << "\n1. Basic unique_ptr usage:" << endl;
        unique_ptr<Resource> ptr1 = make_unique<Resource>("Database", 101);
        ptr1->use();
        
        // Automatic cleanup when ptr1 goes out of scope
    }
    
    // Transfer ownership
    {
        cout << "\n2. Transfer ownership:" << endl;
        unique_ptr<Resource> ptr1 = make_unique<Resource>("FileHandle", 102);
        
        // Transfer ownership using move
        unique_ptr<Resource> ptr2 = move(ptr1);
        
        // ptr1 is now nullptr
        if (ptr1 == nullptr) {
            cout << "ptr1 is now nullptr after move" << endl;
        }
        
        if (ptr2) {
            ptr2->use();
        }
        
        // Only ptr2 will clean up the resource
    }
    
    // Array management
    {
        cout << "\n3. Array management:" << endl;
        unique_ptr<int[]> arrayPtr = make_unique<int[]>(5);
        
        for (int i = 0; i < 5; i++) {
            arrayPtr[i] = (i + 1) * 10;
        }
        
        cout << "Array contents: ";
        for (int i = 0; i < 5; i++) {
            cout << arrayPtr[i] << " ";
        }
        cout << endl;
        
        // Automatic array cleanup
    }
    
    // Custom deleter
    {
        cout << "\n4. Custom deleter:" << endl;
        
        auto customDeleter = [](Resource* ptr) {
            cout << "Custom deleter called for resource: " << ptr->getName() << endl;
            delete ptr;
        };
        
        unique_ptr<Resource, decltype(customDeleter)> ptr(
            new Resource("CustomResource", 103), 
            customDeleter
        );
        
        ptr->use();
    }
}

// Factory function using unique_ptr
class ResourceFactory {
public:
    static unique_ptr<Resource> createResource(const string& type, int id) {
        if (type == "database") {
            return make_unique<Resource>("Database Connection", id);
        } else if (type == "file") {
            return make_unique<Resource>("File Handle", id);
        } else if (type == "network") {
            return make_unique<Resource>("Network Socket", id);
        } else {
            return nullptr;
        }
    }
};

void demonstrateResourceFactory() {
    cout << "\n=== Resource Factory with unique_ptr ===" << endl;
    
    vector<unique_ptr<Resource>> resources;
    
    // Create different types of resources
    resources.push_back(ResourceFactory::createResource("database", 201));
    resources.push_back(ResourceFactory::createResource("file", 202));
    resources.push_back(ResourceFactory::createResource("network", 203));
    
    // Use all resources
    cout << "\nUsing all resources:" << endl;
    for (auto& resource : resources) {
        if (resource) {
            resource->use();
        }
    }
    
    // Resources automatically cleaned up when vector is destroyed
    cout << "\nResources will be automatically cleaned up..." << endl;
}

int main() {
    demonstrateUniquePtr();
    demonstrateResourceFactory();
    return 0;
}
```

### 2. **shared_ptr - Shared Ownership**
```cpp
#include <iostream>
#include <memory>
#include <vector>
#include <string>
using namespace std;

class Document {
private:
    string title;
    string content;
    mutable int accessCount;

public:
    Document(const string& t, const string& c) 
        : title(t), content(c), accessCount(0) {
        cout << "Document '" << title << "' created" << endl;
    }
    
    ~Document() {
        cout << "Document '" << title << "' destroyed (accessed " 
             << accessCount << " times)" << endl;
    }
    
    void read() const {
        accessCount++;
        cout << "Reading document '" << title << "' - Access #" << accessCount << endl;
        cout << "Content: " << content.substr(0, 50) 
             << (content.length() > 50 ? "..." : "") << endl;
    }
    
    const string& getTitle() const { return title; }
    void setContent(const string& newContent) { content = newContent; }
};

class DocumentManager {
private:
    vector<shared_ptr<Document>> documents;

public:
    shared_ptr<Document> createDocument(const string& title, const string& content) {
        auto doc = make_shared<Document>(title, content);
        documents.push_back(doc);
        return doc;
    }
    
    shared_ptr<Document> findDocument(const string& title) {
        for (auto& doc : documents) {
            if (doc->getTitle() == title) {
                return doc;
            }
        }
        return nullptr;
    }
    
    void listDocuments() const {
        cout << "\nDocument Manager Contents:" << endl;
        for (const auto& doc : documents) {
            cout << "- " << doc->getTitle() 
                 << " (ref count: " << doc.use_count() << ")" << endl;
        }
    }
    
    size_t getDocumentCount() const { return documents.size(); }
};

void demonstrateSharedPtr() {
    cout << "=== shared_ptr Demonstration ===" << endl;
    
    DocumentManager manager;
    
    // Create documents
    auto doc1 = manager.createDocument("Report.txt", "This is an important business report with detailed analysis...");
    auto doc2 = manager.createDocument("Manual.txt", "User manual for the software application...");
    
    cout << "\nInitial state:" << endl;
    manager.listDocuments();
    
    // Share documents
    {
        cout << "\n--- Entering inner scope ---" << endl;
        
        // Multiple references to the same document
        auto sharedDoc1 = manager.findDocument("Report.txt");
        auto anotherRef = doc1;  // Another reference
        
        if (sharedDoc1) {
            cout << "Found document, ref count: " << sharedDoc1.use_count() << endl;
            sharedDoc1->read();
        }
        
        if (anotherRef) {
            anotherRef->read();
        }
        
        // Create more shared references
        vector<shared_ptr<Document>> recentDocs;
        recentDocs.push_back(doc1);
        recentDocs.push_back(doc2);
        recentDocs.push_back(sharedDoc1);
        
        cout << "\nAfter creating multiple references:" << endl;
        manager.listDocuments();
        
        cout << "\nUsing documents from vector:" << endl;
        for (auto& doc : recentDocs) {
            if (doc) {
                doc->read();
            }
        }
        
        cout << "--- Exiting inner scope ---" << endl;
        // recentDocs vector and local shared_ptrs destroyed here
    }
    
    cout << "\nAfter inner scope:" << endl;
    manager.listDocuments();
    
    // Documents still exist because manager holds references
    doc1->read();
    doc2->read();
}

// Circular reference problem and solution
class Node {
public:
    int data;
    shared_ptr<Node> next;
    weak_ptr<Node> parent;  // Use weak_ptr to break cycles
    
    Node(int value) : data(value) {
        cout << "Node " << data << " created" << endl;
    }
    
    ~Node() {
        cout << "Node " << data << " destroyed" << endl;
    }
    
    void addChild(shared_ptr<Node> child) {
        next = child;
        child->parent = shared_from_this();
    }
};

// Make Node inherit from enable_shared_from_this
class SafeNode : public enable_shared_from_this<SafeNode> {
public:
    int data;
    shared_ptr<SafeNode> next;
    weak_ptr<SafeNode> parent;
    
    SafeNode(int value) : data(value) {
        cout << "SafeNode " << data << " created" << endl;
    }
    
    ~SafeNode() {
        cout << "SafeNode " << data << " destroyed" << endl;
    }
    
    void addChild(shared_ptr<SafeNode> child) {
        next = child;
        child->parent = shared_from_this();
    }
    
    void printPath() {
        cout << "Path: ";
        auto current = shared_from_this();
        while (current) {
            cout << current->data << " -> ";
            current = current->next;
        }
        cout << "NULL" << endl;
    }
};

void demonstrateCircularReference() {
    cout << "\n=== Circular Reference Handling ===" << endl;
    
    {
        cout << "\nCreating linked structure:" << endl;
        
        auto node1 = make_shared<SafeNode>(1);
        auto node2 = make_shared<SafeNode>(2);
        auto node3 = make_shared<SafeNode>(3);
        
        // Build a chain: 1 -> 2 -> 3
        node1->addChild(node2);
        node2->addChild(node3);
        
        cout << "Reference counts:" << endl;
        cout << "Node 1: " << node1.use_count() << endl;
        cout << "Node 2: " << node2.use_count() << endl;
        cout << "Node 3: " << node3.use_count() << endl;
        
        // Access parent through weak_ptr
        if (auto parent = node2->parent.lock()) {
            cout << "Node 2's parent: " << parent->data << endl;
        }
        
        node1->printPath();
        
        cout << "\nNodes will be properly destroyed..." << endl;
    }
    // All nodes properly destroyed due to weak_ptr breaking cycles
}

int main() {
    demonstrateSharedPtr();
    demonstrateCircularReference();
    return 0;
}
```

### 3. **weak_ptr - Non-owning Observer**
```cpp
#include <iostream>
#include <memory>
#include <vector>
#include <string>
using namespace std;

class Publisher;
class Subscriber;

// Observer pattern implementation using weak_ptr
class Subscriber {
private:
    string name;
    weak_ptr<Publisher> publisher;

public:
    Subscriber(const string& n) : name(n) {
        cout << "Subscriber '" << name << "' created" << endl;
    }
    
    ~Subscriber() {
        cout << "Subscriber '" << name << "' destroyed" << endl;
    }
    
    void subscribe(shared_ptr<Publisher> pub) {
        publisher = pub;
        cout << "'" << name << "' subscribed to publisher" << endl;
    }
    
    void notify(const string& message) {
        cout << "'" << name << "' received: " << message << endl;
    }
    
    void checkPublisher() {
        if (auto pub = publisher.lock()) {
            cout << "'" << name << "' - Publisher is still alive" << endl;
        } else {
            cout << "'" << name << "' - Publisher no longer exists" << endl;
        }
    }
    
    const string& getName() const { return name; }
    weak_ptr<Publisher> getPublisher() const { return publisher; }
};

class Publisher : public enable_shared_from_this<Publisher> {
private:
    string name;
    vector<weak_ptr<Subscriber>> subscribers;

public:
    Publisher(const string& n) : name(n) {
        cout << "Publisher '" << name << "' created" << endl;
    }
    
    ~Publisher() {
        cout << "Publisher '" << name << "' destroyed" << endl;
    }
    
    void addSubscriber(shared_ptr<Subscriber> subscriber) {
        subscriber->subscribe(shared_from_this());
        subscribers.push_back(subscriber);
    }
    
    void publish(const string& message) {
        cout << "\nPublisher '" << name << "' publishing: " << message << endl;
        
        // Clean up expired weak_ptrs and notify active subscribers
        auto it = subscribers.begin();
        while (it != subscribers.end()) {
            if (auto subscriber = it->lock()) {
                subscriber->notify(message);
                ++it;
            } else {
                // Remove expired weak_ptr
                cout << "Removing expired subscriber reference" << endl;
                it = subscribers.erase(it);
            }
        }
    }
    
    void showSubscribers() {
        cout << "\nActive subscribers for '" << name << "':" << endl;
        for (auto& weakSub : subscribers) {
            if (auto subscriber = weakSub.lock()) {
                cout << "- " << subscriber->getName() << endl;
            }
        }
    }
    
    size_t getActiveSubscriberCount() {
        size_t count = 0;
        for (auto& weakSub : subscribers) {
            if (weakSub.lock()) {
                count++;
            }
        }
        return count;
    }
};

void demonstrateWeakPtr() {
    cout << "=== weak_ptr Demonstration ===" << endl;
    
    auto publisher = make_shared<Publisher>("NewsChannel");
    
    // Create subscribers in different scopes
    {
        cout << "\n--- Creating subscribers ---" << endl;
        
        auto sub1 = make_shared<Subscriber>("Alice");
        auto sub2 = make_shared<Subscriber>("Bob");
        auto sub3 = make_shared<Subscriber>("Charlie");
        
        // Subscribe to publisher
        publisher->addSubscriber(sub1);
        publisher->addSubscriber(sub2);
        publisher->addSubscriber(sub3);
        
        publisher->showSubscribers();
        
        // Publish some messages
        publisher->publish("Breaking News: Market Update");
        publisher->publish("Weather Alert: Storm Warning");
        
        cout << "\n--- Checking publisher from subscribers ---" << endl;
        sub1->checkPublisher();
        sub2->checkPublisher();
        
        cout << "Active subscribers: " << publisher->getActiveSubscriberCount() << endl;
        
        // Remove one subscriber explicitly
        sub2.reset();
        cout << "\nAfter removing Bob:" << endl;
        
        publisher->publish("Tech News: New Product Launch");
        
        cout << "--- Exiting subscriber scope ---" << endl;
        // sub1 and sub3 destroyed here
    }
    
    cout << "\n--- After subscribers destroyed ---" << endl;
    publisher->publish("Final Message: End of Day Summary");
    cout << "Active subscribers: " << publisher->getActiveSubscriberCount() << endl;
}

// Cache implementation using weak_ptr
template<typename Key, typename Value>
class WeakCache {
private:
    mutable map<Key, weak_ptr<Value>> cache;

public:
    shared_ptr<Value> get(const Key& key) const {
        auto it = cache.find(key);
        if (it != cache.end()) {
            if (auto value = it->second.lock()) {
                cout << "Cache hit for key: " << key << endl;
                return value;
            } else {
                cout << "Cache expired for key: " << key << endl;
                cache.erase(it);
            }
        }
        
        cout << "Cache miss for key: " << key << endl;
        return nullptr;
    }
    
    void put(const Key& key, shared_ptr<Value> value) {
        cache[key] = value;
        cout << "Cached value for key: " << key << endl;
    }
    
    void cleanup() {
        auto it = cache.begin();
        while (it != cache.end()) {
            if (it->second.expired()) {
                cout << "Removing expired cache entry: " << it->first << endl;
                it = cache.erase(it);
            } else {
                ++it;
            }
        }
    }
    
    size_t size() const {
        return cache.size();
    }
};

void demonstrateWeakCache() {
    cout << "\n=== Weak Cache Demonstration ===" << endl;
    
    WeakCache<string, string> cache;
    
    {
        cout << "\n--- Creating and caching values ---" << endl;
        
        auto value1 = make_shared<string>("Expensive Computation Result 1");
        auto value2 = make_shared<string>("Expensive Computation Result 2");
        
        cache.put("key1", value1);
        cache.put("key2", value2);
        
        cout << "Cache size: " << cache.size() << endl;
        
        // Access cached values
        auto cached1 = cache.get("key1");
        auto cached2 = cache.get("key2");
        auto cached3 = cache.get("key3");  // Miss
        
        if (cached1) cout << "Retrieved: " << *cached1 << endl;
        if (cached2) cout << "Retrieved: " << *cached2 << endl;
        
        // value2 goes out of scope, but value1 is still referenced
        value2.reset();
        
        cout << "\nAfter resetting value2:" << endl;
        cache.get("key1");  // Should hit
        cache.get("key2");  // Should be expired
        
        cout << "--- Exiting scope ---" << endl;
        // value1 goes out of scope here
    }
    
    cout << "\n--- After all values destroyed ---" << endl;
    cache.get("key1");  // Should be expired
    cache.cleanup();
    cout << "Cache size after cleanup: " << cache.size() << endl;
}

int main() {
    demonstrateWeakPtr();
    demonstrateWeakCache();
    return 0;
}
```

## 🏭 Memory Pools and Custom Allocators

### 1. **Memory Pool Implementation**
```cpp
#include <iostream>
#include <vector>
#include <memory>
#include <chrono>
using namespace std;

class MemoryPool {
private:
    struct Block {
        Block* next;
    };
    
    void* memoryPool;
    Block* freeBlocks;
    size_t blockSize;
    size_t blockCount;
    size_t allocatedBlocks;

public:
    MemoryPool(size_t blockSize, size_t blockCount) 
        : blockSize(max(blockSize, sizeof(Block))), blockCount(blockCount), allocatedBlocks(0) {
        
        // Allocate the entire pool
        size_t poolSize = this->blockSize * blockCount;
        memoryPool = malloc(poolSize);
        
        if (!memoryPool) {
            throw bad_alloc();
        }
        
        // Initialize free list
        freeBlocks = static_cast<Block*>(memoryPool);
        Block* current = freeBlocks;
        
        for (size_t i = 0; i < blockCount - 1; i++) {
            current->next = reinterpret_cast<Block*>(
                static_cast<char*>(static_cast<void*>(current)) + this->blockSize
            );
            current = current->next;
        }
        current->next = nullptr;
        
        cout << "Memory pool created: " << blockCount << " blocks of " 
             << this->blockSize << " bytes each" << endl;
    }
    
    ~MemoryPool() {
        free(memoryPool);
        cout << "Memory pool destroyed" << endl;
    }
    
    void* allocate() {
        if (!freeBlocks) {
            cout << "Memory pool exhausted!" << endl;
            return nullptr;
        }
        
        Block* allocated = freeBlocks;
        freeBlocks = freeBlocks->next;
        allocatedBlocks++;
        
        return allocated;
    }
    
    void deallocate(void* ptr) {
        if (!ptr) return;
        
        Block* block = static_cast<Block*>(ptr);
        block->next = freeBlocks;
        freeBlocks = block;
        allocatedBlocks--;
    }
    
    size_t getAllocatedCount() const { return allocatedBlocks; }
    size_t getFreeCount() const { return blockCount - allocatedBlocks; }
    size_t getTotalCount() const { return blockCount; }
    
    void printStatus() const {
        cout << "Pool Status - Allocated: " << allocatedBlocks 
             << ", Free: " << getFreeCount() 
             << ", Total: " << blockCount << endl;
    }
};

// Example class that uses memory pool
class PooledObject {
private:
    static MemoryPool pool;
    int data;

public:
    PooledObject(int value) : data(value) {
        cout << "PooledObject " << data << " constructed" << endl;
    }
    
    ~PooledObject() {
        cout << "PooledObject " << data << " destructed" << endl;
    }
    
    static void* operator new(size_t size) {
        cout << "Using pooled allocation for PooledObject" << endl;
        return pool.allocate();
    }
    
    static void operator delete(void* ptr) {
        cout << "Using pooled deallocation for PooledObject" << endl;
        pool.deallocate(ptr);
    }
    
    int getValue() const { return data; }
    void setValue(int value) { data = value; }
};

// Initialize static memory pool
MemoryPool PooledObject::pool(sizeof(PooledObject), 10);

void demonstrateMemoryPool() {
    cout << "=== Memory Pool Demonstration ===" << endl;
    
    vector<PooledObject*> objects;
    
    // Allocate objects from pool
    cout << "\nAllocating objects from pool:" << endl;
    for (int i = 1; i <= 5; i++) {
        objects.push_back(new PooledObject(i * 10));
        PooledObject::pool.printStatus();
    }
    
    // Use objects
    cout << "\nUsing objects:" << endl;
    for (auto obj : objects) {
        cout << "Object value: " << obj->getValue() << endl;
    }
    
    // Deallocate some objects
    cout << "\nDeallocating first 3 objects:" << endl;
    for (int i = 0; i < 3; i++) {
        delete objects[i];
        objects[i] = nullptr;
        PooledObject::pool.printStatus();
    }
    
    // Allocate more objects (should reuse freed blocks)
    cout << "\nAllocating new objects (reusing freed blocks):" << endl;
    for (int i = 0; i < 3; i++) {
        objects[i] = new PooledObject((i + 6) * 10);
        PooledObject::pool.printStatus();
    }
    
    // Clean up remaining objects
    cout << "\nCleaning up remaining objects:" << endl;
    for (auto obj : objects) {
        if (obj) {
            delete obj;
        }
    }
    
    PooledObject::pool.printStatus();
}

// Performance comparison
void compareAllocationPerformance() {
    cout << "\n=== Allocation Performance Comparison ===" << endl;
    
    const int iterations = 100000;
    
    // Test regular allocation
    auto start = chrono::high_resolution_clock::now();
    
    vector<int*> regularPtrs;
    for (int i = 0; i < iterations; i++) {
        regularPtrs.push_back(new int(i));
    }
    
    for (auto ptr : regularPtrs) {
        delete ptr;
    }
    
    auto end = chrono::high_resolution_clock::now();
    auto regularTime = chrono::duration_cast<chrono::microseconds>(end - start);
    
    // Test pooled allocation
    MemoryPool intPool(sizeof(int), iterations);
    
    start = chrono::high_resolution_clock::now();
    
    vector<int*> pooledPtrs;
    for (int i = 0; i < iterations; i++) {
        int* ptr = static_cast<int*>(intPool.allocate());
        if (ptr) {
            *ptr = i;
            pooledPtrs.push_back(ptr);
        }
    }
    
    for (auto ptr : pooledPtrs) {
        intPool.deallocate(ptr);
    }
    
    end = chrono::high_resolution_clock::now();
    auto pooledTime = chrono::duration_cast<chrono::microseconds>(end - start);
    
    cout << "Regular allocation time: " << regularTime.count() << " μs" << endl;
    cout << "Pooled allocation time: " << pooledTime.count() << " μs" << endl;
    cout << "Speedup: " << static_cast<double>(regularTime.count()) / pooledTime.count() << "x" << endl;
}

int main() {
    demonstrateMemoryPool();
    compareAllocationPerformance();
    return 0;
}
```

### 2. **Custom Allocators for STL Containers**
```cpp
#include <iostream>
#include <vector>
#include <memory>
#include <map>
using namespace std;

// Simple tracking allocator
template<typename T>
class TrackingAllocator {
private:
    static size_t totalAllocations;
    static size_t totalDeallocations;
    static size_t currentAllocated;

public:
    using value_type = T;
    
    TrackingAllocator() = default;
    
    template<typename U>
    TrackingAllocator(const TrackingAllocator<U>&) {}
    
    T* allocate(size_t n) {
        size_t bytes = n * sizeof(T);
        totalAllocations++;
        currentAllocated += bytes;
        
        cout << "Allocated " << bytes << " bytes for " << n 
             << " objects of type " << typeid(T).name() << endl;
        
        return static_cast<T*>(malloc(bytes));
    }
    
    void deallocate(T* ptr, size_t n) {
        size_t bytes = n * sizeof(T);
        totalDeallocations++;
        currentAllocated -= bytes;
        
        cout << "Deallocated " << bytes << " bytes for " << n 
             << " objects of type " << typeid(T).name() << endl;
        
        free(ptr);
    }
    
    static void printStats() {
        cout << "\n=== Allocator Statistics ===" << endl;
        cout << "Total allocations: " << totalAllocations << endl;
        cout << "Total deallocations: " << totalDeallocations << endl;
        cout << "Currently allocated: " << currentAllocated << " bytes" << endl;
        cout << "Outstanding allocations: " << (totalAllocations - totalDeallocations) << endl;
    }
    
    static void resetStats() {
        totalAllocations = 0;
        totalDeallocations = 0;
        currentAllocated = 0;
    }
};

template<typename T>
size_t TrackingAllocator<T>::totalAllocations = 0;

template<typename T>
size_t TrackingAllocator<T>::totalDeallocations = 0;

template<typename T>
size_t TrackingAllocator<T>::currentAllocated = 0;

template<typename T, typename U>
bool operator==(const TrackingAllocator<T>&, const TrackingAllocator<U>&) {
    return true;
}

template<typename T, typename U>
bool operator!=(const TrackingAllocator<T>&, const TrackingAllocator<U>&) {
    return false;
}

// Pool-based allocator
template<typename T>
class PoolAllocator {
private:
    static MemoryPool* pool;
    static const size_t poolSize = 1000;

public:
    using value_type = T;
    
    PoolAllocator() {
        if (!pool) {
            pool = new MemoryPool(sizeof(T), poolSize);
        }
    }
    
    template<typename U>
    PoolAllocator(const PoolAllocator<U>&) {
        if (!pool) {
            pool = new MemoryPool(sizeof(T), poolSize);
        }
    }
    
    T* allocate(size_t n) {
        if (n == 1) {
            return static_cast<T*>(pool->allocate());
        } else {
            // Fall back to regular allocation for multiple objects
            return static_cast<T*>(malloc(n * sizeof(T)));
        }
    }
    
    void deallocate(T* ptr, size_t n) {
        if (n == 1) {
            pool->deallocate(ptr);
        } else {
            free(ptr);
        }
    }
    
    static void cleanup() {
        delete pool;
        pool = nullptr;
    }
};

template<typename T>
MemoryPool* PoolAllocator<T>::pool = nullptr;

void demonstrateCustomAllocators() {
    cout << "=== Custom Allocators Demonstration ===" << endl;
    
    TrackingAllocator<int>::resetStats();
    
    {
        cout << "\n1. Using TrackingAllocator with vector:" << endl;
        vector<int, TrackingAllocator<int>> trackedVector;
        
        for (int i = 0; i < 10; i++) {
            trackedVector.push_back(i * i);
        }
        
        cout << "Vector contents: ";
        for (int val : trackedVector) {
            cout << val << " ";
        }
        cout << endl;
        
        TrackingAllocator<int>::printStats();
    }
    
    cout << "\nAfter vector destruction:" << endl;
    TrackingAllocator<int>::printStats();
    
    {
        cout << "\n2. Using TrackingAllocator with map:" << endl;
        map<string, int, less<string>, TrackingAllocator<pair<const string, int>>> trackedMap;
        
        trackedMap["one"] = 1;
        trackedMap["two"] = 2;
        trackedMap["three"] = 3;
        trackedMap["four"] = 4;
        
        cout << "Map contents:" << endl;
        for (const auto& pair : trackedMap) {
            cout << pair.first << " -> " << pair.second << endl;
        }
        
        TrackingAllocator<pair<const string, int>>::printStats();
    }
    
    cout << "\nFinal statistics:" << endl;
    TrackingAllocator<int>::printStats();
    TrackingAllocator<pair<const string, int>>::printStats();
}

int main() {
    demonstrateCustomAllocators();
    return 0;
}
```

## 🔍 Memory Debugging and Tools

### 1. **Memory Leak Detection**
```cpp
#include <iostream>
#include <memory>
#include <map>
#include <string>
#include <cassert>
using namespace std;

// Simple memory tracker for debugging
class MemoryTracker {
private:
    struct AllocationInfo {
        size_t size;
        string file;
        int line;
        string function;
    };
    
    static map<void*, AllocationInfo> allocations;
    static size_t totalAllocated;
    static size_t peakAllocated;
    static size_t allocationCount;

public:
    static void* allocate(size_t size, const string& file, int line, const string& function) {
        void* ptr = malloc(size);
        if (ptr) {
            allocations[ptr] = {size, file, line, function};
            totalAllocated += size;
            allocationCount++;
            
            if (totalAllocated > peakAllocated) {
                peakAllocated = totalAllocated;
            }
            
            cout << "[ALLOC] " << size << " bytes at " << ptr 
                 << " (" << file << ":" << line << " in " << function << ")" << endl;
        }
        return ptr;
    }
    
    static void deallocate(void* ptr, const string& file, int line, const string& function) {
        if (!ptr) return;
        
        auto it = allocations.find(ptr);
        if (it != allocations.end()) {
            cout << "[FREE] " << it->second.size << " bytes at " << ptr 
                 << " (" << file << ":" << line << " in " << function << ")" << endl;
            
            totalAllocated -= it->second.size;
            allocations.erase(it);
        } else {
            cout << "[ERROR] Attempting to free untracked pointer " << ptr 
                 << " (" << file << ":" << line << " in " << function << ")" << endl;
        }
        
        free(ptr);
    }
    
    static void printReport() {
        cout << "\n=== Memory Tracker Report ===" << endl;
        cout << "Total allocations made: " << allocationCount << endl;
        cout << "Peak memory usage: " << peakAllocated << " bytes" << endl;
        cout << "Current memory usage: " << totalAllocated << " bytes" << endl;
        
        if (!allocations.empty()) {
            cout << "\nLEAKED MEMORY:" << endl;
            for (const auto& pair : allocations) {
                const auto& info = pair.second;
                cout << "- " << info.size << " bytes at " << pair.first 
                     << " allocated in " << info.function 
                     << " (" << info.file << ":" << info.line << ")" << endl;
            }
        } else {
            cout << "\nNo memory leaks detected!" << endl;
        }
    }
    
    static void reset() {
        allocations.clear();
        totalAllocated = 0;
        peakAllocated = 0;
        allocationCount = 0;
    }
};

// Static member definitions
map<void*, MemoryTracker::AllocationInfo> MemoryTracker::allocations;
size_t MemoryTracker::totalAllocated = 0;
size_t MemoryTracker::peakAllocated = 0;
size_t MemoryTracker::allocationCount = 0;

// Macros for convenient usage
#define TRACKED_MALLOC(size) MemoryTracker::allocate(size, __FILE__, __LINE__, __FUNCTION__)
#define TRACKED_FREE(ptr) MemoryTracker::deallocate(ptr, __FILE__, __LINE__, __FUNCTION__)

// Example functions with and without memory leaks
void functionWithLeak() {
    cout << "\n--- Function with memory leak ---" << endl;
    
    int* ptr1 = static_cast<int*>(TRACKED_MALLOC(sizeof(int) * 10));
    int* ptr2 = static_cast<int*>(TRACKED_MALLOC(sizeof(int) * 5));
    
    // Use the memory
    for (int i = 0; i < 10; i++) {
        ptr1[i] = i * i;
    }
    
    for (int i = 0; i < 5; i++) {
        ptr2[i] = i + 100;
    }
    
    // Only free ptr1, ptr2 is leaked!
    TRACKED_FREE(ptr1);
    
    cout << "Function completed (with leak)" << endl;
}

void functionWithoutLeak() {
    cout << "\n--- Function without memory leak ---" << endl;
    
    int* ptr1 = static_cast<int*>(TRACKED_MALLOC(sizeof(int) * 8));
    char* ptr2 = static_cast<char*>(TRACKED_MALLOC(100));
    
    // Use the memory
    for (int i = 0; i < 8; i++) {
        ptr1[i] = i * 2;
    }
    
    strcpy(ptr2, "Hello, tracked memory!");
    cout << "String in tracked memory: " << ptr2 << endl;
    
    // Properly free both allocations
    TRACKED_FREE(ptr1);
    TRACKED_FREE(ptr2);
    
    cout << "Function completed (no leak)" << endl;
}

void demonstrateMemoryTracking() {
    cout << "=== Memory Tracking Demonstration ===" << endl;
    
    MemoryTracker::reset();
    
    functionWithoutLeak();
    MemoryTracker::printReport();
    
    functionWithLeak();
    MemoryTracker::printReport();
    
    // Demonstrate double-free detection
    cout << "\n--- Testing double-free detection ---" << endl;
    int* ptr = static_cast<int*>(TRACKED_MALLOC(sizeof(int)));
    TRACKED_FREE(ptr);
    TRACKED_FREE(ptr);  // This should be detected as an error
}

int main() {
    demonstrateMemoryTracking();
    return 0;
}
```

## 💡 Best Practices and Guidelines

### Memory Management Best Practices
```cpp
#include <iostream>
#include <memory>
#include <vector>
#include <string>
using namespace std;

// 1. RAII (Resource Acquisition Is Initialization)
class RAIIExample {
private:
    string* data;
    size_t size;

public:
    // Constructor acquires resources
    RAIIExample(size_t n) : size(n) {
        data = new string[size];
        cout << "RAII: Acquired resources for " << size << " strings" << endl;
    }
    
    // Destructor releases resources
    ~RAIIExample() {
        delete[] data;
        cout << "RAII: Released resources" << endl;
    }
    
    // Delete copy operations to prevent double-delete
    RAIIExample(const RAIIExample&) = delete;
    RAIIExample& operator=(const RAIIExample&) = delete;
    
    // Move constructor
    RAIIExample(RAIIExample&& other) noexcept : data(other.data), size(other.size) {
        other.data = nullptr;
        other.size = 0;
        cout << "RAII: Moved resources" << endl;
    }
    
    // Move assignment
    RAIIExample& operator=(RAIIExample&& other) noexcept {
        if (this != &other) {
            delete[] data;  // Clean up existing resources
            
            data = other.data;
            size = other.size;
            
            other.data = nullptr;
            other.size = 0;
            
            cout << "RAII: Move assigned resources" << endl;
        }
        return *this;
    }
    
    void setString(size_t index, const string& value) {
        if (index < size) {
            data[index] = value;
        }
    }
    
    const string& getString(size_t index) const {
        if (index < size) {
            return data[index];
        }
        static const string empty;
        return empty;
    }
    
    size_t getSize() const { return size; }
};

// 2. Modern C++ memory management patterns
class ModernMemoryExample {
public:
    // Prefer stack allocation when possible
    static void demonstrateStackVsHeap() {
        cout << "\n=== Stack vs Heap Allocation ===" << endl;
        
        // Stack allocation - automatic cleanup, fast
        {
            string stackString = "I'm on the stack!";
            vector<int> stackVector = {1, 2, 3, 4, 5};
            cout << "Stack string: " << stackString << endl;
            cout << "Stack vector size: " << stackVector.size() << endl;
            // Automatic cleanup when scope ends
        }
        
        // Heap allocation - only when necessary
        {
            auto heapString = make_unique<string>("I'm on the heap!");
            auto heapVector = make_unique<vector<int>>(initializer_list<int>{6, 7, 8, 9, 10});
            cout << "Heap string: " << *heapString << endl;
            cout << "Heap vector size: " << heapVector->size() << endl;
            // Smart pointers handle cleanup automatically
        }
    }
    
    // Use containers instead of raw arrays
    static void demonstrateContainerUsage() {
        cout << "\n=== Container vs Raw Array ===" << endl;
        
        // Avoid this (raw array)
        // int* rawArray = new int[10];
        // ... use array ...
        // delete[] rawArray;  // Easy to forget!
        
        // Prefer this (container)
        vector<int> container(10);
        for (size_t i = 0; i < container.size(); i++) {
            container[i] = i * i;
        }
        
        cout << "Container contents: ";
        for (int value : container) {
            cout << value << " ";
        }
        cout << endl;
        // Automatic cleanup, bounds checking available, rich interface
    }
    
    // Factory functions with smart pointers
    template<typename T, typename... Args>
    static unique_ptr<T> createUnique(Args&&... args) {
        return make_unique<T>(forward<Args>(args)...);
    }
    
    template<typename T, typename... Args>
    static shared_ptr<T> createShared(Args&&... args) {
        return make_shared<T>(forward<Args>(args)...);
    }
};

// 3. Exception safety levels
class ExceptionSafeClass {
private:
    vector<string> data;
    string* backup;

public:
    ExceptionSafeClass() : backup(nullptr) {}
    
    ~ExceptionSafeClass() {
        delete backup;
    }
    
    // Basic exception safety - no resource leaks
    void basicSafety(const string& value) {
        try {
            data.push_back(value);  // May throw
        } catch (...) {
            // Clean up any partial state if needed
            throw;  // Re-throw
        }
    }
    
    // Strong exception safety - commit or rollback
    void strongSafety(const vector<string>& newData) {
        vector<string> temp = data;  // Copy current state
        
        try {
            data = newData;  // May throw
        } catch (...) {
            data = temp;  // Rollback to previous state
            throw;
        }
    }
    
    // No-throw guarantee
    void noThrowOperation() noexcept {
        // Only operations that are guaranteed not to throw
        if (!data.empty()) {
            data.pop_back();
        }
    }
    
    size_t size() const noexcept { return data.size(); }
    bool empty() const noexcept { return data.empty(); }
};

// 4. Memory optimization techniques
class MemoryOptimized {
private:
    // Use smallest appropriate data types
    uint8_t flags;        // Instead of bool[8] or int
    uint16_t count;       // Instead of int if range is small
    
    // Pack related data together
    struct PackedData {
        uint32_t id;
        float value;
        uint16_t category;
        uint8_t status;
        // Compiler may add padding here
    } __attribute__((packed));  // Force no padding (compiler-specific)

public:
    // Object pooling for frequently created/destroyed objects
    static vector<unique_ptr<MemoryOptimized>> pool;
    
    static unique_ptr<MemoryOptimized> acquire() {
        if (!pool.empty()) {
            auto obj = move(pool.back());
            pool.pop_back();
            obj->reset();
            cout << "Reused object from pool" << endl;
            return obj;
        } else {
            cout << "Created new object" << endl;
            return make_unique<MemoryOptimized>();
        }
    }
    
    static void release(unique_ptr<MemoryOptimized> obj) {
        if (pool.size() < 10) {  // Limit pool size
            pool.push_back(move(obj));
            cout << "Returned object to pool" << endl;
        } else {
            cout << "Pool full, object destroyed" << endl;
        }
    }
    
private:
    void reset() {
        flags = 0;
        count = 0;
        // Reset other members
    }
};

vector<unique_ptr<MemoryOptimized>> MemoryOptimized::pool;

void demonstrateBestPractices() {
    cout << "=== Memory Management Best Practices ===" << endl;
    
    // 1. RAII demonstration
    {
        cout << "\n1. RAII Example:" << endl;
        RAIIExample raii(5);
        raii.setString(0, "First");
        raii.setString(1, "Second");
        
        cout << "String 0: " << raii.getString(0) << endl;
        cout << "String 1: " << raii.getString(1) << endl;
        
        // Move semantics
        RAIIExample moved = move(raii);
        cout << "After move - original size: " << raii.getSize() << endl;
        cout << "After move - moved size: " << moved.getSize() << endl;
        
        // Automatic cleanup when scope ends
    }
    
    // 2. Modern memory management
    ModernMemoryExample::demonstrateStackVsHeap();
    ModernMemoryExample::demonstrateContainerUsage();
    
    // 3. Smart pointer factories
    {
        cout << "\n3. Smart Pointer Factories:" << endl;
        auto uniqueStr = ModernMemoryExample::createUnique<string>("Unique String");
        auto sharedStr = ModernMemoryExample::createShared<string>("Shared String");
        
        cout << "Unique: " << *uniqueStr << endl;
        cout << "Shared: " << *sharedStr << " (ref count: " << sharedStr.use_count() << ")" << endl;
    }
    
    // 4. Exception safety
    {
        cout << "\n4. Exception Safety:" << endl;
        ExceptionSafeClass safeClass;
        
        try {
            safeClass.basicSafety("Safe string");
            safeClass.strongSafety({"One", "Two", "Three"});
            safeClass.noThrowOperation();
            
            cout << "Safe class size: " << safeClass.size() << endl;
        } catch (const exception& e) {
            cout << "Exception caught: " << e.what() << endl;
        }
    }
    
    // 5. Object pooling
    {
        cout << "\n5. Object Pooling:" << endl;
        
        // Acquire and release objects
        auto obj1 = MemoryOptimized::acquire();
        auto obj2 = MemoryOptimized::acquire();
        
        MemoryOptimized::release(move(obj1));
        MemoryOptimized::release(move(obj2));
        
        // Reuse pooled objects
        auto obj3 = MemoryOptimized::acquire();
        auto obj4 = MemoryOptimized::acquire();
    }
}

// Common memory management mistakes to avoid
void memoryMistakesToAvoid() {
    cout << "\n=== Common Memory Mistakes to Avoid ===" << endl;
    
    cout << "\n❌ DON'T DO THESE:" << endl;
    cout << "1. Raw pointers for ownership: int* ptr = new int(42);" << endl;
    cout << "2. Manual memory management: delete ptr;" << endl;
    cout << "3. Mixing new/delete with malloc/free" << endl;
    cout << "4. Double deletion" << endl;
    cout << "5. Memory leaks" << endl;
    cout << "6. Dangling pointers" << endl;
    cout << "7. Buffer overruns" << endl;
    cout << "8. Returning pointers to local variables" << endl;
    
    cout << "\n✅ DO THESE INSTEAD:" << endl;
    cout << "1. Use smart pointers: auto ptr = make_unique<int>(42);" << endl;
    cout << "2. Prefer stack allocation when possible" << endl;
    cout << "3. Use containers instead of raw arrays" << endl;
    cout << "4. Follow RAII principles" << endl;
    cout << "5. Use move semantics for expensive copies" << endl;
    cout << "6. Implement proper copy/move constructors" << endl;
    cout << "7. Use const correctness" << endl;
    cout << "8. Return by value or smart pointer" << endl;
}

int main() {
    demonstrateBestPractices();
    memoryMistakesToAvoid();
    return 0;
}
```

## 🔗 Related Topics
- [Pointers & References](./07-pointers-references.md)
- [Object-Oriented Programming](./08-oop.md)
- [Exception Handling](./13-exception-handling.md)
- [STL](./15-stl.md)
- [Advanced Topics](./17-advanced-topics.md)

---
*Previous: [STL](./15-stl.md) | Next: [Advanced Topics](./17-advanced-topics.md)*
