

## 6. CompletableFuture

### 6.1 자바 Concurrent 프로그래밍 소개

> 200930 (Wed)

* Concurrent 개념 및 Java Thread 사용법에 대한 기초 개념 학습

---

### 6.2 Executors

> 200930 (Wed)

* executorService.shutdown()

``` java
ExecutorService executorService = Executors.newSingleThreadExecutor();
executorService.execute(() -> {
    try {
        Thread.sleep(300L);
        System.out.println("Thred " + Thread.currentThread().getName());
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
});

executorService.shutdown(); // Thread를 Graceful하고 종료시킨다.

## Output
Thred pool-1-thread-1
```


---

* executorService.shutdownNow()


``` java
ExecutorService executorService = Executors.newSingleThreadExecutor();
executorService.execute(() -> {
    try {
        Thread.sleep(300L);
        System.out.println("Thred " + Thread.currentThread().getName());
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
});

executorService.shutdownNow(); // 강제로 Thread를 죽인다.

## Output
java.lang.InterruptedException: sleep interrupted
	at java.base/java.lang.Thread.sleep(Native Method)
	at be.goodgid.GoodgidApplication.lambda$main$0(GoodgidApplication.java:16)
	at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1128)
	at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:628)
	at java.base/java.lang.Thread.run(Thread.java:834)
```

* 강제로 Thread를 종료시키기 때문에 

  InterruptedException이 발생된다.

---

* Executors.newFixedThreadPool()

  사용하고자 하는 Thread Count 지정이 가능하다.


``` java
public static void main(String[] args) {
    ExecutorService executorService = Executors.newFixedThreadPool(3);
    executorService.submit(getRunnable("1"));
    executorService.submit(getRunnable("2"));
    executorService.submit(getRunnable("3"));
    executorService.submit(getRunnable("4"));
    executorService.submit(getRunnable("5"));
    executorService.submit(getRunnable("6"));
    executorService.shutdown();
}

private static Runnable getRunnable(String message) {
    return () -> {
        System.out.println(message + " " + Thread.currentThread().getName());
    };
}

## Output
2 pool-1-thread-2
3 pool-1-thread-3
1 pool-1-thread-1
4 pool-1-thread-3
5 pool-1-thread-3
6 pool-1-thread-3
```

---

* Executors.newSingleThreadScheduledExecutor() && schedule()

* Scheduled 설정이 가능하다.

  *schedule(Runnable command, long delay, TimeUnit unit)*

``` java
public static void main(String[] args) {
    ScheduledExecutorService executorService = Executors.newSingleThreadScheduledExecutor();
    executorService.schedule(getRunnable("Hello"), 3, TimeUnit.SECONDS);
    executorService.shutdown();
}

private static Runnable getRunnable(String message) {
    return () -> {
        System.out.println(message + " " + Thread.currentThread().getName());
    };
}

## Output
// After 3 Seconds
Hello pool-1-thread-1
```


---

* Executors.newSingleThreadScheduledExecutor() && scheduleAtFixedRate()

* Scheduled 설정이 가능하다.

  *scheduleAtFixedRate(Runnable command,long initialDelay, long period, TimeUnit unit)*

``` java
public static void main(String[] args) {
    ScheduledExecutorService executorService = Executors.newSingleThreadScheduledExecutor();
    executorService.scheduleAtFixedRate(getRunnable("Hello"), 3, 5, TimeUnit.SECONDS);
}

private static Runnable getRunnable(String message) {
    return () -> {
        System.out.println(message + " " + Thread.currentThread().getName());
    };
}

## Output
// After 3 Seconds
Hello pool-1-thread-1
// After 5 Seconds
Hello pool-1-thread-1
// After 5 Seconds
Hello pool-1-thread-1
```

* 지속적으로 Thread를 실행시켜야하므로

  executorService.shutdown()를 지운다.


---


### 6.3 Callable과 Future

> 200930 (Wed)


* Callable : Runnable과 같은데 Return 값을 갖을 수 있다.

* Future : 비동기적인 작업의 현재 상태를 조회하거나 결과를 가져올 수 있다.

---

* Check Point

1. Future 사용

2. 작업 상태 확인 = isDone()

``` java
public static void main(String[] args) {
    ScheduledExecutorService executorService = Executors.newSingleThreadScheduledExecutor();

    Callable<String> callable = () -> {
        Thread.sleep(3000L);
        return "hello";
    };

    Future<String> future = executorService.submit(callable);

    System.out.println("Start");
    System.out.println("is done? : " + future.isDone());

    try {
        String s = future.get();
        System.out.println(s);
    } catch (InterruptedException e) {
        e.printStackTrace();
    } catch (ExecutionException e) {
        e.printStackTrace();
    }

    System.out.println("is done? : " + future.isDone());
    System.out.println("End");

    executorService.shutdown();
}

## Output
is done? : false
Start
hello
is done? : true
End
```

---


* 작업 취소하기 = cancel()

``` java
// Method Signature : cancel(boolean mayInterruptIfRunning)
future.cancel(false);
```

* cancel() 이후 get()을 호출하면

  CancellationException이 발생한다.


---

* invokeAll()

  동시에 실행한 작업 중 가장 오래 걸리는 시간만큼 걸린다.

  즉 모든 작업이 동시에 종료된다.

``` java
public static void main(String[] args) throws ExecutionException, InterruptedException {
    ExecutorService executorService = Executors.newFixedThreadPool(5);

    Callable<String> callable_1 = () -> {
        Thread.sleep(1000L);
        return "callable_1";
    };

    Callable<String> callable_2 = () -> {
        Thread.sleep(2000L);
        return "callable_2";
    };

    Callable<String> callable_3 = () -> {
        Thread.sleep(3000L);
        return "callable_3";
    };

    List<Future<String>> futures = null;

    try {
        futures = executorService.invokeAll(List.of(callable_1, callable_2, callable_3));
    } catch (InterruptedException e) {
        e.printStackTrace();
    }

    for (Future<String> f : futures) {
        System.out.println(f.get());
    }

    executorService.shutdown();
}

## Output
// After 3 Seconds
callable_1
callable_2
callable_3
```


---


* invokeAny()

  동시에 실행한 작업 중 가장 짧게 끝나는 시간만큼 걸린다.

``` java

public static void main(String[] args) throws ExecutionException, InterruptedException {
    ExecutorService executorService = Executors.newFixedThreadPool(5);

    Callable<String> callable_1 = () -> {
        Thread.sleep(5000L);
        return "callable_1";
    };

    Callable<String> callable_2 = () -> {
        Thread.sleep(8000L);
        return "callable_2";
    };

    Callable<String> callable_3 = () -> {
        Thread.sleep(9000L);
        return "callable_3";
    };

    String anyCallable = null;

    try {
        anyCallable = executorService.invokeAny(List.of(callable_1, callable_2, callable_3));
    } catch (InterruptedException e) {
        e.printStackTrace();
    }

    System.out.println(anyCallable);

    executorService.shutdown();
}

## Output
// After 1 Seconds
callable_1
```

---

### 6.4 CompletableFuture 1

### 6.5 CompletableFuture 2

> 200930 (Wed)

* 비동기 프로그래밍에 가까운 코딩이 가능하다.

* CompletableFuture 사용법 익히고 싶을 때 다시 보자.
