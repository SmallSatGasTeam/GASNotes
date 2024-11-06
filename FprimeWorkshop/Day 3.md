 - Testing
	 - should be spending as much time unit testing as actual coding 
	 - two different types of tests
		 - unit testing - individual units (FPrime components)
		 - integration testing 
	 - high level goals of unit testing
		 - cover all component level requirements
			 - achieve high code coverage 
		 - want to achieve reasonable amount of state and path coverage 
	 - tests to requirements
		 - component level requirements should drive the tests 
			 - should maintain record of how tests cover the requirements
			 - record the mapping in a table or spreadsheet 
			 - unit test examples can be found in the `Svc` components
	 - writing unit tests
		 - basic approach
			 - write a first complete test which covers a requirement
			 - write a second complete test
				 - where there's overlap, refactor into functions
		 - disciplined approach
			 - write functions that test individual behaviors 
			 - write tests by composing the functions 
			 - FPrime has developed support for this (rule-based testing)
		 - writing test code
			 - treat unit tests as a regular programming problem, avoid dangerous code, avoid duplication...etc
				 - don't write messy code 
		 - picking good inputs
			 - techniques for picking inputs 
				 - use arbitrary values like 42, this is easy but not robust because the tests may be dependent on the type given
				 - use significant values, like boundary values (values that push the boundaries of the program)
				 - use an analysis tool to pick values 
					 - in FPrime, `STest` can create random values for testing
			 - a unit test is conisidered part of a complete system
				 - model external behavior
					 - write a test harness or a "mock system"
						 - think about relevant system behavior - should be abstracted at the appropriate level
				 - write unit tests against a specific interface
					 - in FPrime, interfaces are ports
			 - sometimes components call external libraries 
				 - you can sometimes link against the library in the test 
					 - it's usually better to link against a mock or stub library - this avoids complex library behavior, and makes it easier to induce behaviors for testing
			 - code coverage is defined as all lines of code being ran in the source programs
				 - can use tools such as `gcov` to do this analysis
				 - usually easy to get to around 80% code coverage 
					 - think about "normal" behavior of code, and write tests to exercise that 
				 - the cases typically forgotten are those that are rare or are mimicking off-nominal behaviors 
				 - 100% code coverage isn't necessary 
					 - and it won't even cover all the bases
						 - for example you'll still need to test state machines separately
	 - testing in fprime
		 - classes
			 - TesterBase - auto-generated
			 - GTestBase - derived from TesterBase
				 - this includes headers for the Google Test framework
			 - Tester class - this is developer written from a generated template 
			 - TesterBase class - this is auto generated
				 - for each output port in `C`, an input port called a **from port**
				 - for each input port in `C`, an output port called an **in port**
			 - coder provides a template
				 - you can add tests as public methods, and you can also write tests in a derived *Tester* class
			 - generate the test classes
				 - add public test methods to *Tester*, 
				 - in the component directory, run `fprime-util impl --ut`
					 - then move the classes to the `test/ut` directory
				 - write a `main.cpp` file that calls the test methods
		 - building and running unit tests
			 - to generate starter code for unit testing
				 - go to the component directory run `fprime-util impl --ut`
				 - `fprime-util build --ut`
				 - `fprime-util check`
			 - generate the analysis
				 - run `fprime-util check --coverage`
				 - will create results in the coverage directory 
					 - `coverage.html` - summary
					 - `coverage.[filename].cpp.[hash].html` - details
	 - 
