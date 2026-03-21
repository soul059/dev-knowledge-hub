# 17. Advanced Topics

## 📋 Overview
Advanced C++ topics cover modern language features, concurrency, metaprogramming, and sophisticated programming techniques. These concepts are essential for writing high-performance, maintainable, and future-proof C++ code.

## 🚀 Modern C++ Features (C++11 and Beyond)

### 1. **Auto and Type Deduction**
```cpp
#include <iostream>
#include <vector>
#include <map>
#include <memory>
#include <functional>
using namespace std;

void demonstrateAutoKeyword() {
    cout << "=== Auto Keyword and Type Deduction ===" << endl;
    
    // Basic auto usage
    auto integer = 42;                    // int
    auto floating = 3.14;                 // double
    auto character = 'A';                 // char
    auto text = "Hello World";            // const char*
    auto string_obj = string("C++");      // string
    
    cout << "Integer: " << integer << " (type: int)" << endl;
    cout << "Floating: " << floating << " (type: double)" << endl;
    cout << "Character: " << character << " (type: char)" << endl;
    cout << "Text: " << text << " (type: const char*)" << endl;
    cout << "String: " << string_obj << " (type: string)" << endl;
    
    // Auto with complex types
    vector<pair<string, int>> complexVector = {
        {"Alice", 25}, {"Bob", 30}, {"Charlie", 35}
    };
    
    // Without auto (verbose)
    for (vector<pair<string, int>>::const_iterator it = complexVector.begin(); 
         it != complexVector.end(); ++it) {
        cout << "Name: " << it->first << ", Age: " << it->second << endl;
    }
    
    // With auto (clean)
    cout << "\nUsing auto:" << endl;
    for (const auto& person : complexVector) {
        cout << "Name: " << person.first << ", Age: " << person.second << endl;
    }
    
    // Auto with pointers and references
    int value = 100;
    auto ptr = &value;           // int*
    auto& ref = value;           // int&
    const auto& const_ref = value; // const int&
    
    cout << "\nPointer value: " << *ptr << endl;
    cout << "Reference value: " << ref << endl;
    cout << "Const reference value: " << const_ref << endl;
    
    // Auto with functions
    auto lambda = [](int x, int y) { return x + y; };
    auto result = lambda(10, 20);
    cout << "Lambda result: " << result << endl;
}

// decltype for more precise type deduction
template<typename T, typename U>
auto add(T t, U u) -> decltype(t + u) {
    return t + u;
}

// C++14 simplified return type deduction
template<typename T, typename U>
auto multiply(T t, U u) {
    return t * u;
}

void demonstrateDecltype() {
    cout << "\n=== decltype and Return Type Deduction ===" << endl;
    
    int a = 5;
    double b = 3.14;
    
    auto sum = add(a, b);
    auto product = multiply(a, b);
    
    cout << "Sum: " << sum << " (type deduced from int + double)" << endl;
    cout << "Product: " << product << " (type deduced automatically)" << endl;
    
    // decltype with expressions
    decltype(a + b) result = a + b;
    cout << "Decltype result: " << result << endl;
    
    // decltype(auto) for perfect forwarding
    auto getValue = []() -> int& {
        static int value = 42;
        return value;
    };
    
    decltype(auto) perfect_forward = getValue(); // int&, not int
    perfect_forward = 100;
    cout << "Perfect forwarded value: " << perfect_forward << endl;
}

int main() {
    demonstrateAutoKeyword();
    demonstrateDecltype();
    return 0;
}
```

### 2. **Range-based For Loops and Initializer Lists**
```cpp
#include <iostream>
#include <vector>
#include <map>
#include <set>
#include <algorithm>
using namespace std;

class CustomContainer {
private:
    vector<int> data;

public:
    CustomContainer(initializer_list<int> init) : data(init) {}
    
    void add(int value) { data.push_back(value); }
    
    // Iterator support for range-based for loops
    auto begin() { return data.begin(); }
    auto end() { return data.end(); }
    auto begin() const { return data.begin(); }
    auto end() const { return data.end(); }
    
    size_t size() const { return data.size(); }
};

void demonstrateRangeBasedLoops() {
    cout << "=== Range-based For Loops ===" << endl;
    
    // With containers
    vector<string> fruits = {"apple", "banana", "orange", "grape"};
    
    cout << "Fruits (read-only):" << endl;
    for (const auto& fruit : fruits) {
        cout << "- " << fruit << endl;
    }
    
    cout << "\nModifying fruits:" << endl;
    for (auto& fruit : fruits) {
        fruit = "fresh " + fruit;
        cout << "- " << fruit << endl;
    }
    
    // With maps
    map<string, int> ages = {
        {"Alice", 25},
        {"Bob", 30},
        {"Charlie", 35}
    };
    
    cout << "\nAges:" << endl;
    for (const auto& [name, age] : ages) {  // C++17 structured binding
        cout << name << " is " << age << " years old" << endl;
    }
    
    // With sets
    set<int> numbers = {1, 2, 3, 4, 5};
    
    cout << "\nNumbers:" << endl;
    for (int num : numbers) {
        cout << num << " ";
    }
    cout << endl;
    
    // With custom container
    CustomContainer custom = {10, 20, 30, 40, 50};
    
    cout << "\nCustom container:" << endl;
    for (const auto& value : custom) {
        cout << value << " ";
    }
    cout << endl;
    
    // With C-style arrays
    int arr[] = {1, 4, 9, 16, 25};
    
    cout << "\nC-style array:" << endl;
    for (int value : arr) {
        cout << value << " ";
    }
    cout << endl;
}

void demonstrateInitializerLists() {
    cout << "\n=== Initializer Lists ===" << endl;
    
    // Container initialization
    vector<int> vec = {1, 2, 3, 4, 5};
    map<string, double> prices = {
        {"Coffee", 4.50},
        {"Tea", 3.25},
        {"Sandwich", 8.75}
    };
    
    cout << "Vector: ";
    for (int v : vec) cout << v << " ";
    cout << endl;
    
    cout << "Prices:" << endl;
    for (const auto& [item, price] : prices) {
        cout << "- " << item << ": $" << price << endl;
    }
    
    // Function calls with initializer lists
    auto findMax = [](initializer_list<int> values) {
        return *max_element(values.begin(), values.end());
    };
    
    int maxValue = findMax({10, 5, 8, 15, 3});
    cout << "Max value: " << maxValue << endl;
    
    // Custom class with initializer list
    CustomContainer container = {100, 200, 300};
    cout << "Custom container size: " << container.size() << endl;
}

int main() {
    demonstrateRangeBasedLoops();
    demonstrateInitializerLists();
    return 0;
}
```

