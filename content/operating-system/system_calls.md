---
layout: default
usemathjax: true
permalink: /os/functions
---

# System calls

Some omit full descriptions (for sake of conciseness as an exam reference).

## Process management

| Function  | Parameters                               | Return value                    | Details                                                      | Library  |
| --------- | ---------------------------------------- | ------------------------------- | ------------------------------------------------------------ | -------- |
| `fork`    | `()`                                     | 0 (child); child's PID (parent) | Memory space is copy-on-write.                               | unistd.h |
| `exec_`   | depends (see details)                    | -1 (only if error)              | See [equivalent `exec` code](#equivalent-exec-code)          | unistd.h |
| `exit`    | `(int status)`                           | Status                          |                                                              | stdlib.h |
| `wait`    | `(int *wstatus)`                         | Status                          | Last 8 bits = exit status = `WEXITSTATUS(status)`            | unistd.h |
| `waitpid` | `(pid_t pid, int *wstatus, int options)` | Status                          | PID $\leq 0$: wait for any child. <br />Options: see [wait options](#wait-options) | unistd.h |
| `getpid`  | `()`                                     | PID of calling process          |                                                              | unistd.h |

### Equivalent exec code

List: `execl("/bin/ls", "ls", "-l", "-R", "-a", NULL);`

Vector: `char* arr[] = {"ls", "-l", "-R", "-a", NULL}; execv("/bin/ls", arr);`

PATH environment variable, i.e. mimics shell: `char *cmd = "ls"; execvp(cmd, arr);`

Others: `lp, le, ve, lpe, vpe`

### wait options

`WNOHANG`: return immediately if no child has exited.

`WEXITED, WSTOPPED, WCONTINUED`: wait for children that have been (exited/stopped/continued).

## File management

| Function | Parameters                                | Return value                                          | Details                                                      |
| -------- | ----------------------------------------- | ----------------------------------------------------- | ------------------------------------------------------------ |
| `open`   | `(char *path, int flags)`                 | -1 (error); fd (success)                              | path can be either absolute or relative (to current working directory of process). <br />flags: `O_RDONLY, O_WRONLY, O_RDWR | O_CREAT | O_TRUNC`. |
| `close`  | `(int fd)`                                | -1 (error); 0 (success)                               | if `fd` is the last to refer to the underlying open file, the resources associated with the open file description are freed. |
| `read`   | `(int fd, void *buf, size_t count)`       | -1 (error); bytes read (success)                      |                                                              |
| `write`  | `(int fd, const void *buf, size_t count)` | -1 (error); bytes written (success)                   |                                                              |
| `lseek`  | `(int fd, off_t offset, int whence)`      | -1 (error); offset bytes from start address (success) |                                                              |
| `dup2`   | `(int oldfd, int newfd)`                  | -1 (error); new fd (success)                          | allocates a new fd referring to the same open-file description (*system-wide open-file table*) as the `oldfd` |
| `pipe`   | `(int fd[])`                              | -1 (error); 0 (success)                               | `fd[0]` read end, `fd[1]` write end. For input redirection.  |

## Signal handling

| Function    | Parameters                           | Return value                                           | Details                                                      |
| ----------- | ------------------------------------ | ------------------------------------------------------ | ------------------------------------------------------------ |
| `signal`    | `(int signum, sighandler_t handler)` | `SIG_ERR` (error); previous value of handler (success) | signal handler provided must be of the signature `void handler(int signo)` |
| `sigaction` | `(int signum, sigaction *sa, NULL)`  | -1 (error); 0 (success)                                | Simplified parameters. `sa.sa_sigaction` must be set to the handler with params `(int signo, siginfo_t *info, void *ucontext)`. |

## Shared memory

| Function | Parameters                     | Return value                                                 | Details                                                      |
| -------- | ------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `shmget` | `(key, size_t size, shmflg)`   | -1 (error); valid shared memory identifier (success)         | `key=IPC_PRIVATE`. `size` rounded up to nearest page size. `IPC_CREAT` to create a new region. |
| `shmat`  | `(shmid, NULL, 0)`             | -1 (error); `void*`**address of shared memory segment** (success) | Simplified parameters.                                       |
| `shmdt`  | `(void * shmaddr)`             | -1 (error); 0 (success)                                      | **Detaches** the shared memory segment located at the address. |
| `shmctl` | `(shmid, cmd, shmid_ds * buf)` | -1 (error); 0 (success for void cmds)                        | `shmctl(id, IPC_RMID, 0)` to **destroy** a shm segment       |

