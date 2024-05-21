They recommend you read this https://nasa.github.io/fpp/fpp-users-guide.html#Defining-Types then this https://nasa.github.io/fpp/fpp-spec.html
  - GDS limitations 
	  - F Prime still uses the old XML format that they used to in order to support the GDS. 
		  - Some FPP features aren't supported by the old XML format, so limitations will be noted when necessary 
		  - Because of this, some FPP features aren't supported at all by the GDS yet, this will also be pointed out when needed
  - FPP - F prime prime - Modeling Language
	  - `fpp-check file.fpp` - tool to check if an FPP model is valid - can also use standard in: `fpp-check < file.fpp` - no output means the provided code is good
		  - without any given parameters it'll prompt for you to type in code, then you press ctrl+D to have it check the code 
	  - constants
		  - to use a reserved word (such as `constant`) as a constant's name, you must put the '$' character in front of the name
			  - ex. `$constant` is a valid name
		  - Strings
			  - escape sequence for Strings = `\`
				  - can do `\"` to display quotation marks
			  - multiline strings use triple quotes - `"""`
		  - Arrays - can have at most 256 elements
			  - cannot put different types into the same array, but if they're easily convertible (such as an int and a float), then it's fine
			  - if you're not sure if type conversion is allowed, just check using fpp-check
		  - structs - struct value expression; represents a C or C++ style structure, such as a mapping of names to values
			  - struct member consists of a name, equal sign, and value - `x = 1`
			  - order of members doesn't matter
			  - can use structs in arrays, just using curly braces to denote it's a struct
				  - `constant something = [{x=2, y=1}, {z=1, u=12}]`
		  - order of constant declarations don't matter, thus the following code is fine:
			```
			const a = b;
			const b = 2;
			```
		 - you can separate code by semicolons, or simply by newlines 
		 - can create an explicit line continuation by using \
	 - Comments & annotations
		 - comments begin with a `#` and are only displayed in the FPP file
		 - annotations being with either an `@` or `@<` and are displayed in the generated C++ file - everything after the `@` or `@<` is considered an annotation until the newline
			 - `@` denotes a pre-annotation - annotates the element that follows it
			 - `@<` denotes a post-annotation - annotates the element that's before it
	 - Modules - group of elements - basically used for organizational purposes; used in place of object oriented programming kinda
		 - to access constants declared in a module, you do `ModuleName.constantName`
		 - example:
		```
		module M {
			constant a = 1;
		}
		constant b = M.a;
		# constant b will be equal to 1
		```
		 - you can define modules within other modules, allowing for something like `A.B.c` to be legal
			 - this is accessing constant `c` which is within module `B` which is within module `A`
		 - XML LIMITATION - cannot do nested modules, thus nested modules aren't translated properly to C++ as a workaround, just name the module differently
			 - EX. instead of `module A { module B....` instead write `module A_B {....`
	 - Type definitions - may appear at the top level or inside a module definition - annotatable definition
		 - three kinds of type definitions - Array, Struct, and Abstract
		 - array type - describes the element type and its size 
			 - example: `array A = [3] U32` - array named `A` with 3 values, all of which are unsigned 32-bit integers 
			 - the `size` can be a previously declared constant, just has to be a value greater than 0 and less than or equal to 256
			 - string arrays need to know the maximum size of a string, uses a default maximum size if not specified 
				 - ex. `array A = [3] string size 40` - the maximum amount of characters a string can have is 40
			 - array definitions can have another array definition as its element type, but an array cannot use itself as its type
			 - can assign default values to arrays using the keyword `default` followed by an array - the size of the default array MUST be the same as the size of the array being declared
				 - the array just has to be the correct type; for example if you were to use a default array with booleans for a U8 array declaration that wouldn't work, you can use previously declared arrays as the default 
				 - if no default is specified, then automatic defaults are assigned 
					 - numeric values default to 0, booleans default to 'false', and strings default to `""` 
			 - array's can be supplied with both a default value and a format string
				 - the default value just has to written before the format string
			 - to make an array of arrays, you just make a given array's type='another array'
				 - ex: `array A = [3] U32; array B = [3] A` 
					 - this makes array B an array of 3 A's, which are each an array
			 - to set default values for arrays of arrays
				 - you can set all values to the same thing by making the default one thing, since default values carry over if you use a previously declared array that has a specified default value
					 - ex.  `array A = [2] U32 default 10 # default value is [ 10, 10 ]; array B1 = [2] A # default value is [[10,10], [10,10]]`
					 - since array A has a default value of 10, and array B1 is of type A, the default value for B1 is to make each value 10 also
					 - you can override any inherited defaults by just declaring a new `default` for a given array
				 - you can also just declare the default value of the array of array's if you just re-write the entire structure manually 
					 - ex. `array B4 = [2] A default [[1,2], [3,4]]`
		 - format - use the keyword `format` followed by the character sequence `{}` to represent where values will be 
			 - `{c}` - character value 
			 - `{d}` - decimal value
			 - `{x}` - hexadecimal value
			 - `{o}` - octal value
			 - `{e}` - rational value in exponent notation - ex. `1.234e2`
			 - `{f}` - rational value in fixed-point notation - `123.4`
			 - `{g}` - rational value in general format (fixed-point up until larger sizes, at which point it'll start using exponent notation)
			 - `c, d, x, and o` - require integer type
			 - `e, f, g` - require floating-point type
				 - can also specify precision - ex. `{.3f}`; fixed-point notation with a precision of 3
			 - to use the curly bracket in the output, do `{{` or `}}`
			 - no format specified will just output the element in its default format; equivalent to a format sequence of just `{}`
		 - Struct type - yet another name for the same structure that's called a Dictionary in python and a Hash Table in java. It's not the EXACT same structure as a Dictionary or a Hash Table, but it can be thought of as the same thing. 
			 - it's purpose is to specify serialization for the struct
			 - can add pre and post annotations to a struct and any of the elements declared within it
			 - example declaration of a struct: `struct S {x: U32, y: string} default {x = 1, y = "abc"}`
			 - can also specify default values, if none is specified then the automatic default value for each type is used
				 - ex. `constant s = { x = 1, y = "abc"}; struct S1 {x: U8, y: string} default s`
			 - Member Arrays are used instead of Named Arrays for Structs - member arrays are usually used more often than Named Arrays. THE GDS DOESN'T PROPERLY HANDLE MEMBER ARRAYS, SO AVOID USING THEM FOR THE GDS
				 - members arrays generate less code than types, since Member arrays generate a C++ array, whereas a named array generates a C++ class
				 - size of a member array isn't limited to 256 elements
				 - the size of default values doesn't matter, so the following code is accepted:
					 - `struct S { x: [3] U32 } default { x = 10 }`
						 - the member array `x` will be given 3 copies of the default value 10, whereas a name array would have to be given the default value `[10, 10, 10]` to achieve the same thing
			 - When to use Named arrays:
				 - for a small reusable array
				 - want to use the array outside of a structure
				 - for convenience of a generated array class
				 - NOTE: the class that Named Array's generate have an additional degree of memory safety when accessing array elements
			 - Member format strings
				 - example : `struct Channel { offset: U32 format "offset 0x{x}"}` - displays a hexadecimal value
				 - if the member is an array with a size > 1, then the format is applied to each element
			 - For structs, you can use previously defined named types as the members 
				 - ex: `array A = [2] U32; struct S2 { a: A };`
			 - ORDER OF MEMBERS IS VERY IMPORTANT. This is because the order the members are written in is typically the order in which the members will be generated in the code. This means that when serializing, the code will be read from top down.
				 - How this can cause issues:
					 - lets say we have two structs `struct A { x: U32, y: string}; struct B { y: string, x: U32 }; `
					 - if we're serializing from A to B, that means we're converting `x` from U32 to a string `y`, then taking that string `y` and converting it back to U32: this will just produce an unreadable U32 number
					 - if we were to swap the positioning of the members `x` and `y` in the two structs, then it would convert a string to U32 and then back to a string
				 - default values for members doesn't matter
					 - ex: `struct S { x: U32, y: string } default { y = "abc", x = 5}`
						 - it won't change anything to put `y` before `x` in the default declaration
		 - Abstract type - doesn't have a type definition, very similar to generics in Java
			 - if given the FPP code: `type T; array MyArray = [3] T;`
				 - to convert this to English, this is saying "A type T exists, it's defined in the implementation, not in the model."
				 - the generated code will create: 
					 - `type T` - doesn't generate anything
					 - `array MyArray` - C++ class is generated, creating `MyArrayArrayAc.hpp` and `MyArrayArrayAc.cpp`
						 - within `MyArrayArray.hpp` there's an include for `T.hpp`
						 - the user needs to define the header file `T` in the way they need to
				 - NOTE: there are a few special types that are abstract in the model, but are translated to fully developed implementations during the translation to XML
		 - Enum type - FPP model can contain more than one enum definition
			 - an Enum is kinda like an object, but it can only represent one attribute at any given time
			 - example enum declaration with a few enumerated constants
				 - `enum Decision {YES, NO, MAYBE}`
					 - can also separate each enumerated constant by a newline and without commas
			 - example usage of an enum:
				 - `constant myDecision = Decision.MAYBE`
				 - can also use enums as types and values for array's:
					 - `array Decisions = [3] Decision default Decision.MAYBE`
			 - each enumerated constant has an assigned numeric value, starting from 0 and increasing by 1 for each constant. you can assign specific numeric values yourself:
				 - `enum E { A = 1, B = 2, C = 3 }`
				 - if you choose to give custom values, you must do it for each constant
			 - values must be distinct, meaning that you can't make `A = 1 + 1`, it must be `2`
			 - the default representation type for enum constant values is I32, to specify a different type, put a colon after the name of the enum and add the desired type
				 - ex: `enum HeheHaha : U8 { A, B, C }`
			 - GDS NOTE: The gds will always assume enum constants are I32, so don't change the type if you plan to use an enum with the GDS
			 - default enum constant; you can use constants that are already declared in the enum
				 - `enum Decision { YES, NO, MAYBE } default MAYBE` - don't have to specify `MAYBE`'s type because it's already declared in `Decision`
	 - Ports - ports are connections between two component instances - consist of the port name, the data type that will be carried across the port, and an optional return type 
		 - example port declaration: `port P1()` this is equivalent to `port P`. this port simply has a name and no parameters
			 - `port P2(a: U32, b: F32)` - a port `P2` with two parameters
		 - port parameters can be of any valid type, they can be abstract (generic), arrays, etc...
	 - handler functions - there are output ports and input ports for components
		 - if you invoke an output port, the input port handler function needs to run
			 - and if you invoke an input port, the output port handler function needs to run
		 - each input port can be synchronous or asynchronous, and each port has a type
		 - if the port carries a primitive value (like I32 or U16...) then the port is translated to a C++ value parameter, otherwise the port is translated to a reference parameter
			 - ex. `type T; port P(a: U32, b: T);` will be converted to something like: `virtual void pIn_handler(U32 a, const T& b);`
			 - if an input port is synchronous, then the invocation is a direct call of the input handler function
		 - example input handler:
			 - assume pIn is a synchronous input port `void C2::pIn_handler(U32 a, const T& b) {}` - a is a local copy of a U32 value, b is a constant reference to T data passed in by c1
		 - to make a local copy of the data: `auto b1 = b`
		 - if you wanna have the handler and the invoking component to share data as a parameter, then you can make reference parameters 
			 - ex. `virtual void pIn_handler(U32 a, const T& b, T& c);`
			 - this acts the same as a const reference parameter
			 - the main reason to use a reference parameter is to change the value of the reference parameter and use it as a return value
		 - return values - write an arrow to indicate return value `->` followed by a type - only synchronous ports can do this
			 - ex. `port P1 -> U32` - a port with no parameters that returns a U32
		 - only for synchronous ports does using ref parameters provide another way to return values from a port
			 - ex. `type T; port P(a: U32, b: T, ref c: T)`
				 - the generated code may look something like: `virtual void pIn_handler(U32 a, const T& b, T& c);`
				 - since `b` isn't marked as `ref`, it generates `const T& b`, and since `c` is marked as `ref`, it generates the code `T& c`
			 - when the input port is synchronous, a reference parameter refers to the exact data being given
			 - when the input port is asynchronous, a reference parameter refers to data that was copied and then given 
		 - ports behave very similarly to just a basic function, you can't have more than one thing returned at once, so if you want to return more things you can do so by updating the parameters by having them pass by reference
			 - just be careful about using pass-by-refrence, as it would be very bad if two components were trying to change the same data at the same time
	 - Components
		 - types: active, passive, queued
		 - example of a passive component with no members:
			 - `passive component C {}`
		 - port instances - instantiates a port - definition requires a kind, a name, and a type
			 - kinds:
				 - async input - the input to this component arrives on a message queue, and then dispatched on this components thread (if it's an active component) or on the thread of another port (if this component is queued) - passive components cannot have any amount of these - no return types
				 - sync input - this input invokes a handler defined in the current component, and the runs on the thread of the caller
				 - guarded input - everything is the same as a sync input, except only one thread can use this input at a time
				 - output - output of the component
			 - example component that adds two F32 values and outputs the answer as a F32 value:
				  `port F32Value(value: F32);`
				  `passive component F32Adder {`
					  `sync input port f32ValueIn1: F32Value; `
					  `sync input port f32ValueIn2: F32Value; `
					  `output port f32ValueOut: F32Value;`
		        `}`
				- the input ports instantiated within the component block are the port instances
			- NOTE: port instances are typically referred to as just 'ports'
			- when you're creating a port instance, you're actually creating an array of port instances, the size of which defaults to a size of 1 if not specified and acts as a single element
			- to specify array size:
				- `sync input port f32ValueIn: [2] F32Value` - single array of two input ports 
			- priority - only async ports can be assigned a priority 
				- priority is simply a numerical value, ex: 
					- `async input port f32ValueIn1: F32Value priority 10`
				- default priorities are supplied if none are specified. generally, the priorities are meant to regulate the order that elements are meant to be dispatched from the message queue
					- NOTE: if the message queue is full and now overflowing, a FSW assertion will fail, and F' will have the system restart
					- you can actually specify what you want to happen when the message queue overflows:
						- assert: Fail an FSW assertion (default)
						- block: Block the sender until the queue has space
						- drop: Drop the incoming message and continue as normal
					- example usage of specifying queue full behavior:
						- `async input port f32ValueIn1: F32Value priority 10 block`
						- `async input port f32ValueIn2: F32Value drop`
						- the queue full behavior is specified after priority
			- serial port instances - if a port is a `serial` port, then it can't be of any other type
				- a serial port instance doesn't care about what type of data is going through it, the data is converted to or from a specific type of data at either end of the connection
				- example of a passive component taking serial data and copying it onto 10 streams
					`passive component SerialSplitter {`
						`sync input port serialIn: serial`
						`output port serialOut: [10] serial`
					`}`
					- this allows for sending several unrelated types of data over the same port connection
						- this can be useful by having a component on either side of a network act as a hub that directs all data to and from components on that side of the network
						- the only downside to using this technique is that you loose compile-time type checking 
			- special port instances - needs a predefined behavior that's provided by the F' framework
				- six special port behaviors:
					- commands
					- events
					- telemetry
					- parameters
					- time 
					- data products
			- command ports - a way to send instruction to the satellite to perform an action
				- keywords for the special command behaviors: (if a component receives commands then all three port types are required)
					- `command reg`: a port for sending command registration requests
						- ex: `command reg port cmdRegOut`
					- `command recv`: a port for receiving commands
						- ex: `command recv port cmdIn`
					- `command resp`: a port for sending command responses
						- `command resp port cmdResponseOut`
				- any component can have at most one kind of each command port 
				- as the FPP file is translated to C++, the ports are converted to typed port instances and use a predefined port type:
					- `recv` uses port `Fw.Cmd`
					- `reg` uses port `Fw.CmdReg`
					- `resp` uses port `Fw.CmdResponse`
					- the definitions for these ports can be found in `Fw/Cmd`, however to check basic examples you need to use this simple definition before the component declaration otherwise `fpp-check` won't pass:
						`module Fw {`
							`port Cmd`
							`port CmdReg`
							`port CmdResponse`
						`}`
						- these definitions won't actually work, but it'll allow for you to use `fpp-check` on the file without having to worry about the `Fw` definitions
			- event ports - events are meant to mark when something important has happened
				- two types of ports:
					- `event`: port for emitting events as serialized bytes 
						- ex: `event port eventOut`
						- when converted this uses `Fw.Log`
					- `text event`: port for emitting events as human-readable text (this is typically used for testing and debugging on the ground)
						- ex: `text event port textEventOut`
						- when converted this uses `Fw.LogText`
					- simplified version of the definitions of `Fw.Log` and `Fw.LogText` (MEANT FOR TESTING ONLY)
						`module Fw {`
							`port Log`
							`port LogText`
						`}`
			 - telemetry ports - data regarding the state of the system - any component can at most have one telemetry port
				 - example telemetry port: `telemetry port tlmOut`
				 - basic version of the definition found in `Fw/Tlm`:
				 `module Fw {`
					 `port Tlm`
				 `}`
			 - parameter ports - configurable constant that can be updated from the ground, current parameter values are stored in the F Prime component **parameter database**
				 - `param get`: port for getting the current value of a parameter from the databse
					 - ex: `param get port prmGetOut`
				 - `param set`: port for setting the current value of a parameter in the database
					 - ex: `param set port prmSetOut`
				 - simplified version of the definitions of `Fw.PrmGet` and `Fw.PrmSet` (MEANT FOR TESTING ONLY) - actual definition found in `Fw/Prm`
						`module Fw {`
							`port PrmGet`
							`port PrmSet`
						`}`
			 - time get ports - allows component to get the system time from a time component 
				 - example: `time get port timeGetOut`
				 - simplified version of the definition of `Fw.Time`, actual definition found in `Fw/Time`:
					 `module Fw {`
						 `port Time`
					 `}`
			 - Data product ports - collection of data that can be stored onto an onboard file system, then given a priority, and then downlinked in their priority order - a component can have at most one of each type of product port
				 - data products are stored in containers, these containers hold records which are the units of data, and they also hold a header that describes their contents
				 - different types of product behaviors:
					 - `product get`: port meant for synchronously requesting a memory buffer to store a container - uses `Fw.DpGet`
						 - ex: `product get port productGetOut`
					 - `product request`: port for asynchronously requesting a buffer to store a container - uses `Fw.DpRequest`
						 - ex: `product request port productRequestOut`
					 - `product recv`: port for receiving a response to an asynchronous buffer request - uses `Fw.DpResponse`
						 - ex: `async product recv port productRecvIn priority 10 assert`
					 - `product send`: port for sending a buffer that stores a container, after the container has been filled with data - uses `Fw.DpSend`
						 - ex: `product send port productSendOut`
				 - if there's any amount of data product ports, there must be a product get port and a product send port
				 - definitions for all the typed ports can be found in `Fw/Dp`, here's a simple definition:
					 `module Fw {`
						 `port DpGet`
						 `port DpRequest`
						 `port DpResponse`
						 `port DpSend`
					 `}`
			 - internal ports - port that a component can use to send messages to itself - simply used as a replacement if an output and input port reside in the same component - really only make sense for active and queued components
				 - example, instead of this:
					 `type T`
					 `port P(t: T)`
					 `active component ExternalSelfMessage {`
						 `async input port pIn: P`
						 `output port pOut: P`
					 `}`
				  - we can do this:
					  `type T`
					  `active component InternalSelfMessage {`
						  `internal port pInternal(t: T)`
					  `}`
				  - an internal port is like two ports: an output port and an async input port, fused into one
				  - internal ports don't need a named definition, they can just be declared directly within a component 
				  - you can add priority and queue-full behavior to internal ports
					  - `internal port pInternal(t: T) priority 10 drop`
		  - commands - commands are sent from the GDS 
			  - three kinds of commands:
				  - async: command arrives on a message queue, to be dispatched on this component's thread (if the component is active) or on the thread of a port invocation (if the component is queued)
				  - sync: command invokes a handler defined in the current component, and runs on the thread of the caller
				  - guarded: same as sync input, but it can only be accessed by one thread at a time 
			  - example:
				  `active component Action {`
					  `command recv port cmdIn`
					  `command reg port cmdRegOut`
					  `command resp port cmdResponseOut`
					  `async command START`
					  `async command STOP`
				  `}`
				  - explaination: 
					  - `START` is asynchronous which will make it so that when a `START` command is dispatched to an instance of this component, it's put on a queue. after some time the message will be taken off the queue and called to the corresponding handler on the thread of the component 
					  - `STOP` is synchronous, which means that the command will run immediately on the thread of the invoking component, since the command runs immediately, its handler should be very short. 
						  - for example the handler could set a stop flag and then exit
					  - notice that there's three command ports, this is because all three of those command ports are required for any component that has commands
					  - async commands also REQUIRE a message queue, so they're allowed only for active and queued components
			  - formal parameters:
				  - you may specify one or more formal parameters when specifying a command - parameters are bound to arguments when the command is sent to the satellite 
				  - example of a switch component with two states: `ON` and `OFF`
					  `enum State {`
						  `OFF`
						  `ON`
					  `}`
					  `active component Switch {`
						  `command recv port cmdIn`
						  `command reg port cmdRegOut`
						  `command resp port cmdResponseOut`
						  `async command SET_STATE (`
							  `state: State`
						  `)`
					  `}`
			  - opcodes - number that uniquely identifies the command, also has a name but this is mainly just for human interaction from the ground
				  - typically opcodes start at 0
					  - to specify your own opcode, just use the keyword `opcode` followed by a numerical value
						  - ex: `async command COMMAND_2(a: F32, b: U32) opcode 0x10`
							  - this is also showing the opcode expressed as a hexadecimal to display that an opcode can be expressed as any numerical value
						  - this may be obvious, but don't set two commands to have the same opcode
							  - ex. `async command COMMAND; async command COMMAND_2 opcode 0;` - this is bad because the default opcode given to `COMMAND` is already `0`, so assigning `0` to `COMMAND_2` makes it so the two commands now have the same opcode
			  - you can also assign priority and queue-full behavior
				  - ex. `async command COMMAND_3(a: string) opcode 0x10 priority 30 drop`
		  - Events 
			  - 
			  - 