### 3. **Lambda Expressions and Functional Programming**
```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <functional>
#include <numeric>
using namespace std;

void demonstrateBasicLambdas() {
    cout << "=== Basic Lambda Expressions ===" << endl;
    
    // Simple lambda
    auto greet = []() {
        cout << "Hello from lambda!" << endl;
    };
    greet();
    
    // Lambda with parameters
    auto add = [](int a, int b) {
        return a + b;
    };
    cout << "5 + 3 = " << add(5, 3) << endl;
    
    // Lambda with explicit return type
    auto divide = [](double a, double b) -> double {
        if (b != 0) return a / b;
        return 0.0;
    };
    cout << "10.5 / 3.0 = " << divide(10.5, 3.0) << endl;
    
    // Lambda with capture
    int multiplier = 10;
    auto multiply = [multiplier](int value) {
        return value * multiplier;
    };
    cout << "7 * 10 = " << multiply(7) << endl;
    
    // Lambda with mutable capture
    int counter = 0;
    auto increment = [counter]() mutable {
        return ++counter;
    };
    cout << "Counter: " << increment() << ", " << increment() << ", " << increment() << endl;
    cout << "Original counter: " << counter << endl; // Still 0
    
    // Lambda with reference capture
    auto incrementRef = [&counter]() {
        return ++counter;
    };
    cout << "Reference counter: " << incrementRef() << ", " << incrementRef() << endl;
    cout << "Original counter after ref: " << counter << endl; // Modified
}

void demonstrateAdvancedLambdas() {
    cout << "\n=== Advanced Lambda Usage ===" << endl;
    
    vector<int> numbers = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    
    // Lambda with STL algorithms
    cout << "Original numbers: ";
    for (int n : numbers) cout << n << " ";
    cout << endl;
    
    // Filter even numbers
    vector<int> evens;
    copy_if(numbers.begin(), numbers.end(), back_inserter(evens),
            [](int n) { return n % 2 == 0; });
    
    cout << "Even numbers: ";
    for (int n : evens) cout << n << " ";
    cout << endl;
    
    // Transform with lambda
    vector<int> squares;
    transform(numbers.begin(), numbers.end(), back_inserter(squares),
              [](int n) { return n * n; });
    
    cout << "Squares: ";
    for (int n : squares) cout << n << " ";
    cout << endl;
    
    // Reduce with lambda
    int sum = accumulate(numbers.begin(), numbers.end(), 0,
                        [](int acc, int n) { return acc + n; });
    cout << "Sum: " << sum << endl;
    
    // Complex lambda with multiple captures
    string prefix = "Number: ";
    int base = 100;
    
    auto formatter = [prefix, base](int value) {
        return prefix + to_string(base + value);
    };
    
    cout << "Formatted: " << formatter(42) << endl;
    
    // Generic lambda (C++14)
    auto genericPrint = [](const auto& item) {
        cout << "Item: " << item << endl;
    };
    
    genericPrint(42);
    genericPrint(3.14);
    genericPrint("Hello");
    
    // Lambda returning lambda
    auto createMultiplier = [](int factor) {
        return [factor](int value) {
            return value * factor;
        };
    };
    
    auto times3 = createMultiplier(3);
    auto times5 = createMultiplier(5);
    
    cout << "3 * 7 = " << times3(7) << endl;
    cout << "5 * 7 = " << times5(7) << endl;
}

// Functional programming patterns
class FunctionalUtils {
public:
    // Map function
    template<typename Container, typename Function>
    static auto map(const Container& container, Function func) {
        using ReturnType = decltype(func(*container.begin()));
        vector<ReturnType> result;
        
        for (const auto& item : container) {
            result.push_back(func(item));
        }
        
        return result;
    }
    
    // Filter function
    template<typename Container, typename Predicate>
    static auto filter(const Container& container, Predicate pred) {
        Container result;
        
        for (const auto& item : container) {
            if (pred(item)) {
                result.push_back(item);
            }
        }
        
        return result;
    }
    
    // Reduce function
    template<typename Container, typename Function, typename T>
    static T reduce(const Container& container, Function func, T initial) {
        T result = initial;
        
        for (const auto& item : container) {
            result = func(result, item);
        }
        
        return result;
    }
};

void demonstrateFunctionalProgramming() {
    cout << "\n=== Functional Programming Patterns ===" << endl;
    
    vector<int> data = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    
    // Map: square all numbers
    auto squares = FunctionalUtils::map(data, [](int x) { return x * x; });
    cout << "Squares: ";
    for (int s : squares) cout << s << " ";
    cout << endl;
    
    // Filter: only even numbers
    auto evens = FunctionalUtils::filter(data, [](int x) { return x % 2 == 0; });
    cout << "Evens: ";
    for (int e : evens) cout << e << " ";
    cout << endl;
    
    // Reduce: sum all numbers
    int sum = FunctionalUtils::reduce(data, [](int acc, int x) { return acc + x; }, 0);
    cout << "Sum: " << sum << endl;
    
    // Chain operations
    auto result = FunctionalUtils::reduce(
        FunctionalUtils::map(
            FunctionalUtils::filter(data, [](int x) { return x % 2 == 0; }),
            [](int x) { return x * x; }
        ),
        [](int acc, int x) { return acc + x; },
        0
    );
    
    cout << "Sum of squares of even numbers: " << result << endl;
}

int main() {
    demonstrateBasicLambdas();
    demonstrateAdvancedLambdas();
    demonstrateFunctionalProgramming();
    return 0;
}
```

