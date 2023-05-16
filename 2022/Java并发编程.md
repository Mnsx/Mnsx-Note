# JUC基础

## start线程底层分析

```java
public synchronized void start() {
    /**
         * This method is not invoked for the main method thread or "system"
         * group threads created/set up by the VM. Any new functionality added
         * to this method in the future may have to also be added to the VM.
         *
         * A zero status value corresponds to state "NEW".
         */
    if (threadStatus != 0)
        throw new IllegalThreadStateException();

    /* Notify the group that this thread is about to be started
         * so that it can be added to the group's list of threads
         * and the group's unstarted count can be decremented. */
    group.add(this);

    boolean started = false;
    try {
        start0();
        started = true;
    } finally {
        try {
            if (!started) {
                group.threadStartFailed(this);
            }
        } catch (Throwable ignore) {
            /* do nothing. If start0 threw a Throwable then
                  it will be passed up the call stack */
        }
    }
}
```

```java
private native void start0();
```

* Java线程是通过start方法启动执行的，主要内容在native方法start0中
* openjdk于JNI一般是一一对应的，Thread.java对应的就是Thread.c
* start0其实就是JVM_StartThread，通过操作系统分配一个线程

## Java多线程相关概念

* 一把锁
  * Synchronized
* 两个并
  * 并发（同一时刻其实只有一件事件在发生）
  * 并行（同一时刻多个事件正在发生）
* 三个程
  * 进程
  * 线程
  * 管程

## 用户线程和守护线程

* 用户线程
* 守护线程

```java
public final boolean isDeamon() {
    return deamon;
}
```

```java
public class DaemonDemo {
    public static void main(String[] args) {
        Thread t1 = new Thread(() -> {
            System.out.println(Thread.currentThread().getName() + "\t 开始运行，" + (Thread.currentThread().isDaemon() ? "守护线程" : "用户线程"));
            while (true) {

            }
        }, "t1");
        t1.setDaemon(true); // 必须在start之前设置，不然会返回IllegalThreadStateException
        t1.start();

        try {
            TimeUnit.SECONDS.sleep(3);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println(Thread.currentThread().getName() + "\t-----end 主线程");
    }
}
```

![image-20221015205446442](D:\WorkSpace\Note\Picture\image-20221015205446442.png)

# CompletableFuture

## 基础概念

Future接口定义了操作**异步任务执行**的一些方法

![image-20221015205753802](D:\WorkSpace\Note\Picture\image-20221015205753802.png)

如果主线程需要执行一个**很耗时**的计算任务，我们就可以**通过future把这个任务放到异步线程**中执行

主线程继续处理其他任务或者先行结束，再**通过Future获取计算结果**

## FutureTask异步任务

**目的：异步多线程任务执行且返回有结果，三个特点：多线程/有返回/异步任务**——》底层通过**对象适配器**-FutureTask(Callable callable)

```java
public class CompletableFutureDemo {

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        FutureTask<String> futureTask = new FutureTask<>(new MyThread());

        Thread t1 = new Thread(futureTask, "t1");
        t1.start();

        System.out.println(futureTask.get());
    }
}

class MyThread implements Callable<String> {

    @Override
    public String call() throws Exception {
        System.out.println("-----com in Callable");
        return "hello FutureTask";
    }
}
```

