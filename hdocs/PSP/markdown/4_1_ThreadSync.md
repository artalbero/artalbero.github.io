# 4. Thread Syncronization and coordination
# (Part I)

---

## 1. Basic coordination. Joining threads (1)
The join method is used to wait for another thread. In this example, the main application creates a thread and waits until it finishes before going on:
```java
public static void main(String[] args) 
{
    public static void main(String[] args) 
{
    Thread t = new MyThread();
    t.start();
    try 
    {
        t.join(); //we have to catch the possible exception
    } catch (InterruptedException e) {
        ...
    }
}
}
```

---

## 1. Basic Coordination. Joining threads (2)
If we need a secondary thread to wait for another thread, we have to tell which thread is the latter. To manage this, we usually use an attribute.
```java
public class MyThread extends Thread 
{
    Thread waitThread;

    // We will use this constructor 
    // if thread does not have to wait for anyone
    public MyThread() 
    {
        waitThread = null;
    }

    // We will use this constructor 
    // if thread has to wait for thread "wt"
    public MyThread(Thread wt) 
    {
        waitThread = wt;
    }

    // We check if waitThread attribute is not null, 
    // and then call the join method before keep on running
    public void run() 
    {
        if (waitThread != null)
            waitThread.join();
        ...
    } 
}
```

---

## 1. Basic Coordination. Joining threads (3)
Then, in main application, we create two threads of type MyThread, and ask one of them to wait for the other:
```java
public static void main(String[] args) 
{
    Thread t1 = new MyThread();
    Thread t2 = new MyThread(t1);
    t1.start();
    // We start thread t2, but it will not run until t1 finishes
    t2.start();
}
```

---

## Do exercises 1 and 2 from [this document](https://nachoiborraies.github.io/java/md/en/14d)

---

## 2. Access to shared resources. The need of thread synchronization (1)
- It is quite usual that multiple threads want to get the same resource.
- The piece of code that is in charge of allowing threads to get that shared resource is commonly called critical section. 
- This code must not be executed by more than one thread at the same time. 

---

## 2. Access to shared resources. The need of thread synchronization (2)
To show this behaviour, we are going to create a class with one value which is going to be shared amongst some threads.
```java
public class Counter
{
    int value;

    public Counter(int value)
    {
        this.value = value;
    }

    public void increment()
    {
        value++;
    }

    public void decrement()
    {
        value--;
    }

    public int getValue()
    {
        return value;
    }
}
```

---

## 2. Access to shared resources. The need of thread synchronization (3)
Then, we are going to create two kinds of threads, one to increment the counter and another to decrease it.
```java
public static void main(String[] args)
{
    Counter c = new Counter(100);

    Thread tinc = new Thread(() -> {
        for (int i = 0; i < 100; i++)
        {
            c.increment();
            try 
            {
                Thread.sleep(100);
            } catch (InterruptedException e) { }
        }
        System.out.println("Finishing inc. Final value = "+c.getValue());
    });

    Thread tdec = new Thread(() -> {
        for (int i = 0; i < 100; i++)
        {
            c.decrement();
            try
            {
                Thread.sleep(100);
            } catch (InterruptedException e) { }
        }
        System.out.println("Finishing dec. Final value = "+c.getValue());
    });
    tinc.start();
    tdec.start();
}
```

---


## 2. Access to shared resources. The need of thread synchronization (3)
When we run this code, we can see it doesn't work as expected (having the counter always with value = 100), because sometimes they will access to Counter at the same time and one of the operations will be lost.

There are some mechanisms in JAVA designed to solve this problems of communication using the keyword **synchronized**:
- Synchronizing methods.
- Synchronizing objects.

---

## 2.1. Synchronizing methods
We can use the keyword synchronized on the methods. This way, if one of the methods is accessed by a thread, the other threads will not be able to use any of the synchronized methods. However, the program will perform slower.
```java
public class Counter 
{
    int value;

    public Counter(int value) 
    {
        this.value = value;
    }

    public synchronized void increment() 
    {
        value++;
    }

    public synchronized void decrement() 
    {
        value--;
    }

    public int getValue() 
    {
        return value;
    }
}
```

## 2.2. Synchronizing objects (1)
We can also apply the synchronized keyword to an object, passing it as a parameter:
```java
public void myMethod() 
{
    int someValue;
    ...
    synchronized(this) 
    {
        someValue++;
        System.out.println("Value changed: " + someValue);
    }
    ...
}
```
Then, when a thread A tries to execute the instructions inside this block, it will not be able to do it if another thread B is already executing a critical section affecting the object this. As soon as this thread B finishes the critical section, the other thread A will wake up and enter the critical section.

---

## 2.2 Synchronizing objects (2)
We can use any other object with synchronized keyword. For instance, if we have an object called file and we want to create a critical section around it, we can do it like this:
```java
public void someMethod() 
{
    ...
    synchronized(file) 
    {
        ... // Critical section
    }
    ...
}
```

---

## Do exercises 3 and 4 from [this document](https://nachoiborraies.github.io/java/md/en/14d) 

