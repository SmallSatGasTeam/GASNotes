 - Types - Format: (F' Type = C/C++ equivalent)
	 - I8 = int8_t
	 - I16 = int16_t
	 - I32 = int32_t
	 - I64 = int64_t
	 - U8 = uint8_t
	 - U16 = uint16_t
	 - U32 = uint32_t
	 - U64 = uint64_t
	 - F32 = float
	 - F64 = double
 - GDS - ground data system
	 - to run the GDS server, all you gotta do is run `fprime-gds`
	 - to specify deployment - `fprime-gds -d path/to/deployment`
	 - to disable automatic flight software execution run `fprime-gds -n` 
		 - i don't really understand what this does, it's only used for when the project 'migrates to the embedded system'
	 - to change the server address - `fprime-gds --ip-address 8.8.8.8 --ip-port 12345`
	 - to change internal address - `fprime-gds --tts-addr 8.8.8.8 --tts-port 12345`
	 - list of local addresses and ports
		 - **127.0.0.1:5000**: Hosts the HTML frontend. The browser connects here to access the GUI.
		- **0.0.0.0:50000**: The default communication interface port. The embedded system connects here.
		- **0.0.0.0:50050**: An internal port used for transporting GDS data. Communication and HTML endpoints internally connect here.
	- to run the GDS without a UI - `fprime-gds -g none`
 - XML Specifications - these are used to create the basic framework of a component, then the autocoder takes the XML file and builds the main parts of the component
	 - NOTE: when talking about 'size' in relation to strings, 'size' is simply the max amount of characters you expect the string to be
	 - serializables (data passed between components) - ex. commands, telemetry, events, and parameters
		 - XML-Specified serializable - REQUIRED FOR THE GDS
			 - **For the build system to detect the XML file, you name it: `<SomeName>SerializableAi.xml`** - without the greater than and less than signs
		 - tags - can have multiple uses
			 - serializable
				 - outermost tag to indicate serializable is being defined
				 - namespace - the C++ namespace for the serializable class (optional)
				 - name - class name for the serializable 
				 - typeid - unique id - automatically generated if not specified (optional)
			 - comment - special marker to mark when something is a comment: `<!-- -->`
			 - members 
				 - begins the region of declaration where member types are specified
				 - defines member of the type
				 - name
				 - type - should be build in type ex. ENUM, string, XML-specified, or user-written serializable type
				 - size - required for when the type is a string, otherwise don't use this
				 - array_size - specifies that the member is an array of the type with the given size (array_size not size)
				 - format - like how many decimal places to display 
					 - ex.
						```xml
						<Member type="F32" name="temperature">
						    <Format>%.2f</Format>
						</Member>
						```
				- default - default value of the member (optional)
				- enum 
					- specifies an enumeration when the member type is ENUM
					- name
				- item
					- specifies a member of the enumeration
					- name
					- value - some given value that follows C enumeration rules if not specified
					- comment
				- import_serializable_type - imports an XML definition of another serializable type for use (boolean)
				- import_enum_type - imports an XML definition of enumeration type (boolean)
				- import_array_type - imports XML definition of array type (boolean)
				- include_header - determines if you want to have the autocoder generate the header file (boolean)

 - some examples:
```XML
<serializable name="Switch">
  <members>
    <member name="state" type="ENUM">
      <enum name="SwitchState">
        <item name="OFF" value="0"/>
        <item name="ON" value="1"/>
      </enum>
      <default>OFF</default>
    </member>
  </members>
</serializable>
```
 - This file defines a Serializable type `Switch` with one member `state`. Its type is `SwitchState`, which is an enumeration with enumerated constants `OFF` and `ON`. Using the default child element, the default value will be OFF.
 - importing example (using custom enum XML file):
	 - lets say we have this file:
	```XML
		<enum name="SwitchState" default="OFF">
		  <item name="OFF" value="0"/>
		  <item name="ON" value="1"/>
		</enum>
	```
	 - we can import and use it like this:
```XML 
		<serializable name="Switch">
		  <import_enum_type>SwitchStateEnumAi.xml</import_enum_type>
		  <members>
			<member name="state" type="SwitchState"/>
		  </members>
		</serializable>
```
 - XML-specified enumeration - essentially a custom member type that you make
	 - naming scheme - `<Type>EnumAi.xml` - without the greater than and less than signs
		 - the `Type` is the actual name of the new type you're creating
	 - highest level structure
		 - enum - attributes
			 - name - name of enumeration type
			 - namespace (optional) - the enclosing namespace of the enumeration type
				 - basically just another way to separate tags into sections 
				 - if this isn't specified then the type is placed in the global namespace
				 - can use one or more identifiers by using `::`
				 - ex. `<enum name="E" namespace="A::B">` … `</enum>`
			 - default (optional) - default value of the enumeration
				 - must match the name attribute of one of the item definitions within the enumeration 
				 - if not specified then the value of the enumeration is set to 0
				 - ex. `<enum name="E" default="Item2">` … `</enum>`
					 - 'Item2' is assumed to be the name attribute of one of the item definitions in the enum
			 - serialize_type (optional) - the numeric type of the enumeration when serializing 
				 - if not specified, the serialization type is set to FwEnumStoreType
				 - `<enum name="E" serialize_type="U64">` … `</enum>` - serialization type 'U64'
		 - enum - children
			 - comment
				 - ex. `<comment>` _comment_text_ `</comment>` - becomes a generated comment in the C++ code
			 - item_definition
				 - defines an enumerated constant, XML node named 'item'
				 - name
				 - value - assigns integer value to the enumerated constant
					 - if missing, then the value is assigned in the ordinary way for C and C++ enumerations
				 - ex. Here is an enumerated constant with a name, value, and comment: `<item name="OFF" value="0" comment="The off state"/>`
		 - XML-specified array type
			 - Filename - `<Type>ArrayAi.xml`
				 - `Type` is the array's type
			 - attributes
				 - name
				 - namespace - basically just another way to separate tags into sections 
			 - children
				 - comment - `<comment>` _comment_text_ `</comment>`
				 - `<include_header>` _header_file_ `</include_header>` for including C++ header files into the generated C++.
				- `<import_serializable_type>` _serializable_xml_file_ `</import_serializable_type>` for importing XML-specified serializable types.
				- `<import_enum_type>` _enum_xml_file_ `</import_enum_type>` for importing XML-specified enum types.
				- `<import_array_type>` _array_xml_file_ `</import_array_type>` for importing XML-specified array types.
				- format - `<format>` _format_string_ `</format>`
					- must contain a single conversion specifier starting with `%`
						- the conversion specifier must be legal for C, C++'s printf, and Python in relation to the array's type
						- For example, if the array element type is `U32`, then `<format>%u</format>` is a valid format specifier. So is `<format>%u seconds</time>`. `<format>%s</format>` is not a legal format specifier in this case, because the string format `%s` is not valid for type `U32`.
				- type - `<type` _size_attribute_ `>` _type_ `</type>`
					- `size_attribute` (optional) - specifies decimal integer size, only valid if the element type is `string`
					- specifies the size of the string representation
					- must be an F Prime build-in type such as `U32` or a named type
						- ie. the name of an XML-specified type (Serializable, Enum, or Array) or the name of a C++ class included with `include_header`
				- value - specifies a default value for an array element
					- must be text that means a certain type (ex. U32)
						- gets automatically generated into C++ code that represents the correct type
						- ex. U32 will be given the value `0` by default
						- Suppose the array element type is an array `A` of three U32 values. In the generated C++ code, `A` is a class with a three-argument constructor, and each argument initializes an array element. Therefore `A(1, 2, 3)` is a correct default value.
				- exmaples:
					- `<type>U32</type>` specifies the built-in type `U32`
					- `<type>T</type>` specifies the named type `T`. `T` must be (1) included via `include_header` or (2) imported via `import_serializable_type`, `import_enum_type`, or `import_array_type`.
					- `<type size="40">string</type>` specifies a string type of size 40.
					- `<size>` _integer_ `</size>`
					- `<default>` _value_ … `</default>`
		- Developer-Coded Serializable - just know that you NEED to use XML generated code to interact with the GDS
			- Steps
				1. Define the class, NEEDS to be derived from `Fw::Serializable`
				2. Declare two base class pure virtual functions: `serialize()` and `deserialize()`
					1. these provide a buffer as a destination for the serialized form of the data in the type
				3. implement the functions:
					1. the buffer base class that is the argument to the functions provides a some functions for serializing and deserializing. these functions are specified in the SerializeBufferBase class
					2. for each member that is wanted to be kept, call the serialization functions, members can be serialized in any order. the deserialization function should be able to figure out how it was serialized and then deserialize it
					3. if it's absolutely necessary, you can use the provided raw buffer version to do a bulk copy of the data if serializing individual members isn't feasible.
				4. make an enumeration with a single member called `SERIALIZED_SIZE`
					1. the value of this should be the sum of the sizes of all the members. this can be achieved with the C/C++ builtin function `sizeof()`
					2. something optional that we can do is we can check the type before and after the serialization to make sure it's went through correctly
				5. then implement copy constructors and equal operators 
					1. **Copy Constructor**: This is a special constructor that creates a new object as a copy of an existing object.
					2. **Assignment Operator**: This operator is used to assign the state of one object to another object of the same type. It is usually implemented as a member function named `operator=`.
		 - ports - require no developer written code, fully specified in XML files
			 - filename - `<SomeName>PortAi.xml` - without greater than and less than signs
			 - example port can be found in `Autocoders/Python/templates/ExamplePortAi.xml`
			 - tags
				 - interface - outermost tag
				 - namespace (optional) - just another way to separate blocks of tags
				 - name - class name for the port
				 - args (tag, optional) - begins the region of the declaration where port arguments are specified
					 - arg (tag) - defines an argument to the port method
						 - type - the arguments type, should be a built-in type (ie. ENUM, string, XML-specified type, or a user-written serializable type)
						 - name
						 - size - specifies the size of the argument if it's a string
						 - pass_by (optional) - specifies how the argument should be passed to a handler function
							 - values can be `pointer` or `refrence` 
							 - for a synchronous port handler, the default behavior is to pass by value for primitive values, otherwise by const refrence
							 - for an asynchronous port, if not specified then the data itself is serialized, copied into and out of the queue, then deserialized
							 - something to note about pass_by is that you need to be aware of where the data is at any point, since it could be within one component but originated from another
				 - enum (tag) - specifies an enumeration when the argument type=ENUM 
					 - name
					 - item (tag) - specifies the member of the enumeration
						 - name
						 - value - assigns a value to the enumeration member
							 - member values in the enumeration follow C enumeration rules if not otherwise specified
					 - return - specifies that the method will return a value 
						 - type - specifies the type of the return - can be a build-in type
						 - pass_by - specifies by 'refrence' or by 'pointer' or by 'value'
						 - a port call **CANNOT** be serialized if it contains return values
				- import_serializable_type (tag) - imports an XML definition of an enumeration type
				- import_array_type (tag) - imports an XML definition of an array type
				- include_header (tag) - includes a C/C++ user-written header for argument types
			- components - **this is where all the logic resides**
				- the process is the component is first defined in an XML file, which makes a base class, then the developer implements a derived class that implements from that base class
				- filename - `<SomeName>ComponentAi.xml`
				- tags
					- component (tag) - outermost tag 
						- namespace (optional) - basically another way to separate large groups of tags
						- name - will become the class name for the component
						- kind - the type of component
							- can be passive, active, or queued
						- modeler - boolean - if set to 'true' the autocoder doesn't make ports for commands, telemetry, events, and parameters
							- if it's set to true, then those ports must be declared with the 'role' attribute
					- import_dictionary (tag) - imports a ground dictionary
						- Ground Dictionary - defined outside the component XML that is determined by the command, telemetry, event, and parameter entries
							- allows for external tools to be written by other projects to create dictionaries
					- import_port_type (tag) 
						- imports XML definition of a port type used by the component
					- import_serializable_type (tag) - imports XML definition of a serializable type for use in the component interfaces
					- import_enum_type (tag) - imports an XML definition of an array type
					- ports (tag) - defines the beginning of the port section of the XML file
						- port (tag) - starts the description of a port
							- name
							- data_type - if getting from another XML file then use import_port_type here
							- kind - defines the synchronicity and direction of the port
								- sync_input - invokes the derived class methods directly
								- guarded_input - invokes the derived class methods after locking a mutex (gaining access to all shared info) shared by all guarded ports and commands 
								- async_input - creates a message with the serialized arguments of the port call 
									- when message is dequeued, the arguments are deserialized and the derived class method is called
								- output - invoked from the logic in the derived class
							- priority - the priority of the invocation of the port, only used on asynchronous ports.
								- the priority is given to the message in the underlying message queue if priorities are supported by the target OS
							- max_number - specifies the number of ports of this type
								- allows multiple callers to the same type of port, as long as it's required to know the source
								- can be specified as an Fw/Cfg/AcConstants.ini file $Variable value
									- (this is just saying to use $ variables when doing this)
							- full - specifies the behavior for async ports when the message queue is full
								- can be 'drop', 'block', or 'assert' (assert is default)
							- role - specifies the role of the port when the modeler=true
					- commands (tag) - starts the section where software commands are defined
						- opcode_base - defines the base value for the opcodes in the commands, if specified opcodes will be added to this value
						- command (tag) - starts the definition for a particular command 
							- mnemonic - defines a text label for the command, can be alphanumeric with `_` separators, but no spaces 
							- opcode - defines an integer that represents the opcode used to decode the command, should be a C-compatible constant
							- kind - defines the synchronoicity of the command - the command can be of the following kinds:
								- sync - invokes the derived class methods directly
								- guarded - invokes the derived class methods after locking a mutex shared by all guarded ports and commands
								- async - creates a message with the serialized arguments of the port call, when the message is dequeued, the arguments are deserialized and the derived class method is called
							- priority - sets the command message priority if the message is asynchronous
							- full - specifies the behavior for async commands when the message queue is full
					- args (tag, optional) - starts the region of declaration where command arguments are specified
						- arg (tag) - defines an argument in the command
							- type - the type of the argument should be a built-in type. 
								- note: a string type should be used if a text string is the argument 
							- name
							- size - specifies the size of the argument if it's a string
					- enum (tag) - specifies an enumeration when the argument type=ENUM
						- name
						- item (tag) - member of enumeration
							- name
							- value - assigns value to the enumeration member
							- comment
					- telemetry (tag, optional) - section that defines channelized telemetry
						- telemetry_base - defines the base value for channel ID's
					- channel (tag) - starts definition for telemetry channel
						- id - numerical value identifying the channel
						- name - channel name, cannot contain spaces
						- data_type - the type of the channel
						- size - for when the type=string
						- abbrev - abbreviated name for the channel, required for AMPCS which is just a NASA build ground data system
						- update - how often the channel should be updated
							- values are 'ALWAYS' or 'ON_CHANGE'
						- format_string - like how many decimals to round to 
							- ex. `<channel name="MyExample" format="%0.2f"> </channel>`
					- parameters (tag, optional) - specifies the section that defines parameters for the component 
						- parameter_base - defines base value for the parameter ID's
						- opcode_base - defines base value for the opcodes in the parameter set then saves commands
							- if specified, consider it an 'offset' for all opcodes under this one's control
							- if this isn't specified, then the opcodes specified in the XML will be used exactly as they are
							- this is nice because you can define an 'offset' for an entire subsystem, so that you don't have to keep manually incrementing each command's opcode to avoid conflict
					- parameter (tag) - specifies area defining a parameter
						- id - numeric value that represents the parameter
						- name
						- data_type - the type of the parameter
						- size - size of the parameter if it's of type string
						- default - default value for the parameter for when it cannot be retrieved from non-volatile storage
						- set_opcode - the command opcode used for setting the parameter
						- save_opcode - the command opcode used for saving the parameter
						- enum (tag) - specifies an enumeration when the argument type=ENUM
							- name
							- item (tag) - member of enumeration
								- name
								- value - assigns value to the enumeration member
								- comment
					- events (tag, optional) - specifies the section that defines events for the component
						- event_base - base value for the event ID's
							- if specified this value will be added to all event ID's that are children of this tag
							- if missing, event ID's specified by the children will remain unchanged
						- event (tag) - begins definition for a given event
							- id - numeric value
							- name
							- severity - severity of the event
								- values:
									- DIAGNOSTIC
									- ACTIVITY_LO
									- ACTiVITY_HI
									- COMMAND
									- WARNING_LO
									- WARNING_HI
									- FATAL
						- format_string - like how many decimals to round to 
							- ex. `<channel name="MyExample" format="%0.2f"> </channel>`
						- throttle - max number of events that will be issued
							- when the max is reached, it'll just stop issuing events
							- the throttle must be cleared before more events can be issued
						- args (tag, optional) - starts the region of declaration where command arguments are specified
							- arg (tag) - defines an argument in the command
								- type - the type of the argument should be a built-in type. 
									- note: a string type should be used if a text string is the argument 
								- name
								- size - specifies the size of the argument if it's a string
						- internal_interfaces (tag, optional) - specifies internal interface for the component 
							- internal interface - functions that can be called from implementation code, that are used for internal message dispatching. - very similar to async ports and commands
								- these functions will call a handler on the thread of active or queued components, and that handler does whatever the developer has it do
								- often used for interrupt service routines (program that quickly executes the task while minimizing disruption to the main program)
								- example use case: when a button is pressed an ISR can capture the input from a camera and then save it
							- internal_interface (tag) - specifies an internal interface call
								- priority - sets the internal interface's message priority if the message is async
									- if not specified the message of this internal interface will be ignored
								- full - behavior for when the message queue is full
									- Values
										- drop
										- block
										- assert (default)
								- args (tag, optional) - starts the region of declaration where command arguments are specified
									- arg (tag) - defines an argument in the command
										- type - the type of the argument should be a built-in type. 
											- note: a string type should be used if a text string is the argument 
										- name
										- size - specifies the size of the argument if it's a string