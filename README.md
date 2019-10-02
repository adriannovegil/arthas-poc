# Arthas monitoring tool PoC

`Arthas` is a Java Diagnostic tool open sourced by Alibaba.

Arthas allows developers to troubleshoot production issues for Java applications without modifying code or restarting servers.

## Key features

 * __Check whether a class is loaded?__ Or where the class is loaded from? (Useful for trouble-shooting jar file conflicts)
 * __Decompile__ a class to ensure the code is running as expected.
 * Check classloader statistics, e.g. the number of classloaders, the number of classes loaded per classloader, the classloader hierarchy, possible classloader leaks, etc.
 * Check the __method invocation details__, e.g. method parameter, returned values, exceptions and etc.
 * Check the __stack trace__ of specified method invocation. This is useful when a developer wants to know the caller of the method.
 * __Trace the method invocation__ to find slow sub-invocations.
 * Monitor method invocation statistics, e.g. QPS (Query Per Second), RT (Return Time), success rate and etc.
 * __Monitor system metrics__, thread states and CPU usage, GC statistics and etc.
 * Supports command line interactive mode, with __auto-complete__ feature enabled.
 * Supports __telnet__ and __WebSocket__, which enables both local and remote diagnostics with command line and browsers.
 * Supports __JDK 6+__
 * Supports __Linux/Mac/Windows__

## Prepare the environment

The repository contains the binary to test the application.

To prepare the environment execute the following steps:

 * Open a terminal and execute the demo application:
   ```
   $ cd arthas-3.1.3-bin/
   $ java -jar arthas-demo.jar
   ```

   `arthas-demo` is a simple program that generates a random number every second, then find all prime factors of the number.

 * Open another terminal and execute the Arthas binary:

   The user to run this command MUST have the same privilege as the owner of the target process, as a simple example you can try the following command if the target process is managed by user admin: sudo su admin && java -jar arthas-boot.jar or sudo -u admin -EH java -jar arthas-boot.jar

   If you cannot be able to attach to the target process, please check the logs under ~/logs/arthas for troubleshooting.

   java -jar arthas-boot.jar -h print usage.

   ```
   $ cd arthas-3.1.3-bin/
   $ java -jar arthas-boot
   ```

   Select the target Java process to attach:

   ```
   $ java -jar arthas-boot.jar
     * [1]: 35542
       [2]: 71560 arthas-demo.jar
  ```

  The __Demo__ process is the second as shown above, press __2__ then `Enter`. Arthas will attach to the target process, and start to output:

  ```
  [INFO] Try to attach process 71560
  [INFO] Attach process 71560 success.
  [INFO] arthas-client connect 127.0.0.1 3658
    ,---.  ,------. ,--------.,--.  ,--.  ,---.   ,---.
   /  O  \ |  .--. ''--.  .--'|  '--'  | /  O  \ '   .-'
  |  .-.  ||  '--'.'   |  |   |  .--.  ||  .-.  |`.  `-.
  |  | |  ||  |\  \    |  |   |  |  |  ||  | |  |.-'    |
  `--' `--'`--' '--'   `--'   `--'  `--'`--' `--'`-----'

  wiki: https://alibaba.github.io/arthas
  version: 3.0.5.20181127201536
  pid: 71560
  time: 2018-11-28 19:16:24

  $
  ```

## Commands to test

Following you can see a list of different comands that you can execute to test
the Arthas funcionality

### Dashboard

This is the real time statistics dashboard for the current system, press Ctrl+C to exit.

```
$ dashboard
```

### Search all the classes loaded by JVM

Search classes loaded by JVM.

sc stands for search class. This command can search all possible classes loaded by JVM and show their information. The supported options are: `[d]`,`[E]`,`[f]` and `[x:]`.

Wildcards match search

```
$ sc demo.*
```

View class details

```
$ sc -d demo.MathGame
```

View class fields

```
$ sc -d -f demo.MathGame
```

### Search the method of classes loaded by JVM

Search method from the loaded classes.

sm stands for search method. This command can search and show method information from all loaded classes. `sm` can only view the methods declared on the target class, that is, methods from its parent classes are invisible.

```
$ sm com.taobao.arthas.core.shell.system.JobController
```

### Show class loader info

View hierarchy, urls and classes-loading info for the class-loaders.

`classloader` can search and print out the URLs for a specified resource from one particular classloader. It is quite handy when analyze `ResourceNotFoundException`.

```
$ classloader
```

### Decompile Class

Decompile the specified classes.

`jad` helps to decompile the byte code running in JVM to the source code to assist you to understand the logic behind better.

The decompiled code is syntax highlighted for better readability in Arthas console.

It is possible that there’s grammar error in the decompiled code, but it should not affect your interpretation.

