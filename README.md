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

```
$ dashboard
```

### Search all the classes loaded by JVM

```
$ sc demo.*
```

### View class details

```
$ sc -d demo.MathGame
```

### View class fields

```
$ sc -d -f demo.MathGame
```

### Search the method of classes loaded by JVM

```
$ sm com.taobao.arthas.core.shell.system.JobController
```

### Show class loader info

```
$ classloader
```

### Decompile Class

CFR - yet another java decompiler.

```
$ jad com.taobao.arthas.core.shell.system.JobController
```

### Monitor

Monitor invocation for the method matched with class-pattern and method-pattern.

 * `timestamp` timestamp
 * `class` Java class
 * `method` method (constructor and regular methods)
 * `total` calling times
 * `success` success count
 * `fail` failure count
 * `rt` average RT
 * `fail-rate` failure ratio

So to monitor the method with a monitor interval of 5 seconds, an execution time treshold of 10, we’re using the following command:

```
$ monitor -c 5 -n 10 demo.MathGame primeFactors
```

### Show Stacktrace

#### Filtering by condition expression

```
$ stack demo.MathGame primeFactors 'params[0]<0' -n 2
```

#### Filtering by cost

```
$ stack demo.MathGame primeFactors '#cost>5'
```

### Display thread info, thread stack

```
$ thread
```

### Trace the execution time

```
$ trace demo.MathGame run
```

#### Ignore JDK metehods

```
$ trace -j demo.MathGame run
```

#### Filtering by cost

```
$ trace -j demo.MathGame run '#cost > 10'
```

### Display parameters, return objects, and thrown exceptions for method invocations

Check the out parameters and return value

```
$ watch demo.MathGame primeFactors "{params,returnObj}" -x 2
```

### Check the parameters, return values and exceptions of the methods at different times.

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

```
$ dump demo.*
```

# Reading and Setting System Properties

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