## 🧵 Concurrency and Multithreading

### 1. **std::thread and Basic Threading**
```cpp
#include <iostream>
#include <thread>
#include <vector>
#include <chrono>
#include <mutex>
#include <atomic>
using namespace std;

// Global variables for demonstration
mutex printMutex;
atomic<int> globalCounter{0};

void simpleWorkerFunction(int id, int workAmount) {
    for (int i = 0; i < workAmount; i++) {
        // Simulate work
        this_thread::sleep_for(chrono::milliseconds(10));
        
        // Thread-safe printing
        {
            lock_guard<mutex> lock(printMutex);
            cout << "Thread " << id << " completed work item " << (i + 1) << endl;
        }
        
        // Atomic increment
        globalCounter++;
    }
    
    {
        lock_guard<mutex> lock(printMutex);
        cout << "Thread " << id << " finished all work" << endl;
    }
}

void demonstrateBasicThreading() {
    cout << "=== Basic Threading with std::thread ===" << endl;
    
    const int numThreads = 3;
    const int workPerThread = 5;
    vector<thread> threads;
    
    auto startTime = chrono::high_resolution_clock::now();
    
    // Create and start threads
    for (int i = 0; i < numThreads; i++) {
        threads.emplace_back(simpleWorkerFunction, i + 1, workPerThread);
    }
    
    // Wait for all threads to complete
    for (auto& t : threads) {
        t.join();
    }
    
    auto endTime = chrono::high_resolution_clock::now();
    auto duration = chrono::duration_cast<chrono::milliseconds>(endTime - startTime);
    
    cout << "All threads completed in " << duration.count() << " ms" << endl;
    cout << "Global counter value: " << globalCounter.load() << endl;
}

// Thread-safe class example
class ThreadSafeCounter {
private:
    mutable mutex mtx;
    int count;

public:
    ThreadSafeCounter() : count(0) {}
    
    void increment() {
        lock_guard<mutex> lock(mtx);
        count++;
    }
    
    void decrement() {
        lock_guard<mutex> lock(mtx);
        count--;
    }
    
    int get() const {
        lock_guard<mutex> lock(mtx);
        return count;
    }
    
    void add(int value) {
        lock_guard<mutex> lock(mtx);
        count += value;
    }
};

void counterWorker(ThreadSafeCounter& counter, int operations, int threadId) {
    for (int i = 0; i < operations; i++) {
        if (i % 2 == 0) {
            counter.increment();
        } else {
            counter.add(threadId);
        }
        
        // Simulate some work
        this_thread::sleep_for(chrono::microseconds(100));
    }
}

void demonstrateThreadSafety() {
    cout << "\n=== Thread Safety Demonstration ===" << endl;
    
    ThreadSafeCounter counter;
    const int numThreads = 4;
    const int operationsPerThread = 1000;
    
    vector<thread> threads;
    
    // Start threads
    for (int i = 0; i < numThreads; i++) {
        threads.emplace_back(counterWorker, ref(counter), operationsPerThread, i + 1);
    }
    
    // Monitor progress
    for (int i = 0; i < 10; i++) {
        this_thread::sleep_for(chrono::milliseconds(100));
        cout << "Current counter value: " << counter.get() << endl;
    }
    
    // Wait for completion
    for (auto& t : threads) {
        t.join();
    }
    
    cout << "Final counter value: " << counter.get() << endl;
}

int main() {
    demonstrateBasicThreading();
    demonstrateThreadSafety();
    return 0;
}
```

