# Basic Thread Management
# (Part II)

---

## 3. Basic Thread information

There are some useful methods and properties in Thread class to get and set some information about a thread. We are going to focus on three of them for now:

- How to set and get the thread’s name
- How to get the thread state
- How to get thread’s identifier

---

### 3.1 Setting and getting the thread name
#### Thread extension (I)
We can add a name attribute to the class, however, we don't need to do so. We can use this methods:
```java
Thread t = new MyCounterThread();
t.setName("MyThread A");
t.start();
System.out.println("Thread " + t.getName() + " has been launched.");
```

---

#### Thread extension (II)
If we want to get/set thread’s name inside the thread itself (for instance, from run method of the thread), we can call currentThread method to get a Thread object that points to current thread, and then get/set its name.

```java
@Override
public void run() 
{
    Thread.currentThread().setName("AAA");
    ...
    System.out.println(Thread.currentThread().getName());
}
```

---

#### Runnable implementation
If you run a thread from a Runnable instance, you can set the name directly when creating the thread as the second parameter in the constructor.

```java
Runnable counterRun = () -> {
    System.out.println(Thread.currentThread().getName() + " running");
    for (int i = 1; i <= 10; i++)
        System.out.println("Counting " + i);
};
Thread t = new Thread(counterRun, "CounterThread");
t.start();
```

---

### 3.2 Getting thread state (I)

We can get the state of the thread at any time:
```java
Thread t = new MyCounterThread();
t.start();
...
Thread.State st = t.getState();
```
What getState method returns can be one of the following states, that are represented by constants in Thread.State enum: NEW, RUNNABLE, BLOCKED, WAITING, TIMED_WAITING or TERMINATED

---

### 3.2 Getting thread state (II)
```java
if (st == Thread.State.TERMINATED)
    System.out.println("Thread is terminated.");
```
Alternatively, we can check if the thread is alive or not:
```java
if (!t.isAlive())
    System.out.println("Thread is terminated.");
```

---

### 3.3. Getting thread’s identifier
Java Virtual Machine assigns a unique identifier to every thread that is created. If we want to get it, we only have to call the getId method from Thread class:
```java
@Override
public void run() 
{
    ...
    System.out.println("Thread #" + Thread.currentThread().getId());
}
```

---

## 4. The sleep and yield methods
- The sleep method can stop the thread by a number of miliseconds. 
- As it is a static method of Thread class, we just need to add the instruction in the position we want it to stop.
- Also, we need to catch an exception using this method.
- The thread that calls the method is the one which is going to sleep.

---

#### Example of sleep
```java
try 
{
    Thread.sleep(2000);
} catch (InterruptedException e) {
    ...
}
```
In the following example, the main thread will go to sleep, not the thread t.
```java
public static void main(String[] args) 
{
    Thread t = new MyThread();
    t.start();
    t.sleep(2000);
}
```

---

#### Time Unit
we can also use TimeUnit class (from java.util.concurrent package) and its properties to specify another time unit, that will be automatically converted to milliseconds. For instance, if we want our thread to sleep 5 seconds, we can also do it like this:
```java
import java.util.concurrent.TimeUnit;
...
try 
{
    TimeUnit.SECONDS.sleep(5);
} catch (InterruptedException e) { ... }
```

---

### 4.2. The yield method
- Similar to sleep, but it does not need a number of time to work.
- It leaves the processor free and the task planner can assign it to another thread.
- If no other thread is waiting for the processor, then the thread that yielded gets it back.
- This method is also static, but it does not throw any exception, you can call it just like this:
```java
Thread.yield();
```
- Unfortunately, the task planner of JVM may ignore this instruction.

---

#### Example
A thread that counts from A to Z, sleeping 100ms after printing each letter. Main program will wait for this thread to finish, checking its state after each iteration.
```java
public static void main(String[] args)
{
    Thread t = new Thread(() -> {
        for (char c = 'A'; c <= 'Z'; c++)
        {
            System.out.println(c);
            try
            {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                System.err.println("Error: Thread interrupted");
            }
        }
    });

    t.start();
    do
    {
        try
        {
            Thread.sleep(100);
        } catch (InterruptedException e) { }
    } while (t.isAlive());
    System.out.println("Thread has finished, and so do I");
}
/* Notice that main program just sleeps a few milliseconds 
(they can be 50, 100, 200… it does not matter) on each 
iteration. It only has to wait for the thread to finish, 
it has nothing to do, so it better leave the processor 
free by sleeping or yielding. */
```

---

## Do Exercise 3 of [this document](https://nachoiborraies.github.io/java/md/en/14c)

---

## 5. Finishing and interrupting Threads
There are two ways of forcing a thread to finish its task: 
* Using **boolean flags** to tell the thread that it must stop when it checks those flags.
* Using **interruptions** to make it stop.

---

### 5.1. Finishing threads with boolean flags (1)
We cannot stop a thread at a given moment. There were a stop and a destroy methods, but now they are deprecated. 
However, we can ask the method to stop through a boolean flag. This method works in threads that run in any kind of loop.

---

### 5.1. Finishing threads with boolean flags (2)
Let's define a Thread class killable through a boolean flag
```java
public class KillableThread extends Thread 
{
    boolean finish = false; //The boolean flag

    public void setFinish(boolean finish)
    {
        this.finish = finish;
    }

    @Override
    public void run() 
    {
        while (!finish) 
        {
            ... // Thread task
        }
    }
}
```

---

### 5.1 Finishing threads with boolean flags (3)
Then we can apply the setFinish method like this:
```java
public static void main(String[] args)
{
    KillableThread kt = new KillableThread();
    kt.start();
    ...
    if (someCondition)
        kt.setFinish(true);
}
```

---

## Do Exercise 4 of [this document](https://nachoiborraies.github.io/java/md/en/14c)

---

## 5.2. Finishing threads with interruptions (1)
We can also use the method .interrupt() from the Thread class.
```java
public static void main(String[] args) 
{
    Thread t = new Thread(() -> {
        try 
        {
            while (!Thread.currentThread().isInterrupted()) 
            {
                System.out.println("Running");
                Thread.sleep(100);
            }
        } catch (InterruptedException e) { }
        System.out.println("Finished by an interruption");
    });

    t.start();
    try 
    { 
        // Wait for a while...
        Thread.sleep(1000);
    } catch (InterruptedException e) { }

    t.interrupt();
}
```

---

## 5.2. Finishing threads with interruptions (2)
In the example: 
* We create a thread that checks in every loop if it has been interrupted.
* From the main thread, we wait 1000 miliseconds and then interrupt the thread with the method.
* This method causes an InterruptedException that makes the thread go to the catch section and finish the run method.

---

## 5.2. Finishing threads with interruptions (3)
* The InterruptedException is only thrown because of the sleep call in Runnable.
* This thread would finish just by calling its isInterrupted method.

---

## 5.2 Finishing threads with interruptions (4)
* However, as we call a method that throws this exception (sleep), we have to use the try...catch structure.
* A thread decides if it responds to the interruption or not, by using its isInterrupted method and/or by catching the possible exceptions that can be thrown.




