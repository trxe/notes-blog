---
layout: default
usemathjax: true
title: Miscellaneous programs for reference
permalink: /pcp/misc
---

## Some lock free stuff

```cpp
int fetch_add(std::atomic<int>& value, int add) {
    int old_value = value.load();
    while (true){
        int new_value = old_value + 5;
        if(value.compare_exchange_weak(old_value, new_value))
            return old_value;
    }
}
```

## Thread pool

```cpp
#include <vector>
#include <condition_variable>
#include <queue>
#include <thread>
#include <mutex>
#include <iostream>

using namespace std;

const size_t MAX_THREADS = std::thread::hardware_concurrency();

void run_task(int round, int id) {
    cout << "ROUND " << round << ":"
         << "computing " << id << endl;
}

int main() {
    vector<thread> thread_pool;
    queue<thread> task_queue;
    std::mutex queue_mutex;
    std::condition_variable queue_empty_cv;
    std::mutex round_mutex;
    std::condition_variable round_done_cv;
    bool round_done = false;
    for (size_t i = 0; i < MAX_THREADS; i++){
        thread_pool.push_back(thread([&] {
                while (true) {
                    std::unique_lock lk (queue_mutex);
                    queue_empty_cv.wait(lk, [&task_queue]{
                                return !task_queue.empty(); });
                    thread mythread = std::move(task_queue.front());
                    task_queue.pop();
                    lk.unlock();
                    mythread.join(); // run to completion
                }
            }));
    }

    int nodes = 3;
    int rounds = 20;
    for (int r = 0; r < rounds; r++) {
        for (int i = 0; i < nodes; i++) {
            std::unique_lock lk(queue_mutex);
            task_queue.push(thread([r, i, nodes, &round_done, &round_mutex, &round_done_cv] {
                        run_task(r, i);
                        cout << "ok " << i << endl;
                        if (i == nodes - 1) {
                            std::unique_lock lk(round_mutex);
                            round_done = true;
                            round_done_cv.notify_one();
                        }
                    }));
            queue_empty_cv.notify_one();
        }
        while (true) {
            std::unique_lock lk(round_mutex);
            round_done_cv.wait(lk, [&]{ return round_done; } );
            if (round_done) {
                round_done = false;
                std::this_thread::sleep_for(std::chrono::milliseconds(1000));
                break;
            }
        }
    }

    for (auto& t : thread_pool) {
        t.join();
    }
}
```

```go
package main

import (
    "fmt"
)

func task(round int, node int, amlast bool, coord chan<- struct{}) {
    fmt.Printf("R%v Node%v\n", round, node);
    if amlast {
        coord <- struct{}{}
    }
}

func worker(taskq <-chan func(), done chan struct{}) {
    for {
        select {
        case task := <-taskq:
            task();
        case <-done:
            return;
        }
    }
}

func main() {
    done :=  make(chan struct{});
    taskq := make(chan func());
    workers, nodes, rounds := 8, 3, 10
    for w := 0; w < workers; w++ {
        go worker(taskq, done);
    }

    for r := 0; r < rounds; r++ {
        coord := make(chan struct{});
        for i := 0; i < nodes; i++ {
            id := i
            amlast := i == nodes - 1
            taskq <- func() {
                task(r, id, amlast, coord);
            }
        }
        <-coord;
    }
}
```

## Spawning async threads and joining them

```rust
// From assignment 3:
// run leaf nodes in parallel
while let Some(next) = taskq.pop() {
  *count_map.entry(next.typ).or_insert(0usize) += 1;
  let result: (u64, Vec<Task>) = next.execute();
  output.fetch_xor(result.0, std::sync::atomic::Ordering::SeqCst);
  if next.height == 1 {
    let leaf_nodes = result.1.into_iter()
      .map(|t| { 
          *count_map.entry(t.typ).or_insert(0usize) += 1;
          (t, output.clone())
      })
      .map(|(task, output)| tokio::spawn(
          async move {
              let out = task.execute().0; 
              output.fetch_xor(out, std::sync::atomic::Ordering::SeqCst);
          }
      ));
    futures::future::join_all(leaf_nodes).await;
    continue;
  }
  taskq.extend(result.1.into_iter());
}
```