### 2. **Synchronization Primitives**
```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>
#include <queue>
#include <vector>
#include <chrono>
#include <random>
using namespace std;

// Producer-Consumer pattern with condition variables
template<typename T>
class ThreadSafeQueue {
private:
    mutable mutex mtx;
    queue<T> dataQueue;
    condition_variable condition;
    size_t maxSize;

public:
    ThreadSafeQueue(size_t max = 10) : maxSize(max) {}
    
    void push(T item) {
        unique_lock<mutex> lock(mtx);
        
        // Wait until there's space in the queue
        condition.wait(lock, [this] { return dataQueue.size() < maxSize; });
        
        dataQueue.push(item);
        cout << "Produced: " << item << " (queue size: " << dataQueue.size() << ")" << endl;
        
        // Notify consumers
        condition.notify_one();
    }
    
    T pop() {
        unique_lock<mutex> lock(mtx);
        
        // Wait until there's data in the queue
        condition.wait(lock, [this] { return !dataQueue.empty(); });
        
        T result = dataQueue.front();
        dataQueue.pop();
        
        cout << "Consumed: " << result << " (queue size: " << dataQueue.size() << ")" << endl;
        
        // Notify producers
        condition.notify_one();
        
        return result;
    }
    
    bool empty() const {
        lock_guard<mutex> lock(mtx);
        return dataQueue.empty();
    }
    
    size_t size() const {
        lock_guard<mutex> lock(mtx);
        return dataQueue.size();
    }
};

void producer(ThreadSafeQueue<int>& queue, int id, int itemCount) {
    random_device rd;
    mt19937 gen(rd());
    uniform_int_distribution<> dis(100, 999);
    
    for (int i = 0; i < itemCount; i++) {
        int item = dis(gen);
        queue.push(item);
        
        // Random delay
        this_thread::sleep_for(chrono::milliseconds(dis(gen) % 200 + 50));
    }
    
    cout << "Producer " << id << " finished" << endl;
}

void consumer(ThreadSafeQueue<int>& queue, int id, int itemCount) {
    for (int i = 0; i < itemCount; i++) {
        int item = queue.pop();
        
        // Simulate processing time
        this_thread::sleep_for(chrono::milliseconds(100));
    }
    
    cout << "Consumer " << id << " finished" << endl;
}

void demonstrateProducerConsumer() {
    cout << "=== Producer-Consumer Pattern ===" << endl;
    
    ThreadSafeQueue<int> queue(5); // Buffer size of 5
    
    const int numProducers = 2;
    const int numConsumers = 2;
    const int itemsPerProducer = 5;
    const int itemsPerConsumer = 5;
    
    vector<thread> threads;
    
    // Start producers
    for (int i = 0; i < numProducers; i++) {
        threads.emplace_back(producer, ref(queue), i + 1, itemsPerProducer);
    }
    
    // Start consumers
    for (int i = 0; i < numConsumers; i++) {
        threads.emplace_back(consumer, ref(queue), i + 1, itemsPerConsumer);
    }
    
    // Wait for all threads
    for (auto& t : threads) {
        t.join();
    }
    
    cout << "Final queue size: " << queue.size() << endl;
}

// Reader-Writer pattern with shared_mutex (C++17)
class ReadWriteData {
private:
    mutable shared_mutex mtx;
    vector<string> data;

public:
    void write(const string& value) {
        unique_lock<shared_mutex> lock(mtx);
        data.push_back(value);
        cout << "Written: " << value << " (total items: " << data.size() << ")" << endl;
        
        // Simulate write time
        this_thread::sleep_for(chrono::milliseconds(50));
    }
    
    string read(size_t index) const {
        shared_lock<shared_mutex> lock(mtx);
        
        if (index < data.size()) {
            string result = data[index];
            cout << "Read: " << result << " from index " << index << endl;
            
            // Simulate read time
            this_thread::sleep_for(chrono::milliseconds(10));
            
            return result;
        }
        
        return "";
    }
    
    size_t size() const {
        shared_lock<shared_mutex> lock(mtx);
        return data.size();
    }
    
    void readAll() const {
        shared_lock<shared_mutex> lock(mtx);
        cout << "Reading all data (" << data.size() << " items):" << endl;
        
        for (size_t i = 0; i < data.size(); i++) {
            cout << "  [" << i << "] " << data[i] << endl;
        }
        
        this_thread::sleep_for(chrono::milliseconds(20));
    }
};

void writer(ReadWriteData& data, int id, int writeCount) {
    for (int i = 0; i < writeCount; i++) {
        string value = "Writer" + to_string(id) + "_Item" + to_string(i + 1);
        data.write(value);
        
        this_thread::sleep_for(chrono::milliseconds(100));
    }
}

void reader(ReadWriteData& data, int id, int readCount) {
    random_device rd;
    mt19937 gen(rd());
    
    for (int i = 0; i < readCount; i++) {
        // Read random index or read all
        if (i % 3 == 0) {
            data.readAll();
        } else {
            size_t size = data.size();
            if (size > 0) {
                uniform_int_distribution<> dis(0, size - 1);
                data.read(dis(gen));
            }
        }
        
        this_thread::sleep_for(chrono::milliseconds(80));
    }
}

void demonstrateReaderWriter() {
    cout << "\n=== Reader-Writer Pattern ===" << endl;
    
    ReadWriteData data;
    
    vector<thread> threads;
    
    // Start writers
    threads.emplace_back(writer, ref(data), 1, 5);
    threads.emplace_back(writer, ref(data), 2, 5);
    
    // Start readers
    threads.emplace_back(reader, ref(data), 1, 8);
    threads.emplace_back(reader, ref(data), 2, 8);
    threads.emplace_back(reader, ref(data), 3, 8);
    
    // Wait for all threads
    for (auto& t : threads) {
        t.join();
    }
    
    cout << "Final data size: " << data.size() << endl;
}

int main() {
    demonstrateProducerConsumer();
    demonstrateReaderWriter();
    return 0;
}
```

