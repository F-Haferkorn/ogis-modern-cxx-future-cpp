# ogis-modern-cxx-future-cpp: #
This GITHUB-REPO is a testbed for a special kind of **Modern C++ CORE LANGUAGE-extension**.

Herein I present the **loop()** - **COMPOUNDs**  related to ITERATION
  - loop(N, ...){}
  - typed_loop(type, N, ...)
  - named_loop_up(id, N, ...){}
  - named_loop_down(id, N,  ..){}


With optional opt-opration expressions. N should be of integral type. 
These new COMPOUNDs can be easily imlemented via the cpp-preprocessor.

### Example: matrix_copy with stride ###
	template<typename TPtr, typename TRowSize, typename TColSize, typename TStrideSize >
	TPtr matrix_copy( TPtr tgt, TPtr src, TRowSize nRows, TColSize nColumns, TStrideSize stride)
        {
		// compiler can  mangled the loop-internal integral-types tgt and src in registers.
                loop(nRows,  tgt+=stride, src+=stride)  // apply strid eafter each row to tgt and src
                        loop(nbofColumms, tgt++, src++) 
                             	*tgt = *src;
                return tgt;
        }
**Advantages**
With recent compilers  this gives no really speed advantage, but is no way slower than a full for(;;){} iteration.

And it increases readability.

Future C/C++ compilers can take advantage the **reduced degree of freedom of the iteration**
and make e.g. heavily  use of ALU-register operation of DSP-Architectures


### history ###
in C/C++ there are the well known compound statements.
 - if(<cond>) {} else{}
 - while(<cond>) {}
 - do {} while(<cond>);
 - for(<decl>; <cond>;<expr>) {}
 - switch(<expr>) { case: ... default: ... }

with the related staments
 - break
 - continue.
 
 These compounds have not been changed since the  time of creation K&R-C and ANSI-C.
 
 +----------------------------------------------------------------

 
But was that really all for C++?
One might partly agreed that a 

 	try{
 	} catch(...)   {}
 
 "block" is somehow a compound stament, too.
 
 Will there *never* be more||other compound-statments?

## Details of the C++ core language Extension ##
### Shortcuts: ###
	{}	a single <statement> || <statement-block>  after the compound statement
	rep    	the <count> of targeted repetitions (usually a value of integral-type) ;
	type	the <type> of the (hiddden) iteration variable
	name	the <name> of the iteration variable
	, ...) 	an optional comma separated list of post-operations (expressions)
		
## SYNTAX: ##
These compounds iterate ("loop") the trailing block "{}" rep times

	/////// below <hidden> is an id with a secret, unique ID.
	// loop the {} - block <rep> times 
	
	// the  hidden_loop does NOT change <rep> and uss a hidden index variable.
	loop(rep){}	                 // for(auto hidden=rep; hidden-- ; ++hidden){}   
	
	// a typed-loop has  a type-constained,but hidden index variable.
	typed_loop(type,rep){} 	        // for(type hidden=rep; hidden-- ; ++hidden){}   

	
	// the named_loops_...() are useful, when access to the index  variable is needed.
	// the named index variable "name" is counting upwards from 0 to rep-1.
	named_loop_up(name,rep){} 	 // for(auto name=0; name<rep; ++name){}     
	
	// the named  index variable "name" is  counting downwards, from rep -1 to 0 
	named_loop_down(name,rep){} 	 // for(auto name=rep-1; name--; ++name){}     

The argument _rep_ is _not_ changed an in best case should have integral-type:
Any float or enum is not allowed for rep.
Compiler can take advantage when index vars <rep> are of integal-type and fit into ALU-registers.


	
## EXTENDED_SYNTAX ##
	// each of the above the loop compound statements 
	// can be extened by any number of post-iteration operations.
	
	// loop <block> rep times 
	loop(rep){}		
	
	// loop <block> rep times with 1 post-operation op1.
	loop(rep, op1 ){}
	
	// loop <block> rep times with 2 postoperations op1, op2  (more are optional).
	loop(rep, op1, op2){}	

	typed_loop(type,rep, op1, ...)){}	// hidden loop with type constraint on the index variable
	named_loop_up(name, cnt, op1, op2, ...){}   // loop upwards with type "auto" a named index variable .	
	named_loop_down(name, cnt, op1, op2, ...){}   // loop deonwards with type "auto" a named index variable.

