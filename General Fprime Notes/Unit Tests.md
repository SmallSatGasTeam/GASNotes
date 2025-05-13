### Configuring The Project and Component For Unit Tests

In order to configure the project for the auto coder, first open the CMakeLists.txt file for the component you're writing test for. Below the register_fprime_module() add these lines:

``` cmake
set(UT_SOURCE_FILES
	"${CMAKE_CURRENT_LIST_DIR}/<component name>.fpp"
)

register_fprime_ut()
```

If the directory build-fprime-automatic-native-ut does not already exist in the root project directory, in the project directory run `fprime-util generate --ut`, then run `fprime-util build --ut`. If the ut build folder was previously generated, only run the build command.


With this the project has been configured for unit tests and your component has been registered for those tests.

### Generating Auto Coded Test Framework

First add `set(UT_AUTO_HELPERS ON)` to the CMakeLists.txt file as so:

``` cmake
set(UT_SOURCE_FILES
	"${CMAKE_CURRENT_LIST_DIR}/<component name>.fpp"
)

set(UT_AUTO_HELPERS ON) #!!! Add me !!!

register_fprime_ut()
```

Next, to generate the starter code for the test files move to the sub-directory containing the component under test. Then, in that folder run `fprime-util impl --ut`. This should create `<component name>Tester.cpp`, `<component name>Tester.hpp`, `<component name>TestMain.cpp`, and `<component name>TesterHelpers.cpp` files. Next, these files should all be moved to a specific unit test sub directory by running `mkdir -p test/ut` and `mv <component name>Test* test/ut`. Finally, add the paths for the `<component name>Tester.cpp` and `<component name>TestMain.cpp` files to the CMakeLists like so:

```cmake
set(UT_SOURCE_FILES
	"${CMAKE_CURRENT_LIST_DIR}/<component name>.fpp"
	"${CMAKE_CURRENT_LIST_DIR}/test/ut/<component name>Tester.cpp" # !!! Add
	"${CMAKE_CURRENT_LIST_DIR}/test/ut/<component name>TestMain.cpp" # These two lines !!!
)

set(UT_AUTO_HELPERS ON)

register_fprime_ut()
```

### Writing Unit Tests

The unit tests are run using the google testing framework. Additionally, F' auto codes a large set of asserts to check telemetry, ports, events, and commands. Because of the auto coded asserts it is essential that you have the IntelliSense setup for F'. Make sure you're using visual studios code and have JPL's fpp extension installed.

There is no streamlined process to writing the unit tests, so I'm going to go through each file instead.

#### Tester.hpp

This file is a standard header file. It contains all the function declarations for the tester class. One important part of this file are the constants at the beginning. You don't have to worry about the TEST_INSTANCE_ID and shouldn't have to change the TEST_INSTANCE_QUEUE_DEPTH under usual circumstances. However, the MAX_HISTORY_SIZE is important because it restricts the max number of iterations the component can be run in a single test. Usually this won't be a problem, but if your component has any time sensitive functions such as a delayed response, you need to make the MAX_HISTORY_SIZE greater than the number of iterations needed to test the response of your component.

Outside of potentially changing MAX_HISTORY_SIZE, the only change to this file you should ever have to make is adding function declarations for any functions you choose to add to the tester. This follows standard c++ syntax.

#### Tester.cpp

In this file you define all the tests you want to run on your component. The tests should be written as a c++ function and use the auto coded asserts from F'. As mentioned earlier, there are so many auto coded asserts that it is incredibly difficult to write these tests without having the Fpp extension installed in VS code.

How to check commands, events, telemetry, and ports is pretty clearly outlined in the F' documentation at https://nasa.github.io/fprime/UsersGuide/user/unit-testing.html.

There are a couple things that are unclear in that document that I'll try to clear up. First, `this->component.doDispatch()` is only needed to unload the command sequencer and the queue of async input ports. You do not need to use `this->component.doDispatch()` if you're testing a synchronous input port. I'd also note that sometimes the IntelliSense mistakenly marks  `this->component.doDispatch()` as inaccessible, ignore this, it will compile and run just fine.

If you need to test anything with an output port you can force the output port to run using `this->pushFromPortEntry_<port name>();`

The functions run for the unit tests are run just like normal c++ functions so try to follow c++ programming guidelines. For example, if you find yourself running a lot of the same asserts across multiple tests, write a function to run them and call that each time instead of copying and pasting multiple asserts.

#### TestMain.cpp

In `<component name>TestMain.cpp` you call the tests you wrote in Tester.cpp using the google tests framework. To add more tests to this file use the following format:

```c++
TEST(<Type of test>, <Test name>) {
  Components::<component name>Tester tester;
  tester.<name of test function>();
  // !!! You may call more functions if multiple test functions are required
}
```

### Running Unit Tests

To run the unit tests navigate to `Components/<component name>` , NOT to `<component name>/test/ut`. Once there run `fprime-util build --ut` to build the tests, and then run `fprime-util check` to run the tests.