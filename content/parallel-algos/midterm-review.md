---
layout: default
title: Midterm review
usemathjax: true
permalink: /palgo/midterm
---

# Midterms

## The "Dekker" Qn:

```
want[me] = true;
while (want[them]) {
    if (turn == them) {
        want[me] = false;
        while (want[them] == false && turn == them); // SUPER IMPT MISTAKE
        want[me] = true;
    }
}
// CRIT SECT

turn = them;
want[me] = false;
```

If P1 and P2 both want to enter the critical section and enter the while loop, 
and WLOG suppose turn == 1, then P1 will always fall through and immediately turn 
want[me] back to true.

This causes a deadlock

## Vector Clock Protocol

> Is it possible to design a protocol using only $\lfloor n / 2 \rfloor + 1$ elements?

Ans: No.

*Proof*: Consider $n=3$. None of the processes communicate with each other.

*Lemma 1*: We cannot have $x_i = x_j, y_i = y_j$ for any 2 processes.

![Counterexample](/notes-blog/assets/img/palgo/midterm-vector-clock.png)

Suppose for contradictions that we have, WLOG, $x_1=x_2, y_1=y_2$, and that process 1 never communicates with process 2.

Suppose another event happens on process 1 ($x_4, y_4$).

We must have $x_1 < x_4, y_1 < y_4$ to maintain the happens-before relationship.

However this would make $x_2 < x_4, y_2 < y_4$ despite no happens-before relationship between the two processes' events.

*Lemma 2*: We cannot have $x_i \neq x_j$ for any 2 processes.

Suppose for contradiction that $x_i = x_j$. From *Lemma 1* we know that $y_i \neq y_j$.

Then either process $i$ happens-before $j$ or the other way round. Contradiction as none of the processes across the threads are 

*Lemma 3*: We cannot have $x_i \neq x_j$ AND $y_i \neq y_j$ for any 2 processes.

WLOG suppose $x_1 < x_2 < x_3$. Then $y_1 > y_2 > y_3$ due to *Lemma 1* and *2*.

Suppose another event happens on process 2 ($x_4, y_4$). Then either $x_2 < x_4$ or $y_2 < y_4$.

However we must NOT have happens before relationship between $p_1, p_4$ nor $p_3, p_4$.

$(y_1 - y_2) + (x_3 - x_2) + 1$???