To decompile the code, Artha use [CFR](https://www.benf.org/other/cfr/) (yet another java decompiler).

```
$ jad com.taobao.arthas.core.shell.system.JobController
```

### Monitor method invocation

Monitor invocation for the method matched with `class-pattern` and `method-pattern`.

`monitor` is not a command returning immediately.

A command returning immediately is a command immediately returns with the result after the command is input, while a non-immediate returning command will keep outputting the information from the target JVM process until user presses `Ctrl+C`.

On Arthas’s server side, the command is running as a background job, but the weaved code will not take further effect once the job is terminated, therefore, it will not impact the performance after the job quits. Furthermore, Arthas is designed to have no side effect to the business logic.

Items to monitor:

 * `timestamp` timestamp
 * `class` Java class
 * `method` method (constructor and regular methods)
 * `total` calling times
 * `success` success count
 * `fail` failure count
 * `rt` average RT
 * `fail-rate` failure ratio

To monitor the method with a monitor interval of 5 seconds, an execution time treshold of 10, we’re using the following command:

```
$ monitor -c 5 -n 10 demo.MathGame primeFactors
```

### Show Stacktrace

Print out the full call stack of the current method.

Most often we know one method gets called, but we have no idea on which code path gets executed or when the method gets called since there are so many code paths to the target method. The command `stack` comes to rescue in this difficult situation.

Filtering by condition expression:

```
$ stack demo.MathGame primeFactors 'params[0]<0' -n 2
```

Filtering by cost:

```
$ stack demo.MathGame primeFactors '#cost>5'
```

### Display thread info, thread stack

Check the basic info and stack trace of the target thread.

CPU ratio for a given thread is the CPU time it takes divided by the total CPU time within a specified interval period. It is calculated in the following way: sample CPU times for all the thread by calling `java.lang.management.ThreadMXBean#getThreadCpuTime` first, then sleep for a period (the default value is 100ms, which can be specified by `-i`), then sample CPU times again. By this, we can get the time cost for this period for each thread, then come up with the ratio.

Note: this operation consumes CPU time too (`getThreadCpuTime` is time-consuming), therefore it is possible to observe Arthas’s thread appears in the list. To avoid this, try to increase sample interval, for example: 5000 ms.

```
$ thread
```

thread id, show the running stack for the target thread

```
$ thread 1
```

### Trace the execution time

Trace method calling path, and output the time cost for each node in the path.

`trace` can track the calling path specified by `class-pattern` / `method-pattern`, and calculate the time cost on the whole path.

```
$ trace demo.MathGame run
```

Ignore JDK metehods

```
$ trace -j demo.MathGame run
```

Filtering by cost

```
$ trace -j demo.MathGame run '#cost > 10'
```

### Display parameters, return objects, and thrown exceptions for method invocations

Monitor methods in data aspect including `return values`, `exceptions` and `parameters`.

With the help of [OGNL](https://commons.apache.org/proper/commons-ognl/index.html), you can easily check the details of variables when methods being invoked.

F.Y.I

 * any valid OGNL expression as `"{params,returnObj}"` supported
 * there are four watching points: `-b`, `-e`, `-s` and `-f` (the first three are off in default while -f on);
 * at the watching point, Arthas will use the expression to evaluate the variables and print them out;
 * `in parameters` and `out parameters` are different since they can be modified within the invoked methods; `params` stands for `in parameters` in `-b` while `out parameters` in other watching points;
 * there are no `return values` and `exceptions` when using `-b`.

Check the `out parameters` and `return value`:

```
$ watch demo.MathGame primeFactors "{params,returnObj}" -x 2
```

### Check the parameters, return values and exceptions of the methods at different times.

Check the `parameters`, `return values` and `exceptions` of the methods at different times.

`watch` is a powerful command but due to its feasibility and complexity, it’s quite hard to locate the issue effectively.

In such difficulties, `tt` comes into play.

With the help of `tt` (TimeTunnel), you can check the contexts of the methods at different times in execution history.

 * `INDEX`	the index for each call based on time
 * `TIMESTAMP`	time to invoke the method
 * `COST(ms)`	time cost of the method call
 * `IS-RET`	whether method exits with normal return
 * `IS-EXP`	whether method failed with exceptions
 * `OBJECT`	hashCode() of the object invoking the method
 * `CLASS`	class name of the object invoking the method
 * `METHOD`	method being invoked

```
$ tt -t demo.MathGame primeFactors
```

### Dump the bytecode for the particular classes to the specified directory.

Dump the bytecode for the particular classes to the specified directory.

```
$ dump demo.*
```

### Reading and Setting System Properties

Examine the system properties from the target JVM

```
$ sysprop
```

Now we'll test how to change the user.country propety value:

```
$ sysprop user.country
$ sysprop user.country US
$ sysprop user.country
$ sysprop user.country ES
$ sysprop user.country
```

## References

 * https://github.com/alibaba/arthas
 * https://alibaba.github.io/arthas/en/monitor.html
 * https://github.com/qunarcorp/bistoury
 * https://www.hascode.com/2018/10/analyzing-java-applications-on-the-fly-with-arthas/
