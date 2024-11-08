 - Presentation
	 - need to know what type of band the TPA will do
 - Reducing software risk
	 - Checking numeric bounds
		 - avoid using platform-specific integer types
			 - instead of int and unsigned, use int32_t and uint32_t
			 - in this way we know the bounds of these types are constant across platforms
			 - only use platform specific if it's required by a library or some sort of system interface
		 - checking integer conversions 
			 - properly converting from 64bit integer to 32bit integers is a good way of doing things, so that data doesn't get malformed 
			 - you can use `std::limits` to get the limits of integer types
		 - checking integer overflow
			 - for example if you're adding two int32 values, you can add them into an int64, but need to make sure you type cast those values to int64 during the addition so that you're doing 64bit addition and not 32 bit addition
		 - beware of floating point precision loss
	 - use bounded loops 
		 - make sure your loops are bounded by something
			 - for example, instead of running until the queue is empty, we run until for the length of the queue, because there's the possibility that something continues to feed the queue, which makes the queue essentially infinitely long 
	 - avoid recursion
		 - it's a form of a potentially unbounded loop
			 - if it goes longer than the stack can handle, that's not good
	 - use assertions
		 - macro expression
			 - ASSERT(e)
				 - e is a boolean expression that's expected to evaluate to true
				 - if e does evaluate to true, the macro does nothing
				 - otherwise an assertion failure occurs
					 - in unit testing, this usually halts the pogram
					 - at JPL generally they restart when an assertion failure occurs, unless a restart would cause a mission failure
				 - **test as we fly, fly as we test**
			 - use assertions when failure "cannot happen" as in it's meant to just indicate a bug
		 - ASSUME INCOMING DATA IS HOSTILE UNTIL PROVEN OTHERWISE
			 - you're allowed to assume your code is correct
	 - static analysis
		 - compile with `--Wall -Werror` - turns warnings into errors
		 - run tools
			 - free tools such as GitHub actions
			 - other tools such as Coverity and CodeSonar
		 - run the tools as often as feasible
			 - compiler: on every build
			 - GitHub actions: on every pull request to main
			 - other tools: on every code review
	 - follow a coding standard
		 - create a agreed-upon set of rules for developing code
		 - there's JPL's C and C++ coding standard
			 - there's also MISRA*, AUTOSAR*, and Google's C++ style guide
		 - power of 10 is also a good coding standard 
 - coding style guide
	 - encapsulate operations on data
	 - avoid inline operations on data structures
		 - ex: separate the search operation from the creation of the map
	 - Helper class in C++
		 - you can nest classes in c++
		 - inner classes have access to private members, which can be nice
		 - doesn't have access to members of the outer function
			 - like it cannot use `this.something` like how java does
	 - factor functions into layers
		 - have functions do high-level structure
			 - then have those types of functions call lower level functions that implement the detailed work
	 - avoid multiple return statements within the same function
		 - can make it hard to follow logic, they can also lead to resource leaks
		 - generally, follow this pattern for functions:
			 - claim resources
			 - do work
			 - release resources
			 - return
		 - instead of writing multiple return statements based on several if statements, write out to a variable your return value, then at the very end you should return that variable
	 - avoid using member variables of classes when you're just trying to pass information from one function to another
		 - put data as a member of a class, only if it's relevant to the classes state 
		 - instead move the state update outside of the function, or rename the function to announce the fact that it's updating the state...ex: `calculateSizeAndUpdateState`
	 - avoid using boolean flag arguments
		 - instead use a custom enum, so that you can clearly state what types your variables are 
	 - don't use boolean values for return status
		 - instead use custom enums to return success and fail
			 - true doesn't exactly mean success, and false doesn't exactly mean fail
	 - avoid using %d %u and %x for printf formats
		 - use fixed width equivalents...ex: `PRId32, PRIu32, PRIx32`
	 - don't leave memory uninitialized unless required, 
		 - ex: write `I32 x = 0;` instead of `I32x;`
			 - even if there's no meaningful initial value, just pick one
	 - be carful when using pointers and references wisely
		 - use pointers only when the value is assigned after the variable is created 
	 - AVOID:
		 - using malloc after initalization
		 - allocating large objects on the stack
		 - allocating variable-size arrays on the stack
		 - prefer simple memory allocation when possible
			 - objects are statically sized, thus you can statically allocate them
			 - F' provides a `MemAllocator` interface that you can use
	 - avoid bare array access
		 - not safe due to bounds not being checked 
	 - use `const` as much as possible
		 - learn the different ways to use it
	 - avoid using use of C library String functions
		 - don't use `strcpy` and others things like that as they're not bounds-checked
		 - in fprime, use a subclass of StringBase
			 - for example `operator=` is a safer alternative to `strncpy`
 - data structures in F'
	 - many data structures allocate and free elements on demand
	 - FSW data structures must use a fixed total size of memory to hold their data
	 - insert, remove, find iterate - these are the basic data structure operations
	 - pointers or array indices should be used to mark links between elements
	 - for FSW, prefer using arrays instead of pointers
		 - often you don't need a linked data structure at all, you can just use an array
			 - if you do use links, then make sure all memory is pre-allocated at startup
			 - so all memory has an associated index into a pre-allocated array
			 - using the index is safer, because it can be bound checked 
				 - consider using ETL: https://www.etlcpp.com
		 - recommended use of an array
			 - if it will be dense (most indices are filled and used)
		 - dont use an array if
			 - the number of elements grows or shrinks, or if there's no numeric index set, or if the array is sparsly used
	 - Last in first out stack LIFO
		 - recommended to use if you need this type of behavior
	 - FIFO queue
		 - example can be found in Utils/Types/CircularBuffer.hpp/cpp
	 - set/map data representation 
		 - use if find operation isn't used a lot, or if the number of elements are small
	 - linked list
		 - use if you want several lists to share nodes from a common array
			 - otherwise use an array-based set/map - will be significantly less complex
	 - hash set/map 
		 - us if there's no numeric index set for the data or if the index set is sparse, or if the data set is large and you need fast find capability
			 - example: `TlmChan`component in F Prime
 - 