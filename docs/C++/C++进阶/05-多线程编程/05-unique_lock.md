# 【05】unique_lock

### 1. 为什么会有unique_lock?

因为mutex 在管理方面有瑕疵，因此出现了一个互斥量封装器lock_guard来智能地管理mutex。而lock_guard只是简单的管理，功能比较弱，有瑕疵。因此需要搞出来一个功能更强大的东西出来。而这个东西就是unique_lock 。

本质上来说lock_guard 和unique_lock都是为了更好地使用各种锁而诞生的。但是unique_lock更为灵活，功能更强大，可做的操作比较多。

unique_lock 有些时候也被称为：“灵活锁”。

### 2. lock_guard有啥问题？

std::lock_guard可能存在的问题：

| **序号** | **问题描述**       | **细节**                                                             |
| ------ | -------------- | ------------------------------------------------------------------ |
| 1      | **锁的粒度问题**     | std::lock_guard的锁作用域与对象的作用域相同，可能导致锁的粒度过大，影响并发性能。                   |
| 2      | **无法中途解锁**     | 一旦std::lock_guard对象构造，锁将被自动获取，并在对象析构时自动释放，无法中途手动解锁。                |
| 3      | **不支持条件变量**    | std::lock_guard通常与互斥锁一起使用，但不支持与条件变量（如std::condition_variable）配合使用。 |
| 4      | **不支持递归锁**     | 虽然可以与std::recursive_mutex一起使用，但std::lock_guard本身不提供对递归锁的特殊支持。      |
| 5      | **无法指定锁的尝试获取** | std::lock_guard总是尝试获取锁，不支持非阻塞（尝试）获取锁。                              |
| 6      | **无法与超时机制结合**  | std::lock_guard不提供超时机制，无法设置获取锁的等待时间。                               |

就是说，简单的场景下可以使用它，复杂的场景下它就力不从心。

```C++
#include <iostream>
#include<thread>
#include <vector>
#include <mutex>

using namespace std;

int mycount = 0;
int mycount1 = 0;
int mycount2 = 0;
mutex mymutex;

void sum()
{
	for (size_t i = 0; i < 10000; i++)
	{
		mycount2++; 
	}

	lock_guard<mutex> lock(mymutex); //我保护的是mutex类型的锁，它是mymutex
	for (size_t i = 0; i < 10000; i++)
	{
		mycount++;
	}
	//我想在这里解锁，但是lock_guard对象做不到

	for (size_t i = 0; i < 10000; i++)
	{
		mycount1++; //我不想受lock的锁定，但是没有办法  因为它是lock_guard
	}
	
	//lock_guard是粗粒度的，它一杆子插到底了



}

int main()
{
	vector<thread> mybox;
	for (size_t i = 0; i < 10; i++)
	{
		mybox.emplace_back(sum);
	}

	for (thread& t : mybox)
	{
		t.join();
	}

	//最终结果
	cout << mycount << endl;
	cout << mycount1 << endl;
	cout << mycount2 << endl;

	return 0;
}
```

### 3. 什么是unique_lock？

unique_lock是一个更灵活的互斥量封装器，它提供了更多的控制选项，比如延迟锁定、尝试锁定、递归锁定、定时锁定等。与std::lock_guard相比，std::unique_lock提供了更多的功能，但也需要更多的管理责任。

### 4. 如何定义使用unique_lock

unique_lock的构造。

```C++
std::mutex mtx;
std::unique_lock<std::mutex> lk1(mtx);  //就自动上锁 
// 自动锁定
std::unique_lock<std::mutex> lk2(mtx, std::defer_lock);  // 延迟锁定 不要自动上锁
std::unique_lock<std::mutex> lk3(mtx, std::try_to_lock); // 尝试锁定
std::unique_lock<std::mutex> lk4(mtx, std::adopt_lock);  // 接受已锁定的 mutex
```

std::defer_lock 的具体含义是告诉 std::unique_lock 在构造时不要自动锁定互斥锁，而是延迟锁定操作

std::unique_lock 提供了一些成员函数，用于管理锁定状态：

- **lock()**：锁定关联的 mutex。
- **unlock()**：解锁关联的 mutex。
- **try_lock()**：尝试锁定 mutex，如果锁定成功，返回 true；否则返回 false。
- **owns_lock()**：返回一个布尔值，指示 unique_lock 是否拥有 mutex 的所有权。

  

代码样例：

```C++
#include <iostream>  
#include <thread>  
#include <mutex>  
#include <chrono>  
  
std::mutex mtx;  
  
void print_block(int n, char c) {  
    // 尝试在2秒内锁定互斥量  
    if (std::unique_lock<std::mutex> lock(mtx, std::try_to_lock_t()); 
        lock.owns_lock()) {  
        for (int i = 0; i < n; ++i) {  
            std::cout << c;  
        }  
        std::cout << '\n';  
    } else {  
        std::cout << "Failed to lock mutex!\n";  
    }  
}  
  
int main() {  
    std::thread th1(print_block, 50, '*');  
    std::thread th2(print_block, 50, '$');  
  
    th1.join();  
    th2.join();  
  
    return 0;  
}
```

