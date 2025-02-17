# 【11】栅栏（barrier）

## 1. 啥是栅栏

栅栏（barrier）对象是一种同步原语，用于协调多个线程的执行，使它们能够在某个特定的点（即栅栏）等待，直到所有线程都到达这一点，然后它们才能继续执行。栅栏可以确保并发任务在某些关键时刻同步，比如等待所有线程完成某个阶段的工作，然后再进入下一阶段。

!!! tip
	假设有一场学生答题比赛，比赛分为两个阶段：
	
	1. 第一阶段，所有学生需要完成一道题。
	2. 第二阶段，所有学生都必须等待，直到每个人都完成第一阶段的题目，然后才能继续回答第二阶段的题目。
	
	我们可以把栅栏（barrier）想象成一个比赛中的信号员，只有当所有学生都完成了第一阶段的题目，信号员才会挥动旗帜，表示可以开始第二阶段。

## 2. 栅栏的作用和特点
![](assets/%20栅栏的作用和特点.jpg)

## 3. 定义使用 C++20 中的 `std::barrier`

C++20 引入了 `std::barrier`，它提供了栅栏的功能。以下是一个简单的示例，演示如何使用 `std::barrier` 来同步多个线程：

```C++
#include <iostream>
#include <thread>
#include <barrier>
#include <vector>

void worker(int id, std::barrier<>& sync_point) {
    std::cout << "Worker " << id << " is doing phase 1 work\n";
    std::this_thread::sleep_for(std::chrono::milliseconds(100 * id)); // 模拟不同的工作时间

    // 到达栅栏，等待其他线程
    sync_point.arrive_and_wait();
    std::cout << "Worker " << id << " has completed phase 1 and is doing phase 2 work\n";

    std::this_thread::sleep_for(std::chrono::milliseconds(100 * id)); // 模拟不同的工作时间
}

int main() {
    const int num_threads = 5;
    std::barrier sync_point(num_threads); // 创建一个栅栏，等待所有线程到达

    std::vector<std::thread> threads;
    for (int i = 0; i < num_threads; ++i) {
        threads.emplace_back(worker, i + 1, std::ref(sync_point));
    }

    for (auto& t : threads) {
        t.join();
    }

    std::cout << "All workers have completed their tasks.\n";

    return 0;
}
```

  

1.**栅栏初始化**：

```C++
std::barrier sync_point(num_threads); // 创建一个栅栏，等待所有线程到达
```

创建一个栅栏对象 `sync_point`，它需要 `num_threads` 个线程到达栅栏后才继续执行。

2.**worker 函数**：

	- 每个线程先执行一些“第一阶段”的工作。
	- 然后调用 `sync_point.arrive_and_wait();`，表示到达栅栏并等待其他线程也到达。
	- 所有线程都到达栅栏后，继续执行“第二阶段”的工作。

3.**主线程**：

	- 创建并启动多个线程，执行 `worker` 函数。
	- 等待所有线程完成工作。