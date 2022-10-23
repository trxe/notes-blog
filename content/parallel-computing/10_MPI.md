---
layout: default
usemathjax: true
permalink: /parallel/ch10
---

# MPI

An **abstraction** of 
- parallel computer
- with **distributed address space**

Find the parallelism **explicitly** and coordinating this.

MPI is **standard** that a message-passing library interface should adhere to.

Q: Why use MPI instead of simply making a huge address space?
A: Extent of memory contention will scale with address space

## Program Structure

1. [Single process] C program + MPI include file
2. [FORK, Multiple processes start running] MPI environment initialized
3. [Multiple procs] Work & message passing calls
4. [JOIN, Back to single process] Terminate MPI environment

No shared address space, each process is on a a different processor with distributed address space.

## A process in MPI program

There is a **communicator** in every MPI program.

In the communicator, each process has 

- size (`MPI_Comm_size`)
- rank (`MPI_Comm_rank`)
- 

```cpp
int main(int argc, char **argv)
{
    /* Runs on ONE PROCESSOR */
	int rank, size;
	char hostname[256];
	
    /* Runs on EACH PROCESSOR (multiple) */
	MPI_Init(&argc, &argv);
	MPI_Comm_rank(MPI_COMM_WORLD, &rank);
	MPI_Comm_size(MPI_COMM_WORLD, &size);

	memset(hostname, 0, sizeof(hostname));
	int sc_status = gethostname(hostname, sizeof(hostname)-1);
	if (sc_status) {
		perror("gethostname");
		return sc_status;
	}

	/* From here on, each process is free to execute its own code */
    
    /* Rank can be used to identify processes. */
    if (rank == 0) {
        printf("%s: I am the first process!\n", hostname);
        // size == number of processes.
        for (i = 1; i < size; i++) {
            MPI_Send(hostname, 256, MPI_CHAR, i, tag, MPI_COMM_WORLD);
        }
    } else {
        MPI_Status status;
        char recvHost[256];
        int rc = MPI_Recv(recvHost, 256, MPI_CHAR, 0, tag, MPI_COMM_WORLD, &status);
        printf("%s: recved msg from %s\n", hostname, recvHost);
    }

	printf("Hello world from process %d out of %d on host %s\n", rank, size, hostname);

	MPI_Finalize();

    /* Anything else here runs on host processor again. */

	return 0;
}
```

## MPI Calls (Point to Point communication)

|          | Blocking     | Non-Blocking  |
|----------|--------------|---------------|
| Send     | MPI_Send     | MPI_ISend     |
| Recv     | MPI_Recv     | MPI_IRecv     |
| SendRecv | MPI_SendRecv | MPI_ISendRecv |


MPI is about communication. If our processes are independent,we can just run `srun`/`sbatch`

`slurm` Handles job scheduling, distribution of processes across nodes/cores.

MPI: used forcommunication between processes.

- `srun -n 192 -N 12 /nfs/home/$USER/hello`
- processes must be less than cores
- other wise need to add `--overcommit`