在上面的例子中，std::unique_lock<std::mutex> lock(mtx, std::try_to_lock_t());尝试在构造时锁定互斥量mtx，但不会阻塞。如果互斥量已经被锁定，则lock.owns_lock()将返回false，表示没有成功锁定互斥量。

除了std::try_to_lock_t()之外，std::unique_lock还接受其他类型的参数，如std::defer_lock_t()（延迟锁定）、std::adopt_lock_t()（假设已经锁定）等，以满足不同的需求。此外，std::unique_lock还提供了lock()、unlock()、try_lock()等成员函数来手动管理锁的获取和释放。

### 5. unique_lock的第二个参数

`std::unique_lock` 的构造函数可以接受多个参数，其中第二个参数用于指定如何管理锁的行为。常见的第二个参数有以下几种：

1. `std::defer_lock`
2. `std::try_to_lock`
3. `std::adopt_lock`
4. 超时相关参数（如`std::chrono`的时间段）

#### 5.1. `std::defer_lock`

`std::defer_lock` 表示延迟锁定。即在创建 `std::unique_lock` 对象时，不会立即对互斥锁进行锁定操作。

```C++
#include <iostream>
#include <mutex>
#include <thread>

std::mutex mtx;

void example_defer_lock() {
    std::unique_lock<std::mutex> lock(mtx, std::defer_lock); // 不会立即锁定
    // ... 执行一些操作 ...
    lock.lock(); // 在需要时显式锁定
    std::cout << "Locked with defer_lock" << std::endl;
    lock.unlock(); // 显式解锁
}

int main() {
    std::thread t1(example_defer_lock);
    t1.join();
    return 0;
}
```

#### 5.2. `std::try_to_lock`

`std::try_to_lock` 表示尝试锁定。在创建 `std::unique_lock` 对象时，会尝试锁定互斥锁。如果锁定失败，不会阻塞。

```C++
#include <iostream>
#include <mutex>
#include <thread>

std::mutex mtx;

void example_try_to_lock() {
    std::unique_lock<std::mutex> lock(mtx, std::try_to_lock); // 尝试锁定
    if (lock.owns_lock()) {
        std::cout << "Locked with try_to_lock" << std::endl;
    } else {
        std::cout << "Failed to lock with try_to_lock" << std::endl;
    }
}

int main() {
    std::thread t1(example_try_to_lock);
    t1.join();
    return 0;
}
```

#### 5.3. `std::adopt_lock`

`std::adopt_lock` 表示接管已经锁定的互斥锁。此参数假设互斥锁已经在其他地方被锁定，`std::unique_lock` 对象会接管这个锁的所有权。 使用了 std::adopt_lock，这意味着互斥锁已经被锁定

```C++
#include <iostream>
#include <mutex>
#include <thread>

std::mutex mtx;

void example_adopt_lock() {
    mtx.lock(); // 先锁定互斥锁
    std::unique_lock<std::mutex> lock(mtx, std::adopt_lock); // 接管已锁定的互斥锁
    std::cout << "Locked with adopt_lock" << std::endl;
    // lock goes out of scope and unlocks mtx
}

int main() {
    std::thread t1(example_adopt_lock);
    t1.join();
    return 0;
}
```

#### 5.4. 超时相关参数 mutex

`std::unique_lock` 还支持基于时间的锁定操作，允许使用超时参数。这些参数可以是 `std::chrono` 时间点或时间段，主要用于 `std::timed_mutex` 类型的互斥锁。

#### 5.5. 基于时间点的超时锁定

```C++
#include <iostream>
#include <mutex>
#include <thread>
#include <chrono>

std::timed_mutex mtx;

void example_timed_lock() {
    auto now = std::chrono::system_clock::now();
    std::unique_lock<std::timed_mutex> lock(mtx, now + std::chrono::seconds(1)); // 尝试锁定直到指定时间点
    if (lock.owns_lock()) {
        std::cout << "Locked with timed lock (time_point)" << std::endl;
    } else {
        std::cout << "Failed to lock with timed lock (time_point)" << std::endl;
    }
}

int main() {
    std::thread t1(example_timed_lock);
    t1.join();
    return 0;
}
```

#### 5.6. 基于时间段的超时锁定

```C++
#include <iostream>
#include <mutex>
#include <thread>
#include <chrono>

std::timed_mutex mtx;

void example_timed_lock_duration() {
    std::unique_lock<std::timed_mutex> lock(mtx, std::chrono::seconds(1)); // 尝试锁定指定时间段
    if (lock.owns_lock()) {
        std::cout << "Locked with timed lock (duration)" << std::endl;
    } else {
        std::cout << "Failed to lock with timed lock (duration)" << std::endl;
    }
}

int main() {
    std::thread t1(example_timed_lock_duration);
    t1.join();
    return 0;
}
```

### 6. unique_lock的特点

- **灵活性**：unique_lock提供了更多的控制选项，比如延迟锁定、尝试锁定、递归锁定、定时锁定等。这使得开发者可以根据具体的业务逻辑来精确控制锁的加锁和解锁时机。
- **可移动性**：unique_lock是可移动的，可以拷贝、赋值或移动。这使得开发者可以在需要时轻松地将锁的所有权从一个对象转移到另一个对象。
- **手动控制**：unique_lock支持手动解锁，而lock_guard不支持。这提供了更大的灵活性，但也需要开发者自行管理锁的加锁和解锁过程。