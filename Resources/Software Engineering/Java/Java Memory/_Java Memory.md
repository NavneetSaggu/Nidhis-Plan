# Java Memory

Watch all parts of this [https://www.youtube.com/watch?v=ckYwv4_Qtmo&ab_channel=VirtualPairProgrammers](https://www.youtube.com/watch?v=ckYwv4_Qtmo&ab_channel=VirtualPairProgrammers). This is the best resource for Java Memory.

Also watch this
[https://www.youtube.com/watch?v=TpYIcJN9EV8&t=784s](https://www.youtube.com/watch?v=TpYIcJN9EV8&t=784s)

## Stack

- Every thread has it’s own stack.
- Data from stack can only be seen by thread that owns the stack.
- Java knows exactly when data on a stack can be destroyed
    
    ![Untitled](Java%20Memory/Untitled.png)
    
- Whenever we reach a closing } bracket of a block, any variables declared in the block are popped and destroyed; for eg. when a method returns. 
This is how **scoping** works.
- Stacks are great for local variables, because by definition we want them to have a short lifetime.
- Stores only primitives and references.

## Heap

- To store data that has longer lifetime than a code block or a function for eg. objects to be shared across multiple methods.
- All data outside of stack is on the heap.
- One heap is shared across all the threads; all threads and blocks can potentially access the heap.
- All objects are stored on the heap. Variables are a reference to the object.
    
    ![Screenshot 2022-05-02 at 1.00.08 AM.png](Java%20Memory/Screenshot_2022-05-02_at_1.00.08_AM.png)
    

## Passing By Value vs Reference

- Java only has pass by value
- When objects are passed, their reference is passed by value
- No pass by reference in Java

## Garbage Collection

- When an object is not reachable by any reference from the stack, it is eligible for garbage collection

## Final

- We cannot assign a final variable a new value.
- This means; when a stack variable is assigned an object in the heap, we cannot change which object in the heap this variable points to.
- This, however, does not mean, that we cannot make changes in the object itsel f.
- It’s not possible in Java to fix the state of an object and make it unchangeable. In other languages where this is possible, this feature is known as **Const Correctness**.
- Const Correctness is a powerful concept that helps us write much safer code.

![Screenshot 2022-05-02 at 1.53.46 AM.png](Java%20Memory/Screenshot_2022-05-02_at_1.53.46_AM.png)