---
layout: default
title: Synchronization Primitives
permalink: /palgo/ch2
---

# Synchronization Primitives

## Motiviation

Busy wait is wasteful of CPU cycles.

### Semaphore

```java
boolean value;
BinarySemaphore(boolean initValue) {
    value = initValue;
}
public synchronized void P() {
    while (!value) {
        Util.myWait(this); // insert me into queue of blocked processes. **DOES NOT BUSY WAIT**
    }
    value = false;
}
public synchronized void V() {
    value = true;
    notify();
}

BinarySemaphore mutex = new BinarySemaphore(true);
mutex.P();
cs();
mutex.V();
```

### Monitors

A `class` used in concurrent programs.

- Has data that multiple processes can access simultaneously
- Entry methods that guarantee mutual exclusion

A form of **conditional synchronization**: thread waits for a certain condition to become true before continuing

- `wait`
- `notify`/`signal`

Two queues are present, the **wait** queue and the **monitor** queue.
1. `wait` puts process onto **wait** queue and process exits monitor.
2. `notify` removes some process from **wait** queue and puts onto the **monitor** queue

## Problem set

1. Starvation Free Readers-Writers with Monitors

```java
class ReaderWriter {
    int numReader;
    int numWriter;
    bool writerWaiting;

    void writeFile() {
        synchronized (object) {
            while (numReader > 0 || numWriter > 0) {
                writerWaiting = true;
                object.Wait();
            }
            numWriter = 1;
        }
        // Write
        synchronized (object) {
            numWriter = 0;
            writerWaiting = false;
            object.notifyAll();
        }
    }

    void readFile() {
        synchronized (object) {
            while (numWriter > 0 || writerWaiting) {
                // Since while loop true while writerWaiting, 
                // even when woken by finishing readers, it will continue to wait till
                // writers are done waiting.
                object.wait();
            }
            numReader++;
        }
        synchronized (object) {
            numReader--;
            object.notify(); // notifies any readers/writers waiting./
        }
    }
}
```

2. Sleeping Barber

Monitor ver.
```java
class SleepingBarber {
    int numCustomers = 0;
    int inBuf = 0;
    int outBuf = 0;
    int n;

    void runBarber() {
        // barber waits till there's a customer
        while (true) {
            synchronized (object) {
                while (numCustomers == 0) {
                    object.wait();
                }
            }
            synchronized (object) {
                cut(outBuf);
                numCustomers--;
                outBuf = (outBuf + 1) % n;
                object.notifyAll();
            }
        }
    }

    void hairCut() {
        int myid;
        synchronized (object) {
            if (numCustomers == n) return;
            numCustomers++;
            myid = inBuf;
            inBuf = (inBuf + 1) % n;
        }
        synchronized (object) {
            while (myid != (outBuf - 1) % n) {
                object.wait(); // if i'm not the last one to be freed i continue to wait.
            }
            numCustomers--;
        }
    }
}
```

Semaphore ver.
```java
class SleepingBarber {
    int n;
    int inBuf = 0, outBuf = 0;
    BS mutex = new BS(true);
    CS notEmpty = new CS(0);
    BS[] hair;
    int numCustomers = 0;

    public SleepingBarber(int n) {
        this.n = n;
        hair = new BS[n]; // set all to false
    }

    public synchronized void runBarber() {
        while (true) {
            notEmpty.P();
            mutex.P();
            hair[outBuf].V();
            outBuf = (outBuf + 1) % n;
            mutex.V();
        }
    }

    public synchronized void cutHair() {
        mutex.P();
        if (numCustomers == n) {
            mutex.V();
            return;
        }
        int val = inBuf;
        inBuf++;
        numCustomers++;
        notEmpty.V();
        mutex.V();
        hair[val].P();
        mutex.P();
        numCustomers--;
        mutex.V();

    }

}
```

3. P + Q $\geq$ R

```java
class PQR {
    int p = 0;
    int q = 0;
    int r = 0;
    void printP() {
        while (true) {
            synchronized (object) {
                object.wait();
                p++;
            }
            print("P");
            synchronized (object) {
                object.notifyAll();
            }
        }
    }
    void printQ() {
        while (true) {
            synchronized (object) {
                object.wait();
                q++;
            }
            print("Q");
            synchronized (object) {
                object.notifyAll();
            }
        }
    }
    void printR() {
        while (true) {
            synchronized (object) {
                while (r >= p + q) {
                    object.wait();
                }
                r++;
            }
            print("R");
            synchronized (object) {
                object.notifyAll();
            }
        }
    }
}

```