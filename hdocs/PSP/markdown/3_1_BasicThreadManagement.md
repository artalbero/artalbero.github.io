# Basic Thread Management 
# (PART I)

---
## 0. Threads in Java
Every Java application is a thread. Therefore, Java applications work with threads everytime.

---

## 1. Thread States (I)
<img src="https://nachoiborraies.github.io/java/img/14_statesChanges.png"
     alt="Threads" />

---

## 1. Thread States (II)
- **New**: the process or thread has been just created, but it has not started running yet.
- **Ready**: the process or thread is not currently running, but it is ready to do it.
- **Asleep**: the thread has fallen asleep due to a call to the sleep method that we will see later. As soon as the sleep time expires, it will come back to the Ready state.

---

## 1. Thread States (III)
- **Waiting**: the thread is waiting for other thread to reactivate it. It happens when threads are fighting for limited resources, and the one who gets them is in charge of notifying the others when it has finished. We will also see this feature in other documents in this unit.
- **Running**: the process or thread is currently running on a given core or processor.

---

## 1. Thread States (IV)
- **Locked**: the process or thread is waiting for some event to happen. This event may be a user input, a file unlocking... Whenever this event happens, the process becomes ready.
- **Finished**: the process or thread has finished its task, or the operating system has forced it to finish with an interruption.

---

## 2. Basic Thread handling.
## 2.1 Defining a thread 
If we want to define a thread, we have some ways to do it:
- Inheriting from Thread class
- Implementing Runnable interface

---

### 2.1.1 Inheriting from Thread class
```java
public class MyThread extends Thread
{
    ... // Attributes, constructors and methods of our class
 
    @Override
    public void run()
    {
        // Code to be executed by the thread
    }
}
```

---

### 2.1.1 Implementing Runnable interface
```java
public class MyOtherThread implements Runnable
{
    ... // Attributes, constructors and methods of our class
    @Override
    public void run() 
    {
        // Code to be executed by the thread
    }
}
```
In this last case, we can also use an anonymous class or a lambda expression to define the Runnable object.
```java
Runnable lambdaRun = () -> {
    // Code to be executed by the thread
};
```

---

## 2.2 Creating and launching a Thread
Using the class extended from Thread:
```java
Thread t = new MyThread();
t.start();
```
Using the class that implements Runnable:
```java
Thread t = new Thread(new MyOtherThread());
t.start();
```
Using the lambda expression (stored in a variable):
```java
Thread t = new Thread(lambdaRun);
t.start();
```

---

## 2.3 Extending *Thread* or implementing *Runnable*?
We have two ways of doing the same thing, but each one has its own flavour:
- Extends Thread: 
    - You can't extend from any other class. This option is usual in small, simple applications.
- Implements Runnable:
    - You can extend from other classes. This option is more usual in complex applications.

---

## 2.4 Example (I)
To start with something simple, we are going to create a thread that counts from 1 to 10. As we do not need to extend from any other class, we are going to create a Thread subclass. 
```java
public class MyCounterThread extends Thread 
{
    @Override
    public void run 
    {
        for (int i = 1; i <= 10; i++)
            System.out.println("Counting " + i);
    }
}
```

---

## 2.4 Example (II)
Our main program looks like this:
```java
public class MyMainCounter 
{
    public static void main(String[] args) 
    {
        MyCounterThread t = new MyCounterThread();
        t.start();
    }
}
```

---

### 2.4 Example (III)
```java
public class MyMainCounter 
{
    public static void main(String[] args) 
    {
        MyCounterThread t = new MyCounterThread();
        t.start();
        System.exit(0); //New line
    }
}
```
If you run the program multiple times you will notice:
- It will not always count to 10!

This is because main program is finished unexpectedly and then all its threads are killed. So our thread only can count until its father is killed.

---

### 2.4 Example (IV)
```java
public class MyMainCounter 
{
    public static void main(String[] args) 
    {
        MyCounterThread t = new MyCounterThread();
        t.start();
        System.out.println("Hello!!"); //New Line
    }
}
```
If you run the program multiple times you will notice:
- The word "Hello!!" can appear in any place in the count.

This happens because the System.out instruction can be called before the thread ends.

---

### 2.4 Example (V)
Finally, we try to start the same process twice:
```java
public class MyMainCounter 
{
    public static void main(String[] args)
    {
        MyCounterThread t = new MyCounterThread();
        t.start();
        t.start();
    }
}
```
But an IllegalThreadException is thrown. We can't start the same thread more than once. We need two different Thread objects of the same class to do so.

---

### 2.5 Conclusions
- The thread runs independent from its father.
- The thread keeps running until its father finishes.
- If any thread call the System.exit method, all the threads will finish their execution.
- There is no way to know the exact order in which the main application and the threads will produce their results.
- After launching a thread, we cannot call its start method again.

---

### Do Exercise 1 and Exercise 2 of [this document](https://nachoiborraies.github.io/java/md/en/14c)
