# Basic Thread Management 
# (PART III)

---

## 6. Thread groups and daemons

---

## 6.1. Thread groups (1)
We can create ThreadGroups to group several threads so we can treat them as a unit.
We can create a group with a name:
```java
ThreadGroup g1 = new ThreadGroup("Main group");
```
Or even create a groupt inside another group like this:
```java
ThreadGroup g2 = new ThreadGroup(g1, "Additional group inside main group");
```

---

## 6.1 Thread groups (2)

To add threads to a group, we can use some of the constructors available in Thread class. For instance, if we create a thread by extending Thread class, we can add it to a group with this constructor (and some others, check the API for more details):
```java
public Thread(ThreadGroup group, String name);
```
If we created the thread by implementing Runnable interface, we can add it with these constructors (and some others, check the API for more details):
```java
public Thread(ThreadGroup group, Runnable target);
public Thread(ThreadGroup group, Runnable target, String name);
```

---

Once we have added the threads to a group, there are some useful methods inside ThreadGroup class, such as:

* **activeCount:** returns how many threads in this group (and its subgroups) are currently active (not finished)
* **enumerate(Thread[] array):** copies into the specified array every active thread of the group (and its subgroups)
* **interrupt:** interrupts all threads in the group.
* **setMaxPriority / getMaxPriority:** sets/gets the maximum priority of the threads in the group.

---

### Example of thread groups (1)
The following example creates some threads from a class that implements Runnable interface. These threads are supposed to generate a random number between 1 and 10, sleep the number of seconds specified by this random number, and then print a message in the screen. But as soon as the first thread finishes its task, the whole group is interrupted.

---

### Example of thread groups (2)
The code for the Runnable object is:
```java
import java.util.Random;
import java.util.concurrent.TimeUnit;

public class MyRandomMessage implements Runnable
{
    Random r = new Random (System.currentTimeMillis());
    @Override
    public void run()
    {
        int time = r.nextInt(10) + 1;
        try
        {
            TimeUnit.SECONDS.sleep(time);
            System.out.println("Thread waited " + time + 
                " seconds and finished.");
        } catch (Exception e) {} 
    }
}
```

---
### Example of thread groups (3)
The code for the main is:
```java
public static void main(String[] args)
{
    ThreadGroup g = new ThreadGroup("Random messages");
    MyRandomMessage m = new MyRandomMessage();
    Thread t1 = new Thread(g, m);
    Thread t2 = new Thread(g, m);
    Thread t3 = new Thread(g, m);
    t1.start();
    t2.start();
    t3.start();

    while (g.activeCount() == 3)
    {
        try
        {
            Thread.sleep(100);
        } catch (Exception e) {}
    }
    g.interrupt();
}
```

---

### Example of thread groups (4)
As soon as one thread finishes, activeCount method will return a number lower than 3, and main thread will finish its loop and interrupt all the threads. If the other threads are still waiting for its time to expire, they will be interrupted, an exception will be thrown and they will not print their finish message.

---

### 6.2. Daemon threads (1)
A daemon thread is a special type of thread that executes a periodic task from time to time.
* They have very low priority (i.e., they run when no other “normal” thread needs to run).
* Main program does not wait for them to finish so, if the main program ends, the daemon will be ended as well, even if it has not finished yet.

For this reasons, a daemon should not do any critical task. A good example of daemon is the Java Garbage Collector.

---

### 6.2 Daemon threads (2)
To create a daemon thread, we only have to call the setDaemon method from Thread class before starting the thread:
```java
Thread t = new MyThread();
t.setDaemon(true);
t.start();
```
We can also use the isDaemon method from Thread class to check if a given thread is a daemon or not.

---

