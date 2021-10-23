# safe-c

The goal of this project is the definition of a provable safe subset of C. This is very preliminary and probably full of errors.

The basis is the current working draft for [C23](./c23.mkd) and the draft for the [TS 6010](./ts6010.mkd).

Our preliminary definition for safe code is:

* There is no undefined behavior.
* There are no visible side effects that depend on unspecified behavior.

We also consider a version where certan undefined behavior is defined to trap at run-time.

When using a safe compilation mode, unsafe code must produce a diagnostic, i.e. code which compiles without diagnostic can not encounter UB.

We test with

	gcc -O2 -Wall -fsanitize=undefined


1. We first try to define the existing subset of C which is aleady safe
2. We then define a subset that can easily be proved to be safe
3. We investigate annotations that can be added to make code safe

Race condition: For now, we assume there are no race conditions. We will later see how this restriction can be lifted.


## Safe Code

* trap:
  This code can trap at run-time or call abort():
  
* race:
  Additional assumptions are needed to show that the code does not have race conditions.
  
* provable
  This code can be proved safe using simple proofs.

### Existing Safe Subset

Example 1:
  The following example is safe because unsigned arithmetic is safe
  and the types are promoted to unsigned int.
  ```
	unsigned int foo(unsigned int x)
	{
		return x + 1;
	}
  ```
  
Example 2 (trap):
  This code may trap at run-time.
  ```
	int foo(int x)
	{
	  return x + 1;
	}
  ```

Example 3:
  This example is safe because overflow is avoided using a test.
  ```
	int foo(int x)
	{
		if (x < INT_MAX)
			return x + 1;

		return 0;
	}
  ```
Example 4:
  This example is safe because it uses checked integer functions introduced in [n2684](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2683.pdf)
  ```
  	int foo(int x)
	{
		int result;
		
		if (!ckd_add(&result, x, 1)
			return result;

		return 0;
	}
  ```
  
Example 5 (race):
  This example is safe because 'static' guarantees that the access is valid (i.e. non-null, compatible effective type, not-end-of-life, correctly aligned).
  But need additonal assumptions that there is no race condition (i.e. no ther access or free using 'p').
  ```
	int foo(int p[static 1])
	{
		return p[0];
	}
  ```

### Provable Safe Subset

We start simple: We assume that if-conditions accumulate constraints on the possible range of
integer variables. The condition must be relational operator where one side must be a constant.
Ranges are tracked inside if and else branches.
```
unsigned int y = 1;
if (x < 3)
{
  [[assume: x < 3]];
  if (0 < x)
  {
    [[assume: (0 < x)]];
    [[assume: (x < 3)]];
    y >> x; // ok, requires (0 < x) && (x < width)
  }
}
```


## Undefined Behavior in the C Standard

Classification:

* CTD: compile-time diagnosable
  This is diagnosable at a compile-time.
* RTD: run-time diagnosable (without tracking additional state)
  This is diagnosable at run-time without tracking state which is expensive to track.
* !!!: difficult to diagnose
  This is expensive to diagnose because additional abstract machine state would have to be tracked (e.g. uninitialized memory, etc.)


Current working draft: [n2731](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2731.pdf)

* [Expressions](./expr.mkd)
* [Constant Expressions](./cexpr.mkd)
* [Declarations](./decl.mkd)
* [Statements and Blocks](./stat.mkd)
* [External Definitions](./edef.mkd)