## IMPLEMENTATION: ##
This "core-language extension" can be implemented  solely using the cpp-preprocessor.
	
	#pragma once

	#define CPPMACRO_UNIQUE_ID()  CPPMACRO_UNIQUE_ID_##_##LINE##_##__LINE__##_##__COUNTER__

	// count-down: )inverse  loop: =
	#define CPPMACRO_NTIMES_COUNT_UP(type, varName, nbrOfRepetitions, ...) \
    		for (type varName = 0; varName++<nbrOfRepetitions; __VA_ARGS__)

	// regular loop: counting-up
	#define CPPMACRO_NTIMES_COUNT_DOWN(type, varName, nbrOfRepetitions, ...) \
    		for (type varName = nbrOfRepetitions; varName--; __VA_ARGS__)

	/// todo: choose the fatest depending on the machine
	#define CPPMACRO_NTIMES_FAST(type, varName, nbrOfRepetitions, ...) \
    		CPPMACRO_NTIMES_COUNT_UP(type, varName, nbrOfRepetitions, __VA_ARGS__)

	//#define loop(nbrOfRepetitions)  CPPMACRO_NTIMES(auto, CPPMACRO_UNIQUE_ID(), nbrOfRepetitions)
	#define loop(nbrOfRepetitions, ...)                 CPPMACRO_NTIMES_FAST(auto, CPPMACRO_UNIQUE_ID(), nbrOfRepetitions, ##__VA_ARGS__)
	#define typed_loop(type, nbrOfRepetitions, ...)     CPPMACRO_NTIMES_FAST(type, CPPMACRO_UNIQUE_ID(), nbrOfRepetitions, ##__VA_ARGS__)
	#define named_loop_up(varName, nbrOfRepetitions, ...)  CPPMACRO_NTIMES_COUNT_UP(auto, varName, nbrOfRepetitions, ##__VA_ARGS__)
	#define named_loop_down(varName, nbrOfRepetitions, ...)  CPPMACRO_NTIMES_COUNT_DOWN(auto, varName, nbrOfRepetitions, ##__VA_ARGS__)



Properties of the solution 
 - the current solution is based on the c--preprocessor (cpp) only.	
 - it is a header only solution
	
Creating the hidden index name  
	CPPMACRO_UNIQUE_ID()   generated an unqiue for the hidden index
	
Outcomes for ANSI-C
 - Even if it has been designed for Modern C++ it works also with a plain ANSI-C compiler.
 - It is implementable using the standard 'cpp' c--preprocessor, only,
	

## DISCUSSION ##
 One might argue, loop(){} is is only a plain mapping to a for(;;){} statement.
 
 ### Advantages: ###
 OK, it is all about iterating and it is not a swiss-army knife for all iterations.
 BUT:
 - READability:  It can reduce C/C++ source code size and improve its readability.
 - TEACHability: it will improve the way to teach C/C++  especially for a younger audiencce.
 - ALGORITHMics: It allows/leads the developer(s) to notate code that completely does _NOT depend on the iteration index_.
 - OPTIMIZATION: It opens the way to enhanced optimizations for upcoming compiler implementations.

### REMARK on TEACHing C/C++: ###
  - ! the UK-Government decided to "force" (childern||pupils) form 4-years on to learn how to programm. !
  - 2nd grade (7years old) pupils CAN cope with the concept of PRINTING, LOOPING and generating textual/graphical outputs. But CONDITIONS with the need of using **boolean Expressions** like AND, OR, NOT are a very hard stuff at that age. 
  Suggestion:
  - What about starting to teach C++ for pupils with: putc(), loop() and going on with vars and  assignment.
  - Forming them to statements to creating textual outputs on the (screen || printer).
  - Assigning, and the operations add, subtract, multiply, divide and modulo are enough challenging  at that age.
  - and teaching in then next, the 3rd grade:  
  	- Conditions inklusive Boolean Algebra.
	. and later functions _and then the power of the full for(;;){} iteration_
  
  What do you think  ?
   - How far COULD the teaching-level for bachelor students in 2034 of Programming (C/C++) for the now 4-year old kindergarden children be?
   - Compared to 2020  bachelor students?
  And what would be their favored programming language ?
    
### REMARK on OPTIMIZAtion: ###
The new iterative coumpounds 
 - have less degree of freedom
 - and will give room for futher enhancements of the compiler optimization.
We will see if:
 - it can lead to SHORTER, COMPACTER machine-code as the itearation is SIMPLER.
 - This will definitely apply for large loops with a tiny loop body.
 - This can reduce the _rate of cache-misses_ dramatically.

### Beware of UNINTENDED-USAGE: ###
	YES, in the presented cpp-implementation,
	the underlying itarator index <rep> is not really "hidden" and could be "guessed" by a experienced programmer.

## WORK to to: ##
 - However I  will have to prove, that there are _realworld USE-CASES_,
 - for cases the  described compounds can be applied
 - so that the praised _"unbeatable"_ _compiler_  
 - can be driven to make better/shorer/faster machine code
 - compared to the now-a-days solution for(int i=0; i<rep; ++i){}
  
## CONCLUSion: ##
First experts comments say that it is is a _stony way_ to get the suggested  core-language _compounds_ wil get its way to C/C++. 
But You can take the occasion and try it out yourself. 
It works. Even for plain Ansi-C See the *IMPLEMENTATION* section.

## ADVANCED Reading: ##
	- Teaching programming in the Kindergarden and later:
 	https://www.raspberrypi.org/forums/viewtopic.php?t=762   a news form 2011 
 	https://www.intechopen.com/books/early-childhood-education/evaluating-a-course-for-teaching-advanced-programming-concepts-with-scratch-to-preservice-kindergart


 
	