## 7. Threads, context and shared data. Exercise:
Copy the following code:
```java
public class ContextExample implements Runnable 
{
    // Reference to current thread
    Thread t;

    public void start2Threads() 
    {
        // Create first thread
        t = new Thread(this);
        t.start();

        // Sleep for 5 seconds
        try 
        {
            Thread.sleep(5000);
        } catch (InterruptedException e) { }

        // Create second thread
        t = new Thread(this);                  // Line #1
        t.start();

        // Sleep for 5 seconds
        try 
        {
            Thread.sleep(5000); 
        } catch (InterruptedException e) { }

        // Destroy thread
        t = null;                              // Line #2
    }

    @Override
    public void run() 
    { 
        // Take initial time in milliseconds
        long ini = System.currentTimeMillis();
        while (t == Thread.currentThread()) 
        {
            System.out.println("Running thread (" + ini + ") ");
            // Sleep for 100 ms
            try 
            {
                Thread.sleep(100);
            } catch (InterruptedException e) { } 
        }
        System.out.println("Finishing thread (" + ini + ") ");
    }

    public static void main(String[] args) 
    {
        ContextExample t = new ContextExample();
        t.start2Threads();
    }
}
```

---

## Answer the following questions about the code:

* What does the while condition of run method do?

* Can there be two threads executing their run methods at the same time?

---

## Answer the following questions about the code:

* If the answer to previous question is yes, could those threads come into conflict with variable ini, so that one thread overwrites the value previously written by the other?

* How can we stop a thread in this example without creating a new one?

Some lines are numbered to give you a hint. You can check the answers in [this document](https://nachoiborraies.github.io/java/md/en/14c).

---

## 7.1. Conclusions (1)
After testing this example, we can come to some conclusions:

* Every attribute of the same object is shared among all the threads of our application. That is why, when main program changes the value of t attribute in previous example, both threads see that change.
* If we call a method multiple times, its local variables are different in each call (they are not shared). That is why each thread on previous example has its own ini value.

---

## 7.1. Conclusions (2)
* If we create a Thread subclass instead of implementing Runnable interface, each attribute of this subclass is not shared among threads, since we instantiate every thread and thus we create its own memory space, as we do with every instantiated object.

---

## 7.2. ThreadLocal variables (3)
For instance, if we define this class:

```java
public class MyThread extends Thread 
{
    int num;

    public MyThread(int num) 
    {
        this.num = num;
    }
    ...
}
```

---

## 7.1 Conclusions (4)
then attribute num will be different for every instantiated thread. So if we type something like this:
```java
MyThread t1 = new MyThread(10);
MyThread t2 = new MyThread(20);
```
Then object t1 will have its num attribute with value 10, and t2 will have it with value 20.

---

## 7.2. ThreadLocal variables (1)
If we ned an attribute that is not shared among threads, we can use the ThreadLocal class, that lets us specify a data type to create an attribute of this type, and create multiple values of this attribute, each one assigned to a different thread.

---

## 7.2. ThreadLocal variables (2)
For instance, if we want our threads to have their own creation date, we will do something like this:
```java
public class MyRunnableClass implements Runnable
{
    private static ThreadLocal<LocalDate> creationDate = new ThreadLocal<LocalDate>()
    {
        protected LocalDate initialValue()
        {
            return new LocalDate.now();
        }
    }
}
```

---

## 7.2. ThreadLocal variables (3)
If we want to get the value of this attribute for each thread, we will call its get method, and if we want to assign a new value, we will call its set method. The initialValue method in the code above is executed when the attribute has no value and the thread is trying to get it. There is also a remove method that we can use to remove the value of this attribute from current thread.

---

## 7.2. ThreadLocal variables (4)
Then, we can have a run method like this in our MyRunnableClass class:
```java
@Override
public void run()
{
    System.out.println("This thread was created on " + creationDate.get());
    System.out.println("Updating creation date...");
    creationDate.set(LocalDate.now());
    System.out.println("Now the creation date is " + creationDate.get()); 
    System.out.println("Removing value...");
    creationDate.remove();
    System.out.println("Now the creation date is " + creationDate.get()); 
}
```

---

## 7.2. ThreadLocal variables (5)
The previous will show three different dates for the same thread, one for each call to get, since we set a new value between the first and the second call, and we remove the value after this second call.