sending commands:
```cpp
this->sendCOMMAND_NAME(
arguments
);
this->component.doDispatch(); // must call this to get a command response back, this gives a thread to the activity on the queue
ASSERT_CMD_RESPONSE_SIZE(1);
ASSERT_CMD_RESPONSE(0, Component::OPCODE_COMMAND_NAME,
					cmdSeq, Fw::COMMAND::OK
);
```
checking events
```cpp
ASSERT_EVENTS_SIZE(1);
ASSERT_EVENTS_EventName_SIZE(1);
ASSERT_EVENTS_EventName(0, //index of history
						arg1, //expected value of arg1
						arg2 // expected value of arg2
					   )
```
checking telemetry
```cpp
ASSERT_TLM_SIZE(1);
ASSERT_TLM_ChannelName_SIZE(1);
ASSERT_TLM_ChannelName(0, arg);
```
checking output ports 
```cpp
ASSERT_FROM_PORT_HISTORY_SIZE(1);
ASSERT_from_PortName_SIZE(1);
ASSERT_from_P0rtName(0, expectedValue);
```
setting parameters
```
this->paramSet_ParamName(
	value, //parameter value
	Fw::PARAM_VALID // parameter status
)
```
setting the time
```
this->setTime(time);
```

 - fprime integration test framework
	 - designed for integration testing with F' gds
	 - allows users to write automated tests in python
	 - creates excel test logs
 - ingenuity story
	 - goal - fly handful of flights over 30 days
	 - was designed to fly in a very optimal flat area 
		 - used feature tracking navigation - looking at the terrain to see how fast 
	 - got stuck in sand ripples (basically sand dunes)
		 - no rocks, so it followed its shadow
			 - this caused a fault error, so it landed
		 - wanted to pop up flight
			 - go up and takes photos
		 - so it did a pop up flight, and as it was landing it was following its shadow, so it came down at a angle, and struck its blades against a sand dune
	 - this was a good scenario, since they were able to test the limits of the system and gather the diagnostic data
 - development process
	 - important to have if schedule changes occur
	 - waterfall model
		 - requirements (PDR document) -> design (software architecture) -> implementation (software)-> verification -> maintenance
	 - proof of concept and prototyping
		 - use target OS and hardware platform
		 - be able to compile and execute software on the platform
		 - be able to communicate over the planned interfaces
		 - then check data bandwidth and performance analysis
	 - design phase 
		 - develop list of components with functionality description
		 - have context diagrams, interconnect block diagrams (topologies)
			 - sequence diagrams
			 - data flow diagrams
			 - and component block diagrams
		 - define ports and types to be used across components
		 - analyze resource utilization and performance (memory, cpu, and I/O usage)
		 - address any concurrency issues
	 - implementation phase
		 - conduct component level reviews - requirements, design, implementation and unit-test, integrated test results, closeout
		 - don't want to fully recreate the design once we're implementing 
			 - we can---however---be able to change the design a little bit to accommodate new changes discovered during implementation
		 - there can be different builds
			 - prototypes
			 - internal releases
			 - external releases to support sub-system proof of concept
		 - Artifacts
			 - FSW binaries, non-volatile parameter or config files
			 - documentation, build environment, config...etc.
			 - dictionaries
	 - version control
		 - have a main
			 - then a devel (development)
				 - then add feature branches off of the development branch
	 - verification phase
		 - catching bugs early is cheaper and easier to fix
		 - artifacts
			 - FSW release package 
				 - FSW binaries, non-volatile parameter or config files
				 - documentation, build environment, config etc
				 - dictionaries
			 - test reports
			 - requirements verification and validation matrix
	 - delivery review
		 - release description document (RDD)
			 - change log
			 - version identification
			 - test reports
			 - project overview
			 - controlling documents
			 - requirement verification sumary
			 - known issues
			 -  problem disposition
			 - detailed contents
		 - users guides
		 - software design documents
	 - component level process
		 - develop component software design document (SDD)
			 - component overview
			 - assumptions
			 - component level requirements
			 - design
				 - diagrams
				 - data flows, stat transitions, classes, sequences
				 - port list
				 - custom data types
				 - states
				 - port behaviors
			 - command telemetry event and parameters
		 - reference datasheets
		 - FPP
		 - design review
	 - component implementation
		 - review JPL c-coding standard (JPL Rules DocID 78115) and code checklist being used by the project for coding guidelines
		 - reference other mature F' components for C++ coding style
		 - code development
			 - port handler behaiors
			 - state mangement
			 - command handlers
			 - telemetry and events
			 - parameters
		 - component compilation for all targets
		 - static analysis 
		 - conduct code review 
		 - code checklist walkthrough
		 - component unit testing
		 - traceability of each test-case to component level requirements
		 - code coverage analysis
		 - unit test output and coverage results
		 - conduct unit test review
		 - unit test checklist walkthrough
	 - integrated testing
		 - test component functionality with the integrated FSW build
		 - test venues
			 - simulation
			 - hardware
		 - test scripts
			 - send commands and verify telemetry/events
		 - test reports
			 - test as-runs
			 - telemetry and event logs
		 - requirements V&V
	 - component closeout
		 - verify that all component requirements have been verified
		 - verify everything
	 - checklists
		 - Class D / CubeSats
			 - may use a single simple checklist for the entire component development process
	 - weekly progress summary
		 - report any delays, and highlight accomplishments and progress, and describe current progress against development plan schedule
	 - use completion points
		 - they're basically measured in work months, but you can use whatever definition for it that you'd like
	 - reporting is much more valuable than planning