### 3. **Async and Futures**
```cpp
#include <iostream>
#include <future>
#include <chrono>
#include <vector>
#include <numeric>
#include <random>
using namespace std;

// CPU-intensive task
long long calculateSum(long long start, long long end) {
    cout << "Calculating sum from " << start << " to " << end << endl;
    
    long long sum = 0;
    for (long long i = start; i <= end; i++) {
        sum += i;
    }
    
    // Simulate some processing time
    this_thread::sleep_for(chrono::milliseconds(100));
    
    return sum;
}

// I/O-intensive task simulation
string fetchData(const string& source, int delay) {
    cout << "Fetching data from " << source << "..." << endl;
    
    // Simulate network delay
    this_thread::sleep_for(chrono::milliseconds(delay));
    
    return "Data from " + source + " (delay: " + to_string(delay) + "ms)";
}

void demonstrateAsync() {
    cout << "=== std::async and std::future ===" << endl;
    
    // Async with different launch policies
    cout << "\n1. Different launch policies:" << endl;
    
    // Deferred execution (lazy)
    auto deferredFuture = async(launch::deferred, calculateSum, 1LL, 1000000LL);
    
    // Async execution (eager)
    auto asyncFuture = async(launch::async, calculateSum, 1000001LL, 2000000LL);
    
    // Auto (implementation decides)
    auto autoFuture = async(calculateSum, 2000001LL, 3000000LL);
    
    cout << "Futures created, now getting results..." << endl;
    
    // Get results (this is where deferred actually executes)
    cout << "Deferred result: " << deferredFuture.get() << endl;
    cout << "Async result: " << asyncFuture.get() << endl;
    cout << "Auto result: " << autoFuture.get() << endl;
    
    // Multiple async operations
    cout << "\n2. Multiple async operations:" << endl;
    
    vector<future<string>> futures;
    vector<string> sources = {"Server1", "Server2", "Server3", "Server4"};
    vector<int> delays = {200, 150, 300, 100};
    
    // Start all async operations
    for (size_t i = 0; i < sources.size(); i++) {
        futures.emplace_back(async(launch::async, fetchData, sources[i], delays[i]));
    }
    
    // Wait for results in order
    cout << "Collecting results in order:" << endl;
    for (size_t i = 0; i < futures.size(); i++) {
        cout << "Result " << (i + 1) << ": " << futures[i].get() << endl;
    }
}

// Complex computation that can be parallelized
vector<int> generateRandomNumbers(size_t count, int seed) {
    mt19937 gen(seed);
    uniform_int_distribution<> dis(1, 1000);
    
    vector<int> numbers;
    numbers.reserve(count);
    
    for (size_t i = 0; i < count; i++) {
        numbers.push_back(dis(gen));
    }
    
    this_thread::sleep_for(chrono::milliseconds(50)); // Simulate work
    return numbers;
}

double calculateAverage(const vector<int>& numbers) {
    if (numbers.empty()) return 0.0;
    
    long long sum = accumulate(numbers.begin(), numbers.end(), 0LL);
    this_thread::sleep_for(chrono::milliseconds(30)); // Simulate work
    
    return static_cast<double>(sum) / numbers.size();
}

int findMaximum(const vector<int>& numbers) {
    if (numbers.empty()) return 0;
    
    int maxVal = *max_element(numbers.begin(), numbers.end());
    this_thread::sleep_for(chrono::milliseconds(20)); // Simulate work
    
    return maxVal;
}

void demonstrateComplexAsync() {
    cout << "\n=== Complex Async Computations ===" << endl;
    
    const size_t dataSize = 100000;
    
    auto startTime = chrono::high_resolution_clock::now();
    
    // Start data generation asynchronously
    auto dataFuture = async(launch::async, generateRandomNumbers, dataSize, 42);
    
    // While data is being generated, we can do other work
    cout << "Data generation started, doing other work..." << endl;
    this_thread::sleep_for(chrono::milliseconds(20));
    
    // Get the generated data
    vector<int> data = dataFuture.get();
    cout << "Generated " << data.size() << " random numbers" << endl;
    
    // Now start multiple computations on the data
    auto avgFuture = async(launch::async, calculateAverage, cref(data));
    auto maxFuture = async(launch::async, findMaximum, cref(data));
    
    // Parallel computation of partial sums
    const size_t numChunks = 4;
    const size_t chunkSize = data.size() / numChunks;
    
    vector<future<long long>> sumFutures;
    
    for (size_t i = 0; i < numChunks; i++) {
        size_t start = i * chunkSize;
        size_t end = (i == numChunks - 1) ? data.size() : (i + 1) * chunkSize;
        
        sumFutures.emplace_back(async(launch::async, [&data, start, end]() {
            long long partialSum = 0;
            for (size_t j = start; j < end; j++) {
                partialSum += data[j];
            }
            this_thread::sleep_for(chrono::milliseconds(25)); // Simulate work
            return partialSum;
        }));
    }
    
    // Collect results
    double average = avgFuture.get();
    int maximum = maxFuture.get();
    
    long long totalSum = 0;
    for (auto& future : sumFutures) {
        totalSum += future.get();
    }
    
    auto endTime = chrono::high_resolution_clock::now();
    auto duration = chrono::duration_cast<chrono::milliseconds>(endTime - startTime);
    
    cout << "Results:" << endl;
    cout << "- Average: " << average << endl;
    cout << "- Maximum: " << maximum << endl;
    cout << "- Total Sum: " << totalSum << endl;
    cout << "- Computation time: " << duration.count() << " ms" << endl;
}

// Exception handling with futures
int riskyComputation(int value) {
    if (value < 0) {
        throw invalid_argument("Negative values not allowed");
    }
    
    if (value == 0) {
        throw runtime_error("Zero division risk");
    }
    
    return 1000 / value;
}

void demonstrateAsyncExceptions() {
    cout << "\n=== Exception Handling with Async ===" << endl;
    
    vector<int> testValues = {10, 5, 0, -1, 2};
    vector<future<int>> futures;
    
    // Start computations
    for (int value : testValues) {
        futures.emplace_back(async(launch::async, riskyComputation, value));
    }
    
    // Collect results with exception handling
    for (size_t i = 0; i < futures.size(); i++) {
        try {
            int result = futures[i].get();
            cout << "Value " << testValues[i] << " -> Result: " << result << endl;
        } catch (const exception& e) {
            cout << "Value " << testValues[i] << " -> Exception: " << e.what() << endl;
        }
    }
}

int main() {
    demonstrateAsync();
    demonstrateComplexAsync();
    demonstrateAsyncExceptions();
    return 0;
}
```

## 🔬 Template Metaprogramming

