# Brief Introduction to Mutual Exclusion
The subsequent posts will be a recap of some parallel computing concepts learnt from [CS4231](https://nusmods.com/courses/CS4231/parallel-and-distributed-algorithms). I highly recommend taking this course
for its interesting content. This is to help improve my understanding of some of the concepts shared during this class. The class covers distributed algorithms too but I feel more inclined to share
a more overlooked part which is the parallel portion of things. 

## Shared Memory System
Suppose there are 2 processes running a system. The code for each process is as follows:
```cpp
void foo() {
  x++;
}
```
`x` is a global variable, shared by bother processes. There are 2 possible outcomes when both processes are done running:
- Increment happens sequentially (ie `x` is increased twice)
- `x` becomes something else

Both outcomes **can be correct**. It heavily depends on the **agreement between the program and the system**.

## Mutual Exclusion
There are 2 main APIs for mutual exclusion:
- `RequestCS` (tries to access a particular shared resource)
- `ReleaseCS` (process is done with shared resource, tries to release it back)
Each of these API requires the process id as the parameter for it. 
The next sections will share various attempts at designing and solving the mutual exclusion problem.

### Attempt 1: Shared Boolean Variable
Suppose that the shared resource is a room with a door. When the door is open, anyone can enter. Upon entering the room, close the door behind them.
When someone leaves the room, he/she opens the door behind them for the next person. With this in mind, the APIs can be designed as such:
```
void requestCS(int pid) {
  while door is close
    keep waiting
  door opens, enter the room
  close the door
}

void releaseCS(int pid) {
  leave the room
  open the door
}
```

This example fails in some cases. To intuitively see why, consider the following:
- Suppose the door is initially open.
- Bob enters the room.
- Before Bob can close the door, Alice enters the room.
- Bob and Alice are now in the room.

### Attempt 2: Shared Boolean Array
Consider the same room from above. Now, instead of checking whether the door is open, Alice and Bob have to raise their hand if they want to enter the room. 
Alice and Bob starts with both hands down. 
```
// Alice code
void requestCS(int pid) {
  raise hand
  while Bob's hand is raised, wait
}

void releaseCS(int pid) {
  put hand down
}

// Bob code
void requestCS(int pid) {
  raise hand
  while Alice's hand is raised, wait
}

void releaseCS(int pid) {
  put hand down
}
```
Obviously from above, it is possible for Alice and Bob to both have their hands raised at the same time. 
This causes both of them to not be able to enter the room. This is known as **deadlock**.

### Attempt 3: Turn-based shared variable
Consider the same room as above. Instead of raising hands, Alice or Bob takes turns to enter instead. 
This is different from the second attempt since this just takes 1 step for either Alice and Bob to enter the room.
Recall in Attempt 2, Alice/Bob have to raise their hand **then** enter the room. 
```
// Alice code
void requestCS(int pid) {
  while its Bob's turn, wait outside
}

void releaseCS(int pid) {
  leave the room and give Bob his turn
}

// Bob code
void requestCS(int pid) {
  while its Alice's turn, wait outside
}

void releaseCS(int pid) {
  leave the room and give Alice his turn
}
```
Suppose Alice enters first. Upon leaving, suppose Bob gets distracted and decides he doesn't want to enter the room anymore. Now Alice will never get her turn back
since Bob is the only one who can give her her turn. Selfish Bob! This is known as **starvation**.

## Conclusion
Clearly from the above examples, mutual exclusion is quite tricky to solve. To solve this problem, alogrithms and APIs must have the following properties:
- Mutual Exclusion: Only 1 person allowed in the room at any one time
- Progress: If the room is empty, someone will eventually enter it
- No starvation: If someone wants to enter the room, he/she will eventually enter it.

Thanks for taking the time to read this! How will you solve this mutual exclusion problem? :-)
