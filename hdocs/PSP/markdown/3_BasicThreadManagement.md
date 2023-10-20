# Basic Thread Management

---
## 0. Threads in Java
Every Java application is a thread. Therefore, Java applications work with threads everytime.

---

## 1. Thread States (I)
![](/artalbero.github.io/hdocs/PSP/img/3_/14_statesChanges.png)

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