### 1. **SFINAE and Type Traits**
```cpp
#include <iostream>
#include <type_traits>
#include <string>
#include <vector>
using namespace std;

// SFINAE - Substitution Failure Is Not An Error
template<typename T>
typename enable_if<is_integral<T>::value, void>::type
processValue(T value) {
    cout << "Processing integral value: " << value << endl;
}

template<typename T>
typename enable_if<is_floating_point<T>::value, void>::type
processValue(T value) {
    cout << "Processing floating-point value: " << value << " (precise)" << endl;
}

template<typename T>
typename enable_if<is_same<T, string>::value, void>::type
processValue(const T& value) {
    cout << "Processing string value: \"" << value << "\"" << endl;
}

// C++17 if constexpr version
template<typename T>
void processValueModern(T value) {
    if constexpr (is_integral_v<T>) {
        cout << "Modern: Processing integral value: " << value << endl;
    } else if constexpr (is_floating_point_v<T>) {
        cout << "Modern: Processing floating-point value: " << value << endl;
    } else if constexpr (is_same_v<T, string>) {
        cout << "Modern: Processing string value: \"" << value << "\"" << endl;
    } else {
        cout << "Modern: Processing other type" << endl;
    }
}

// Custom type traits
template<typename T>
struct is_container : false_type {};

template<typename T, typename Alloc>
struct is_container<vector<T, Alloc>> : true_type {};

// You can extend this for other containers
template<typename T>
constexpr bool is_container_v = is_container<T>::value;

// SFINAE for method detection
template<typename T>
class has_size_method {
private:
    template<typename C>
    static auto test(int) -> decltype(declval<C>().size(), true_type{});
    
    template<typename>
    static false_type test(...);

public:
    static constexpr bool value = decltype(test<T>(0))::value;
};

template<typename T>
constexpr bool has_size_method_v = has_size_method<T>::value;

template<typename T>
enable_if_t<has_size_method_v<T>, size_t>
getSize(const T& container) {
    return container.size();
}

template<typename T>
enable_if_t<!has_size_method_v<T>, size_t>
getSize(const T&) {
    return 1; // Assume single element for non-containers
}

void demonstrateSFINAE() {
    cout << "=== SFINAE and Type Traits ===" << endl;
    
    // Test different overloads
    processValue(42);           // integral
    processValue(3.14);         // floating-point
    processValue(string("test")); // string
    
    cout << "\nModern C++17 version:" << endl;
    processValueModern(42);
    processValueModern(3.14);
    processValueModern(string("test"));
    processValueModern('c');    // other type
    
    // Test custom type traits
    cout << "\nCustom type traits:" << endl;
    cout << "vector<int> is container: " << is_container_v<vector<int>> << endl;
    cout << "int is container: " << is_container_v<int> << endl;
    
    // Test method detection
    vector<int> vec = {1, 2, 3, 4, 5};
    int single = 42;
    
    cout << "Vector size: " << getSize(vec) << endl;
    cout << "Integer 'size': " << getSize(single) << endl;
}

// Compile-time computation
template<int N>
struct Factorial {
    static constexpr int value = N * Factorial<N - 1>::value;
};

template<>
struct Factorial<0> {
    static constexpr int value = 1;
};

// C++14 constexpr function version
constexpr int factorial(int n) {
    return n <= 1 ? 1 : n * factorial(n - 1);
}

// Compile-time string processing
template<size_t N>
constexpr size_t constexpr_strlen(const char (&str)[N]) {
    return N - 1; // Exclude null terminator
}

constexpr bool constexpr_equal(const char* a, const char* b, size_t len) {
    return len == 0 ? true : (*a == *b && constexpr_equal(a + 1, b + 1, len - 1));
}

void demonstrateMetaprogramming() {
    cout << "\n=== Template Metaprogramming ===" << endl;
    
    // Compile-time factorial
    constexpr int fact5_template = Factorial<5>::value;
    constexpr int fact5_function = factorial(5);
    
    cout << "5! (template): " << fact5_template << endl;
    cout << "5! (constexpr): " << fact5_function << endl;
    
    // Compile-time string processing
    constexpr auto len = constexpr_strlen("Hello, World!");
    cout << "String length (compile-time): " << len << endl;
    
    constexpr bool same = constexpr_equal("test", "test", 4);
    cout << "Strings equal (compile-time): " << same << endl;
    
    // Type information at compile time
    cout << "\nType information:" << endl;
    cout << "int is signed: " << is_signed_v<int> << endl;
    cout << "unsigned int is signed: " << is_signed_v<unsigned int> << endl;
    cout << "int is arithmetic: " << is_arithmetic_v<int> << endl;
    cout << "string is arithmetic: " << is_arithmetic_v<string> << endl;
}

int main() {
    demonstrateSFINAE();
    demonstrateMetaprogramming();
    return 0;
}
```

