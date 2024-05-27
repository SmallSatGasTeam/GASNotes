They recommend you read this https://nasa.github.io/fpp/fpp-users-guide.html#Defining-Component-Instances_Init-Specifiers_Execution-Phases then this https://nasa.github.io/fpp/fpp-spec.html
Also this is a good place to find info pertaining to FPP tools https://github.com/nasa/fpp/wiki/Tools
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
			```FPP
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
		```FPP
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
				```FPP
				  port F32Value(value: F32);
				  passive component F32Adder {
					  sync input port f32ValueIn1: F32Value; 
					  sync input port f32ValueIn2: F32Value; 
					  output port f32ValueOut: F32Value;
		        }
		        ```
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
					```FPP
					passive component SerialSplitter {
						sync input port serialIn: serial
						output port serialOut: [10] serial
					}
					```
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
						```FPP
						module Fw {
							port Cmd
							port CmdReg
							port CmdResponse
						}
						```
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
						```FPP
						module Fw {
							port Log
							port LogText
						}
						```
			 - telemetry ports - data regarding the state of the system - any component can at most have one telemetry port
				 - example telemetry port: `telemetry port tlmOut`
				 - basic version of the definition found in `Fw/Tlm`:
				```FPP
				 module Fw {
					 port Tlm
				 }
				```
			 - parameter ports - configurable constant that can be updated from the ground, current parameter values are stored in the F Prime component **parameter database**
				 - `param get`: port for getting the current value of a parameter from the databse
					 - ex: `param get port prmGetOut`
				 - `param set`: port for setting the current value of a parameter in the database
					 - ex: `param set port prmSetOut`
				 - simplified version of the definitions of `Fw.PrmGet` and `Fw.PrmSet` (MEANT FOR TESTING ONLY) - actual definition found in `Fw/Prm`
					```FPP
						module Fw {
							port PrmGet
							port PrmSet
						}
					```
			 - time get ports - allows component to get the system time from a time component 
				 - example: `time get port timeGetOut`
				 - simplified version of the definition of `Fw.Time`, actual definition found in `Fw/Time`:
				```FPP
					 module Fw {
						 port Time
					 }
				```
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
					 ```FPP
					 module Fw {`
						 port DpGet
						 port DpRequest
						 port DpResponse
						 port DpSend
					 }
					 ```
			 - internal ports - port that a component can use to send messages to itself - simply used as a replacement if an output and input port reside in the same component - really only make sense for active and queued components
				 - example, instead of this:
					 ```FPP
					 type T
					 port P(t: T)
					 active component ExternalSelfMessage {
						 async input port pIn: P
						 output port pOut: P
					 }
					 ```
				  - we can do this:
					  ```FPP
					  type T
					  active component InternalSelfMessage {
						  internal port pInternal(t: T)
					  }
					  ```
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
				  ```FPP
				  active component Action {
					  command recv port cmdIn
					  command reg port cmdRegOut
					  command resp port cmdResponseOut
					  async command START
					  async command STOP
				  }
				  ```
				  - explaination: 
					  - `START` is asynchronous which will make it so that when a `START` command is dispatched to an instance of this component, it's put on a queue. after some time the message will be taken off the queue and called to the corresponding handler on the thread of the component 
					  - `STOP` is synchronous, which means that the command will run immediately on the thread of the invoking component, since the command runs immediately, its handler should be very short. 
						  - for example the handler could set a stop flag and then exit
					  - notice that there's three command ports, this is because all three of those command ports are required for any component that has commands
					  - async commands also REQUIRE a message queue, so they're allowed only for active and queued components
			  - formal parameters:
				  - you may specify one or more formal parameters when specifying a command - parameters are bound to arguments when the command is sent to the satellite 
				  - example of a switch component with two states: `ON` and `OFF`
					  ```FPP
					enum State {
						  `OFF`
						  `ON`
					  }
					  active component Switch {
						  command recv port cmdIn
						  command reg port cmdRegOut
						  command resp port cmdResponseOut
						  async command SET_STATE (
							  state: State
						  )
					  }
					  ```
			  - opcodes - number that uniquely identifies the command, also has a name but this is mainly just for human interaction from the ground
				  - typically opcodes start at 0
					  - to specify your own opcode, just use the keyword `opcode` followed by a numerical value
						  - ex: `async command COMMAND_2(a: F32, b: U32) opcode 0x10`
							  - this is also showing the opcode expressed as a hexadecimal to display that an opcode can be expressed as any numerical value
						  - this may be obvious, but don't set two commands to have the same opcode
							  - ex. `async command COMMAND; async command COMMAND_2 opcode 0;` - this is bad because the default opcode given to `COMMAND` is already `0`, so assigning `0` to `COMMAND_2` makes it so the two commands now have the same opcode
			  - you can also assign priority and queue-full behavior
				  - ex. `async command COMMAND_3(a: string) opcode 0x10 priority 30 drop`
		  - Events - something that emits a serialized event report that can be stored on-board the satellite or sent to the ground 
			  - have levels of severity
				  - activity high
				  - activity low
					  - ex. `event Event1 severity activity low format "Event 1 occured"`
				  - command
				  - diagnostic
				  - fatal
				  - warning high
					  - ex. `event Event3 severity warning high format "Event 3 occured"`
				  - warning low
					  - ex. `event Event2 severity warning low format "Event 2 occured"`
			  - format - literal string for use in a display or event log
			  - formal parameters - bound to arguments, when the component instance emits the event; argument values appear in the formatted text that describes the event
				  - these are the exact same as for port definitions, except there's no reference parameter (aka no parameter can be pass-by-reference)
			  - NOTE: the order in which the parameters are stated are the order in which they're gonna have to be used in the format string. example:
				```FPP
					enum Case {A, B, C};
					array ThreeThings = [3] F64;
					passive component EventParameters {
					   @ Sample output: "Saw case A with value [ 1.2, 2.7, 3.2 ]"
					   event Event2( case: Case, value: ThreeThings ) \
					   severity warning low \
					   format "Saw case {} with value {}"
					}
				```
			  - Identifiers - like command opcodes, it's just a unique identifier for events
				  - example usage:
					  - `event Evvent1 severity activity low id 0x10 format "Event 1 occured"`
			  - Throttling - just in case the code decides to send out a lot of warnings at once
				  - example usage:
					  - `event Event1 severity warning high format "Event 1 occured" throttle 10` - the component that this event is a part of will stop emitting this event after it has already emitted it 10 times
		  - Telemetry channels - a channel to transfer data - one or more telemetry channels can be defined in a component 
			  - Telemetry point - individual piece of data being sent as telemetry
			  - A component is REQUIRED to have a telemetry port and a time get port if it has any amount of telemetry channels.
				```FPP
				passive component BasicTelemetry {
					telemetry port tlmOut;
					time get port timeGetOut;
				
					telemetry Channel1: U32;
				}
				```
			  - Example telemetry channel:
				  - `telemetry Channel1: U32` - you can also make it's type a previously defined constant
			  - Telemetry channels also have identifiers
				  - ex. `telemetry Channel1: U32 id 0x10;`
			  - Update frequency
				  - `always` - Emit a telemetry point (new piece of data) whenever the component implementation calls the auto generated function that emits the telemetry
				  - `on change` - Emit a telemetry point (new piece of data) if:
					  1. The implementation calls the auto generated function
					  2. The auto generated function hasn't ever been called 
					  3. The last time the auto generated function was called, it had a different value (telemetry point)
				 - Choosing which frequency depends on how much load you want to put on the system at once, since `always` would be sending information more often than `on change` in most scenarios, but would be providing more up-to-date data points (telemetry points) than `on change` would
				 - example usage - uses the keyword `update` followed by either `always` or `on change`:
					 - `telemetry Channel2: F64 id 0x10 update on change`
					 - `telemetry Channel3: F64 id 0x11 update always`
			 - format - can also format the telemetry - uses the keyword `format` that comes after the `update` keyword if there is one
				 - example: `telemetry Channel2: F64 id 0x10 update on change format "{.3f}"`
			 - limits - bounds for the expected values carried by the channel
				 - two kinds of limits
					 - low - telemetry should stay above this limit
					 - high - telemetry should stay below this limit
				 - each kind of limit can have one of three levels of severity:
					 - yellow: crossing the limit is a low concern
					 - orange: crossing the limit is a medium concern
					 - red: crossing the limit is a high concern
				 - example - the numbers that come after the colors are the threshold limits for each of the severities:
				  ```FPP
					  telemetry Channel2: F64 id 0x10 \
					  update on change \
					  format "{.3f}" \
					  low { red -3, orange -2, yellow -1} \
					  high {red 3, orange 2, yellow 1 }
				```
				- NOTE: XML representation doesn't allow for limits for telemetry channels that are specifically of array or struct type
		- Parameters - typed constant value that can be updated by a command - F Prime has a built-in database system to deal with parameters 
			- example parameter:
				- `param Param1: U32`
			- there's also some required commands and parameter ports are required if you want to use any amount of parameters:
				```FPP
				command recv port cmdIn
				command reg port cmdRegOut
				command resp port cmdResponseOut

				param get port prmGetOut
				param set port prmSetOut
				```
				 - F prime will use these to automatically generate commands for setting the local parameter in the component, and saving the local parameter to a system-wide database
			 - default values - uses the word `default` followed by the default values
				 - example:
					 - `param Param3: F64 default 2.0`
			 - identifiers - unique id to represent the parameter
				 - example
					 - `param Param2: F64 default 2.0 id 0x10`
			 - Set & Save opcodes
				 - set - setting the opcode locally in the component
				 - save - opcode to be saved to the system-wide database
				 - if no other opcodes were created before these, the set will start at opcode 0, and the save will start at opcode 1
				 - example 
				```FPP
					@ Implicitly stated that set opcode is 0x00 and save is 0x01
					param Param1: U32 default 1;
			
					param Param2: F64 \
						default 2.0 \
						id 0x10 \
						set opcode 0x10 \
						save opcode 0x11
					
					@ Param3's set opcode is 0x12, and its save is 0x20
					param Param3: F64 \
						default [ 1.0, 2.0, 3.0 ] \
						save opcode 0x20
				```
		- Data products - collection of related data that's stored on the satellite and then sent to the ground
			- F Prime's built in components for dealing with Data Products include:
				- Managing buffers that store data products in memory
				- Writing data products to the file system
				- Cataloging stored data products for downlink using priority as order
			- Container - what a data product is represented as - Components can only use data products that they created
				- one container holds one data product, each data product is stored in its own file
				- container consists of:
					- A header which provides info about the container (such as the size of the data)
					- Binary data representing a list of serialized records
						- Record = unit of data
				- example record and container 
					- `product record Record1: I32`
					- `product container Container1`
					- NOTE: records aren't tied to specific containers, however if a Component has a container or a record specifier, it also needs to have a record or a container specifier
				- Example usage with id's
					- `product record Record2: Data id 0x10`
					- `product container Container1 id`
				- Array & Struct records - records have to have a static size, so just make sure that the array or struct that you're using has a specified size 
					- `product record DataArrayRecord: Data array` - the number of elements remains unspecified, but is provided when the record is serialized into a container
		- Notes on constants and types
			- Constants and type definitions can be written within components - typically this should be done for organizational purposes
				- to access these definitions outside of the component, it's very similar to accessing something that's a part of a module 
					- if *T* is a constant defined within a component *C* then to access *T* you would have to write `C.T`
						- when converted to C++ it will look like `C::T` 
			- Note about XML limitation:
				- XML cannot convert the definition of constants and types to be members of C++ components 
					- instead it makes everything a C++ class
		- FPP include files
			- if you don't wanna have a really long FPP file, and want to separate it out you can
				- you just have to use the suffix `.fppi` instead of `.fpp` to make it an include file 
					- for example you could define a bunch of commands in the `.fppi` file and then import it to your `.fpp` file
			- to include a `.fppi` file: 
				- `include "Example.fppi"` - this is called an Include Specifier 
		- Matched ports
			- basically if you have two ports, an input and an output, that have the same amount of ports, you can use the special keyword `match` to match the two together and gain special support from F Prime
				- it will then automatically number a topology 
			- Why use matched ports?
				- they make it super easy to identify what's connected and provides more modularity 
			- example:
			```FPP
			ouput port pingOut: [3] Svc.Ping
			async input port pingIn: [3] Svc.Ping
			match pingOut with pingIn
			```
	- Topology - Component Instances 
		- kinds of component instance definitions: passive, queued, active
		- parts of a component instance 
			- name of the component definition it will be representing
			- keywords `base id` followed by that component's base ID
				- this is so that different components are on different id's entirely, and all the types and members within the components will have id's that build off of this `base id`
		- passive component
			- example:
				```FPP
				module Sensors {
					passive component EngineTemp {
						......
					}
				}
				
				module FSW {
					instance engineTemp: Sensors.EngineTemp base id 0x100
				}
				```
			 - XML note: the XML dictionary requires that each component instance have a distinct base ID
			 - queued components 
		 - queued components
			 - just like instantiating a passive component but with one extra element
				 - `queue size` - comes after `base id` 
					 - specifies how big the queue will be 
			 - example 
				  ```FPP
					 module Sensors {
						 queued component EngineTemp {
							 ....
						 }
					 }
					
					module FSW {
						instance engineTemp: Sensors.EngineTemp base id 0x100 \
							queue size 10
					}
				```
		- Active components 
			- just the same as instantiating a queued component, but with a few new things
				- Queue size (required)
				- Stack size (optional) - keywords `stack size` followed by the desired size in bytes
				- Priority (optional) - keyword `priority` followed by a numeric value representing the priority
			- example (also displays how you can create modules to organize stuff):
				```FPP
					module Utils {
						active component DataCompressor {
							.....
						}
					}
					
					module FSW {
						module Default {
							constant queueSize = 10;
							constant stackSize = 10 * 1024;
						}
						
						instance dataCompressor: Utils.DataCompressor base id 0x100 \
							queue size Default.queueSize \
							stack size Default.stackSize \
							priority 30
					}
				```
			 - CPU affinity (optional) - its meaning depends on the platform, but it's usually an instruction to the operating system to run the thread of the active component on a particular CPU, identified by a number
				 - example usage:
					 ```FPP
					 instance dataCompressor: Util.DataCompressor base id 0x100 \
						queue size Default.queueSize \
						stack size Default.stackSize \
						priority 30 \
						cpu 0	 
					```
			 - Implementation type
				 - The translator can automatically know the implementation type if its C++ class name matches with the name of the FPP component 
					 - ex. the C++ class `A::B` matches with the FPP component name `A.B`
				 - but if it for whatever reason doesn't you can specify the type like this:
					```FPP
					instance dataCompressor: Utils.DataCompressor base id 0x100 \
						type "Utils::SpecialDataCompressor" \
						queue size Default.queueSize \
						cpu 0
					```
			 - Header file
				 - the generator is able to automatically identify the header file if it conforms to these rules:
					 - the name of the header file is `Name.hpp` where `Name` is also the name of the component in the FPP model
					 - the header fil is in the same directory as the FPP source file that defines the component 
				 - if the file doesn't follow these rules, you can specify the location of the header file using the keyword `at` followed by it's location relative to the location of the FPP file you're currently using
				 - example:
					```FPP
					instance linuxTime: Svc.Time base id 0x4500 \
						type "Svc::LinuxTime" \ 
						at "../../Svc/LinuxTime/LinuxTime.hpp"
					```
			- Init specifiers - it would be more accurate to call these "setup or teardown specifiers", as they're ran when the FSW sets up or tears down everything 
				- you can write one or more *init specifiers* to specify any special things you'd like to do upon the FSW's startup
					- these are snippets of C++ code that you write, and that will be ran upon startup or when the FSW closes
				- Execution phases 
					- A topology is a unit of an FPP model that specifies the top-level structure of an F Prime application (component instances and their connections)
					- When the C++ code for topology is generated, it'll create `NameTopologyAc.hpp` and `NameTopologyAc.cpp`
					- some definitions:
						- phase: the symbol denoting the execution phase
							- generated file: the file for the topology that contains the definition: either the `.hpp` or `.cpp` file
							- intended use: the intended use of the C++ code snippet 
							- where placed: where fpp places the code snippet in the generated file
							- default code: whether fpp generates default code if there's no init specifier. if there is then it replaces the default code
					- the generated code is separated into many phases of execution:
						- configConstants:
							- generates: `TopologyAc.hpp`
							- use: C++ constants for use in constructing and initializing an instance
							- where: in the namespace `ConfigConstants::ComponentInstance`
								- `ComponentInstance` is just an example name meant to represent the name of a component instance
							- default: none
						- configObjects:
							- generated: `TopologyAc.cpp`
							- use: Statically declared C++ objects for use in constructing and initializing the given instance 
							- where: in the namespace `ConfigObjects::ComponentInstance`
							- default: none
						- instances:
							- generated: `TopologyAc.cpp`
							- use: constructor for a given instance that has an unconventional constructor format
							- where: anonymous (file-private) namespace
							- default code: standard constructor call for the given instance
						- initComponents:
							- generated: `TopologyAc.cpp`
							- use: initialization code for a given instance that also has an unconventional constructor format
							- where: in the file-private function `initComponents`
							- default code: standard call to `init` for the current instance
						- configComponents:
							- generated: `TopologyAc.cpp`
							- use: implementation specific configuration code for the given instance
							- where: in the file-private function `configComponents`
							- default code: none
						- regCommands:
							- generated: `TopologyAc.cpp`
							- use: meant for registering the commands of an instance (if there are any) with the command dispatcher.
								- Only required if an instance has a non-standard command registration format
							- where: in the file-private function `regCommands`
							- default code: standard call to `regCommands` if the instance has commands, otherwise there is no default code
						- readParameters:
							- generated: `TopologyAc.cpp`
							- use: meant for reading parameters from a file
								- normally used when an instance is the parameter database
							- where: in the file-private function `readParameters`
							- default code: none
						- loadParameters:
							- generated: `TopologyAc.cpp`
							- use: for loading parameter values from the parameter database
								- required if an instance has an unconventional parameter-loading format
							- where: in the file-private function `loadParameters`
							- default code: none
						- startTasks: 
							- generated: `TopologyAc.cpp`
							- use: code for starting the task of an instance (if there is one)
							- where: the file-private function `startTasks`
							- default code: standard call to `startTasks` if a given instance is an active component; otherwise there's no default code
						- stopTasks:
							- generated: `TopologyAc.cpp`
							- use: code for stopping the task of an instance (if there is one)
							- where: in the file-private function `stopTasks`
							- default code: standard call to `exit` if an instance is an active component; otherwise there's no default code
						- freeThreads:
							- generated: `TopologyAc.cpp`
							- use: code for freeing the thread associated with a given instance
							- where: in the file-private function `freeThreads`
							- default code: standard call to `join` if a given instance is an active component; otherwise there's no default code
						- tearDownComponents:
							- generated: `TopologyAc.cpp`
							- use: code for deallocating all the allocated memory (if there's any) associated with an instance
							- where: in the file-private function `tearDownComponents`
							- default: none
					- Most often `configConstants`, `configObjects`, and `configComponents` are the parts that need code written for. 
						 - often require specific input, that cannot be provided in any other way other than writing an init specifier 
					 - `instances`, `initComponents`, `regCommands`, `readParameters`, and `loadParameters` should never need to be changed unless something didn't follow standard procedure 
						 - the only thing that does need to be changed is that for the parameter database instance, one line of 'special code' is needed to read its parameters
							 - according to ChatGPT-4 free version, this one line of 'special code' is literally just an invocation of the `readParameters` function to initialize it
								 - ex: `paramDb.readParameters();` or `paramDb.init();`
					 - `startTasks`, `stopTasks`, `freeThreads` are only required if the user-written implementation of a component instance manages its own F Prime task
						 - if a standard F Prime active component is used, then the framework manages the task and this code is all autogenerated 
					 - `tearDownComponents` is only required if a component instance needs to deallocate memory or release resources when the program exits
				 - creating an init specifier
					 - may write one or more init specifiers for a component instance definition 
					 - they come at the end of the component definition, and are enclosed by curly braces
					 - uses the keyword `phase` followed by the execution phase and the code snippet that is represented by three quotes
					 - example:
					```FPP
						instance cmdSeq: Svc.CmdSequencer base id 0x0700 \
							queue size Default.queueSize \
							stack size Defulat.stakSize \
							priority 100 \
						{
							phase Fpp.ToCpp.Phases.configConstants """
							enum {
								BUFFER_SIZE = 5*1024
							}
							"""
							
							phase Fpp.ToCpp.Phases.configComponents """
							cmdSeq.allocateBuffer(
								0,
								Allocation::mallocator,
								ConfigConstants::cmdSeq::BUFFER_SIZE
							);
							"""
						}
					```
		 - defining topologies - aka connection graphs - defines what component instances are used in the application and how their port instances are connected
			 - use the keyword `topology` to declare a new topology 
				 - within topologies you define connections between component ports
			 - basic example:
			```FPP
					port P
					passive component C {
						sync input port pIn: P
						output port pOut: P
					}  
					
					instance c1: C base id 0x100
					instance c2: C base id 0x200
					
					topology Simple {
						@ This is saying that these instances are part of the topology
						instance c1
						instance c2
						
						@ specifies a connection graph called C1
						connections C1 {
							c1.pOut -> c2.pIn
						}
					
						@ specifies a connection graph called C2
						connections C2 {
							c2.pOut -> c1.pIn
						}
					}
			```
			- notice that you have to re-declare the instances to specify that they're part of the topology
				- `connections` are the Graph Specifiers, and the `instance` declarations are the Instance Specifiers 
					- if an instance is defined within a module, you can still use it, ex:
						- `instance A.B`
				- arrows (`->`) represent a connection from an endpoint to another endpoint 
			- connection graphs
				- two ways to specify connection graphs:
					- direct graph specifiers
						- specific graph specifiers that the user manually creates
					- pattern graph specifiers 
						- this is like having F Prime automatically connect all `timeGetOut` ports instead of writing it all out manually
				- you can also define more than one connection per connection graph, thus these two sets of connections are the same: 
					```FPP
						connections C {
							a.p -> b.p
						}
						connections C {
							c.p -> d.p
						}
					
						connections C {
							a.p -> b.p
							c.p -> d.p
						}
						
						@ you can also seperate connections by commas
						connections C {a.p -> b.p, c.p -> d.p}
					```
				- there are some very common topology connections, such as connecting all the `timeGetOut` ports to `sysTime.timeGetPort`
					- since this is so often done, you can use the **pattern graph specifier**:
						- `time connections instance sysTime`
							- the keyword `time` is the **kind** of the pattern graph specifier
							- `sysTime` is the **source instance** of the pattern specifier
						- this will automatically construct a direct graph specifier called `Time` that will automatically connect all instances that have a `timeGetOut` port to the `sysTime.timeGetPort` port
				- pattern graph specifiers 
					- rules:
						- At most one occurrence of each pattern kind is allowed in each topology 
						- For each pattern, required ports shown in the table must exist and must be unambiguous. 
							- Ex. `time connections instance sysTime`
							- if `sysTime` has no input ports of type `Fw.Time`, or if `sysTime` has two or more such ports then an error will occur
					- key parts of Pattern Graph Specifiers:
						- kind - keyword(s) denoting the kind, these appear right before the keyword `connections`
						- source instance - source
						- target instance - target
						- graph name
						- connections - connections generated by the pattern
					- kinds:
						- command - generates three connection graphs
							- command: 
								- Name - `Command`
								- all connections from any port of type `Fw::Cmd` from the source connects to the `command recv` port of each target
							- command:
								- Name - `CommandRegistration`
								- Source instance: `Svc.CommandDispatcher`
									- This instance must have a unique output port of type `Fw.Cmd`, unique input port of type `Fw.CmdReg`, and a unique input port of type `Fw.CmdResponse`
								- Target instances: Every instance that has command ports
								- Connections: All connections from the `command reg` port of each target to the unique input port of type `Fw.CmdReg` of the source 
							- command: 
								- Name - CommandResponse
								- all connections from the command resp` port of each target to the unique input port of type `Fw.CmdResponse` of the source 
						- event:
							- Source: instance of `Svc.ActiveLogger`or a similar component meant for logging event reports. 
								- Must have a unique input port of type `Fw.Log`
							- Target: Each instance that has an `event` port
							- Name - `Events`
							- Connections: All connections from the `event` port of each target instance to the unique input port of type `Fw.Log` of the source instance
						- health:
							- Source: instance of `Svc.Health` or a similar component for health monitoring. The instance must have a unique output port of type `Svc.Ping` and a unique input port of type `Svc.Ping`
							- Target: each instance other than the source that has a unique output port of type `Svc.Ping` and a unique input port of type `Svc.Ping`
							- Name - `Health`
							- Connections: all connections from the unique output port of type `Svc.Ping` of each target to the unique input port of type `Svc.Ping` of the source instance. All connections from the unique output port of type `Svc.Ping` of the source instance to the unique input port of type `Svc.Ping` of each target
						- param:
							- Source: An instance of `Svc.PrmDb` or a similar component representing a database of parameters. The instance must have a unique input port of type `Fw.PrmGet` and a unique input port of type `Fw.PrmSet`
							- Target: Each instance that has `parameter` ports 
							- Name - `Parameters`
							- Connections: All connections from the `param get` port of each instance to the unique input port of type `Fw.PrmGet` of the source instance. All connections from the `param set` port of each target to the unique input port of type `Fw.PrmSet` of the source instance 
						- telemetry:
							- Source: An instance of `Svc.TlmChan` or a similar component for storing telemetry. Requires a unique input port of type `Fw..Tlm`
							- Target: each instance that has a telemetry port
							- Name - `Telemetry`
							- Connections: All connections from the `telemetry` port of each target to the unique input port of type `Fw.Tlm` of the source
						- text event:
							- Source: An instance of `Svc.PassiveTextLogger` or a similar component for logging event reports in textual form. Must have a unique input port of type `Fw.LogText`
							- Target: Each instance has a `text event` port
							- Name - `TextEvents`
							- Connections: All connections from the `text event` port of each instance to the unique input port of type `Fw.LogText` of the source
						- time:
							- Source: An instance of `Svc.Time` or a similar component for providing the system time. The instance must have a unique input port of type `Fw.Time`
							- Target: Each instance that has a `time get` port
							- Name - `Time`
							- Connections: All connections from the `time get` port of each target to the unique input port of type `Fw.Time` that's from the source
					- you can specify what connections you want to generate with a graph pattern specifier
						- ex. if you have instances `a, b, c`, but you only wanna connect `a` and `b`n to time, then you can write:
							- `time connections instance sysTime {a, b}`
				- port numbering 
					- three ways to specify port numbers: explicit numbering, matched numbering, and general numbering
					- explicit numbering:
						- requires an explicit number for a connection endpoint
						- `RateGroups` graph of the Ref topology in the F Prime repository defines the rate group connections. It contains the following connections (in the first line, `Ports.RateGroups.rateGroup1` is an enumerated constant):
							```FPP
							rateGroupDriverComp.CycleOut[Ports.RateGroups.rateGroup1] -> rateGroup1Comp.CycleIn
							rateGroup1Comp.RateGroupMemberOut[0] -> SG1.schedIn
							rateGroup1Comp.RateGroupMemberOut[1] -> SG2.schedIn
							rateGroup1Comp.RateGroupMemberOut[2] -> chanTlm.Run
							rateGroup1Comp.RateGroupMemberOut[3] -> fileDownlink.Run
							```
						- the port number is specified in the brackets
						- notice how we can use predefined constants or explicit integers to specify what ports to use
						- when it makes sense to use constants then do, otherwise like with the example above you can just use the literal numbers if there's no clear pattern 
					- multiple connections can go to the same input port, but only one connection can go from each output port
					- the goal is to avoid literal port numbering, and instead use named constants, so that things are more readable 
						- also don't explicitly write the 0 index in, just have FPP infer it by just stating the port without a port number
							- only use the 0 index if you're reusing that same port multiple times
					- Matched numbering - after explicit numbering is done being processed, the translator applies `matched numbering
						- ex:
							 ```FPP
							connections Sequencer {
							  cmdSeq.comCmdOut -> cmdDisp.seqCmdBuff
							  cmdDisp.seqCmdStatus -> cmdSeq.cmdResponseIn
							}
							
							connections Uplink {
							  ...
							  uplink.comOut -> cmdDisp.seqCmdBuff
							  cmdDisp.seqCmdStatus -> uplink.cmdResponseIn
							  ...
							}
							```
							- it would make sense to just match `cmdDisp.seqCmdBuff` with `cmdDisp.seqCmdStatus` within the Command Sequencer component
						-  some rules for matching:
							- `match p1 with p2`: for every connection between `p1` and another component, there must be a corresponding connection between the other component and `p2`
							- you can use explicit numbering, and the automatic matching will work around the explicit numbering if it is able. 
								- for example if we wanted to do `match p1 with p2`, but if you also explicitly connected `p1` and `p2` to the same component then that wouldn't allow for the matching of the two, since they're now both just inputting into the same component 
									- an error will occur if something like this happens
						- manual matching:
							- Automatic matched numbering only works when each matched pair of connections goes between the same two components
								- one of the components must have a manually matched pair of ports that is matched in the expected way
					- general numbering - done after the explicit numbering and matched numbering have been calculated
						- the translator uses the following algorithm to fill any still unsigned port numbers:
							- traverse the connections in a deterministic order (described in the *FPP Language Specification* document)
							- for each connection:
								- if the output port is unassigned, set to the lowest available port number
								- if the port number is unassigned, set it to zero
				- Importing Topologies:
					- A project might have the following topologies:
						- topology for command and data handling with components such as a command dispatcher, event logger, telemetry data base, parameter database, and components for managing files
						- various subsystem topologies, for things such as power, thermal, attitude control...
						- a release topology
					- it's also super helpful to have each topology be able to run on its own to allow for modular testing 
					- each of the topologies might have their own C&DH (command and data handling) topology 
					- to accomplish some of this you can import one topology into another
					- example:
						```FPP
						topology B {
							import A
							...
						}
						```
						- what the translator will do from here is:
							- Resolve all pattern graph specifiers in A, and resolve all explicit port numbers in A. The resulting topology will be called `T`
							- Take the union of the instances specified in `T` and the instances specified in `B`, counting any duplicates. These are the instances of `B`
							- Take the union of the connection graphs specified in `T` and the connection graphs specified in `B`. If each of `T` and `B` has a connection between the same ports, each becomes a separate connection in `B`
							- Resolve the pattern graph specifiers of `B`, and apply matched numbering and general numbering to `B`
					- private instances:
						- if you ever want to make a component instance private, you can just declare something like: `private instance d` instead of `instance d`
					- transitive imports are allowed:
						- ex: topology A can import topology B which can import topology C
						- just be careful because C cannot import A, otherwise the translator will error 
					- You can also include code from another file through the use of header files. This is done by writing an *include specifier*, this will be able to import a header file from any component instance that has been declared in a topology
		- Specifying models as files - this will explain how to assemble a collection of elements specified in several files into a model
			- FPP doesn't need to be split into certain files unlike XML
				- just be sure to have a good naming scheme and to keep things logically organized
			- include specifiers - main purpose is to split up large syntactic units into many files
				- these can be defined inside a module, component, or topology definition
				- to declare an include specifier, you use the keyword `include` followed by a string that denotes the file path to the include specifier
				- example:
					- you could have the line `constant a = 1` in the file `"a.fppi"`
					- if you import the file you have access to the constant in the file that's importing `.fppi` file
						- ex. `include "a.fpp"` - the `.fpp` file that's importing the `.fppi` file now has full access to everything in the `.fppi` file as if its contents were declared within the `.fpp` file itself 
				- to convert a regular `.fpp` file into a `.fppi` file, run: `fpp-format -i myFile.fpp`
				- you can also import a `.fppi` file from subdirectories
					- ex. `include "A/a.fppi"` or `include "../a.fppi"`
			- Dependencies:
				- lets say you have two files:
					- `a.fpp` - contents:  `constant a = 0`
					- `b.fpp` - contents: `constant b = a`
					- if you do `fpp-check a.fpp b.fpp` the check should pass
						- however if you just do `fpp-check b.fpp` the check will fail stating that `a` is undefined 
				- different steps that occur at different phases of development 
					- `fpp-locate-defs` - generates location specifiers for all the definitions that *could be used* in a set of files *F*
					- `fpp-depend` - passes the files *F* and the location specifiers that were generated in the previous step