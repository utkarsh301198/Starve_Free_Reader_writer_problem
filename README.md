# Starve_Free_Readers_writers_problem
Here is the complete design detail that one can analyse.
## Soltuion (Starve Free)
The traditional solution to the problem results in the starvation of either the reader or the writer. In this solution, I attempted to suggest a starvation-free solution. Just one assumption is made when proposing the solution: Semaphore maintains the first in first out (FIFO) order while locking and releasing processes.( Semaphore uses a FIFO queue to maintain the list of blocked processes)
### Semaphore
Designing a Semaphore with FIRST-IN-FIRST-OUT(FIFO) queue to maintain the list of blocked processes
```cpp
// The code for a FIFO semaphore.
struct Semaphore{
  int val = 1;
  FIFO_Queue* Qf = new FIFO_Queue();
}
    
void wait(Semaphore *Sm,int* process_id){
  Sm->val--;
  if(Sm->val < 0){
  Sm->Qf->push(process_id);
  block(); //this function will block the process by using system call and will transfer it to the waiting queue
           //the process will remain in the waiting queue till it is waken up by the wakeup() system calls
           //this is a type of non busy waiting
  }
}
    
void signal(Semaphore *Sm){
  Sm->val++;
  if(Sm->val <= 0){
  int* PID = Sm->Qf->pop();
  wakeup(PID); //this function will wakeup the process with the given pid using system calls
  }
}


//The code for the queue which will allow us to make a FIFO semaphore.
struct FIFO_Queue{
    ProcessBlock* front, rear;
    int* pop(){
        if(front == NULL){
            return -1;            // Error : underflow.
        }
        else{
            int* value = front->val;
            front = front->next;
            if(front == NULL)
            {
                rear = NULL;
            }
            return value;
        }
    }
    void* push(int* value){
        ProcessBlock* blck = new ProcessBlock();
        blck->val = value;
        if(rear == NULL){
            front = rear = n;
            
        }
        else{
            rear->next = blck;
            rear = blck;
        }
    }
    
}

// A Process Block.
struct ProcessBlock{
    ProcessBlock* next;
    int* process_block;
}
```
### Intialization
In this solution we have used 3 semaphores. 'turn' semaphore is used to specific whose chance is it next to enter the critical section.  The process holding this semaphore gets the next chance to enter the critical section.'rwt' semaphore is the semaphore which is required to access the critical section.'r_mutexs' semaphore is required to change the 'read_count' variable which maintain the number of active readers.
```cpp
int read_count = 0;                      //Integer representing the number of reader executing critical section
Semaphore turn = new Semaphore();        //seamphore representing the order in which the writer and 
                                         //reader are requesting access to critical section
Semaphore rwt = new Semaphore();         //seamphore required to access the critical section
Semaphore r_mutex = new Semaphore();     //seamphore required to change the read_count variable
```
### Reader's Code
```cpp
do{
/* ENTRY SECTION */
       wait(turn,process_id);              //waiting for its turn to get executed
       wait(r_mutex,process_id);           //requesting access to change read_count
       read_count++;                       //update the number of readers trying to access critical section 
       if(read_count==1)                   // if I am the first reader then request access to critical section
         wait(rwt);                        //requesting  access to the critical section for readers
       signal(turn);                       //releasing turn so that the next reader or writer can take the token
                                           //and can be serviced
       signal(r_mutex);                    //release access to the read_count
/* CRITICAL SECTION */
       
/* EXIT SECTION */
       wait(r_mutex,process_id)            //requesting access to change read_count         
       read_count--;                       //a reader has finished executing critical section so read_count decrease by 1
       if(read_count==0)                   //if all the reader have finished executing their critical section
        signal(rwt);                       //releasing access to critical section for next reader or writer
       signal(r_mutex);                    //release access to the read_count  
       
/* REMAINDER SECTION */
       
}while(true);
```
### Writer's code
```cpp
do{
/* ENTRY SECTION */
      wait(turn,process_id);              //waiting for its turn to get executed
      wait(rwt,process_id);               //requesting  access to the critical section
      signal(turn,process_id);            //releasing turn so that the next reader or writer can take the token
                                          //and can be serviced
/* CRITICAL SECTION */

/* EXIT SECTION */
      signal(rwt)                         //releasing access to critical section for next reader or writer

/* REMAINDER SECTION */

}while(true);
```
## Analysis of Correctness of Solution

### Bounded Waiting
Before accessing the critical section any reader or writer have to first acquire the 'turn' semaphore in which a FIFO queue is used for the blocked processes. Thus as the queue uses a FIFO principle , every process has to wait for a finite amount of time before it can access the critical section thus meeting the requirement of bounded waiting.

### Mutual Exclusion principle
If we talk about rwt semaphore, it ensures that only a single writer can access the critical section at any moment of time which in turn ensuring mutual exclusion between the writers. Also when the first reader try to access the critical section it has to acquire the rwt mutex lock to access the critical section thus ensuring mutual exclusion between the readers and writers as well.

### Progress Requirement
The code is structured so that there are no chances for deadlock and also the readers and writers takes a finite amount of time to pass through the critical section and also at the end of each reader writer code they release the semaphore for other processes to enter into critical section.

## Conclusion 
None of these ideas are feasible, and our approach addresses all of their shortcomings. However, there are recent solutions to a broader issue, the community mutual exclusion problem. Our solution is a more straightforward solution to a more straightforward problem (the readers-writers problem versus the group mutual exclusion problem).

## References
- Abraham Silberschatz, Peter B. Galvin, Greg Gagne - Operating System Concepts
- Operating Systems Concepts (9th Edition), Abraham Silberchatz, Peter B Galvin and Greg Gagne
- [Wikipedia](https://en.wikipedia.org/wiki/Readers%E2%80%93writers_problem)