### 2. **Variadic Templates and Perfect Forwarding**
```cpp
#include <iostream>
#include <memory>
#include <utility>
#include <string>
#include <vector>
using namespace std;

// Basic variadic template
template<typename... Args>
void print(Args... args) {
    ((cout << args << " "), ...); // C++17 fold expression
    cout << endl;
}

// C++11/14 version using recursion
template<typename First>
void printRecursive(First&& first) {
    cout << first << endl;
}

template<typename First, typename... Rest>
void printRecursive(First&& first, Rest&&... rest) {
    cout << first << " ";
    printRecursive(rest...);
}

// Variadic template class
template<typename... Types>
class Tuple;

template<>
class Tuple<> {
public:
    static constexpr size_t size() { return 0; }
};

template<typename Head, typename... Tail>
class Tuple<Head, Tail...> : private Tuple<Tail...> {
private:
    Head head;

public:
    template<typename H, typename... T>
    Tuple(H&& h, T&&... t) : Tuple<Tail...>(forward<T>(t)...), head(forward<H>(h)) {}
    
    Head& getHead() { return head; }
    const Head& getHead() const { return head; }
    
    Tuple<Tail...>& getTail() { return static_cast<Tuple<Tail...>&>(*this); }
    const Tuple<Tail...>& getTail() const { return static_cast<const Tuple<Tail...>&>(*this); }
    
    static constexpr size_t size() { return 1 + Tuple<Tail...>::size(); }
};

// Helper function to get element by index
template<size_t Index, typename... Types>
auto get(Tuple<Types...>& tuple) {
    if constexpr (Index == 0) {
        return tuple.getHead();
    } else {
        return get<Index - 1>(tuple.getTail());
    }
}

// Perfect forwarding factory
template<typename T, typename... Args>
unique_ptr<T> make_unique_perfect(Args&&... args) {
    return unique_ptr<T>(new T(forward<Args>(args)...));
}

// Universal reference and perfect forwarding
template<typename T>
void processReference(T&& value) {
    if constexpr (is_lvalue_reference_v<T>) {
        cout << "Lvalue reference: " << value << endl;
    } else {
        cout << "Rvalue reference: " << value << endl;
    }
}

// Variadic fold expressions (C++17)
template<typename... Args>
auto sum(Args... args) {
    return (args + ...); // Unary right fold
}

template<typename... Args>
auto multiply(Args... args) {
    return (args * ...);
}

template<typename... Args>
bool all_true(Args... args) {
    return (args && ...);
}

template<typename... Args>
bool any_true(Args... args) {
    return (args || ...);
}

// Function composition using variadic templates
template<typename F>
auto compose(F&& f) {
    return forward<F>(f);
}

template<typename F, typename... Fs>
auto compose(F&& f, Fs&&... fs) {
    return [f = forward<F>(f), composed = compose(forward<Fs>(fs)...)](auto&& x) {
        return f(composed(forward<decltype(x)>(x)));
    };
}

void demonstrateVariadicTemplates() {
    cout << "=== Variadic Templates ===" << endl;
    
    // Basic variadic function
    print("Hello", 42, 3.14, "World");
    
    cout << "Recursive version:" << endl;
    printRecursive("Hello", 42, 3.14, "World");
    
    // Variadic tuple
    Tuple<int, string, double> tuple(42, "hello", 3.14);
    
    cout << "Tuple contents:" << endl;
    cout << "Element 0: " << get<0>(tuple) << endl;
    cout << "Element 1: " << get<1>(tuple) << endl;
    cout << "Element 2: " << get<2>(tuple) << endl;
    cout << "Tuple size: " << tuple.size() << endl;
    
    // Perfect forwarding
    cout << "\nPerfect forwarding:" << endl;
    
    string lvalue = "lvalue string";
    processReference(lvalue);                    // lvalue
    processReference(string("rvalue string"));  // rvalue
    processReference("string literal");         // rvalue
    
    // Factory with perfect forwarding
    auto ptr = make_unique_perfect<string>("constructed with perfect forwarding");
    cout << "Created string: " << *ptr << endl;
    
    // Fold expressions
    cout << "\nFold expressions:" << endl;
    cout << "Sum: " << sum(1, 2, 3, 4, 5) << endl;
    cout << "Multiply: " << multiply(2, 3, 4) << endl;
    cout << "All true: " << all_true(true, true, true) << endl;
    cout << "Any true: " << any_true(false, true, false) << endl;
    
    // Function composition
    auto add_one = [](int x) { return x + 1; };
    auto multiply_two = [](int x) { return x * 2; };
    auto to_string = [](int x) { return to_string(x); };
    
    auto composed = compose(to_string, multiply_two, add_one);
    cout << "Composed function (5): " << composed(5) << endl; // ((5 + 1) * 2) -> "12"
}

// Advanced: Type list manipulation
template<typename... Types>
struct TypeList {};

template<typename TypeList>
struct Length;

template<typename... Types>
struct Length<TypeList<Types...>> {
    static constexpr size_t value = sizeof...(Types);
};

template<typename TypeList>
constexpr size_t Length_v = Length<TypeList>::value;

template<size_t Index, typename TypeList>
struct TypeAt;

template<size_t Index, typename Head, typename... Tail>
struct TypeAt<Index, TypeList<Head, Tail...>> {
    using type = typename TypeAt<Index - 1, TypeList<Tail...>>::type;
};

template<typename Head, typename... Tail>
struct TypeAt<0, TypeList<Head, Tail...>> {
    using type = Head;
};

template<size_t Index, typename TypeList>
using TypeAt_t = typename TypeAt<Index, TypeList>::type;

// Apply a template to all types in a type list
template<template<typename> class F, typename TypeList>
struct Transform;

template<template<typename> class F, typename... Types>
struct Transform<F, TypeList<Types...>> {
    using type = TypeList<F<Types>...>;
};

template<template<typename> class F, typename TypeList>
using Transform_t = typename Transform<F, TypeList>::type;

void demonstrateTypeListManipulation() {
    cout << "\n=== Type List Manipulation ===" << endl;
    
    using MyTypes = TypeList<int, double, string>;
    
    cout << "Type list length: " << Length_v<MyTypes> << endl;
    
    // This would be compile-time only, but we can show the concept
    static_assert(is_same_v<TypeAt_t<0, MyTypes>, int>);
    static_assert(is_same_v<TypeAt_t<1, MyTypes>, double>);
    static_assert(is_same_v<TypeAt_t<2, MyTypes>, string>);
    
    cout << "Type at index 0 is int: " << is_same_v<TypeAt_t<0, MyTypes>, int> << endl;
    cout << "Type at index 1 is double: " << is_same_v<TypeAt_t<1, MyTypes>, double> << endl;
    cout << "Type at index 2 is string: " << is_same_v<TypeAt_t<2, MyTypes>, string> << endl;
    
    // Transform all types to pointers
    using PointerTypes = Transform_t<add_pointer, MyTypes>;
    static_assert(is_same_v<TypeAt_t<0, PointerTypes>, int*>);
    static_assert(is_same_v<TypeAt_t<1, PointerTypes>, double*>);
    static_assert(is_same_v<TypeAt_t<2, PointerTypes>, string*>);
    
    cout << "Transformed types to pointers successfully" << endl;
}

int main() {
    demonstrateVariadicTemplates();
    demonstrateTypeListManipulation();
    return 0;
}
```

## 🎯 Modern C++ Best Practices