```java
public class FutureThreadPoolDemo {
    public static void main(String[] args) {
        long starTime = System.currentTimeMillis();

        method1(); // 常规操作

//        method2(); // 线程池+异步

        long endTime = System.currentTimeMillis();
        System.out.println("-----costTime:" + (endTime - starTime) + "ms");
        System.out.println(Thread.currentThread().getName() + "\t -----end");
    }

    private static void method2() {
        ExecutorService threadPool = Executors.newFixedThreadPool(3);

        FutureTask<String> futureTask1 = new FutureTask<>(() -> {
            try {
                TimeUnit.MILLISECONDS.sleep(500);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "task1 over";
        });

        threadPool.submit(futureTask1);

        FutureTask<String> futureTask2 = new FutureTask<>(() -> {
            try {
                TimeUnit.MILLISECONDS.sleep(500);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "task1 over";
        });

        threadPool.submit(futureTask2);

        try {
            TimeUnit.MILLISECONDS.sleep(500);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        threadPool.shutdown();
    }

    private static void method1() {
        try {
            TimeUnit.MILLISECONDS.sleep(500);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        try {
            TimeUnit.MILLISECONDS.sleep(500);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        try {
            TimeUnit.MILLISECONDS.sleep(500);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

![image-20221015215931512](D:\WorkSpace\Note\Picture\image-20221015215931512.png)

![image-20221015215937052](D:\WorkSpace\Note\Picture\image-20221015215937052.png)

优点：**使用线程池+异步，程序运行速率显著提升**

缺点：

**get是阻塞方法**，一般建议放在程序最后，一旦调用，只有得到结果，才允许执行后端的程序

```java
public class FutureAPIDemo {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        FutureTask<String> futureTask = new FutureTask<String>(() -> {
            System.out.println(Thread.currentThread().getName() + "\t -----come in");
            try {
                TimeUnit.MILLISECONDS.sleep(5000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "task over";
        });

        Thread t1 = new Thread(futureTask, "t1");
        t1.start();

        // get方法会阻塞
public class FutureAPIDemo {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        FutureTask<String> futureTask = new FutureTask<String>(() -> {
            System.out.println(Thread.currentThread().getName() + "\t -----come in");
            try {
                TimeUnit.MILLISECONDS.sleep(5000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "task over";
        });

        Thread t1 = new Thread(futureTask, "t1");
        t1.start();

        // get方法会阻塞
        // System.out.println(futureTask.get());
        System.out.println(futureTask.get(3, TimeUnit.SECONDS));

        System.out.println(Thread.currentThread().getName() + "\t -----其他任务正在执行。。。");
    }
}

        System.out.println(Thread.currentThread().getName() + "\t -----其他任务正在执行。。。");
    }
}
```

![image-20221016191713873](D:\WorkSpace\Note\Picture\image-20221016191713873.png)

**轮询的方式会消耗无谓的CPU资源，而且也不见得能及时地得到计算结果**

## CompletableFuture

CompletableFuture = Future + CompletionStage

**CompletionStage代表异步计算过程中的某一个阶段，一个阶段完成以后会触发另外一个阶段**

![image-20221016194845593](D:\WorkSpace\Note\Picture\image-20221016194845593.png)

**四个静态方法，创建CompletableFuture**

* runAsync 无返回值

  * `public static CompletableFuture<Void> runAsync(Runnable runnable)`

  * `public static COmpletableFuture<Void> runAsync(Runnable runnable, Executor executor)`

* supplyAsync 有返回值

  * `public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier)`
  * `public static <U> 0CompletableFuture<U> supplyAsync(Supplier<u> supplier, Executor executor)`

上述Executor参数说明

没有指定Executor的方法，直接使用**默认的ForkJoinPool.commonPool()**

```java
public class CompletableFutureBuildDemo {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        CompletableFuture<Void> voidCompletableFuture = CompletableFuture.runAsync(() -> {
            System.out.println(Thread.currentThread().getName());
            try {
                TimeUnit.MILLISECONDS.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });

        System.out.println(voidCompletableFuture.get());
    }
}
```

![image-20221016201205762](D:\WorkSpace\Note\Picture\image-20221016201205762.png)

```java
public class CompletableFutureBuildDemo {
    public static void main(String[] args) throws ExecutionException, InterruptedException {

        ExecutorService threadPool = Executors.newFixedThreadPool(3);

        CompletableFuture<Void> voidCompletableFuture = CompletableFuture.runAsync(() -> {
            System.out.println(Thread.currentThread().getName());
            try {
                TimeUnit.MILLISECONDS.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, threadPool);

        System.out.println(voidCompletableFuture.get());

        threadPool.shutdown();
    }
}
```

![image-20221016201325733](D:\WorkSpace\Note\Picture\image-20221016201325733.png)

```java
public class CompletableFutureBuildDemo {
    public static void main(String[] args) throws ExecutionException, InterruptedException {

        ExecutorService threadPool = Executors.newFixedThreadPool(3);

        CompletableFuture<String> completableFuture = CompletableFuture.supplyAsync(() -> {
            System.out.println(Thread.currentThread().getName());
            try {
                TimeUnit.MILLISECONDS.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            return "Hello supplyAsync";
        });

        System.out.println(completableFuture.get());

        threadPool.shutdown();
    }
}
```

![image-20221016201549420](D:\WorkSpace\Note\Picture\image-20221016201549420.png)

```java
public class CompletableFutureBuildDemo {
    public static void main(String[] args) throws ExecutionException, InterruptedException {

        ExecutorService threadPool = Executors.newFixedThreadPool(3);

        CompletableFuture<String> completableFuture = CompletableFuture.supplyAsync(() -> {
            System.out.println(Thread.currentThread().getName());
            try {
                TimeUnit.MILLISECONDS.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            return "Hello supplyAsync";
        }, threadPool);

        System.out.println(completableFuture.get());

        threadPool.shutdown();
    }
}
```

![image-20221016201820453](D:\WorkSpace\Note\Picture\image-20221016201820453.png)

**CompletableFuture可以设置回调**

```java
public class CompletableFutureUseDemo {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        CompletableFuture<Integer> completableFuture = CompletableFuture.supplyAsync(() -> {
            System.out.println(Thread.currentThread().getName() + "....come in");
            int result = ThreadLocalRandom.current().nextInt(10);
            try {
                TimeUnit.MILLISECONDS.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("---1s后，结果为：" + result);
            return result;
        }).whenComplete((result, e) -> {
            if (e == null) {
                System.out.println("...计算完成，更新系统UpdateValue：" + result);
            }
        }).exceptionally(e -> {
            e.printStackTrace();
            System.out.println("异常情况：" + e.getCause() + "\t" + e.getMessage());
            return null;
        });

        System.out.println(Thread.currentThread().getName() + "...在忙");
    }

    private static void future1() throws InterruptedException, ExecutionException {
        CompletableFuture<Integer> completableFuture = CompletableFuture.supplyAsync(() -> {
            System.out.println(Thread.currentThread().getName() + "....come in");
            int result = ThreadLocalRandom.current().nextInt(10);
            try {
                TimeUnit.MILLISECONDS.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("---1s后，结果为：" + result);
            return result;
        });

        System.out.println(Thread.currentThread().getName() + "...在忙");

        System.out.println(completableFuture.get());
    }
}
```

![image-20221016203029804](D:\WorkSpace\Note\Picture\image-20221016203029804.png)

主线程不能立即结束，**否则CompletableFuture默认使用的线程池会立即关闭**

**ForkJoinPool类似于守护线程**

```
public class CompletableFutureUseDemo {
    public static void main(String[] args) throws ExecutionException, InterruptedException {

        ExecutorService threadPool = Executors.newFixedThreadPool(3);

        CompletableFuture<Integer> completableFuture = CompletableFuture.supplyAsync(() -> {
            System.out.println(Thread.currentThread().getName() + "....come in");
            int result = ThreadLocalRandom.current().nextInt(10);
            try {
                TimeUnit.MILLISECONDS.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("---1s后，结果为：" + result);
            if (result > 5) {
                int i = 10 / 0;
            }
            return result;
        }, threadPool).whenComplete((result, e) -> {
            if (e == null) {
                System.out.println("...计算完成，更新系统UpdateValue：" + result);
            }
        }).exceptionally(e -> {
            e.printStackTrace();
            System.out.println("异常情况：" + e.getCause() + "\t" + e.getMessage());
            return null;
        });

        System.out.println(Thread.currentThread().getName() + "...在忙");
    }

    private static void future1() throws InterruptedException, ExecutionException {
        CompletableFuture<Integer> completableFuture = CompletableFuture.supplyAsync(() -> {
            System.out.println(Thread.currentThread().getName() + "....come in");
            int result = ThreadLocalRandom.current().nextInt(10);
            try {
                TimeUnit.MILLISECONDS.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("---1s后，结果为：" + result);
            return result;
        });

        System.out.println(Thread.currentThread().getName() + "...在忙");

        System.out.println(completableFuture.get());
    }
}
```

* 异步任务结束时，会自动回调某个对象的方法
* 主线程设置好回调后，不再关心异步任务的执行，异步任务之间可以顺序执行
* 异步任务出错时，会自动调用某个对象的方法

```java
public class CompletableFutureMallDemo {
    public static void main(String[] args) {

        CompletableFuture<String> completableFuture = CompletableFuture.supplyAsync(() -> {
            return "hello 123";
        });

//        System.out.println(completableFuture.get());
        System.out.println(completableFuture.join());
        
        Student student = new Student();
        student.setId(1);
        student.setStudentName("mnsx");
        student.setMajor("cs");
        
        student.setId(12).setStudentName("li4").setMajor("english");
    }
}

@NoArgsConstructor
@AllArgsConstructor
@Data
@Accessors(chain = true)
class Student {
    private Integer id;
    private String studentName;
    private String major;
}
```

@Accessors(chain = true)链式编程开启

join()方法，与get()类似，但是不会在编译过程中产生异常

## 电商案例

* 使用普通方式

```java
public class CompletableFutureMallDemo {

    static List<NetMail> list = Arrays.asList(
            new NetMail("jd"),
            new NetMail("dnagdang"),
            new NetMail("taobao")
    );

    public static List<String> getPrice(List<NetMail> list, String produceName) {

        return list.stream()
                .map(netMail -> String.format(produceName + "in %s price is %.2f",
                        netMail.getNetMailName(), netMail.calcPrice(produceName)))
                .collect(Collectors.toList());
    }

    public static void main(String[] args) {
        // 同一款产品，同时搜索出同款产品在各大电商平台的售价
        long startTime = System.currentTimeMillis();
        List<String> mysql = getPrice(list, "mysql");
        for (String s : mysql) {
            System.out.println(s);
        }
        long endTime = System.currentTimeMillis();
        System.out.println("---costTime" + (endTime - startTime) + "ms");
    }
}

class NetMail {
    @Getter
    private String netMailName;

    public NetMail(String netMailName) {
        this.netMailName = netMailName;
    }

    public double calcPrice(String produceName) {
        try {
            TimeUnit.MILLISECONDS.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        return ThreadLocalRandom.current().nextDouble() * 2 + produceName.charAt(0);
    }
}
```

![image-20221016223832253](D:\WorkSpace\Note\Picture\image-20221016223832253.png)

* 使用CompletableFuture

```java
public static List<String> getPriceByCompletableFuture(List<NetMail> list, String productName) {
    ExecutorService threadPool = Executors.newFixedThreadPool(3);
    return list.stream()
        .map(netMail -> CompletableFuture.supplyAsync(() -> String.format(productName + "in %s price is %.2f",
                                                                          netMail.getNetMailName(), netMail.calcPrice(productName)), threadPool)).collect(Collectors.toList())
        .stream().map(CompletableFuture::join).collect(Collectors.toList());
}
```

![image-20221017192138815](D:\WorkSpace\Note\Picture\image-20221017192138815.png)

## CompletableFuture API

* 获得结果和触发计算

  * 获得结果

    ```java
    public T get();
    public T get(long timeout, TimeUnit unit);
    public T join();
    public getNow(T valueIfAbsent);
    ```

  * 主动触发计算

    ```java
    public boolean complete(T value);
    ```

  ```java
  public class CompletableFutureAPIDemo {
      public static void main(String[] args) throws ExecutionException, InterruptedException, TimeoutException {
          CompletableFuture<String> completableFuture = CompletableFuture.supplyAsync(() -> {
              try {
                  TimeUnit.MILLISECONDS.sleep(1000);
              } catch (InterruptedException e) {
                  e.printStackTrace();
              }
              return "abc";
          });
  
          // 获得结果
  //        System.out.println(completableFuture.get()); // abc
  //        System.out.println(completableFuture.get(2, TimeUnit.MILLISECONDS)); // abc
  //        System.out.println(completableFuture.join()); // abc
  //        System.out.println(completableFuture.getNow("还没有结果")); // 还没有结果
          // 主动触发计算
  /*        System.out.println(completableFuture.complete("打断计算，立即返回该值")); // true
          System.out.println(completableFuture.get()); // 打断计算，立即返回该值*/
      }
  }
  ```

* 对计算结果进行处理

  **计算对结果存在依赖关系，这两个线程串行化，有异常仍旧能够通过**

  * thenApply

    ```java
    public class CompletableFutureAPI2Demo {
        public static void main(String[] args) {
            ExecutorService threadPool = Executors.newFixedThreadPool(3);
            CompletableFuture.supplyAsync(() -> {
                try {
                    TimeUnit.MILLISECONDS.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                return 1;
            }, threadPool).thenApply(f -> {
                System.out.println("222");
                return f + 2;
            }).thenApply(f -> {
                System.out.println("333");
                return f + 3;
            }).whenComplete((v, e) -> {
                if (e == null) {
                    System.out.println(v);
                }
            }).exceptionally(e -> {
                e.printStackTrace();
                System.out.println(e.getMessage());
                return null;
            });
    
            System.out.println(Thread.currentThread().getName() + "...忙");
        }
    }
    ```

  * handle

    ```java
    public class CompletableFutureAPI2Demo {
        public static void main(String[] args) {
            ExecutorService threadPool = Executors.newFixedThreadPool(3);
            CompletableFuture.supplyAsync(() -> {
                try {
                    TimeUnit.MILLISECONDS.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                return 1;
            }, threadPool).handle((f, e) -> {
                System.out.println("222");
                return f + 2;
            }).handle((f, e) -> {
                int i = 10 / 0;
                System.out.println("333");
                return f + 3;
            }).whenComplete((v, e) -> {
                if (e == null) {
                    System.out.println(v);
                }
            }).exceptionally(e -> {
                e.printStackTrace();
                System.out.println(e.getMessage());
                return null;
            });
    
            System.out.println(Thread.currentThread().getName() + "...忙");
        }
    }
    ```

* 对计算结果进行消费

  **接收任务的处理结果，并消费处理，无返回结果**

  * thenAccept

    ```java
    public class CompletableFutureAPI3Demo {
        public static void main(String[] args) {
            CompletableFuture.supplyAsync(() -> 1).thenApply(v -> v + 2).thenApply(v -> v + 3).thenAccept(System.out::println);
        }
    }
    ```

* 不需要计算结果新起一个线程

  * thenRun

    ```java
    public class CompletableFutureAPI3Demo {
        public static void main(String[] args) {
            System.out.println(CompletableFuture.supplyAsync(() -> 2).thenRun(() -> {
                System.out.println("fff");
            }).join());
        }
    }
    ```

* 线程池的选择

  ```java
  public class CompletableFutureWithThreadPoolDemo {
      public static void main(String[] args) throws ExecutionException, InterruptedException, TimeoutException {
          ExecutorService threadPool = Executors.newFixedThreadPool(5);
  
          CompletableFuture<Void> completableFuture = CompletableFuture.supplyAsync(() -> {
              try {
                  TimeUnit.MILLISECONDS.sleep(20);
              } catch (InterruptedException e) {
                  e.printStackTrace();
              }
              System.out.println("1号任务" + "\t" + Thread.currentThread().getName());
              return "abc";
          }, threadPool).thenRun(() -> {
              try {
                  TimeUnit.MILLISECONDS.sleep(20);
              } catch (InterruptedException e) {
                  e.printStackTrace();
              }
              System.out.println("2号任务" + "\t" + Thread.currentThread().getName());
          }).thenRun(() -> {
              try {
                  TimeUnit.MILLISECONDS.sleep(20);
              } catch (InterruptedException e) {
                  e.printStackTrace();
              }
              System.out.println("3号任务" + "\t" + Thread.currentThread().getName());
          }).thenRun(() -> {
              try {
                  TimeUnit.MILLISECONDS.sleep(20);
              } catch (InterruptedException e) {
                  e.printStackTrace();
              }
              System.out.println("4号任务" + "\t" + Thread.currentThread().getName());
          });
  
          completableFuture.get(2L, TimeUnit.SECONDS);
  
          threadPool.shutdown();
      }
  }
  ```

  **没有传入线程池，那么使用ForkJoinThreadPool**

  **thenRun的任务会跟随第一个任务使用的线程池**

  ![image-20221017202319908](D:\WorkSpace\Note\Picture\image-20221017202319908.png)

  **thenRunAsync的任务不会跟随**

  ![image-20221017202421131](D:\WorkSpace\Note\Picture\image-20221017202421131.png)

  **如果处理太快了，那么可能根据底层原理，可能直接main线程**

	```java
	private static final boolean useCommonPool =
    	(ForkJoinPool.getCommonPoolParallelism() > 1); // cpu数量大于1为true

	private static final Executor asyncPool = useCommonPool ?
    	ForkJoinPool.commonPool() : new ThreadPerTaskExecutor(); // 使用	ForkJoinPool
	```

* 对计算速度进行选用

  ```java
  public class CompletableFutureFastDemo {
      public static void main(String[] args) {
          CompletableFuture<String> playA = sCompletableFuture.supplyAsync(() -> {
              System.out.println("a come in");
              try {
                  TimeUnit.MILLISECONDS.sleep(2000);
              } catch (InterruptedException e) {
                  e.printStackTrace();
              }
              return "playA";
          });
  
          CompletableFuture<String> playB = CompletableFuture.supplyAsync(() -> {
              System.out.println("b come in");
              try {
                  TimeUnit.MILLISECONDS.sleep(1000);
              } catch (InterruptedException e) {
                  e.printStackTrace();
              }
              return "playB";
          });
  
          System.out.println(playA.applyToEither(playB, f -> {
              return f + "is winner";
          }).join());
      }
  }
  ```
  
  **竞争谁快使用谁**
  
* 对计算结果进行合并

  两个CompletionState任务都完成后，**最终能把两个任务的结果一起交给thenCombine来处理**

  ```java
  public class CompletableFutureCombineDemo {
      public static void main(String[] args) {
          CompletableFuture<Integer> completableFuture1 = CompletableFuture.supplyAsync(() -> {
              System.out.println(Thread.currentThread().getName() + "\t---启动");
              try {
                  TimeUnit.MILLISECONDS.sleep(1000);
              } catch (InterruptedException e) {
                  e.printStackTrace();
              }
              return 10;
          });
  
          CompletableFuture<Integer> completableFuture2 = CompletableFuture.supplyAsync(() -> {
              System.out.println(Thread.currentThread().getName() + "\t---启动");
              try {
                  TimeUnit.MILLISECONDS.sleep(1000);
              } catch (InterruptedException e) {
                  e.printStackTrace();
              }
              return 10;
          });
  
          CompletableFuture<Integer> result = completableFuture1.thenCombine(completableFuture2, Integer::sum);
  
          System.out.println(result.join());
      }
  }
  ```

# 多线程锁

## 乐观锁和悲观锁

* 悲观锁——synchronized关键字和lock的实现类都是悲观锁

  **适合写操作多的场景，先加锁可以保证写操作时数据正确**

  显示的锁定后再操作同步资源

* 乐观锁——认为自己在使用数据时**不会有别的线程来修改数据或资源**，所以不加锁

  在Java中通过无锁编程实现，只是在更新数据时去判断，之前没有别的线程更新这个数据

  **判断规则**

  **版本号机制Version**

  **CAS算法，Java原子类的递增操作就是通过CAS自旋实现的**

  **适合读操作多的场景**，不加锁的特点能够使其**读操作的性能大大提升**

  乐观锁则直接去操作同步资源，是一种无锁算法

## 8锁案例

1. 标准访问有ab两个线程，先打印邮件还是短信——》邮件

2. sendEmail方法中加入暂停3s，先打印邮件还是短信——》邮件

   > 一个对象里面如果有多个synchronized方法，某一时刻，只要一个线程去调用其中的一个synchronized方法了，其他线程都只能等待
   >
   > **某个时刻内，只能有唯一的一个线程去访问这些synchronized方法**
   >
   > **锁的是当前的对象this，被锁定后其他线程都不能进入到当前对象的其他的synchronized方法**

3. 添加一个普通的hello方法，先打印邮件还是hello——》hello

4. 有两部手机，先打印短信和邮件——》短信

   > 普通方法与同步锁无关不冲突，**只有声明了synchronized才会争抢锁**
   >
   > 两个对象，争抢的不是同一把锁，**锁的是this对象**

5. 有两个静态同步方法，先打印短信还是邮件——》邮件

6. 有两个静态同步方法，两部手机，先打印短信还是邮件——》邮件

   > **对于静态方法，锁的是类不是对象**
   >
   > * 对于普通同步方法，锁的是当前对象this
   > * 对于静态同步方法，锁的是当前对象的类
   > * 对于同步代码块，所得是括号中的对象

7. 有一个静态同步方法，有一个普通同步方法，有一部手机，先打印邮件还是短信——》短信

8. 有一个静态同步方法，有一个普通同步方法，有两部手机，先打印邮件还是短信——》短信

> `javap -c xx.class`
>
> 可以对代码进行反汇编
>
> 如果需要更多的信息
>
> `javap -v xx.class`
>
> `-v --verbose` 
>
> 输出附加信息（行号、本地变量表、反汇编等详细信息）

* 同步代码块

```java
public class LockSyncDemo {
    Object obj = new Object();

    public void m1() {
        synchronized (obj) {
            System.out.println("---hello synchronized code block");
        }
    }

    public static void main(String[] args) {

    }
}
```

![image-20221018213744918](D:\WorkSpace\Note\Picture\image-20221018213744918.png)

```shell
 public void m1();
    Code:
       0: aload_0
       1: getfield      #3                  // Field obj:Ljava/lang/Object;
       4: dup
       5: astore_1
       6: monitorenter
       7: getstatic     #4                  // Field java/lang/System.out:Ljava/io/PrintStream;
      10: ldc           #5                  // String ---hello synchronized code block
      12: invokevirtual #6                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      15: aload_1
      # 正常退出
      16: monitorexit
      17: goto          25
      20: astore_2
      21: aload_1
      # 异常退出
      22: monitorexit
      23: aload_2
      24: athrow
      25: return
    Exception table:
       from    to  target type
           7    17    20   any
          20    23    20   any
```

同步代码块**实现使用的是monitorenter和monitorexit指令**

一般情况一个monitorenter对两个exit

* 同步方法

```java
public class LockSyncDemo {

    public synchronized void m1() {
        System.out.println("---hello synchronized code block");
    }

    public static void main(String[] args) {

    }
}
```

![image-20221018214133998](D:\WorkSpace\Note\Picture\image-20221018214133998.png)

调用指令将会检查方法的**ACC_SYNCHTONIZED访问标志**是否被设置

如果设置，**执行线程时会将先持有monitor锁**，然后再执行方法

**最后方法完成后释放monitor**

JVM要求**执行线程要求先成功持有管程**

![image-20221018214954038](D:\WorkSpace\Note\Picture\image-20221018214954038.png)

**每一个对象创建时，底层都是c++，都会创建一个ObjectMonitor，所以每个对象都自带一个monitor**

**_owner——》代表被锁住的对象**

## 公平锁和非公平锁

```java
class TicketDemo {
    private int number = 50;
    ReentrantLock lock = new ReentrantLock(true);

    public void sale() {
        lock.lock();
        try {
            if (number > 0) {
                System.out.println(Thread.currentThread().getName() + "卖出第: \t" + (number--) + "\t 还剩: " + number);
            }
        } finally {
            lock.unlock();
        }
    }
}

public class SaleTicketDemo {
    public static void main(String[] args) {
        TicketDemo ticket = new TicketDemo();

        new Thread(() -> {
            for (int i = 0; i < 55; ++i) {
                ticket.sale();
            }
        }, "a").start();;

        new Thread(() -> {
            for (int i = 0; i < 55; ++i) {
                ticket.sale();
            }
        }, "b").start();;

        new Thread(() -> {
            for (int i = 0; i < 55; ++i) {
                ticket.sale();
            }
        }, "c").start();;
    }
}
```

公平锁：多个线程按照申请锁的顺序来获取锁，**先来先得**

非公平锁：多个线程获取锁的顺序不是按照申请锁的顺序，**后来的可能先得到锁**

**默认非公平锁**

恢复挂起线程对于cpu而言这个时间差存在的很明显，**使用非公平锁能够尽量减少CPU空闲状态时间**

**减少切换线程的开销**

## 可重入锁（递归锁）

**同一线程**在外层方法获取锁的时候，再次进入该线程的内部方法会**自动获取锁**

可重入锁分类

* 隐式锁（synchronized）默认就是可重入锁

  ```java
  public class ReEntryLockDemo {
      public static void main(String[] args) {
          final Object obj = new Object();
  
          new Thread(() -> {
              synchronized (obj) {
                  System.out.println(Thread.currentThread().getName() + "\t ---外层调用");
                  synchronized (obj) {
                      System.out.println(Thread.currentThread().getName() + "\t ---中层调用");
                      synchronized (obj) {
                          System.out.println(Thread.currentThread().getName() + "\t ---内层调用");
                      }
                  }
              }
          }, "a").start();
      }
  }
  ```

  ```java
  public class ReEntryLockDemo {
      public static synchronized void m1() {
          System.out.println(Thread.currentThread().getName() + "\t---come in");
          m2();
          System.out.println(Thread.currentThread().getName() + "\t---come end");
      }
  
      public static synchronized void m2() {
          System.out.println(Thread.currentThread().getName() + "\t---come in");
          m3();
          System.out.println(Thread.currentThread().getName() + "\t---come end");
      }
  
      public static synchronized void m3() {
          System.out.println(Thread.currentThread().getName() + "\t---come in");
          System.out.println(Thread.currentThread().getName() + "\t---come end");
      }
  
      public static void main(String[] args) {
          m1();
      }
  }
  ```

* 显式锁（lock）

**每个锁对象拥有一个锁计数器和一个指向持有该锁的线程的指针**

当锁计数器为0时，有线程进入加1，如果再次进入的相同线程，那么会再次加1，释放时则减1，为0表示全部释放

## 死锁和排查

两个或两个以上的线程在执行过程中，因争夺资源，导致相互等待的现象

```java
public class DeadLockDemo {
    public static void main(String[] args) {
        final Object o1 = new Object();
        final Object o2 = new Object();

        new Thread(() -> {
            synchronized (o1) {
                System.out.println(Thread.currentThread().getName() + "\t自己持有1锁，希望获得2锁");
                try {
                    TimeUnit.MILLISECONDS.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                synchronized (o2) {
                    System.out.println(Thread.currentThread().getName() + "\t成功获得2锁");
                }
            }
        }, "A").start();

        new Thread(() -> {
            synchronized (o2) {
                System.out.println(Thread.currentThread().getName() + "\t自己持有2锁，希望获得1锁");
                try {
                    TimeUnit.MILLISECONDS.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                synchronized (o1) {
                    System.out.println(Thread.currentThread().getName() + "\t成功获得1锁");
                }
            }
        }, "B").start();
    }
}
```

* **系统资源不足**
* **进程运行推进的顺序不合适**
* **资源分配不当**

> 排查死锁
>
> 1. **jps -l查看执行进程号**
> 2. **通过jstack查看堆栈情况**

# 线程中断

## 中断机制

**一个线程不应该被其他线程中断，应该由线程自己自行停止**

提供一种用于停止线程的协商机制——中断，也称中断标识协商机制

**中断过程完全需要程序员自己实现**

需要手动调用该线程interrupt方法，**该方法仅仅将线程对象中带你标识设为true**

## 三个中断方法

`public void interrupt()`

实例方法，**仅仅是设置线程的中断状态为true，发起一个协商而不会立即停止线程**

`public static boolean interrupted()`

静态方法，**判断线程是否被中断并清除当前中断状态**

1. 判断线程是否为中断状态
2. 清除线程的当前中断状态，并将中断状态设置为false

`public void boolean isInterrupted()`

实例方法，**判断当前线程是否被中断**

## 深入中断机制

* 如何停止中断运行中的线程

  * **使用volatile修饰的变量，通过volatile的可见性**

    ```java
    public class InterruptDemo {
    
         static volatile boolean isStop = false;
    
        public static void main(String[] args) {
            interrupt1();
        }
    
        private static void interrupt1() {
            new Thread(() -> {
                while (true) {
                    if (isStop) {
                        System.out.println(Thread.currentThread().getName() + "\t isStop被修改为true，线程停止");
                        break;
                    }
                    System.out.println("---hello volatile");
                }
            }, "t1").start();
    
            try {
                TimeUnit.MILLISECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
    
            new Thread(() -> {
                isStop = true;
            }, "t2").start();
        }
    }
    ```

  ![image-20221019200025444](D:\WorkSpace\Note\Picture\image-20221019200025444.png)

  * **通过AtomicBoolean，通过Atomic的原子性**

    ```java
    public class InterruptDemo {
         static AtomicBoolean isStop = new AtomicBoolean(false);
    
        public static void main(String[] args) {
            new Thread(() -> {
                while (true) {
                    if (isStop.get()) {
                        System.out.println(Thread.currentThread().getName() + "\t isStop被修改为true，线程停止");
                        break;
                    }
                    System.out.println("---hello volatile");
                }
            }, "t1").start();
    
            try {
                TimeUnit.MILLISECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
    
            new Thread(() -> {
                isStop.set(true);
            }, "t2").start();
        }
    }
    ```

    ![image-20221019201051478](D:\WorkSpace\Note\Picture\image-20221019201051478.png)

  * 使用原生API

    ```java
    public class InterruptDemo {
        public static void main(String[] args) {
            Thread t1 = new Thread(() -> {
                while (true) {
                    if (Thread.currentThread().isInterrupted()) {
                        System.out.println(Thread.currentThread().getName() + "\t isStop被修改为true，线程停止");
                        break;
                    }
                    System.out.println("---hello volatile");
                }
            }, "t1");
            t1.start();
    
            try {
                TimeUnit.MILLISECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
    
            new Thread(t1::interrupt, "t2").start();
        }
    }
    ```

    ![image-20221019201844539](D:\WorkSpace\Note\Picture\image-20221019201844539.png)

* 如果中断标识符被设置为true，不会立即停止线程

  ```java
  public class InterruptDemo2 {
      public static void main(String[] args) {
          Thread t1 = new Thread(() -> {
              for (int i = 1; i <= 300; ++i) {
                  System.out.println("-----" + i);
              }
              System.out.println("2" + Thread.currentThread().isInterrupted());
          }, "t1");
          t1.start();
          try {
              TimeUnit.MILLISECONDS.sleep(1);
          } catch (InterruptedException e) {
              e.printStackTrace();
          }
          t1.interrupt();
          System.out.println("1" + t1.isInterrupted());
          try {
              TimeUnit.MILLISECONDS.sleep(2000);
          } catch (InterruptedException e) {
              e.printStackTrace();
          }
          System.out.println("3" + t1.isInterrupted());
      }
  }
  ```

  ```java
  public class InterruptDemo3 {
      public static void main(String[] args) {
          Thread t1 = new Thread(() -> {
              while (true) {
                  if (Thread.currentThread().isInterrupted()) {
                      System.out.println(Thread.currentThread().getName() + "\t" + Thread.currentThread().isInterrupted());
                      break;
                  }
  
                  try {
                      Thread.sleep(200);
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  }
  
                  System.out.println("-----hello");
              }
          }, "t1");
          t1.start();
  
          try {
              TimeUnit.MILLISECONDS.sleep(1000);
          } catch (InterruptedException e) {
              e.printStackTrace();
          }
  
          new Thread(t1::interrupt).start();
      }
  }
  ```

  ![image-20221019211735192](D:\WorkSpace\Note\Picture\image-20221019211735192.png)

  **死循环**

  > 解决方案——
  >
  > ```java
  > try {
  >     Thread.sleep(200);
  > } catch (InterruptedException e) {
  >     Thread.currentThread().interrupt();
  >     e.printStackTrace();
  > }
  > ```
  >
  > ![image-20221019212339163](D:\WorkSpace\Note\Picture\image-20221019212339163.png)

* 静态方法interrupted

  ```java
  public class InterruptDemo4 {
      public static void main(String[] args) {
          System.out.println(Thread.currentThread().getName() + "\t" + Thread.interrupted());
          System.out.println(Thread.currentThread().getName() + "\t" + Thread.interrupted());
          System.out.println("-----1");
          Thread.currentThread().interrupt();
          System.out.println("-----2");
          System.out.println(Thread.currentThread().getName() + "\t" + Thread.interrupted());
          System.out.println(Thread.currentThread().getName() + "\t" + Thread.interrupted());
      }
  }
  ```

  ![image-20221019213317844](D:\WorkSpace\Note\Picture\image-20221019213317844.png)

  ```java
  // ClearInterrupt——是否清除中止标识
  public static boolean interrupted() {
      return currentThread().isInterrupted(true);
  }
  
  public boolean isInterrupted() {
      return isInterrupted(false);
  }
  
  private native boolean isInterrupted(boolean ClearInterrupted);
  ```

## 源码分析

```java
public void interrupt() {
    if (this != Thread.currentThread())
        checkAccess();

    synchronized (blockerLock) {
        Interruptible b = blocker;
        if (b != null) {
            interrupt0();           // Just to set the interrupt flag
            b.interrupt(this);
            return;
        }
    }
    interrupt0();
}
```

```java
private native void interrupt0();
```

**调用的原生interrupt0，被native修饰，表明调用的操作系统方法或者第三方代码库**

如果**被阻塞的线程**被调用了interrupt方法，那么就会导致InterruptedException

**中断不活动的线程不会产生任何的影响**

```java
public boolean isInterrupted() {
    return isInterrupted(false);
}

private native boolean isInterrupted(boolean ClearInterrupted);
```

**调用被native修饰的isInterrupted方法**，并且将中断标志位终止

# LockSupport

## 等待唤醒机制对比

* Object中wait()与notify()

  * 正常情况

  ```java
  public class LockSupportDemo {
      public static void main(String[] args) {
          Object objLock = new Object();
  
          new Thread(() -> {
              synchronized (objLock) {
                  System.out.println(Thread.currentThread().getName() + "\t ---come it");
                  try {
                      objLock.wait();
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  }
                  System.out.println(Thread.currentThread().getName() + "\t ---wake up");
              }
          }, "t1").start();
  
          try {
              TimeUnit.MILLISECONDS.sleep(1000);
          } catch (InterruptedException e) {
              e.printStackTrace();
          }
  
          new Thread(() -> {
              synchronized (objLock) {
                  objLock.notify();
                  System.out.println(Thread.currentThread().getName() + "\t ---notify");
              }
          },"t2").start();
      }
  }
  ```

  ![image-20221019220920913](D:\WorkSpace\Note\Picture\image-20221019220920913.png)

  * 异常情况1

  ```java
  public class LockSupportDemo {
      public static void main(String[] args) {
          Object objLock = new Object();
  
          new Thread(() -> {
  //            synchronized (objLock) {
                  System.out.println(Thread.currentThread().getName() + "\t ---come it");
                  try {
                      objLock.wait();
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  }
                  System.out.println(Thread.currentThread().getName() + "\t ---wake up");
  //            }
          }, "t1").start();
  
          try {
              TimeUnit.MILLISECONDS.sleep(1000);
          } catch (InterruptedException e) {
              e.printStackTrace();
          }
  
          new Thread(() -> {
  //            synchronized (objLock) {
                  objLock.notify();
                  System.out.println(Thread.currentThread().getName() + "\t ---notify");
  //            }
          },"t2").start();
      }
  }
  ```

  ![image-20221019220955333](D:\WorkSpace\Note\Picture\image-20221019220955333.png)

  **使用wait和notify方法需要加锁**

  * 异常情况2

  ```java
  public class LockSupportDemo {
      public static void main(String[] args) {
          Object objLock = new Object();
  
          new Thread(() -> {
              synchronized (objLock) {
                  System.out.println(Thread.currentThread().getName() + "\t ---come it");
                  objLock.notify();
                  System.out.println(Thread.currentThread().getName() + "\t ---wake up");
              }
          }, "t1").start();
  
          try {
              TimeUnit.MILLISECONDS.sleep(1000);
          } catch (InterruptedException e) {
              e.printStackTrace();
          }
  
          new Thread(() -> {
              synchronized (objLock) {
                  try {
                      objLock.wait();
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  }
                  System.out.println(Thread.currentThread().getName() + "\t ---notify");
              }
          },"t2").start();
      }
  }
  ```

  ![image-20221019221235154](D:\WorkSpace\Note\Picture\image-20221019221235154.png)

  **无法唤醒程序**

* JUC中Condition的await()与signal()

  * 异常情况1

  ```java
  public class LockSupportDemo {
      public static void main(String[] args) {
          Lock lock = new ReentrantLock();
          Condition condition = lock.newCondition();
  
          new Thread(() -> {
  //            lock.lock();
              try {
                  System.out.println(Thread.currentThread().getName() + "\t ---come in");
                  condition.await();
                  System.out.println(Thread.currentThread().getName() + "\t ---被唤醒");
              } catch (InterruptedException e) {
                  e.printStackTrace();
              } finally {
  //                lock.unlock();
              }
          }, "t1").start();
  
          try {
              TimeUnit.MILLISECONDS.sleep(1000);
          } catch (InterruptedException e) {
              e.printStackTrace();
          }
  
          new Thread(() -> {
  //            lock.lock();
              try {
                  condition.signal();
                  System.out.println(Thread.currentThread().getName() + "\t ---发出通知");
              } finally {
  //                lock.unlock();
              }
          }, "t2").start();
      }
  }
  ```

  ![image-20221020200000915](D:\WorkSpace\Note\Picture\image-20221020200000915.png)

  * 异常情况2

  ```java
  public class LockSupportDemo {
      public static void main(String[] args) {
          Lock lock = new ReentrantLock();
          Condition condition = lock.newCondition();
  
          new Thread(() -> {
              lock.lock();
              try {
                  condition.signal();
                  System.out.println(Thread.currentThread().getName() + "\t ---发出通知");
              } finally {
                  lock.unlock();
              }
          }, "t2").start();
  
          try {
              TimeUnit.MILLISECONDS.sleep(1000);
          } catch (InterruptedException e) {
              e.printStackTrace();
          }
  
          new Thread(() -> {
              lock.lock();
              try {
                  System.out.println(Thread.currentThread().getName() + "\t ---come in");
                  condition.await();
                  System.out.println(Thread.currentThread().getName() + "\t ---被唤醒");
              } catch (InterruptedException e) {
                  e.printStackTrace();
              } finally {
                  lock.unlock();
              }
          }, "t1").start();
      }
  }
  ```

  ![image-20221020200323251](D:\WorkSpace\Note\Picture\image-20221020200323251.png)

> Object和Condition使用的限制条件
>
> * 线程先要获得并持有锁，必须在锁块（synchronized或lock）中
> * 必须要先等待后唤醒，线程才能够被唤醒

* LockSupport的pack()与unpack()

  ```java
  public static void park() {
      UNSAFE.park(false, 0L);
  }
  ```

  **UNSAFE类功能强大但不推荐使用，因为容易造成内存泄漏**

  ```java
  public native void park(boolean var1, long var2);
  ```

  ![image-20221020200955164](D:\WorkSpace\Note\Picture\image-20221020200955164.png)

  **permit许可证默认没有**，不能放行，调用park方法当前线程就会阻塞，直到别的线程给当前线程发放**permit**，park方法才会被唤醒

  ```java
  public static void unpark(Thread thread) {
      if (thread != null)
          UNSAFE.unpark(thread);
  }
  ```

  ```java
  public native void unpark(Object var1);
  ```

  **不需要锁块，也可以使用**

  ```java
  public class LockSupportDemo {
      public static void main(String[] args) {
          Thread t1 = new Thread(() -> {
              try {
                  TimeUnit.MILLISECONDS.sleep(1000);
              } catch (InterruptedException e) {
                  e.printStackTrace();
              }
              System.out.println(Thread.currentThread().getName() + "\t ---come in");
              LockSupport.park();
              System.out.println(Thread.currentThread().getName() + "\t ---wake up");
          }, "t1");
          t1.start();
  
          new Thread(() -> {
              System.out.println(Thread.currentThread().getName() + "\t ---notice");
              LockSupport.unpark(t1);
          }, "t2").start();
      }
  }
  ```

  **先唤醒后等待LockSupport照样可以支持**

  ```java
  public class LockSupportDemo {
      public static void main(String[] args) {
          Thread t1 = new Thread(() -> {
              try {
                  TimeUnit.MILLISECONDS.sleep(1000);
              } catch (InterruptedException e) {
                  e.printStackTrace();
              }
              System.out.println(Thread.currentThread().getName() + "\t ---come in");
              LockSupport.park();
              LockSupport.park();
              System.out.println(Thread.currentThread().getName() + "\t ---wake up");
          }, "t1");
          t1.start();
  
          new Thread(() -> {
              System.out.println(Thread.currentThread().getName() + "\t ---notice");
              LockSupport.unpark(t1);
              LockSupport.unpark(t1);
              LockSupport.unpark(t1);
              LockSupport.unpark(t1);
          }, "t2").start();
      }
  }
  ```

  **unpark重复使用无效，因为只能拥有一个permit**

  ![image-20221020211949553](D:\WorkSpace\Note\Picture\image-20221020211949553.png)

# Java内存模型

## Java Memory Model

![image-20221020213131785](D:\WorkSpace\Note\Picture\image-20221020213131785.png)

CPU的运行并**不是直接操作内存而是先把内存里面的数据读到缓存**，而内存的读和写操作的时候就会造成不一致的问题

JVM规范中视图定义一种Java内存模型（JMM）来**屏蔽各种硬件和操作系统的内存访问差异**，以实现让Java程序在各种平台下都能达到一致的内存访问效果

JMM本身是一种**抽象概念并不真实存在**，仅仅描述一组约定或规范，通过这组规范定义了程序中各个变量的读写访问方式并决定一个线程对共享变量的读写访问方式并决定一个线程对共享变量的写入何时以及何时变成对另一个线程可见，关键技术点都是围绕**多线程的原子性、可见性和有序性展开的**

> 作用：
>
> 1. 通过JMM来实现**线程和主内存之间的抽象关系**
> 2. **屏蔽各个硬件平台和操作系统的内存访问差异**以实现让Java线程在各种平台下达到一致的内存访问效果

## 三大特性

* 可见性

  **当一个线程修改了共享变量的值，其他线程是否能够立即知道该变更**，JMM规定了所有的变量都存储在主内存中

  ![image-20221020221642073](D:\WorkSpace\Note\Picture\image-20221020221642073.png)

  系统主内存**共享变量**数据修改被写的时机是不确定的，**多线程并发下很可能出现脏读**，所以每个线程都有自己的**工作内存**，线程自己的工作内存保存了该线程使用到的变量的**主内存副本拷贝**，线程对变量的所有操作（读取、赋值等）都必须在线程自己的工作内存中进行，而不能够直接读写主内存的变量，不同线程之间也无法直接访问对方工作内存中的变量，线程间变量值的传递均需要通过主内存来完成

* 原子性

  一个操作是不可打断的，即多线程环境下，操作不能被其他线程干扰

* 有序性

  为了提升性能，编译器和处理器通常会对指令序列进行重新排序，**Java对丁JVM线程内部维持顺序化语义，即只要结果不受影响，那么允许指令执行顺序与代码顺序不同**，即指令重排序

  ![image-20221020223507108](D:\WorkSpace\Note\Picture\image-20221020223507108.png)

  ![image-20221020222410151](D:\WorkSpace\Note\Picture\image-20221020222410151.png)

## 多线程对共享变量读写

![image-20221020223030791](D:\WorkSpace\Note\Picture\image-20221020223030791.png)

![image-20221020223241423](D:\WorkSpace\Note\Picture\image-20221020223241423.png)

## happens-before

在JMM中**如果一个线程执行的结果需要对另一个操作可见**或者**代码重排序**，那么两个操作之间必须存在happens-before原则

**保证可见性和有序性**

* **总原则**
  1. **如果一个操作hanppens-before另一个操作，那么第一个操作的执行结果对第二个操作可见，而且第一个操作的执行顺序排在第二个操作之前**
  2. **两个操作之间存在hapens-before关系，并以为一定按照原则指定的顺序执行，如果重拍的结果一致，那么这种重排序并不非法**

* 八条细则

  1. 次序规则

     **一个线程内**，按照代码顺序写在前面的操作先行发生于写在后面的操作

  2. 锁定规则

     一个unlock操作**先行发生于**后面对同同一把锁的lock操作

  3. volatile变量规则

     对一个volatile变量的写操作先行发生于后面对这个变量的读操作，**前面的写对后面的读是可见的**，这里的后面同样是指时间上的先后

  4. 传递规则

     A先于B，B先于C，那么A先于C

  5. 线程启动原则

     Thread对象的start()方法先行发生于此线程的每一个动作

  6. 线程中断规则

     需要先设置中断标志，才能检测到中断事件的发生

  7. 线程终止规则

     线程中的是所有操作都先行发生于对此线程的终止检测，我们可以通过**isAlive()等手段检测线程是否已经停止执行**

  8. 对象终结规则

     一个对象的初始化完成先行发生于它的finalize()方法的开始

# volatile

## volatile两大特性

**可见性和有序性**

volatile内存语义

* 当写一个volatile变量时，JMM会把该线程对应的本地内存中的共享变量值**立即刷新回主内存**中
* 当读一个volatile变量时，JMM会把该线程对应的本地内存设置为无效，重新回到主内存中读取最新的共享变量

## 内存屏障

内存屏障其实就是一种JVM指令，Java内存模型的重排规则会**要求Java编译器在涩会给你成JVM指令时插入特定的内存屏障指令**，通过这些指令，volatile实现了Java内存模型中的可见性和重排序，**但volatile无法保证原子性**

![image-20221021220851443](D:\WorkSpace\Note\Picture\image-20221021220851443.png)

**分类**

* 粗分

  * 读屏障——在读指令之前插入读屏障，让工作内存或CPU告诉缓存当中的缓存数据失效，重新回到主内存中获取最最新数据
  * 写屏障——在写指令之后插入写屏障，强制把写缓冲区的数据刷回主内存中

* 细分

  ```java
  //Unsave.class
  public native void loadFence();
  
  public native void storeFence();
  
  public native void fullFence();
  ```

  ![image-20221021221341379](D:\WorkSpace\Note\Picture\image-20221021221341379.png)

> 在每一个**volatile读操作后面**插入一个**LoadLoad屏障**——禁止处理器把上面的volatile读于下面的普通读重排序
>
> 在每一个**volatile读操作后面**插入一个**LoadStore屏障**——禁止处理器把上面的volatile读与下面的普通写重排序
>
> 在每一个**volatile写操作后面**插入一个**StoreStore屏障**——可以保证volatile之前，其前面的所有普通写操作都已经刷新到主内存中
>
> 在每一个**volatile写操作后面**插入一个**StoreLoad屏障**——作用是避免volatile写于后面可能有的volatile读/写操作重排序

## volatile特性

**保证可见性**

```java
public class VolatileSeeDemo {
    static volatile boolean flag = true;
    
    public static void main(String[] args) {
        new Thread(() -> {
            System.out.println(Thread.currentThread().getName() + "\t ---come in");
            while (flag) {
                
            }
            System.out.println(Thread.currentThread().getName() + "\t ---end");
        }, "t1").start();    
        
        try {
            TimeUnit.MILLISECONDS.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        
        flag = false;

        System.out.println(Thread.currentThread().getName() + "\t ---flag被设置为false，程序停止");
    }
}
```

> read -> load -> use -> assign -> store -> write -> **lock -> unlock**
>
> **lock和unlock提供原子性**

**不具有原子性**

```java
class MyNumber {
    int number;

    public synchronized void addPlusPlus() {
        number++;
    }
}

public class VolatileNoAtomicDemo {
    public static void main(String[] args) {
        MyNumber myNumber = new MyNumber();

        for (int i = 0; i <= 10; ++i) {
            new Thread(() -> {
                for (int j = 1; j <= 1000; ++j) {
                    myNumber.addPlusPlus();
                }
            }, String.valueOf(i)).start();
        }

        try {
            TimeUnit.MILLISECONDS.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println(myNumber.number);
    }
}
```

![image-20221021223448763](D:\WorkSpace\Note\Picture\image-20221021223448763.png)

```java
// 不加锁
class MyNumber {
    int number;

    public void addPlusPlus() {
        number++;
    }
}

public class VolatileNoAtomicDemo {
    public static void main(String[] args) {
        MyNumber myNumber = new MyNumber();

        for (int i = 0; i <= 10; ++i) {
            new Thread(() -> {
                for (int j = 1; j <= 1000; ++j) {
                    myNumber.addPlusPlus();
                }
            }, String.valueOf(i)).start();
        }

        try {
            TimeUnit.MILLISECONDS.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println(myNumber.number);
    }
}
```

![image-20221021223728969](D:\WorkSpace\Note\Picture\image-20221021223728969.png)

volatile不能保证原子性，A如果修改值后，B需要立即去获取A修改后的值，**可能会造成修丢失的情况**

**指令禁重排**

**不存在数据依赖关系**，那么重排序是合法的

**存在数据依赖关系**，那么重排序是非法的

## 使用场景

* 单一赋值

* 状态标志，判断业务是否结束

* 开销较低的读，写锁策略

* DCL双端锁的发布，单例设计模式


# CAS

## CAS简介

**compare and swap**的缩写，实现并发算法时，常用到的一种技术

包含**三个操作数**——**内存位置**、**预期原值**及**更新值**

执行CAS操作的时候，将内存位置的值与预期原值比较

* 如果相匹配，**那么处理器会自动将该位置值改为更新值**
* 如果不匹配，处理器不做任何操作，**多个线程同时执行CAS只有一个会成功**

```java
public class CASDemo {
    public static void main(String[] args) {
        AtomicInteger atomicInteger = new AtomicInteger(5);

        System.out.println(atomicInteger.compareAndSet(5, 2048));
        System.out.println(atomicInteger.compareAndSet(5, 2048));
    }
}
```

CAS是JDK提供的**非阻塞原子性**操作，**它通过硬件保证了比较-更新的原子性**

CAS是一条CPU的**原子指令**（cmpxchg指令），不会造成所谓的数据不一致，**Unsafe提供的CAS方法底层实现即CPU指令cmpxchg**

> 执行cmpxchg指令的时候，会判断当前线程是否为多核系统，如果是就给总线枷锁，只有一个线程会对总线加锁成功，枷锁成功之后会执行cas操作，**也就是CAS的原子性实际上是CPU实现独占的**

```java
public final boolean compareAndSet(int expect, int update) {
    return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
}
```

```java
public final native boolean compareAndSwapObject(Object var1, long var2, Object var4, Object var5);

public final native boolean compareAndSwapInt(Object var1, long var2, int var4, int var5);

public final native boolean compareAndSwapLong(Object var1, long var2, long var4, long var6);
```

* var1：表示要操作的对象
* var2：表示要操作对象中属性地址的偏移量
* var4：表示需要修改数据的期望值
* var5/var6：表示需要修改为的新值

## CAS底层原理（Unsafe类）

![image-20221022192318065](D:\WorkSpace\Note\Picture\image-20221022192318065.png)

Unsafe是CAS的核心类，由于Java方法无法直接访问底层系统，需要通过本地（native）方法来访问，**Unsafe相当于是一个后门**，基于该类可以**直接操作特定的内存数据**，**其内部方法操作可以像C的指针一样直接操作内存**

**Unsafe类中的所有方法都是native修饰的，也就是说Unsafe类中的方法都直接调用操作系统底层资源执行相应任务**

变量valueOffset，表示**该变量在内存中的偏移地址**，因为Unsafe类需要根据内存偏移地址来获取数据

AtomicInteger类主要利用**CAS + volatile 和 native方法来保证原子操作**，从而避免synchronized的高开销

```java
public final int getAndIncrement() {
    return unsafe.getAndAddInt(this, valueOffset, 1);
}
```

```java
public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    do {
        var5 = this.getIntVolatile(var1, var2);
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

    return var5;
}
```

调用CAS方法，JVM会实现CAS汇编指令

这是一种完全依赖于**硬件**的功能，通过它可以实现原子操作

CAS是一种系统原语，原语属于操作系统用于范畴，是由若干条指令组成，用于完成某个功能的一个过程，**并且原语的执行必须是连续的，在执行过程中不允许被中断，也就是说CAS是一条CPU的原子指令，不会造成所谓的数据不一致的问题**

**JDK提供的CAS机制，在汇编层级会禁止变量两侧的指令优化，然后使用cmpxchg指令比较并更新变量值（原子性）**

## 原子应用AtomicReference

```java
public class AtomicReferenceDemo {
    public static void main(String[] args) {
        AtomicReference<User> atomicReference = new AtomicReference<>();

        User z3 = new User("z3", 22);
        User l4 = new User("l4", 38);

        atomicReference.set(z3);

        System.out.println(atomicReference.compareAndSet(z3, l4) + "\t" + atomicReference.get().toString());
    }
}

@Data
@AllArgsConstructor
class User {
    String username;
    int age;
}
```

## 自旋锁

**采用循环的方式尝试获取锁**，好处**减少线程上下文切换的消耗**，缺点**循环消耗CPU**

```java
public class SpinLockDemo {
    AtomicReference<Thread> atomicReference = new AtomicReference<>();

    public void lock() {
        Thread thread = Thread.currentThread();
        System.out.println(Thread.currentThread().getName() + "\t ---come in");
        while (!atomicReference.compareAndSet(null, thread)) {

        }
    }

    public void unlock() {
        Thread thread = Thread.currentThread();
        atomicReference.compareAndSet(thread, null);
        System.out.println(Thread.currentThread().getName() + "\t ---unlock");
    }

    public static void main(String[] args) {
        SpinLockDemo spinLockDemo = new SpinLockDemo();

        new Thread(() -> {
            spinLockDemo.lock();
            try {
                TimeUnit.MILLISECONDS.sleep(5000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            spinLockDemo.unlock();
        }, "A").start();

        try {
            TimeUnit.MILLISECONDS.sleep(500);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        new Thread(() -> {
            spinLockDemo.lock();

            spinLockDemo.unlock();
        }, "B").start();
    }
}
```

## CAS缺点

* 循环时间长开销过大

  如果CAS长时间一直不成功，可能带来巨大的CPU开销

* ABA问题

  线程1获取A后，A就是预期值，在线程1将预期值和地址原值作比较的中间时间，线程2获取A将其改为B后放回，但是如果线程2又将B改为A放回地址，A这时候比较返回的结果就将会是true，**线程2中间的操作将会被忽视**

  **尽管线程1CAS操作成功，但是不代表这个过程就是没有问题的**

## AtomicStampedReference

简单API调用

```java
public class AtomicStampedDemo {
    public static void main(String[] args) {
        Book javaBook = new Book(1, "java");

        AtomicStampedReference<Book> stampedReference = new AtomicStampedReference<>(javaBook, 1);

        System.out.println(stampedReference.getReference() + "\t" + stampedReference.getStamp());

        Book mysqlBook = new Book(2, "mysql");

        boolean flag = stampedReference.compareAndSet(javaBook, mysqlBook, stampedReference.getStamp(), stampedReference.getStamp() + 1);
        System.out.println(flag + "\t" + stampedReference.getReference() + "\t" + stampedReference.getStamp());

        flag = stampedReference.compareAndSet(javaBook, mysqlBook, stampedReference.getStamp(), stampedReference.getStamp() + 1);
        System.out.println(flag + "\t" + stampedReference.getReference() + "\t" + stampedReference.getStamp());
    }
}

@Data
@AllArgsConstructor
class Book {
    private int id;
    private String bookName;
}
```

ABA问题展示

```java
public class ABADemo {
    static AtomicInteger atomicInteger = new AtomicInteger(100);

    public static void main(String[] args) {
        new Thread(() -> {
            atomicInteger.compareAndSet(100, 101);
            try {
                TimeUnit.MILLISECONDS.sleep(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            atomicInteger.compareAndSet(101, 100);
        }, "t1").start();

        new Thread(() -> {
            try {
                TimeUnit.MILLISECONDS.sleep(200);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(atomicInteger.compareAndSet(100, 2022) + "\t" + atomicInteger.get());
        }, "t2").start();
    }
}
```

使用AtomicStampedReference解决

```java
public class ABADemo {
    static AtomicStampedReference<Integer> stampedReference = new AtomicStampedReference<>(100, 1);

    public static void main(String[] args) {
        new Thread(() -> {
            int stamp = stampedReference.getStamp();
            System.out.println(Thread.currentThread().getName() + "\t 1" + stamp);
            try {
                TimeUnit.MILLISECONDS.sleep(500);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            stampedReference.compareAndSet(100, 101, stampedReference.getStamp(), stampedReference.getStamp() + 1);
            System.out.println(Thread.currentThread().getName() + "\t 2" + stampedReference.getStamp());
            stampedReference.compareAndSet(101, 102, stampedReference.getStamp(), stampedReference.getStamp() + 1);
            System.out.println(Thread.currentThread().getName() + "\t 3" + stampedReference.getStamp());
        }, "t1").start();

        new Thread(() -> {
            int stamp = stampedReference.getStamp();
            System.out.println(Thread.currentThread().getName() + "\t 1" + stamp);
            try {
                TimeUnit.MILLISECONDS.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(stampedReference.compareAndSet(100, 2022, stamp, stamp + 1));
            System.out.println(stampedReference.getReference() + "\t 2" + stampedReference.getStamp());
        }, "t2").start();
    }
}
```

# 原子操作类

## 基础类型原子类

> * AtomicInteger
> * AtomicBoolean
> * AtomicLong

```java
public class AtomicIntegerDemo {
    public static final int SIZE = 50;

    public static void main(String[] args) throws InterruptedException {
        MyNumberAtomic myNumberAtomic = new MyNumberAtomic();
        CountDownLatch countDownLatch = new CountDownLatch(SIZE);

        for (int i = 0; i < SIZE; i++) {
            new Thread(() -> {
                try {
                    for (int j = 0; j < 1000; ++j) {
                        myNumberAtomic.addPlusPlus();
                    }
                } finally {
                    countDownLatch.countDown();
                }
            }, String.valueOf(i)).start();
        }

        countDownLatch.await();

        System.out.println(Thread.currentThread().getName() + "\t" +myNumberAtomic.atomicInteger.get());
    }
}
```

## 数组类型原子类

```java
public class AtomicIntegerArrayDemo {
    public static void main(String[] args) {
        AtomicIntegerArray atomicIntegerArray = new AtomicIntegerArray(new int[5]);

        for (int i = 0;  i < atomicIntegerArray.length(); ++i) {
            System.out.println(atomicIntegerArray.get(i));
        }

        int tmpInt = 0;

        tmpInt = atomicIntegerArray.getAndSet(0, 1122);
        System.out.println(atomicIntegerArray.get(0));
        tmpInt = atomicIntegerArray.getAndSet(1, 1123);
        System.out.println(atomicIntegerArray.get(1));
    }
}
```

## 引用类型原子类

```java
public class AtomicMarkReferenceDemo {
    static AtomicMarkableReference markableReference = new AtomicMarkableReference(100, false);

    public static void main(String[] args) {
        new Thread(() -> {
            boolean marked = markableReference.isMarked();
            System.out.println(Thread.currentThread().getName() + "\t1" + marked);
            try {
                TimeUnit.MILLISECONDS.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + "\t2" + markableReference.isMarked());
        }, "t1").start();

        new Thread(() -> {
            boolean marked = markableReference.isMarked();
            System.out.println(Thread.currentThread().getName() + "\t2" + marked);
            try {
                TimeUnit.MILLISECONDS.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            markableReference.compareAndSet(100, 200, marked, !marked);
            System.out.println(Thread.currentThread().getName() + "\t2" + markableReference.isMarked());
        }, "t2").start();
    }
}
```

## 对象的属性修改原子类

> AtomicIntegerFieldUpdater——原子更新对象中int类型字段的值
>
> AtomicLongFieldUpdater——原子更新对象中Long类型字段的值
>
> AtomicReferenceFieldUpdater——原子更新引用类型字段的值

**字段需要使用volatile字段标志**

以一种**线程安全**的方式操作**非线程安全**对象内的某些字段

更新的对象属性必须使用**pulbic volatile**修饰符

因为对象属性修改类型原子类都是抽象类，所以每次使用都必须**使用静态方法newUpdater()**创建一个更新器，而且需要设**置想要更新的类和属性**

```java
class BankAccount {
    String bankName = "CCB";
    int money = 0;
    public synchronized void add() {
        money++;
    }
}

public class AtomicIntegerFieldUpdaterDemo {
    public static void main(String[] args) throws InterruptedException {
        BankAccount bankAccount = new BankAccount();
        CountDownLatch countDownLatch = new CountDownLatch(10);

        for (int i = 1; i <= 10; ++i) {
            new Thread(() -> {
                try {
                    for (int j = 1; j <= 1000; ++j) {
                        bankAccount.add();
                    }
                } finally {
                    countDownLatch.countDown();
                }
            }, String.valueOf(i)).start();
        }

        countDownLatch.await();

        System.out.println(bankAccount.money);
    }
}
```

```java
class BankAccount {
    String bankName = "CCB";
    public volatile int money = 0;
    public void add() {
        money++;
    }

    AtomicIntegerFieldUpdater<BankAccount> fieldUpdater = AtomicIntegerFieldUpdater.newUpdater(BankAccount.class, "money");

    public void transMoney(BankAccount bankAccount) {
        fieldUpdater.getAndIncrement(bankAccount);
    }
}

public class AtomicIntegerFieldUpdaterDemo {
    public static void main(String[] args) throws InterruptedException {
        BankAccount bankAccount = new BankAccount();
        CountDownLatch countDownLatch = new CountDownLatch(10);

        for (int i = 1; i <= 10; ++i) {
            new Thread(() -> {
                try {
                    for (int j = 1; j <= 1000; ++j) {
                        bankAccount.transMoney(bankAccount);
                    }
                } finally {
                    countDownLatch.countDown();
                }
            }, String.valueOf(i)).start();
        }

        countDownLatch.await();

        System.out.println(bankAccount.money);
    }
}
```

```java
class MyVar {
    public volatile Boolean isInit = Boolean.FALSE;

    AtomicReferenceFieldUpdater<MyVar, Boolean> referenceFieldUpdater = AtomicReferenceFieldUpdater.newUpdater(MyVar.class, Boolean.class, "isInit");

    public void init(MyVar myVar) {
        if (referenceFieldUpdater.compareAndSet(myVar, Boolean.FALSE, Boolean.TRUE)) {
            System.out.println(Thread.currentThread().getName() + "\t" + "---start init");
            try {
                TimeUnit.MILLISECONDS.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + "\t" + "---end init");
        } else {
            System.out.println(Thread.currentThread().getName() + "\t" + "---已经有线程开始进行初始化工作");
        }
    }
}

public class AtomicReferenceFieldUpdaterDemo {
    public static void main(String[] args) {
        MyVar myVar = new MyVar();

        for (int i = 0; i < 5; i++) {
            new Thread(() -> {
                myVar.init(myVar);
            }, String.valueOf(i)).start();
        }
    }
}
```

## 原子操作增强类

```java
class ClickNumber {
    int number = 0;

    public synchronized void click() {
        number++;
    }

    AtomicLong atomicLong = new AtomicLong(0);

    public void click2() {
        atomicLong.getAndIncrement();
    }

    LongAdder longAdder = new LongAdder();

    public void click3() {
        longAdder.increment();
    }

    LongAccumulator longAccumulator = new LongAccumulator(Long::sum, 0);

    public void click4() {
        longAccumulator.accumulate(1);
    }
}
public class AccumulatorCompareDemo {
    public static final int _1w = 10000;
    public static final int threadNumber = 50;

    public static void main(String[] args) throws InterruptedException {
        ClickNumber clickNumber = new ClickNumber();

        CountDownLatch countDownLatch1 = new CountDownLatch(threadNumber);
        CountDownLatch countDownLatch2 = new CountDownLatch(threadNumber);
        CountDownLatch countDownLatch3 = new CountDownLatch(threadNumber);
        CountDownLatch countDownLatch4 = new CountDownLatch(threadNumber);

        long start = 0L;
        long end = 0L;

        start = System.currentTimeMillis();
        for (int i = 1; i <= threadNumber; ++i) {
            new Thread(() -> {
                try {
                    for (int j = 0; j < _1w; ++j) {
                        clickNumber.click();
                    }
                } finally {
                    countDownLatch1.countDown();
                }
            }, String.valueOf(i)).start();
        }
        countDownLatch1.await();
        end = System.currentTimeMillis();
        System.out.println("---const time" + (end - start) + "ms" + "\t" + clickNumber.number);

        start = System.currentTimeMillis();
        for (int i = 1; i <= threadNumber; ++i) {
            new Thread(() -> {
                try {
                    for (int j = 0; j < _1w; ++j) {
                        clickNumber.click2();
                    }
                } finally {
                    countDownLatch2.countDown();
                }
            }, String.valueOf(i)).start();
        }
        countDownLatch2.await();
        end = System.currentTimeMillis();
        System.out.println("---const time" + (end - start) + "ms" + "\t" + clickNumber.atomicLong.get());

        start = System.currentTimeMillis();
        for (int i = 1; i <= threadNumber; ++i) {
            new Thread(() -> {
                try {
                    for (int j = 0; j < _1w; ++j) {
                        clickNumber.click3();
                    }
                } finally {
                    countDownLatch3.countDown();
                }
            }, String.valueOf(i)).start();
        }
        countDownLatch3.await();
        end = System.currentTimeMillis();
        System.out.println("---const time" + (end - start) + "ms" + "\t" + clickNumber.longAdder.sum());

        start = System.currentTimeMillis();
        for (int i = 1; i <= threadNumber; ++i) {
            new Thread(() -> {
                try {
                    for (int j = 0; j < _1w; ++j) {
                        clickNumber.click4();
                    }
                } finally {
                    countDownLatch4.countDown();
                }
            }, String.valueOf(i)).start();
        }
        countDownLatch4.await();
        end = System.currentTimeMillis();
        System.out.println("---const time" + (end - start) + "ms" + "\t" + clickNumber.longAccumulator.get());
    }
}
```

![image-20221023235841717](D:\WorkSpace\Note\Picture\image-20221023235841717.png)

```java
public class LongAdder extends Striped64 implements Serializable
```

```java
@sun.misc.Contended static final class Cell {
    volatile long value;
    Cell(long x) { value = x; }
    final boolean cas(long cmp, long val) {
        return UNSAFE.compareAndSwapLong(this, valueOffset, cmp, val);
    }

    // Unsafe mechanics
    private static final sun.misc.Unsafe UNSAFE;
    private static final long valueOffset;
    static {
        try {
            UNSAFE = sun.misc.Unsafe.getUnsafe();
            Class<?> ak = Cell.class;
            valueOffset = UNSAFE.objectFieldOffset
                (ak.getDeclaredField("value"));
        } catch (Exception e) {
            throw new Error(e);
        }
    }
}

static final int NCPU = Runtime.getRuntime().availableProcessors();

transient volatile Cell[] cells
    
transient volatile long base;

transient volatile int cellsBusy;
```

* NCPU——电脑CPU个数
* cells——单元格数组
* base
* cellsBusy——自旋锁

**Striped64中一些变量或者方法的定义**

* base：类似于AtomicLong中全局的value值，在没有竞争情况下数据直接累加到base上，或者cells扩容时，也需要将数据写到base上
* collide：表示扩容意向，false一定不会扩容，true可能会扩容
* cellsBudy：初始化cells或者扩容cells需要获取锁，0表示无所，1表示其他线程已经拥有锁
* casCellsBudy()：通过CAS操作修改cellsBusy的值，CAS成功代表获取锁，返回true
* NCPU：当前计算机CPU数量，Cell数组扩容时会使用到
* getProbe()：获取当前线程的hash值
* advanceProbe()：重置当前线程的hash值

## LongAdder高性能原理

当只有少量线程时，他与Atomic类似只有一个base再进行数据处理，但是一旦存在大量线程访问时，cell[]也将同时处理数据

LongAdder内部有一个base变量和一个Cell数组

* base变量：低并发，直接累加再该变量上
* Cell数组，高并发，累加进各个线程自己的槽cell[i]中

槽位分配与HashMap类似，**哈希映射**

```java
public void increment() {
    add(1L);
}
```

```java
public void add(long x) {
    Cell[] as; long b, v; int m; Cell a;
    if ((as = cells) != null || !casBase(b = base, b + x)) {
        boolean uncontended = true;
        if (as == null || (m = as.length - 1) < 0 ||
            (a = as[getProbe() & m]) == null ||
            !(uncontended = a.cas(v = a.value, v + x)))
            longAccumulate(x, null, uncontended);
    }
}
```

* as表示cells引用
* b表示获取的base值
* v表示期望值
* m表示cells数组的长度
* a表示当前程序命中的cell单元格

```java
final void longAccumulate(long x, LongBinaryOperator fn,
                          boolean wasUncontended) {
    int h;
    if ((h = getProbe()) == 0) {
        ThreadLocalRandom.current(); // force initialization
        h = getProbe();
        wasUncontended = true;
    }
    boolean collide = false;                // True if last slot nonempty
    for (;;) {
        Cell[] as; Cell a; int n; long v;
        if ((as = cells) != null && (n = as.length) > 0) {
            if ((a = as[(n - 1) & h]) == null) {
                if (cellsBusy == 0) {       // Try to attach new Cell
                    Cell r = new Cell(x);   // Optimistically create
                    if (cellsBusy == 0 && casCellsBusy()) {
                        boolean created = false;
                        try {               // Recheck under lock
                            Cell[] rs; int m, j;
                            if ((rs = cells) != null &&
                                (m = rs.length) > 0 &&
                                rs[j = (m - 1) & h] == null) {
                                rs[j] = r;
                                created = true;
                            }
                        } finally {
                            cellsBusy = 0;
                        }
                        if (created)
                            break;
                        continue;           // Slot is now non-empty
                    }
                }
                collide = false;
            }
            else if (!wasUncontended)       // CAS already known to fail
                wasUncontended = true;      // Continue after rehash
            else if (a.cas(v = a.value, ((fn == null) ? v + x :
                                         fn.applyAsLong(v, x))))
                break;
            else if (n >= NCPU || cells != as)
                collide = false;            // At max size or stale
            else if (!collide)
                collide = true;
            else if (cellsBusy == 0 && casCellsBusy()) {
                try {
                    if (cells == as) {      // Expand table unless stale
                        Cell[] rs = new Cell[n << 1];
                        for (int i = 0; i < n; ++i)
                            rs[i] = as[i];
                        cells = rs;
                    }
                } finally {
                    cellsBusy = 0;
                }
                collide = false;
                continue;                   // Retry with expanded table
            }
            h = advanceProbe(h);
        }
        else if (cellsBusy == 0 && cells == as && casCellsBusy()) {
            boolean init = false;
            try {                           // Initialize table
                if (cells == as) {
                    Cell[] rs = new Cell[2];
                    rs[h & 1] = new Cell(x);
                    cells = rs;
                    init = true;
                }
            } finally {
                cellsBusy = 0;
            }
            if (init)
                break;
        }
        else if (casBase(v = base, ((fn == null) ? v + x :
                                    fn.applyAsLong(v, x))))
            break;                          // Fall back on using base
    }
}
```

1. 最初无竞争时只更新base
2. 如果更新base失败，首次新建一个Cell[]数组
3. 当多个线程竞争同一个Cell比较激烈时，可能就要对Cell[]扩容，每次加2

![image-20221024210817771](D:\WorkSpace\Note\Picture\image-20221024210817771.png)

longAccumulate()方法的入参

* long x需要增加的值，一般默认都是1
* LongBinaryOperator fn默认传递的是null
* wasUncontended竞争标识，如果是false则代表有竞争，只有cells初始化之后，并且当前线程CAS竞争修改失败，才会是false

```java
static final int getProbe() {
    return UNSAFE.getInt(Thread.currentThread(), PROBE);
}
```

```java
private static final sun.misc.Unsafe UNSAFE;
private static final long BASE;
private static final long CELLSBUSY;
private static final long PROBE;
static {
    try {
        UNSAFE = sun.misc.Unsafe.getUnsafe();
        Class<?> sk = Striped64.class;
        BASE = UNSAFE.objectFieldOffset
            (sk.getDeclaredField("base"));
        CELLSBUSY = UNSAFE.objectFieldOffset
            (sk.getDeclaredField("cellsBusy"));
        Class<?> tk = Thread.class;
        PROBE = UNSAFE.objectFieldOffset
            (tk.getDeclaredField("threadLocalRandomProbe"));
    } catch (Exception e) {
        throw new Error(e);
    }
}
```

`getProbe()`得到**当前线程的Hash值**

如果没有hash值，那么会**在longAccumulate中强制获取一个hash值**

longAccumulate分为三个case——

* cells已经被初始化了
  * 当前cell为null——创建cell并赋值
  * 竞争修改value失败——设置wasUncontended=true
  * CAS修改数组cell——cell中value=value+x
  * 数组长度大于NCPU——设置collide=false
  * 数组长度小于NCPU——设置collide=true
  * 允许扩容——扩容2倍并拷贝数组
* cells**没有加锁且没有初始化**，则尝试对它进行加锁，并初始化cells数组
* cells**正在进行初始化**，则尝试直接在基数base上进行累加操作

# ThreadLocal

## Thread基础功能

实现**每一个线程都有自己专属的本地变量副本**

```java
class House {
    int saleCount = 0;
    public synchronized void saleHouse() {
        ++saleCount;
    }

    /*ThreadLocal<Integer> saleVolume = new ThreadLocal<Integer>() {
        @Override
        protected Integer initialValue() {
            return 0;
        }
    };*/

    ThreadLocal<Integer> saleVolume = ThreadLocal.withInitial(() -> 0);

    public void saleVolumeByThreadLocal() {
        saleVolume.set(1 + saleVolume.get());
    }
}
public class ThreadLocalDemo {
    public static void main(String[] args) {
        House house = new House();

        for (int i = 0; i < 5; i++) {
            
            
            new Thread(() -> {
                int size = new Random().nextInt(5) + 1;
                try {
                    for (int i1 = 0; i1 < size; i1++) {
//                    house.saleHouse();
                        house.saleVolumeByThreadLocal();
                    }
                    System.out.println(Thread.currentThread().getName() + "\t 共计卖出多少套: " + house.saleVolume.get());
                } finally {
                    house.saleVolume.remove();
                }
            }, String.valueOf(i)).start();
        }

        try {
            TimeUnit.MILLISECONDS.sleep(300);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

    }
}
```

使用Thread Local之后必须将其remove

```java
class MyData {
    ThreadLocal<Integer> threadLocalField = ThreadLocal.withInitial(() -> 0);

    public void add() {
        threadLocalField.set(1 + threadLocalField.get());
    }
}
public class ThreadLocalDemo2 {
    public static void main(String[] args) {
        MyData myData = new MyData();
        ExecutorService threadPool = Executors.newFixedThreadPool(3);

        try {
            for (int i = 0; i < 10; i++) {
                threadPool.submit(() -> {
                    System.out.println(Thread.currentThread().getName() + "before" + myData.threadLocalField.get());
                    myData.add();
                    System.out.println(Thread.currentThread().getName() + "after" + myData.threadLocalField.get());
                });
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            threadPool.shutdown();
        }
    }
}
```

线程池线程复用出现问题

![image-20221025200158388](D:\WorkSpace\Note\Picture\image-20221025200158388.png)

## 源码解析

```java
public class Thread implements Runnable {
    ThreadLocal.ThreadLocalMap threadLocals = null;
}
```

创建Thread就会生成一个ThreadLocal

ThreadLocal中包含了一个静态内部类ThreadMap

![image-20221025200929226](D:\WorkSpace\Note\Picture\image-20221025200929226.png)

**ThreadLocalMap实际上就是一个以threadLocal实例为key，任意对象为value的Entry对象**

## ThreadLocal内存泄漏分析

内存泄漏——**不再会被使用的对象或者变量占用的内存不能被回收，就是内存泄漏**

```java
static class Entry extends WeakReference<ThreadLocal<?>> {
    /** The value associated with this ThreadLocal. */
    Object value;

    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    }
}
```

Jave技术允许使用finalize()方法，**当垃圾收集器确定没有对该对象的引用时，由对象上的垃圾收集器调用**

* 强引用

  当内存不足时，JVM开始垃圾回收，对于强引用的对象，就算**出现了OOM也不会对该对象进行回收**，因此强引用是造成Java内存泄漏的主要原因

  ```java
  class MyObject {
      @Override
      protected void finalize() throws Throwable {
          System.out.println("---invoke finalize method");
      }
  }
  public class ReferenceDemo {
      public static void main(String[] args) {
          MyObject myObject = new MyObject();
          System.out.println("gc before: " + myObject);
  
          myObject = null;
          System.gc();
  
          try {
              TimeUnit.MILLISECONDS.sleep(5000);
          } catch (InterruptedException e) {
              e.printStackTrace();
          }
  
          System.out.println("gc after: " + myObject);
      }
  }
  ```

![image-20221025203114504](D:\WorkSpace\Note\Picture\image-20221025203114504.png)

* 软引用

  软引用是一种相对强引用弱化了一些的引用，需要用SoftReference类来实现，**可以让对象豁免一些垃圾收集**

  对于只有软引用的对象来说

  * 当系统内存充足时，不会被回收
  * 当系统内存不足时，会被回收

```java
class MyObject {
    @Override
    protected void finalize() throws Throwable {
        System.out.println("---invoke finalize method");
    }
}
public class ReferenceDemo {
    public static void main(String[] args) {
        SoftReference<MyObject> myObjectSoftReference = new SoftReference<>(new MyObject());
        System.gc();
        try {
            TimeUnit.MILLISECONDS.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        // System.out.println("---gc after内存够用: " + myObjectSoftReference.get());

        try {
            byte[] bytes = new byte[20 * 1024 * 1024];
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            System.out.println("---gc after内存不够用: " + myObjectSoftReference.get());
        }
    }
}
```

![image-20221025210612002](D:\WorkSpace\Note\Picture\image-20221025210612002.png)

* 弱引用

  弱引用需要使用WeakReference类来实现，**它比软引用的生存期更短**

  **只要gc执行马上就回收**

  ```java
  class MyObject {
      @Override
      protected void finalize() throws Throwable {
          System.out.println("---invoke finalize method");
      }
  }
  public class ReferenceDemo {
      public static void main(String[] args) {
          WeakReference<MyObject> myObjectWeakReference = new WeakReference<>(new MyObject());
          System.out.println("---before gc" + myObjectWeakReference);
          System.gc();
          System.out.println("---after gc" + myObjectWeakReference);
      }
  }
  ```

![image-20221025211008839](D:\WorkSpace\Note\Picture\image-20221025211008839.png)

* 虚引用

  1. 虚引用必须和引用队列联合使用

     需要使用PhantomReference类来实现

     **如果一个对象仅持有虚引用，那么它就与没有引用一样，任何时候都可能被垃圾回收**

     **不能直接访问该对象**

  2. PhantomReference‘的get方法总是返回null

     **仅仅是提供了一种确保对象被finalize以后，做某些事的通知机制**

  3. 处理监控通知使用

     被垃圾回收时收到一个通知

  ```java
  class MyObject {
      @Override
      protected void finalize() throws Throwable {
          System.out.println("---invoke finalize method");
      }
  }
  public class ReferenceDemo {
      public static void main(String[] args) {
          MyObject myObject = new MyObject();
          ReferenceQueue<MyObject> referenceQueue = new ReferenceQueue<>();
          PhantomReference<MyObject> myObjectPhantomReference = new PhantomReference<>(myObject, referenceQueue);
          myObject = null;
          System.gc();
  
  //        System.out.println(myObjectPhantomReference.get()); // null
  
          List<byte[]> list = new ArrayList<>();
  
          new Thread(() -> {
              while (true) {
                  list.add(new byte[1 * 1024 * 1024]);
                  try {
                      TimeUnit.MILLISECONDS.sleep(500);
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  }
                  System.out.println(myObjectPhantomReference.get());
              }
          }, "t1").start();
  
          new Thread(() -> {
              while (true) {
                  Reference<? extends MyObject> reference = referenceQueue.poll();
                  if (reference != null) {
                      System.out.println("---有虚引用对象加入队列");
                      break;
                  }
              }
          }, "t2").start();
      }
  }
  ```

  ![image-20221025224605277](D:\WorkSpace\Note\Picture\image-20221025224605277.png)

对象将会有一个强引用指向ThreadLocal，当对象被设置为null时，那么就会调用gc，进行垃圾回收，如果ThreadLcoal中ThreadLocalMap中的不是弱引用，那么可能存在对象与ThreadLocal的引用被删除，但是ThreadLocal和ThreadLocalMap的引用没有删除的情况

但是使用弱引用仍旧可能存在内存泄露的情况，因为弱引用被删除后，Entry的key将会被设置为null，如果没有调用expungeStaleEntry那么内存中将永远存在一个key为null的Entry

```java
private int expungeStaleEntry(int staleSlot) {
    Entry[] tab = table;
    int len = tab.length;

    // expunge entry at staleSlot
    tab[staleSlot].value = null;
    tab[staleSlot] = null;
    size--;

    // Rehash until we encounter null
    Entry e;
    int i;
    for (i = nextIndex(staleSlot, len);
         (e = tab[i]) != null;
         i = nextIndex(i, len)) {
        ThreadLocal<?> k = e.get();
        if (k == null) {
            e.value = null;
            tab[i] = null;
            size--;
        } else {
            int h = k.threadLocalHashCode & (len - 1);
            if (h != i) {
                tab[i] = null;

                // Unlike Knuth 6.4 Algorithm R, we must scan until
                // null because multiple entries could have been stale.
                while (tab[h] != null)
                    h = nextIndex(h, len);
                tab[h] = e;
            }
        }
    }
    return i;
}
```

# Java对象内存布局

## 对象在堆内存中的布局

![image-20221026192454856](D:\WorkSpace\Note\Picture\image-20221026192454856.png)

* 对象头

  * 对象标记 MarkWord

    ```java
    public class ObjectHeadDemo {
        public static void main(String[] args) {
    
            Object o = new Object(); // new一个对象，占多少内存
    
            System.out.println(o.hashCode()); // 这个hashcode记录在对象的什么地方
    
            synchronized (o) {
                // 为什么知道这个对象被加锁
            }
    
            System.gc(); // 手动收集垃圾——如果15次没有被回收，那么就可以从新生代到养老区
        }
    }
    ```

    ![image-20221026193451332](D:\WorkSpace\Note\Picture\image-20221026193451332.png)

    | 存储内容                             | 标志位 | 状态               |
    | ------------------------------------ | ------ | ------------------ |
    | 对象哈希码、对象分代年龄             | 01     | 未锁定             |
    | 指向锁记录的指针                     | 00     | 轻量级锁定         |
    | 指向重量级锁的指针                   | 10     | 膨胀（重量级锁定） |
    | 空，不需要记录信息                   | 11     | GC标记             |
    | 偏向线程ID、偏向时间戳、对象分代年龄 | 01     | 可偏向             |

    **在64位系统中，Mark Word占了8字节，类型指针占了8字节，一共是16个字节**

    > 默认存储对象是HashCode、分代年龄和锁标志位等信息
    >
    > 这些信息都是与对象自身定义无关的数据，所以MarkWord被设计成一个**非固定的数据结构**以便**在极小的空间内存储尽量多的数据**。他会根据对象的状态复用自己的存储空间，也就是说**在运行期间MarkWord里存储的数据会随锁标志位的变化而变化**

  * 类元信息 类型指针+

    存储的时指向对象元数据（klass）的首地址

    指向方法区中klass类元信息

    ![image-20221026195307562](D:\WorkSpace\Note\Picture\image-20221026195307562.png)

* 实例数据

  存放类的属性（Field）数据信息，包括父类的属性信息

* 对齐填充

  虚拟机要求对象起始地址必须是8字节的整数倍

  填充数据不是必须存在的，仅仅是为了字节对齐

**数组对象比普通对象在对象头中多一个Length长度**

## 深入对象头

![image-20221026200611893](D:\WorkSpace\Note\Picture\image-20221026200611893.png)

hash：存储对象的哈希码

age：保存对象的分代年龄

biased_lock：偏向锁标识位

lock：锁状态标识位

JavaThread*：保存持有偏向锁的线程ID

epoch：保存偏向时间戳

**GC年龄采用4位bit存储，最大为15**

**默认开启了压缩指针**，所以实际操作中，类型指针为4字节，而不是8字节

# synchronized锁升级 

## synchronized的性能变化

重量级锁，如果锁的竞争强烈，那么性能就会下降

> **Java5之前，用户态和内核态之间的转换**
>
> java的线程是映射到操作系统原生线程之上的，如果要阻塞或唤醒一个线程就需要**操作系统介入**，需要**在用户态和核心态之间切换**， 这种切换回**消耗大量的系统资源**，因为**用户态与内核态都有各自专用的内存空间，专用的寄存器等**，用户态切换内核态，需要传递许多变量、参数给内核，内核也需要保护好用户态在切换时的一些寄存器值和变量等，**以便内核态调用结束后切换回用户态继续工作**

早期的**synchronized属于重量级锁，效率低，因为监视器锁（monitor）是依赖于底层操作系统的Mutex Lock（系统互斥锁）来实现的**，挂起和恢复线程都需要转入内核态去完成

针对这种情况，java6之后，为了减少获得锁和释放锁所带来的性能消耗，引入了轻量级锁和偏向锁

## synchronized锁的种类和升级流程

![image-20221028215819050](D:\WorkSpace\Note\Picture\image-20221028215819050.png)

* 偏向锁：MarkWord存储的是偏向的线程ID
* 轻量级锁：MarkWord存储的是指向线程栈中LockRecord的指针
* 重量级锁：MarkWord存储的是指向堆中的monitor对象的指针

**无锁**：初始状态，一个对象被实例化侯，如果还没有被任何线程竞争锁，那么它就是五锁状态(001)

```java
class Ticket {
    private int number = 50;

    Object lockObject = new Object();

    public void sale() {
        synchronized (lockObject) {
            if (number > 0) {
                System.out.println(Thread.currentThread().getName() + "\t" + (number--) + "\t" + number);
            }
        }
    }
}
public class SynchronizedUpDemo {
    public static void main(String[] args) {
        Object o = new Object();

        System.out.println(o.hashCode());
        System.out.println(Integer.toHexString(o.hashCode()));
        System.out.println(Integer.toBinaryString(o.hashCode()));

        System.out.println(ClassLayout.parseInstance(o).toPrintable());
    }
}
```

![image-20221028221121535](D:\WorkSpace\Note\Picture\image-20221028221121535.png)

**偏向锁**：单线程竞争

> 当线程A第一次竞争到锁时，通过操作修改MarkWord中的偏向线程ID、偏向模式
>
> 如果不存在其他线程竞争，那么持有偏向锁的线程将永远不需要竞争

**当一段同步代码一直被同一个线程多次访问，由于只有一个线程那么该线程在后续访问时便会自动获得锁**

**偏向锁会偏向第一个访问锁的线程**，如果不存在其他线程竞争，那么持有偏向锁的线程将不需要同步

将该线程的id放在对象头中，下次访问线程ID于对象头中的ID相同，那么无需每次加锁去cas更新对象头，如果不相等，那么才需要通过cas更改对象头的数据

**启动偏向锁需要4s的延迟进行开启**

```java
// 将启动时间改为0——-XX:BiasedLockingStartupDelay=0
// 睡眠5s
class Ticket {
    private int number = 50;

    Object lockObject = new Object();

    public void sale() {
        synchronized (lockObject) {
            if (number > 0) {
                System.out.println(Thread.currentThread().getName() + "\t" + (number--) + "\t" + number);
            }
        }
    }
}
public class LockSaleTicketDemo {
    public static void main(String[] args) {
        Object o = new Object();

        synchronized (o) {
            System.out.println(ClassLayout.parseInstance(o).toPrintable());
        }
    }
}
```

![image-20221028223913809](D:\WorkSpace\Note\Picture\image-20221028223913809.png)

第一个线程进入后会将第一个线程ID存入对象头

```java
class Ticket {
    private int number = 50;

    Object lockObject = new Object();

    public void sale() {
        synchronized (lockObject) {
            if (number > 0) {
                System.out.println(Thread.currentThread().getName() + "\t" + (number--) + "\t" + number);
            }
        }
    }
}
public class LockSaleTicketDemo {
    public static void main(String[] args) {
        try {
            TimeUnit.MILLISECONDS.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        Object o = new Object();

/*        synchronized (o) {
            System.out.println(ClassLayout.parseInstance(o).toPrintable());
        }*/

        new Thread(() -> {
            synchronized (o) {
                System.out.println(ClassLayout.parseInstance(o).toPrintable());
            }
        }, "t1").start();
    }
}
```

![image-20221028224122094](D:\WorkSpace\Note\Picture\image-20221028224122094.png)

当第二个线程进入，抢夺资源后，就不能在使用偏向锁了，要升级位轻量级锁

竞争线程尝试CAS更新对象头失败，会等待到**全局安全点**（此时不再执行任何代码）撤销偏向锁

![image-20221028224525501](D:\WorkSpace\Note\Picture\image-20221028224525501.png)

**轻锁**：多线程竞争，但是任意时刻最多只有一个线程竞争，即不存在锁竞争太过激烈的情况，也就没有线程阻塞

**为了在线程近乎交替执行同步块时提高性能**，在没有多线程竞争的前提下，**通过CAS减少**重量级锁使用操作系统互斥量产生的性能消耗，先进行自旋，不行才进行阻塞

**当关闭偏向锁功能或多线程竞争偏向锁时**会导致锁升级为轻量级锁

JVM会为每个线程在当前线程的栈帧中创建用于存储锁记录的空间——Displaced Mark Word

如果一个线程获得锁时发现是轻量级锁，会把锁的MarkWord复制到自己的Displaced Mark Word里面，然后线程尝试用CAS将锁的MarkWord替换为指向所记录的指针，如果成功，当前线程获得锁，如果失败，表示MarkWord已经被替换成了其他线程的锁记录，说明在与其他线程竞争锁，当前线程尝试使用自旋获取锁

```java
class Ticket {
    private int number = 50;

    Object lockObject = new Object();

    public void sale() {
        synchronized (lockObject) {
            if (number > 0) {
                System.out.println(Thread.currentThread().getName() + "\t" + (number--) + "\t" + number);
            }
        }
    }
}
public class LockSaleTicketDemo {
    public static void main(String[] args) {
        Object o = new Object();

        new Thread(() -> {
            System.out.println(ClassLayout.parseInstance(o).toPrintable());
        }, "t1").start();
    }
}
```

![image-20221029215422555](D:\WorkSpace\Note\Picture\image-20221029215422555.png)

![锁升级](D:\WorkSpace\Note\Picture\锁升级.png)

> **自适应自旋锁的大致原理**
>
> 线程如果自旋成功了，那么下次自旋的最大次数会增加，因为JVM认为既然上次成功了，那么这一次也很大概率会成功
>
> 反之，如果很少会自旋成功，那么下次会减少自旋的次数甚至不自选，避免CPU空转

**重锁**：有大量的线程参与锁的竞争，冲突性很高

> **hashcode与锁升级的关系**
>
> 当一个对象已经计算过一致性哈希码后，**他就再也无法进入偏向锁状态了**
>
> 而且当一个对象当前正处于偏向锁状态，有收到需要计算其一致性哈希码请求时，**它的偏向状态会被立即撤销，并且锁会膨胀为重量级锁**
>
> 轻量级锁，JVM会在当前线程中创建一个锁记录空间，**用于存储锁对象的MarkWord拷贝，该拷贝中包含identity hash code**，所以轻量级锁可以和identity hash code共存
>
> 代表重量级锁的ObjectMonitor类里**有字段可以记录非加锁状态下的MarkWord**，自然可以存储原来的hashcode
>
> ```java
> public class HashCodeSynchronizedDemo {
>     public static void main(String[] args) {
>         try {
>             TimeUnit.MILLISECONDS.sleep(5000);
>         } catch (InterruptedException e) {
>             e.printStackTrace();
>         }
> 
>         Object o = new Object();
>         System.out.println(ClassLayout.parseInstance(o).toPrintable());
> 
>         o.hashCode();
> 
>         synchronized (o) {
>             System.out.println(ClassLayout.parseInstance(o).toPrintable());
>         }
>     }
> }
> ```
>
> ![image-20221029221854776](D:\WorkSpace\Note\Picture\image-20221029221854776.png)
>
> ```java
> public class HashCodeSynchronizedDemo {
>     public static void main(String[] args) {
> 
>         try {
>             TimeUnit.MILLISECONDS.sleep(5000);
>         } catch (InterruptedException e) {
>             e.printStackTrace();
>         }
> 
>         Object o = new Object();
> 
>         synchronized (o) {
>             o.hashCode();
>             System.out.println(ClassLayout.parseInstance(o).toPrintable());
>         }
>     }
> }
> ```
>
> ![image-20221029222424665](D:\WorkSpace\Note\Picture\image-20221029222424665.png)

![image-20221029222614473](D:\WorkSpace\Note\Picture\image-20221029222614473.png)

## JIT编译器对锁的优化

JT（即时编译器）

* 锁消除问题

```java
public class LockClearUpDemo {
    static Object objectLock = new Object();

    public static void main(String[] args) {
        LockClearUpDemo lockClearUpDemo = new LockClearUpDemo();

        for (int i = 1; i <= 10; ++i) {
            new Thread(() -> {
                m1();
            }, String.valueOf(i)).start();
        }
    }

    private static void m1() {
        /*synchronized (objectLock) {
            System.out.println("---hello");
        }*/

        Object o = new Object();
        synchronized (o) {
            System.out.println("---hello" + "\t" + o.hashCode() + "\t" + objectLock.hashCode());
        }
    }
}
```

![image-20221029223119628](D:\WorkSpace\Note\Picture\image-20221029223119628.png)

从JIT角度相当于无视他，synchronized(o)不存在，这个锁对象并没有被共用扩散到其他线程使用，极端的说根本没有加这个锁对象的底层机器码，消除了锁的使用

* 锁粗化问题

```java
public class LockBigDemo {
    static Object objectLock = new Object();

    public static void main(String[] args) {
        new Thread(() -> {
            synchronized (objectLock) {
                System.out.println("111");
            }
            synchronized (objectLock) {
                System.out.println("222");
            }
            synchronized (objectLock) {
                System.out.println("333");
            }
            
            synchronized (objectLock) {
                System.out.println("111");
                System.out.println("222");
                System.out.println("333");
            }
        }, "t1").start();
    }
}
```

# AQS

## 入门理论知识

![image-20221029225200258](D:\WorkSpace\Note\Picture\image-20221029225200258.png)

CLH：是一个单向链表，AQS中队列是CLH变体的虚拟双向队列FIFO

**整体就是一个抽象的FIFO队列来完成资源获取的排队工作，并通过一个int类型变量表示持有锁的状态**

![image-20221030184645995](D:\WorkSpace\Note\Picture\image-20221030184645995.png)

如果共享资源被占用，**就需要一定的阻塞等待唤醒机制来保证锁分配**，主要用CLH的变体实现的，将暂时获取不到锁的线程加入到队列中，这个队列就是AQS同步队列的抽象表现

它将要请求共享资源的线程及自身的等待状态封装成队列的节点对象（node），通过CAS、自旋以及LockSupport.park()的方式，维护state变量的状态，是并发达到同步的效果

![image-20221030185659956](D:\WorkSpace\Note\Picture\image-20221030185659956.png)

![image-20221030185709177](D:\WorkSpace\Note\Picture\image-20221030185709177.png)

AQS使用一个**volatile的int类型的成员变量来表示同步状态**，通过内置的FIFO队列来完成资源获取的排队工作将每条要去抢占资源的线程封装成一个Node节点来实现锁的分配，通过CAS完成对State值的修改

![image-20221030190113345](D:\WorkSpace\Note\Picture\image-20221030190113345.png)

## 源码分析前置知识

* AQS自身

  * AQS的int变量

    AQS的同步状态State成员变量

    ```java
    private volatile int state;
    ```

  * AQS的CLH队列

    双向队列，通过自旋等待，state变量判断是否阻塞，从尾部入队，从头部出队

* 内部类Node

  * Node的int变量

    Node的等待状态waitState成员变量

    ```java
    volatile int waitStatus;
    ```

  * Node内部结构

    ```java
    static final class Node {
        // 共享
        static final Node SHARED = new Node();
    	// 独占
        static final Node EXCLUSIVE = null;
    
    	// 线程被取消
        static final int CANCELLED =  1;
    	// 后续线程需要唤醒
        static final int SIGNAL    = -1;
    	// 等待condition唤醒
        static final int CONDITION = -2;
    	// 共享式同步状态获得将无条件的传播下去
        static final int PROPAGATE = -3;
    
       	// 初始为0，状态时上面的几种
        volatile int waitStatus;
    	
        // 前置节点
        volatile Node prev;
    
       	// 后继节点
        volatile Node next;
    }
    ```

## 源码深度讲解和分析

```java
public class AQSDemo {
    public static void main(String[] args) {
        Lock lock = new ReentrantLock();

        lock.lock();
        try {

        } finally {
            lock.unlock();
        }
    }
}
```

默认使用非公平锁，公平锁和非公平锁的区别

![image-20221031135311535](D:\WorkSpace\Note\Picture\image-20221031135311535.png)

![image-20221031135402732](D:\WorkSpace\Note\Picture\image-20221031135402732.png)

* 非公平锁

```java
static final class NonfairSync extends Sync {
    private static final long serialVersionUID = 7316153563782823691L;

    /**
         * Performs lock.  Try immediate barge, backing up to normal
         * acquire on failure.
         */
    final void lock() {
        if (compareAndSetState(0, 1))
            setExclusiveOwnerThread(Thread.currentThread());
        else
            acquire(1);
    }

    protected final boolean tryAcquire(int acquires) {
        return nonfairTryAcquire(acquires);
    }
}
```

* 公平锁

```java
static final class FairSync extends Sync {
    private static final long serialVersionUID = -3000897897090466540L;

    final void lock() {
        acquire(1);
    }
	
    protected final boolean tryAcquire(int acquires) {
        final Thread current = Thread.currentThread();
        int c = getState();
        if (c == 0) {
            if (!hasQueuedPredecessors() &&
                compareAndSetState(0, acquires)) {
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        else if (current == getExclusiveOwnerThread()) {
            int nextc = c + acquires;
            if (nextc < 0)
                throw new Error("Maximum lock count exceeded");
            setState(nextc);
            return true;
        }
        return false;
    }
}
```

非公平锁，加锁时AQS流程——

1. 调用接口中的lock方法

```java
public interface Lock {

    void lock();
}
```

2. 子类ReentrantLock实现了接口的方法

```java
public void lock() {
    sync.lock();
}
```

3. 调用类内部类Sync的lock抽象方法，Sync结成了AQS抽象类

```java
abstract static class Sync extends AbstractQueuedSynchronizer {
        private static final long serialVersionUID = -5179523762034025860L;

        /**
         * Performs {@link Lock#lock}. The main reason for subclassing
         * is to allow fast path for nonfair version.
         */
        abstract void lock();
}
```

4. Sync子类NonfairSync实现了lock抽象方法

```java
final void lock() {
    if (compareAndSetState(0, 1))
        setExclusiveOwnerThread(Thread.currentThread());
    else
        acquire(1);
}
```

5. 修改state状态标志位成功则抢占锁成功，失败则调用acqure方法

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

6. 调用tryAcquire方法抢占锁，使用模板设计模式，父类不提供实现实现方法

```java
protected boolean tryAcquire(int arg) {
    throw new UnsupportedOperationException();
}
```

7. 调用子类的tryAcquire方法

```java
protected final boolean tryAcquire(int acquires) {
    return nonfairTryAcquire(acquires);
}
```

8. 调用nonfaireTryAcquire方法（参数为1）

```java
final boolean nonfairTryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```

9. 抢占成功，返回true，抢占失败返回false

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

10. 抢占成功则直接退出，抢占失败则继续推进

```java
private Node addWaiter(Node mode) {
    Node node = new Node(Thread.currentThread(), mode);
    // Try the fast path of enq; backup to full enq on failure
    Node pred = tail;
    if (pred != null) {
        node.prev = pred;
        if (compareAndSetTail(pred, node)) {
            pred.next = node;
            return node;
        }
    }
    enq(node);
    return node;
}
```

11. 创建一个节点，因为tail为空，直接将节点入队

```java
private Node enq(final Node node) {
    for (;;) {
        Node t = tail;
        if (t == null) { // Must initialize
            if (compareAndSetHead(new Node()))
                tail = head;
        } else {
            node.prev = t;
            if (compareAndSetTail(t, node)) {
                t.next = node;
                return t;
            }
        }
    }
}
```

12. 创建虚拟结点，CAS将头节点指向虚拟结点，让尾节点也指向虚拟结点，循环，将需要插入的元素的前继节点指向虚拟结点，CAS将虚拟结点的后虚节点执行虚拟结点，如对成功，然后试图抢占锁

```java
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

```java
final Node predecessor() throws NullPointerException {
    Node p = prev;
    if (p == null)
        throw new NullPointerException();
    else
        return p;
}
```

13. pre是头节点，但是抢占锁失败，调用shouldParkAfterFailedAcquire

```java
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    int ws = pred.waitStatus;
    if (ws == Node.SIGNAL)
        /*
             * This node has already set status asking a release
             * to signal it, so it can safely park.
             */
        return true;
    if (ws > 0) {
        /*
             * Predecessor was cancelled. Skip over predecessors and
             * indicate retry.
             */
        do {
            node.prev = pred = pred.prev;
        } while (pred.waitStatus > 0);
        pred.next = node;
    } else {
        /*
             * waitStatus must be 0 or PROPAGATE.  Indicate that we
             * need a signal, but don't park yet.  Caller will need to
             * retry to make sure it cannot acquire before parking.
             */
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
    }
    return false;
}
```

14. waitStatus等于0，将waitStatus设置为-1，然后循环，再次进入该方法，因为等于-1直接返回true

```java
private final boolean parkAndCheckInterrupt() {
    LockSupport.park(this);
    return Thread.interrupted();
}
```

15. 调用park方法，阻塞线程

非公平锁，加锁时AQS流程——

1. 调用Lock接口中的unlock方法

```java
void unlock();
```

2. 调用接口实现类ReentrantLock的unlock方法

```java
public void unlock() {
    sync.release(1);
}
```

3. 调用 内部类的release方法

```java
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}
```

4. 调用AQS的tryRelease方法，这里使用了模板模式，无法直接使用父类的该方法

```java
protected boolean tryRelease(int arg) {
    throw new UnsupportedOperationException();
}
```

5. 调用子类ReentrantLock中的tryRelease方法，CAS将当前占用设置为null，返回true

```java
protected final boolean tryRelease(int releases) {
    int c = getState() - releases;
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();
    boolean free = false;
    if (c == 0) {
        free = true;
        setExclusiveOwnerThread(null);
    }
    setState(c);
    return free;
}
```

6. 执行unparkSuccessor方法

```java
private void unparkSuccessor(Node node) {
    /*
         * If status is negative (i.e., possibly needing signal) try
         * to clear in anticipation of signalling.  It is OK if this
         * fails or if status is changed by waiting thread.
         */
    int ws = node.waitStatus;
    if (ws < 0)
        compareAndSetWaitStatus(node, ws, 0);

    /*
         * Thread to unpark is held in successor, which is normally
         * just the next node.  But if cancelled or apparently null,
         * traverse backwards from tail to find the actual
         * non-cancelled successor.
         */
    Node s = node.next;
    if (s == null || s.waitStatus > 0) {
        s = null;
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    if (s != null)
        LockSupport.unpark(s.thread);
}
```

7. 将虚拟结点的waitStatus设置为0，然后唤醒unpark，虚拟结点的下一个节点，虚拟结点的下一个节点抢占锁，返回true

```java
final boolean nonfairTryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```

8. 将当前节点的值设置为null，然后断开与虚拟结点的联系，回收虚拟节点

```java
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

取消节点排队流程

![image-20221031144932064](D:\WorkSpace\Note\Picture\image-20221031144932064.png)

```java
private void cancelAcquire(Node node) {
    // Ignore if node doesn't exist
    if (node == null)
        return;

    node.thread = null;

    // Skip cancelled predecessors
    Node pred = node.prev;
    while (pred.waitStatus > 0)
        node.prev = pred = pred.prev;

    // predNext is the apparent node to unsplice. CASes below will
    // fail if not, in which case, we lost race vs another cancel
    // or signal, so no further action is necessary.
    Node predNext = pred.next;

    // Can use unconditional write instead of CAS here.
    // After this atomic step, other Nodes can skip past us.
    // Before, we are free of interference from other threads.
    node.waitStatus = Node.CANCELLED;

    // If we are the tail, remove ourselves.
    if (node == tail && compareAndSetTail(node, pred)) {
        compareAndSetNext(pred, predNext, null);
    } else {
        // If successor needs signal, try to set pred's next-link
        // so it will get one. Otherwise wake it up to propagate.
        int ws;
        if (pred != head &&
            ((ws = pred.waitStatus) == Node.SIGNAL ||
             (ws <= 0 && compareAndSetWaitStatus(pred, ws, Node.SIGNAL))) &&
            pred.thread != null) {
            Node next = node.next;
            if (next != null && next.waitStatus <= 0)
                compareAndSetNext(pred, predNext, next);
        } else {
            unparkSuccessor(node);
        }

        node.next = node; // help GC
    }
}
```

# lock锁升级

## 读写锁使用

无锁——>独占锁——>读写锁——>邮戳锁

ReentrantReadWriteLock——一个资源能够被**多个读线程访问**，或者被**一个写线程访问**，但是不能同时存在读写线程

**读写锁，只允许读读共存，而写读、读写、写写都是互斥的**，只能存在一个写锁但是可以同时存在多个读锁，但是不能同时存在写锁和读锁

**只有在读多写少的场景下，读写锁才具有较高的性能体现**

```java
class MyResource {
    Map<String, String> map = new HashMap<>();
    Lock lock = new ReentrantLock();
    ReentrantReadWriteLock rwLock = new ReentrantReadWriteLock();

    public void write(String key, String value) {
        lock.lock();
        try {
            System.out.println(Thread.currentThread().getName() + "\t" + "正在写入");
            map.put(key, value);
            try {
                TimeUnit.MILLISECONDS.sleep(500);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + "\t" + "完成写入");
        } finally {
            lock.unlock();
        }
    }

    public void read(String key) {
        lock.lock();
        try {
            System.out.println(Thread.currentThread().getName() + "\t" + "正在读取");
            String result = map.get(key);
            try {
                TimeUnit.MILLISECONDS.sleep(200);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + "\t" + "读取完成" + result);
        } finally {
            lock.unlock();
        }
    }
}
public class ReentrantReadWriteLockDemo {
    public static void main(String[] args) {
        MyResource myResource = new MyResource();

        for (int i = 0; i < 10; i++) {
            int finalI = i;
            new Thread(() -> {
                myResource.write(finalI + "", finalI + "");
            }, String.valueOf(i)).start();
        }

        for (int i = 0; i < 10; i++) {
            int finalI = i;
            new Thread(() -> {
                myResource.read(finalI + "");
            }, String.valueOf(i)).start();
        }
    }
}
```

**独占锁不允许被打断**

```java
class MyResource {
    Map<String, String> map = new HashMap<>();
    Lock lock = new ReentrantLock();
    ReentrantReadWriteLock rwLock = new ReentrantReadWriteLock();

    public void write(String key, String value) {
        rwLock.writeLock().lock();
        try {
            System.out.println(Thread.currentThread().getName() + "\t" + "正在写入");
            map.put(key, value);
            try {
                TimeUnit.MILLISECONDS.sleep(500);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + "\t" + "完成写入");
        } finally {
            rwLock.writeLock().unlock();
        }
    }

    public void read(String key) {
        rwLock.readLock().lock();
        try {
            System.out.println(Thread.currentThread().getName() + "\t" + "正在读取");
            String result = map.get(key);
            try {
                TimeUnit.MILLISECONDS.sleep(200);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + "\t" + "读取完成" + result);
        } finally {
            rwLock.readLock().unlock();
        }
    }
}
public class ReentrantReadWriteLockDemo {
    public static void main(String[] args) {
        MyResource myResource = new MyResource();

        for (int i = 0; i < 10; i++) {
            int finalI = i;
            new Thread(() -> {
                myResource.write(finalI + "", finalI + "");
            }, String.valueOf(i)).start();
        }

        for (int i = 0; i < 10; i++) {
            int finalI = i;
            new Thread(() -> {
                myResource.read(finalI + "");
            }, String.valueOf(i)).start();
        }
    }
}
```

使用读写锁，可以存在读读的情况

![image-20221031153621247](D:\WorkSpace\Note\Picture\image-20221031153621247.png)

```java
class MyResource {
    Map<String, String> map = new HashMap<>();
    Lock lock = new ReentrantLock();
    ReentrantReadWriteLock rwLock = new ReentrantReadWriteLock();

    public void write(String key, String value) {
        rwLock.writeLock().lock();
        try {
            System.out.println(Thread.currentThread().getName() + "\t" + "正在写入");
            map.put(key, value);
            try {
                TimeUnit.MILLISECONDS.sleep(500);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + "\t" + "完成写入");
        } finally {
            rwLock.writeLock().unlock();
        }
    }

    public void read(String key) {
        rwLock.readLock().lock();
        try {
            System.out.println(Thread.currentThread().getName() + "\t" + "正在读取");
            String result = map.get(key);
            try {
                TimeUnit.MILLISECONDS.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + "\t" + "读取完成" + result);
        } finally {
            rwLock.readLock().unlock();
        }
    }
}
public class ReentrantReadWriteLockDemo {
    public static void main(String[] args) {
        MyResource myResource = new MyResource();

        for (int i = 0; i < 10; i++) {
            int finalI = i;
            new Thread(() -> {
                myResource.write(finalI + "", finalI + "");
            }, String.valueOf(i)).start();
        }

        for (int i = 0; i < 10; i++) {
            int finalI = i;
            new Thread(() -> {
                myResource.read(finalI + "");
            }, String.valueOf(i)).start();
        }

        try {
            TimeUnit.MILLISECONDS.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        for (int i = 0; i < 3; i++) {
            int finalI = i;
            new Thread(() -> {
                myResource.write(finalI + "", finalI + "");
            }, "new" + i).start();
        }
    }
}
```

![image-20221031153846533](D:\WorkSpace\Note\Picture\image-20221031153846533.png)

读锁没有完成时，写锁无法获得，读写互斥

## 读写锁-锁降级

ReentrantReadWriteLock锁降级：将写入锁降级为读锁，**锁的严苛程度增强成为升级，反之称为降级**

| 特性       | 说明                                                         |
| ---------- | ------------------------------------------------------------ |
| 公平性选择 | 支持非公平（默认）和公平的锁获取方法，吞吐量还是非公平优于公平 |
| 重进入     | 该锁支持重进入，以读写线程为例：读线程在获取了读锁之后，能够再次获取读锁，而写线程在获取了写锁之后可以再次获取写锁，同时也可以获取读锁 |
| 锁降级     | 遵循获取写锁、获取读锁在释放锁的次序，写锁能够降级为读锁     |

1. 如果同一个线程持有了写锁，在没有释放写锁的情况下，它还可以继续获得读锁，这激素hi写锁的降级，降级成为了读锁
2. 规则惯例，先获取写锁，然后获取读锁，再释放写锁的次序
3. 如果释放了写锁，那么就完全转换为读锁

**读锁属于悲观锁**，不能升级为写锁

## StampedLock

stamp——代表了锁的状态，当stamp返回零时，表示线程获取锁失败，而且，但是放锁或者转换锁的时候，都要传入最初获取的stamp值

写锁饥饿问题——

1. 使用公平锁可以一定程度解决这个问题，但是是以系统吞吐量作为代价

2. StampedLock采取乐观锁获取锁后，其他线程尝试获取写锁时**不会被阻塞**，这其实是对读锁的优化，所以，**再获取乐观读锁后，还是需要对结果进行校验**

StampedLock**是不可重入的，危险**（如果一个线程已经持有写锁，再去获取写锁的花就会造成死锁）

三种访问模式——

* Reading（读模式悲观）：功能与ReentrantReadWriteLock的读锁类似
* Writing（写模式）：功能与ReentrantReadWriteLock的写锁类似
* Optimistic reading（乐观读模式）：无锁机制，支持读写并发，**很乐观的认为读取时没有人修改，加入被修改再实现升级为悲观读模式**

```java
    static int number = 37;
    static StampedLock stampedLock = new StampedLock();

    public void write() {
        long stamp = stampedLock.writeLock();
        System.out.println(Thread.currentThread().getName() + "\t" + "---写线程准备");
        try {
            number = number + 13;
        } finally {
            stampedLock.unlockWrite(stamp);
        }
        System.out.println(Thread.currentThread().getName() + "\t" + "---写线程结束修改");
    }

    public void read() {
        long stamp = stampedLock.readLock();
        System.out.println(Thread.currentThread().getName() + "\t" + "---读线程准备");
        try {
            TimeUnit.MILLISECONDS.sleep(4000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        try {
            int result = number;
            System.out.println(Thread.currentThread().getName() + "\t" + "---读线程完成获取" + result);
            System.out.println("写线程没有修改成功");
        } finally {
            stampedLock.unlockRead(stamp);
        }
    }

    public static void main(String[] args) {
        StampedLockDemo stampedLockDemo = new StampedLockDemo();

        new Thread(stampedLockDemo::read, "read").start();

        new Thread(stampedLockDemo::write, "read").start();
    }
}
```

乐观读模式

```java
public class StampedLockDemo {
    static int number = 37;
    static StampedLock stampedLock = new StampedLock();

    public void write() {
        long stamp = stampedLock.writeLock();
        System.out.println(Thread.currentThread().getName() + "\t" + "---写线程准备");
        try {
            number = number + 13;
        } finally {
            stampedLock.unlockWrite(stamp);
        }
        System.out.println(Thread.currentThread().getName() + "\t" + "---写线程结束修改");
    }

    public void optimisticRead() {
        long stamp = stampedLock.tryOptimisticRead();
        int result = number;
        System.out.println(Thread.currentThread().getName() + "\t" + "---读取数据" + stampedLock.validate(stamp));
        for (int i = 0;i < 4; ++i) {
            try {
                TimeUnit.MILLISECONDS.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + "\t" + "---读取数据" + i + "s\t" + stampedLock.validate(stamp));
        }
        if (!stampedLock.validate(stamp)) {
            System.out.println("error");
            long stamp2 = stampedLock.readLock();
            try {
                System.out.println("锁升级——》悲观");
                result = number;
                System.out.println(result);
            } finally {
                stampedLock.unlockRead(stamp2);
            }
        }
        System.out.println(Thread.currentThread().getName() + "\t" + "---读取数据成功" + "\t" + stampedLock.validate(stamp));
    }

    public static void main(String[] args) {
        StampedLockDemo stampedLockDemo = new StampedLockDemo();

        new Thread(stampedLockDemo::optimisticRead, "read").start();

        try {
            TimeUnit.MILLISECONDS.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        new Thread(() -> {
            System.out.println(Thread.currentThread().getName() + "\t写线程进入");
            stampedLockDemo.write();
        }, "write").start();
    }
}
```

StampedLock缺点——

* StampedLock不支持重入，没有Re开头
* StampedLock的悲观读锁和写锁都不支持条件变量
* 使用StampedLock一定不要调用中断操作，不要调用interrupt()方法
