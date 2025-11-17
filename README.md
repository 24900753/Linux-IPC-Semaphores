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
```
#include <stdio.h>	 /* standard I/O routines.              */
#include <stdlib.h>      /* rand() and srand() functions        */
#include <unistd.h>	 /* fork(), etc.                        */
#include <time.h>	 /* nanosleep(), etc.                   */
#include <sys/types.h>   /* various type definitions.           */
#include <sys/ipc.h>     /* general SysV IPC structures         */
#include <sys/sem.h>	 /* semaphore functions and structs.    */
#define NUM_LOOPS	20	 /* number of loops to perform. */
#if defined(__GNU_LIBRARY__) && !defined(_SEM_SEMUN_UNDEFINED)
/* union semun is defined by including <sys/sem.h> */
#else
/* according to X/OPEN we have to define it ourselves */
union semun {
        int val;                    /* value for SETVAL */
        struct semid_ds *buf;       /* buffer for IPC_STAT, IPC_SET */
        unsigned short int *array;  /* array for GETALL, SETALL */
        struct seminfo *__buf;      /* buffer for IPC_INFO */
};
#endif

int main(int argc, char* argv[])
{
    int sem_set_id;	      /* ID of the semaphore set.       */
    union semun sem_val;      /* semaphore value, for semctl(). */
    int child_pid;	      /* PID of our child process.      */
    int i;		      /* counter for loop operation.    */
    struct sembuf sem_op;     /* structure for semaphore ops.   */
    int rc;		      /* return value of system calls.  */
    struct timespec delay;    /* used for wasting time.         */

    /* create a private semaphore set with one semaphore in it, */
    /* with access only to the owner.                           */
    sem_set_id = semget(IPC_PRIVATE, 1, 0600);
    if (sem_set_id == -1) {
	perror("main: semget");
	exit(1);
    }
    printf("semaphore set created, semaphore set id '%d'.\n", sem_set_id);

    /* intialize the first (and single) semaphore in our set to '0'. */
    sem_val.val = 0;
    rc = semctl(sem_set_id, 0, SETVAL, sem_val);
    if (rc == -1) {
	perror("main: semctl SETVAL");
	exit(1);
    }

    /* fork-off a child process, and start a producer/consumer job. */
    child_pid = fork();
    switch (child_pid) {
	case -1:	/* fork() failed */
	    perror("fork");
	    exit(1);
	case 0:		/* child process here */
	    for (i=0; i<NUM_LOOPS; i++) {
		/* block on the semaphore, unless it's value is non-negative. */
		sem_op.sem_num = 0;
		sem_op.sem_op = -1;
		sem_op.sem_flg = 0;
		rc = semop(sem_set_id, &sem_op, 1);
		if (rc == -1) {
		    perror("child: semop");
		    exit(1);
		}
		printf("consumer: '%d'\n", i);
		fflush(stdout);
	    }
	    break;
	default:	/* parent process here */
	    srand(time(NULL)); // Initialize random seed
	    for (i=0; i<NUM_LOOPS; i++) {
		printf("producer: '%d'\n", i);
		fflush(stdout);
		/* increase the value of the semaphore by 1. */
		sem_op.sem_num = 0;
		sem_op.sem_op = 1;
		sem_op.sem_flg = 0;
		rc = semop(sem_set_id, &sem_op, 1);
		if (rc == -1) {
		    perror("parent: semop");
		    exit(1);
		}
		/* pause execution for a little bit, to allow the */
		/* child process to run and handle some requests. */
		/* this is done about 25% of the time.            */
		if (rand() > 3*(RAND_MAX/4)) {
	    	    delay.tv_sec = 0;
	    	    delay.tv_nsec = 1000000; // 1 millisecond
	    	    nanosleep(&delay, NULL);
		}
	    }
	    if(NUM_LOOPS>=10) {
		rc = semctl(sem_set_id, 0, IPC_RMID, sem_val);
		if (rc == -1) {
		    perror("semctl IPC_RMID");
		    exit(1);
		}
	    }
	    break;
    }

    return 0;
}

```


```
#include <stdio.h>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <sys/sem.h>
#include <sys/msg.h>

int main() {
    key_t key1 = ftok(".", 1);  // Shared Memory key
    key_t key2 = ftok(".", 2);  // Semaphore key
    key_t key3 = ftok(".", 3);  // Message Queue key

    // ---------- Create Shared Memory ----------
    int shmid = shmget(key1, 1024, 0666 | IPC_CREAT);
    if (shmid < 0) {
        perror("shmget");
        return 1;
    }

    // ---------- Create Semaphore ----------
    int semid = semget(key2, 1, 0666 | IPC_CREAT);
    if (semid < 0) {
        perror("semget");
        return 1;
    }

    // ---------- Create Message Queue ----------
    int msqid = msgget(key3, 0666 | IPC_CREAT);
    if (msqid < 0) {
        perror("msgget");
        return 1;
    }

    printf("Shared Memory ID : %d\n", shmid);
    printf("Semaphore ID     : %d\n", semid);
    printf("Message Queue ID : %d\n", msqid);

    return 0;
}
```




## OUTPUT
$ ./sem.o 

<img width="608" height="800" alt="Screenshot from 2025-11-17 21-34-00" src="https://github.com/user-attachments/assets/75315c33-1c10-4127-a271-28e8a02a3a8b" />


$ ipcs

<img width="752" height="461" alt="Screenshot from 2025-11-17 21-37-43" src="https://github.com/user-attachments/assets/22ae2300-d5b0-4aba-a07b-4bfcb9be666c" />






# RESULT:
The program is executed successfully.
