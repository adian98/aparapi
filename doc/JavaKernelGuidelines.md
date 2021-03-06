#JavaKernelGuidelines
*What code can and can't be converted to OpenCL by Aparapi. Updated Sep 13, 2011 by frost.g...@gmail.com*
##Aparapi Java Kernel Guidelines
Certain practices can improve the chances of your Java kernel being converted to OpenCL and executing on a GPU.

The following guidelines/restrictions only apply to the Kernel.run() method and any method reachable from run() (called” run-reachable methods” in this documentation), clearly any methods executed via a normal Java execution path will not be subject to these restrictions.

Some restrictions/guidelines may be removed or augmented in a future Aparapi releases.

##Data Types
* Only the Java primitive data types boolean, byte, short, int, long, and float and one-dimensional arrays of these primitive data types are supported by Aparapi.
* Aparapi support for the primitive data type double will depend on your graphics card, driver, and OpenCL version. Aparapi will query the device/platform to determine if double is supported (at runtime). If your platform does not support double, Aparapi will drop back to (Java Thread Pool) (JTP) mode.
* The primitive data type char is not supported.

##Fields
* Elements of primitive array fields can be read from kernel code.
* Elements of primitive array fields can be written to by kernel code.
* Note that Java creates 'hidden' fields for captured final primitive arrays (from anonymous inner classes) and they can be accessed as if they were fields of the kernel.
* Primitive scalar fields can only be read by the kernel code. Because kernel run-reachable methods execute in parallel in an indeterminate order, any reliance on the result of modifications to primitive scalar fields is discouraged even when executing in Java Thread Pool mode.
* Static final fields can be read from kernel code.
* Static non-final fields are not supported for either read or write. Try to make them final.

##Arrays
* Only one-dimensional arrays are supported.
* Arrays cannot be aliased either by direct local assignment or by passed arguments to other methods.
* Java 5’s extended 'for' syntax for (int i: arrayOfInt){} is not supported, because it causes a shallow copy of the original array under the covers.

##Methods
* References to or through a Java Object other than your kernel instance will cause Aparapi to abandon attempting to create OpenCL (note the following exceptions).
* There are a few very specific exceptions to the above rule to allow accesses through getters/setters of objects held in arrays of objects referenced from the kernel code.
* Static methods are not supported by Aparapi.
* Recursion is not supported, whether direct or indirect. Aparapi tries to detect this recursion statically, but the developer should not rely on Aparapi to do so.
* Methods with varargs argument lists are not supported by Aparapi.
* Overloaded methods (i.e. methods with the same name but different signatures) are not supported by Aparapi. OpenCL is C99 based so we are constrained by OpenCL's lack of support for overloading.
* The kernel base class contains wrappers around most of the functions offered by java.lang.Math.  When run in a thread pool these wrappers delegate back to java.lang.Math when executing in OpenCL they translate to OpenCL equivalents.

##Other Restrictions

* Exceptions are not supported (no throw, catch. or finally).
* New is not supported either for arrays or objects
* Synchronized blocks and synchronized methods are not supported.
* Only simple loops and conditionals are supported; switch, break, and continue are not supported.
* A variable cannot have its first assignment be the side effect of an expression evaluation or a method call.  For example, the following will not be translated to run on the GPU.


        int foo(int a) {
           // . . .
        }
        public void run() {
          int z;
          foo(z = 3);
        }


* This should be regarded as an error which needs to be addressed, as a workaround, explicitly initialize variables (even to 0) when declared.

## Beware Of Side Effects
OpenCL is C99-based and as such the result of expressions depending on side effects of other expressions can differ from what one might expect from Java, please avoid using code that assumes Java's tighter rules.  Generally code should be as simple as possible.
For example, although Java explicitly defines

    arra[i++] = arrb[i++];
  to be equivalent to

    arra[i] = arrb[i+1];
    i += 2;

The C99/OpenCL standard does not define this and so the result would be undefined.

##Runtime Exceptions
* When run on the GPU, array accesses will not generate an ArrayIndexOutOfBoundsException.  Instead the behavior will be unspecified.
* When run on the GPU, ArithmeticExceptions will not be generated, for example with integer division by zero. Instead the behavior will be unspecified.
Attribution
