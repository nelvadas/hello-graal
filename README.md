# Getting started with GraalVM 

This helloworld project introduce you to Graal Technology 


## Setup 

Download and install [GraalVM](oracle.com/graalvm) 
```
$ sdk install java 23-graal
$ java version "23" 2024-09-17
Java(TM) SE Runtime Environment Oracle GraalVM 23+37.1 (build 23+37-jvmci-b01)
Java HotSpot(TM) 64-Bit Server VM Oracle GraalVM 23+37.1 (build 23+37-jvmci-b01, mixed mode, sharing)
```

## How fast is GraalVM JIT Compiler 

1. Run the application with Open JDK JIT 

```sh
$sdk use java 23-open

Using java version 23-open in this shell.
$ java -version
openjdk version "23" 2024-09-17
OpenJDK Runtime Environment (build 23+37-2369)
OpenJDK 64-Bit Server VM (build 23+37-2369, mixed mode, sharing)
$ time java  -cp target/*.jar com.oracle.graalvm.App
Hello World!
java -cp target/*.jar com.oracle.graalvm.App  0,04s user 0,03s system 14% cpu 0,444 total

2. Running the application GraalVM JIT Compiler 

```sh 
$sdk use java 23-graal

Using java version 23-graal in this shell.
$ time java  -cp target/*.jar com.oracle.graalvm.App
Hello World!
java -cp target/*.jar com.oracle.graalvm.App  0,06s user 0,02s system 125% cpu 0,067 total
```

you can rely on :
* `cpulimit` command to constraint your application on a specific number of cores.
* `time` command to display the time spent by your application in both user and system space 
*  `-XX:StartFlightRecording=duration=60s,filename=jit23-graal.jfr` option to collect JFR event 

```sh 
$ cpulimit -l 2 time java -XX:StartFlightRecording=duration=60s,filename=jit23-graal.jfr -cp target/*.jar com.oracle.graalvm.App
[0,264s][info][jfr,startup] Started recording 1. The result will be written to:
[0,264s][info][jfr,startup]
[0,264s][info][jfr,startup] …/hello-graal/jit23-graal.jfr
Hello World!
        0,31 real         0,36 user         0,06 sys

```



## Build  Native Image from application 
``$ mvn clean package 
``

``` 
$ native-image -cp target/*.jar  com.oracle.graalvm.App helloworld
=======================================================================================================================
GraalVM Native Image: Generating 'MyApp' (executable)...
========================================================================================================================
[1/8] Initializing...                                                                                    (3,7s @ 0,17GB)
 Java version: 23+37, vendor version: Oracle GraalVM 23+37.1
 Graal compiler: optimization level: 2, target machine: armv8-a, PGO: ML-inferred
 C compiler: cc (apple, arm64, 15.0.0)
 Garbage collector: Serial GC (max heap size: 80% of RAM)
 1 user-specific feature(s):
 - com.oracle.svm.thirdparty.gson.GsonFeature
------------------------------------------------------------------------------------------------------------------------
Build resources:
 - 26,49GB of memory (73,6% of 36,00GB system memory, determined at start)
 - 11 thread(s) (100,0% of 11 available processor(s), determined at start)
[2/8] Performing analysis...  [*****]                                                                    (2,6s @ 0,19GB)
    1 985 reachable types   (57,5% of    3 452 total)
    1 769 reachable fields  (38,2% of    4 631 total)
    8 548 reachable methods (35,2% of   24 316 total)
      730 types,     7 fields, and    90 methods registered for reflection
       49 types,    33 fields, and    48 methods registered for JNI access
        4 native libraries: -framework Foundation, dl, pthread, z
[3/8] Building universe...                                                                               (0,4s @ 0,24GB)
[4/8] Parsing methods...      [*]                                                                        (0,7s @ 0,24GB)
[5/8] Inlining methods...     [***]                                                                      (0,5s @ 0,29GB)
[6/8] Compiling methods...    [***]                                                                      (6,5s @ 0,39GB)
[7/8] Laying out methods...   [*]                                                                        (0,7s @ 0,24GB)
[8/8] Creating image...       [*]                                                                        (0,6s @ 0,30GB)
   2,83MB (44,16%) for code area:     3 861 compilation units
   3,38MB (52,73%) for image heap:   52 735 objects and 86 resources
 203,65kB ( 3,11%) for other data
   6,40MB in total
```
A new binary is available in the project directory 
```sg
$ ls -rtlah
total 14600
-rw-r--r--@  1 enono  staff   943B 26 sep 13:28 graalvm-helloworld-nativeimage.iml
-rw-r--r--@  1 enono  staff   940B 26 sep 13:30 pom.xml
drwxr-xr-x@  4 enono  staff   128B 26 sep 13:40 src
-rw-r--r--@  1 enono  staff    86B 26 sep 13:55 .gitignore
drwxr-xr-x@ 15 enono  staff   480B 26 sep 13:56 .git
drwxr-xr-x@  6 enono  staff   192B 26 sep 13:56 ..
drwxr-xr-x@ 10 enono  staff   320B 26 sep 15:17 target
-rw-r--r--@  1 enono  staff   350K 26 sep 15:32 jit23-open.jfr
-rw-r--r--@  1 enono  staff   368K 26 sep 15:46 jit23-graal.jfr
-rw-r--r--@  1 enono  staff   4,4K 26 sep 15:51 README.md
drwxr-xr-x@ 12 enono  staff   384B 26 sep 15:53 .
-rwxr-xr-x@  1 enono  staff   6,4M 26 sep 15:53 helloworld
```

Use the `--emit build-report` option introduces with GraalVM 23 to generate a build report .
[]!(./images/image.png)

## Run the native Binary 
with GraalVM Native image you can run your application whitout a JVM!
 
```shell 
$ ./helloworld 
Hello World!
```

## Handling Reflexion 

Build the JAR and run the AppWithReflection class.
It depends on `Greeter.sayHello` ( Reflexion) to greet guests using the provided name as a parameter.

```sh
java -cp target/helloworld-1.0-SNAPSHOT.jar com.oracle.graalvm.AppWithReflexion com.oracle.graalvm.Greeter sayHello Marc
Hello Marc!

```

### Naive Native image 
Build the native Binary  using :
```sh
native-image -cp target/*.jar  com.oracle.graalvm.AppWithReflexion helloworld2
```

Run the helloworld2 script with the previous parameters

```sh
❯ ./helloworld2  com.oracle.graalvm.Greeter sayHello Marc

Exception in thread "main" java.lang.ClassNotFoundException: com.oracle.graalvm.Greeter
	at org.graalvm.nativeimage.builder/com.oracle.svm.core.hub.ClassForNameSupport.forName(ClassForNameSupport.java:215)
	at org.graalvm.nativeimage.builder/com.oracle.svm.core.hub.ClassForNameSupport.forName(ClassForNameSupport.java:183)
	at java.base@24/java.lang.Class.forName(DynamicHub.java:1509)
	at java.base@24/java.lang.Class.forName(DynamicHub.java:1468)
	at java.base@24/java.lang.Class.forName(DynamicHub.java:1461)
	at com.oracle.graalvm.AppWithReflexion.main(AppWithReflexion.java:36)
	at java.base@24/java.lang.invoke.LambdaForm$DMH/sa346b79c.invokeStaticInit(LambdaForm$DMH)
  ```

You have a ClassNotFoundException


### Fixing the Relfexion 

#### With Native Image tracing Agent 

1. Create a  `META-INF/native-image` directory

```sh 
 mkdir -p META-INF/native-image
 ```
 
2. Run the application with the GraalVM Tracing Agent as follow

```sh
java  -agentlib:native-image-agent=config-output-dir=META-INF/native-image -cp target/helloworld-1.0-SNAPSHOT.jar com.oracle.graalvm.AppWithReflexion com.oracle.graalvm.Greeter sayHello Marc
Hello Marc!
```

The execution also create a configuration file `reachability-metadata.json`

```sh
hello-graal on  main [!?] is 📦 v1.0-SNAPSHOT via ☕ v24 on ☁️  (us-east-2)
❯ ls META-INF/native-image
reachability-metadata.json
``` 

3. Explore the configuration 

```sh
❯ cat reachability-metadata.json
{
  "reflection": [
    {
      "type": "com.oracle.graalvm.Greeter",
      "methods": [
        {
          "name": "sayHello",
          "parameterTypes": [
            "java.lang.String"
          ]ETA-INF/native-image
        }
      ]
    }
  ]
}
```

4. Rebuild the native image 

```sh
native-image -cp target/helloworld-1.0-SNAPSHOT.jar:META-INF/native-image/reachability-metadata.json com.oracle.graalvm.AppWithReflexion helloworld2
```


5. ReRun the application 
By default the native image process will lookup configuraiton in META-INF/native-image/


#### With Reachability Meta Data
