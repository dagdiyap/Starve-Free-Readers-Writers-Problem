# Starve Free Readers-Writers Problem
Problem deals with multiple processes reading from and writing to a shared resource synchronously where neither of them is starved for indefinate time. 

## Solution
Solution consists of pseudocodes for the problem and all of them are in c++ syntax.

We design a semaphore that handles processes in a First-In-First-Out order. The processes are given access to the semaphore in the order they called wait. Each process gets blocked and pushed to the queue when it calls wait and is then woken up when it reaches the head of the queue. The following are the pseudo code for the FIFO queue and the semaphore that utilizes the queue for resource allocation. 

```cpp

//This represents a process in the FIFO queue. 
Struct Proc {
    Proc* next;
    int process_id;
}

//Implementation of a FIFO queue of Proc nodes. 
struct Queue {
    Proc* head, back;
    
   	void push(int pid) {
        Proc* proc = new Proc();
        proc->process_id = pid;
        if(back == NULL) {
            head = proc;
            back = proc; 
        } else {
            back->next = proc;
            back = proc;
        }
    }
    
    int pop() {
        if(head == NULL) {
            return -1; // underflow 
        } else {
            int pid = head->process_id;
            head = head->next;
            if(head == NULL) {
                back = NULL;
            }
            
            return pid;
        }
    }
}

struct Semaphore {
    int value;
    Queue* queue = new Queue();
    
    void wait(int pid) {
        value--;
        if(value < 0) {
            queue->push(pid);
            block(pid); //block the process until wake is called. 
        }
    }
    
    void signal() {
        value++;
        if(value <= 0) {
            int pid = queue->pop();
            wake(pid); //wake the process at the head of the queue. 
        }
    }
}
```

We declare the following global variables

```cpp 
Semaphore* in, out, write; 
in->value = 1; 
out->value = 1; 
write->value = 0; 

int in_ctr = 0; // #readers started reading
int out_ctr = 0; // #readers completed reading
bool wait_write = false; // true if a writer is waiting 
```

Then, the reader and writer processes are as follows  
Reader Process
```cpp 
// Assume the process_id is the process id
in->wait(process_id); // Call wait 
in_ctr++;             // increment in_ctr as the process starts reading
in->signal();         

//Read the data(Critical section)

out->wait(process_id); // wait on the out semaphore 
out_ctr++;             // increment out_ctr after reading complete
if(in_ctr == out_ctr && wait_write) {   // if writer is waiting and 
    write->signal();                    // this is last reader then 
}                                       // signal write semaphore
out->signal(); 
```

Writer Process
```cpp 
//Assume process_id is the process id
in->wait(process_id); 
out->wait(process_id); 
if(in_ctr == out_ctr) {   // if no processes are currently reading
    out->signal(); 
} else {                  // wait for readers to finish
    wait_write = true; 
    out->signal(); 
    writer->wait(); 
    wait_write = false; 
}

//Write the data(Critical section)

in->signal(); 
```

## Explanation

### Semaphores

Three semaphores are used namely - 
- mutex_of_r - It provides mutual exclusion to r_count variable which counts number of readers in critical section.
- queue_of_rw - It maintains order of arrival of readers and writers.
- resources - It prevents readers and writers or multiple writers to be present in critical section at same time.

### Shared variables

- r_count - It counts number of readers present in critical section at given time.
- data - It is data which is shared among various threads.

### Reader function

- Entry section - It acquires required semaphores to enter critical section.
Reader tries to acquires queue_of_rw initially, if it is unavailable, reader is added to queue for the given semaphore.
After aquiring queue_of_rw, it tries to aquire mutex_of_r to modify r_count.
If it is the first reader, it tries to aquire resources to confirm there are no writers in the critical section.
It releases queue_of_rw and mutex_of_r before entering critical section.

- Critical section - Reader reads the shared data in this section.

- Exit section - It releases the aquired semaphores.
Reader tries to aquire mutex_of_r to modify r_count.
If it is the last reader, it releases resources.
It releases mutex_of_r at the end.

### Writer function

- Entry section - This section acquires required semaphores to enter critical section.
Writer tries to acquires queue_of_rw  initially, if it is unavailable, writer is added to queue for the given semaphore.
After aquiring queue_of_rw, it tries to aquire resources and enter critical section.
It releases queue_of_rw before entering critical section.

- Critical section - Writer modifies the shared data in this section.

- Exit section - This section releases the aquired semaphores.
Writer releases the resources SEMAPHORE.
