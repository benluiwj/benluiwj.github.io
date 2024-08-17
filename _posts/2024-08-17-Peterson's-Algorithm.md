# Peterson's Algorithm
This post continues from the previous one about mutual exclusion. If you have not read that, you can read it (here)[https://benluiwj.github.io/2024/08/15/mutual-exclusion.html]. 
This post will describe a possible solution to solving the mutual exclusion problem. After sharing the algorithm, I will attempt to prove its correctness as well. 

## The Algorithm
In the real world, it is possible to want something but not actually have it. To solve the mutual exclusion problem, let's set up the parameters. There is a room with 1 door. 
There are 2 people, Alice and Bob, who want exclusive access in the room. For either one to enter the room, they **must have both** of the following:
- it is their turn to enter the room
- they want to enter the room

With that in mind, here is the pseudo code for the algorithm. 
```
// Globals
wantCS = [false, false]
turn = 0

// Alice APIs
requestCS() {
  wantCS[0] = true
  turn = 1
  while (wantCS[1] == true and turn == 1);
}

releaseCS() {
  wantCS[0] = false
}

// Bob APIs
requestCS() {
  wantCS[1] = true
  turn = 0
  while (wantCS[0] == true and turn == 0);
}

releaseCS() {
  wantCS[1] = false
}
```
## Proof outline
To prove the correctness, we need to show that all 3 properties of mutual exclusion holds.
1. Mutual Exclusion
2. Progress
3. No Starvation
Since no starvation implies progress, we will just prove no starvation. The proof for progress will be left as an exercise for the reader :-)

### Mutual Exclusion Proof
We have to show that mutual exclusion holds for 2 cases:
- when `turn = 0`
- when `turn = 1`
The 2 cases are symmetric so we will show the case for `turn = 0` and the other case is similar.
We proof this by contradiction.
Suppose that both Alice and Bob are in the room when `turn = 0`. A possible execution that led to this stage could be as follows:
- Alice wants to enter and sets it to her turn
- Bob wants to enter and set it to his turn as well
The end result of the above execution is `wantCS = [true, true]` and `turn = 0`.
If both Alice and Bob are in the same room, this means that the `while` loop for both of them are false conditions.
In other words, this means that Alice must have seen `turn = 0` to enter and Bob must have seen `turn = 1`.
However, this is not possible since we said that `turn = 0` at the start. So Bob should not have seen `turn = 1`.

### No Starvation
To proof no starvation, we show what happens when both Alice and Bob are waiting to enter the room. Suppose Alice is stuck forever.
If Alice is stuck forever, this means that Bob is in the room. When Bob exits the room, he indicates that he does not want to enter the room by setting `wantCS[1] = false`.
Now, it is possible for the following scenarios to occur:
- Bob does not try to enter again immediately
- Bob tries to enter again immediately
#### Bob doesn't try to enter again immediately
If Bob does not enter, then Alice will enter. This means that Alice will not be stuck forever.
#### Bob tries to enter again immediately
Bob comes back and indicates we wants to enter by setting `wantCS[1] = true`. Since Alice is already waiting on the while condition in her `requestCS`, she cannot change any variables. 
Bob sets `turn = 0` and is then forced to wait because of his while loop conditions. Since `turn  = 0`, Alice's while loop condition breaks and she enters the room.
This contradicts that Alice is stuck forever. 

### Progress
This is left as an exercise for the reader. Some hint to help the reader derive this part of the proof on their own.
Consider 2 cases:
- Nobody in the room and Alice and Bob wants to enter
- Nobody in the room and Alice/Bob wants to enter
Furthermore, consider the value of turn when both Alice and Bob are waiting.

## Conclusion
Peterson's Algorithm is a possible solution to the mutual exclusion problem. 
There is another solution known as Lamport's Bakery Algorithm, which I will share in the next post so stay tune!
