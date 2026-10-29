# Linux-IPC-Semaphores
Ex05-Linux IPC-Semaphores

# AIM:
To Write a C program that implements a producer-consumer system with two processes using Semaphores.

# DESIGN STEPS:

### Step 1:

Navigate to any Linux environment installed on the system or installed inside a virtual environment like virtual box/vmware or online linux JSLinux (https://bellard.org/jslinux/vm.html?url=alpine-x86.cfg&mem=192) or docker.

### Step 2:

Write the C Program using Linux Process API - Sempahores

### Step 3:

Execute the C Program for the desired output. 

# PROGRAM:

## Write a C program that implements a producer-consumer system with two processes using Semaphores.
~~~
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/ipc.h>
#include <sys/sem.h>
#include <sys/wait.h>
#include <sys/mman.h>

#define N 5

union semun {
    int val;
};

void P(int semid, int num) {
    struct sembuf op = {num, -1, 0};
    semop(semid, &op, 1);
}

void V(int semid, int num) {
    struct sembuf op = {num, 1, 0};
    semop(semid, &op, 1);
}

int main(void) {
    int semid;
    int *buffer;
    union semun arg;

    buffer = mmap(NULL, sizeof(int),
                  PROT_READ | PROT_WRITE,
                  MAP_SHARED | MAP_ANONYMOUS, -1, 0);

    if (buffer == MAP_FAILED) {
        perror("mmap");
        return 1;
    }

    semid = semget(IPC_PRIVATE, 2, 0666 | IPC_CREAT);
    if (semid == -1) {
        perror("semget");
        return 1;
    }

    arg.val = 1;
    semctl(semid, 0, SETVAL, arg);  // Empty

    arg.val = 0;
    semctl(semid, 1, SETVAL, arg);  // Full

    printf("Semaphore set created: %d\n", semid);
    fflush(stdout);

    pid_t pid = fork();

    if (pid == -1) {
        perror("fork");
        semctl(semid, 0, IPC_RMID);
        return 1;
    }

    if (pid == 0) {
        // Producer
        for (int i = 1; i <= N; i++) {
            P(semid, 0);
            *buffer = i;
            printf("Producer produced: %d\n", *buffer);
            V(semid, 1);
        }
        munmap(buffer, sizeof(int));
        exit(0);
    } else {
        // Consumer
        for (int i = 1; i <= N; i++) {
            P(semid, 1);
            printf("Consumer consumed: %d\n", *buffer);
            V(semid, 0);
        }

        wait(NULL);
        semctl(semid, 0, IPC_RMID);
        munmap(buffer, sizeof(int));
        printf("Semaphore removed. Done.\n");
    }

    return 0;
}
~~~



## OUTPUT
$ ./sem.o


<img width="423" height="292" alt="image" src="https://github.com/user-attachments/assets/f462d4e3-76fa-46cf-ab26-653f351e1852" />


$ ipcs


<img width="517" height="111" alt="image" src="https://github.com/user-attachments/assets/33973df7-4935-42c7-a1ab-8f6502482ed9" />





# RESULT:
The program is executed successfully.