### Best Practices Summary
```cpp
#include <iostream>
#include <memory>
#include <vector>
#include <algorithm>
#include <string>
#include <optional>
#include <variant>
using namespace std;

// 1. Modern C++ Resource Management
class ModernResource {
private:
    unique_ptr<int[]> data;
    size_t size;

public:
    // Constructor
    explicit ModernResource(size_t s) : data(make_unique<int[]>(s)), size(s) {
        cout << "Created resource with " << size << " elements" << endl;
    }
    
    // Destructor (automatic cleanup)
    ~ModernResource() {
        cout << "Resource automatically cleaned up" << endl;
    }
    
    // Delete copy operations (unique ownership)
    ModernResource(const ModernResource&) = delete;
    ModernResource& operator=(const ModernResource&) = delete;
    
    // Move operations
    ModernResource(ModernResource&& other) noexcept 
        : data(move(other.data)), size(other.size) {
        other.size = 0;
        cout << "Resource moved" << endl;
    }
    
    ModernResource& operator=(ModernResource&& other) noexcept {
        if (this != &other) {
            data = move(other.data);
            size = other.size;
            other.size = 0;
            cout << "Resource move-assigned" << endl;
        }
        return *this;
    }
    
    // Safe access
    int& at(size_t index) {
        if (index >= size) {
            throw out_of_range("Index out of range");
        }
        return data[index];
    }
    
    size_t getSize() const noexcept { return size; }
};

// 2. Optional and Variant (C++17)
optional<string> findUserName(int id) {
    static map<int, string> users = {
        {1, "Alice"}, {2, "Bob"}, {3, "Charlie"}
    };
    
    auto it = users.find(id);
    if (it != users.end()) {
        return it->second;
    }
    return nullopt;
}

using Value = variant<int, double, string>;

Value processInput(const string& input) {
    if (input.find('.') != string::npos) {
        return stod(input);  // double
    } else if (all_of(input.begin(), input.end(), ::isdigit)) {
        return stoi(input);  // int
    } else {
        return input;        // string
    }
}

void demonstrateModernFeatures() {
    cout << "=== Modern C++ Features ===" << endl;
    
    // Resource management
    {
        auto resource = make_unique<ModernResource>(10);
        for (size_t i = 0; i < resource->getSize(); i++) {
            resource->at(i) = i * i;
        }
        
        cout << "Resource element 5: " << resource->at(5) << endl;
        
        // Move resource
        auto moved = move(resource);
        cout << "Original resource size after move: " 
             << (resource ? resource->getSize() : 0) << endl;
        cout << "Moved resource size: " << moved->getSize() << endl;
    } // Automatic cleanup
    
    // Optional usage
    cout << "\nOptional usage:" << endl;
    for (int id : {1, 2, 4, 3}) {
        if (auto name = findUserName(id)) {
            cout << "User " << id << ": " << *name << endl;
        } else {
            cout << "User " << id << ": Not found" << endl;
        }
    }
    
    // Variant usage
    cout << "\nVariant usage:" << endl;
    vector<string> inputs = {"42", "3.14", "hello", "123"};
    
    for (const auto& input : inputs) {
        Value value = processInput(input);
        
        visit([&input](const auto& v) {
            cout << "Input '" << input << "' -> " << v 
                 << " (type: " << typeid(v).name() << ")" << endl;
        }, value);
    }
}

// 3. Algorithm usage and functional style
void demonstrateAlgorithms() {
    cout << "\n=== Modern Algorithm Usage ===" << endl;
    
    vector<int> numbers = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    
    // Find elements
    auto it = find_if(numbers.begin(), numbers.end(), 
                     [](int n) { return n > 5; });
    if (it != numbers.end()) {
        cout << "First number > 5: " << *it << endl;
    }
    
    // Transform and filter in functional style
    vector<int> evens;
    copy_if(numbers.begin(), numbers.end(), back_inserter(evens),
            [](int n) { return n % 2 == 0; });
    
    cout << "Even numbers: ";
    for (int n : evens) cout << n << " ";
    cout << endl;
    
    // Transform
    vector<int> squares;
    transform(numbers.begin(), numbers.end(), back_inserter(squares),
              [](int n) { return n * n; });
    
    cout << "Squares: ";
    for (int n : squares) cout << n << " ";
    cout << endl;
    
    // Accumulate with lambda
    auto sum = accumulate(numbers.begin(), numbers.end(), 0,
                         [](int acc, int n) { return acc + n; });
    cout << "Sum: " << sum << endl;
    
    // All/any/none of
    bool allPositive = all_of(numbers.begin(), numbers.end(),
                             [](int n) { return n > 0; });
    bool anyEven = any_of(numbers.begin(), numbers.end(),
                         [](int n) { return n % 2 == 0; });
    bool noneNegative = none_of(numbers.begin(), numbers.end(),
                               [](int n) { return n < 0; });
    
    cout << "All positive: " << allPositive << endl;
    cout << "Any even: " << anyEven << endl;
    cout << "None negative: " << noneNegative << endl;
}

// 4. Best practices summary
void bestPracticesSummary() {
    cout << "\n=== Modern C++ Best Practices Summary ===" << endl;
    
    cout << "\n✅ DO:" << endl;
    cout << "1. Use smart pointers (unique_ptr, shared_ptr, weak_ptr)" << endl;
    cout << "2. Prefer stack allocation and automatic storage" << endl;
    cout << "3. Use move semantics for expensive objects" << endl;
    cout << "4. Use const and constexpr liberally" << endl;
    cout << "5. Use auto for type deduction" << endl;
    cout << "6. Use range-based for loops" << endl;
    cout << "7. Use lambda expressions for short functions" << endl;
    cout << "8. Use STL algorithms instead of manual loops" << endl;
    cout << "9. Use RAII for resource management" << endl;
    cout << "10. Use exceptions for error handling" << endl;
    cout << "11. Use optional/variant instead of null pointers" << endl;
    cout << "12. Use uniform initialization {}" << endl;
    cout << "13. Use override and final keywords" << endl;
    cout << "14. Use enum class instead of enum" << endl;
    cout << "15. Use std::array instead of C arrays" << endl;
    
    cout << "\n❌ AVOID:" << endl;
    cout << "1. Raw pointers for ownership" << endl;
    cout << "2. Manual memory management (new/delete)" << endl;
    cout << "3. C-style casts" << endl;
    cout << "4. Macros (prefer constexpr/templates)" << endl;
    cout << "5. Global variables" << endl;
    cout << "6. using namespace std in headers" << endl;
    cout << "7. C-style arrays" << endl;
    cout << "8. Manual resource management" << endl;
    cout << "9. Unchecked array access" << endl;
    cout << "10. Ignoring compiler warnings" << endl;
}

int main() {
    demonstrateModernFeatures();
    demonstrateAlgorithms();
    bestPracticesSummary();
    return 0;
}
```

## 📚 Additional Resources and Learning Path

### Learning Progression
1. **Master the Fundamentals** (Files 1-8)
   - Focus on syntax, basic OOP, and memory management
   - Practice with small projects and exercises

2. **Advanced OOP Concepts** (Files 9-12)
   - Inheritance hierarchies and design patterns
   - Template programming and generic design

3. **Professional Development** (Files 13-17)
   - Error handling and robust code design
   - Modern C++ features and best practices
   - Concurrency and high-performance programming

### Recommended Next Steps
- **Practice Projects**: Build real applications using these concepts
- **Code Review**: Study open-source C++ projects
- **Standards**: Keep up with new C++ standards (C++20, C++23)
- **Tools**: Learn debugging, profiling, and static analysis tools
- **Design Patterns**: Study common software design patterns
- **Performance**: Learn about optimization and performance tuning

## 🔗 Related Topics
- [Basics](./01-basics.md) - Start your C++ journey
- [Memory Management](./16-memory-management.md) - Essential for advanced C++
- [STL](./15-stl.md) - Modern C++ relies heavily on STL
- [Templates](./12-templates.md) - Foundation for metaprogramming

---
*Previous: [Memory Management](./16-memory-management.md) | [Back to README](./README.md)*
