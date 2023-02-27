---
layout: default
title: Synchronization Primitives
usemathjax: true
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

- **Wait** queue: processes waiting for **re-entry** into the monitor. 
- **Monitor** queue: processes waiting for **first entrance** into the monitor

1. `wait` puts process onto **wait** queue and process exits monitor.
2. `notify` removes some process from **wait** queue and puts onto the **monitor** queue
3. `notifyAll` unblocks all processes on the wait queue.

## Problem set

1. Starvation Free Readers-Writers with Monitors


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