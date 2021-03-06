#UsersGuide
*Aparapi User's Guide. Updated Sep 14, 2011 by frost.g...@gmail.com*
##User’s Guide
Aparapi is: An API used to express data parallel workloads in Java and a runtime system capable of running compatible workloads on a compatible GPU.

Where your workload runs depends on

Whether you have a compatible GPU and OpenCL capable device driver
Whether your Java parallel code can be converted to OpenCL by Aparapi
For information about restrictions on the code that Aparapi can convert to OpenCL, see JavaKernelGuidelines.
Aparapi depends on AMD’s OpenCL™ driver to execute on the GPU and therefore shares the same device, driver, and platform compatibility requirements as AMD APP SDK V2.5®.

* 32-bit Microsoft® Windows® 7
* 32-bit Microsoft® Windows Vista® SP2
* 64-bit Microsoft® Windows® 7
* 64-bit Microsoft® Windows Vista® SP2
* 32-bit Linux® OpenSUSE™ 11.2,   Ubuntu® 10.04/9.10, or Red Hat® Enterprise Linux® 5.5/5.4
* 64-bit Linux® OpenSUSE™ 11.2,   Ubuntu® 10.04/9.10, or Red Hat® Enterprise Linux® 5.5/5.4
* An OpenCL GPU and suitable OpenCL enabled device driver
* An installed AMD APP SDK v2.5 or later

If you prefer to test Aparapi in JTP mode (Java Thread Pool) then you will only need Aparapi.jar and Oracle Java 6 or later JRE or JDK.
The following fragment of Java code takes an input float array and populates an output array with the square of each element.

    final float in[8192]; // initialization of in[0..8191] omitted
    final float out[in.length];

    for(int i=0; i<in.length; i++){
       out[i]=in[i]*in[i];
    }
This code segment illustrates an ideal data parallel candidate, each pass through the loop is independent of the others. Traversing the loop in any order should provide the same result.

To convert the above code to Aparapi we use an anonymous inner-class (a common Java idiom) to express the data parallel nature of the above sequential loop.

    Kernel kernel = new Kernel(){
       @Override public void run(){
          int i = getGlobalId();
          out[i]=in[i]*in[i];
       }
    };
    kernel.execute(in.length);
Java developers should recognize the general pattern as similar to that used to launch a new Thread.

    Thread thread = new Thread(new Runnable(){
       @Override public void run(){
           System.out.println(“In another thread!”);
       }
    });
    thread.start();
    thread.join();
The Aparapi developer extends the com.amd.aparapi.Kernel and overrides the public void Kernel.run() method. It is this Kernel.run() method that is executed in parallel.

The base class also exposes the Kernel.execute(range) method which is used to initiate the execution of Kernel.run() over the range 0...n.

Kernel.execute(range) will block until execution has completed. Any code within the overridden ‘void run()’ method of Kernel (and indeed any method or methods reachable from that method) is assumed to be data-parallel and it is the developer’s responsibility to ensure that it is. Aparapi can neither detect nor enforce this.

Within the executing kernel (on the GPU device or from the thread pool) the Kernel.getGlobalId() method is used to identify which (of the range 0..n) a particular execution represents.

## Compiling an Aparapi application
Aparapi has only two compilation requirements:

Aparapi.jar must be in the class path at compile time.
The generated class files must contain debug information (javac –g)
A typical compilation might be:
    $ javac –g –cp ${APARAPI_DIR}/aparapi.jar Squares.java
Aparapi requires this classfile debug information so that can extract the name and scope of local variables for the generated OpenCL.

## Running an Aparapi application
At runtime an Aparapi-enabled application requires aparapi.jar to be in the class path to be able to execute in a Java Thread Pool (no GPU offload).

    $ java–cp ${APARAPI_DIR}/aparapi.jar;. Squares
To take advantage of the GPU, the directory containing the platform-dependent Aparapi shared library is passed via the java.library.path property.

    $ java –Djava.library.path=${APARAPI_DIR} –cp ${APARAPI_DIR}/aparapi.jar;. Squares

Aparapi detects whether the JNI shared library is available. If the library cannot be located your code will be executed using a Java Thread Pool.

An application can detect whether a kernel was executed on the GPU or by a Java Thread Pool (JTP) by querying the execution mode ‘after’ Kernel.execute(range) has returned. This is achieved using the Kernel.getExecutionMode() method.

    Kernel kernel = new Kernel(){
       @Override public void run(){
          int i = getGlobalId();
          out[i]=in[i]*in[i];
       }
    };
    kernel.execute(in.length);
    if (!kernel.getExecutionMode().equals(Kernel.EXECUTION_MODE.GPU)){
       System.out.println(“Kernel nid not execute on the GPU!”);
    }

To obtain a runtime report of the execution mode of all kernel executions, set the com.amd.aparapi.enableExecutionModeReporting property to true when the JVM is launched.

    $ java –Djava.library.path=${APARAPI_DIR} –Dcom.amd.aparapi.enableExecutionModeReporting=true –cp ${APARAPI_DIR}/aparapi.jar;. Squares

##Running the sample applications
Aparapi includes two sample applications in the /samples subdirectory of the binary distribution zip file.

samples/squares	simple example that computes an array of squares of integers
samples/mandel	computes and displays the Mandelbrot set
The jar file for each sample is included (so you can run a sample without having to build it) as well as both Linux® and Microsoft Windows® script files for launching the samples.

You will need an appropriate GPU card, OpenCL® enabled Catalyst® driver and a compatible Oracle Java 6 JRE for your platform. To execute a sample:

Set the environment variable JAVA_HOME to point to the root of your JRE or JDK.
Change to the appropriate samples directory (samples/squares or samples/mandel)
Run either the .bat or .sh script. On Linux® , you might have to initially chmod +x script.sh to add execute permissions.
The sample scripts pass the first arg (%1 or $1) to -Dcom.amd.aparapi.executionMode when the JVM is launched. This allows the sample to be tested in either GPU or JTP execution modes by passing the requested mode.

    $ cd samples/mandel
    $ bash ./mandel.sh GPU
    <executes in GPU mode here>
    $ bash ./mandel.sh JTP
    <executes in JTP mode here>

## Building the sample applications
To build a sample, install Oracle® JDK 6 and Apache Ant (at least 1.7.1).

Set the environment variable ANT_HOME to point to the root of your ant install.
Ensure that the %ANT_HOME%/bin or ${ANT_HOME}/bin is in your path.
Set the environment variable JAVA_HOME to point to the root of your JDK.
Change to the appropriate samples directory (sample/squares or sample/mandel).
Initiate a build using ant.
    $ cd samples/mandel
    $ ant
    $ bash ./mandel.sh GPU
Attribution