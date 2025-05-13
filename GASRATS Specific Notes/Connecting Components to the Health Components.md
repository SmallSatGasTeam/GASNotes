### Warnings
- Only active components can be pinged by the Health component
- You should not have to edit the health component at all, it is built into the F' framework and should already be included in your deployment if you're using the F' build tools

## Add component to the Ping Entries list (TopologyDef.hpp)
In `<deployment name>/Top/<deployment name>TopologyDef.hpp` there should be a list of ping entries. Each namespace matches a component instance name. The WARN and FATAL elements denote the number of seconds without a return ping before a warning or fatal event is transmitted. By default these are set to 3 and 5 seconds respectively.

To add a new component, copy one of the previous entries, rename it to match the name of the component instance you are connecting, and then set the WARN and FATAL times.

The code block should look like this:

```c++
namespace PingEntries {
namespace blockDrv {
enum { WARN = 3, FATAL = 5 };
}
namespace tlmSend {
enum { WARN = 3, FATAL = 5 };
}
namespace cmdDisp {
enum { WARN = 3, FATAL = 5 };
}
namespace cmdSeq {
enum { WARN = 3, FATAL = 5 };
}
namespace eventLogger {
enum { WARN = 3, FATAL = 5 };
}
namespace fileDownlink {
enum { WARN = 3, FATAL = 5 };
}
namespace fileManager {
enum { WARN = 3, FATAL = 5 };
}
namespace fileUplink {
enum { WARN = 3, FATAL = 5 };
}
namespace prmDb {
enum { WARN = 3, FATAL = 5 };
}
namespace rateGroup1 {
enum { WARN = 3, FATAL = 5 };
}
namespace rateGroup2 {
enum { WARN = 3, FATAL = 5 };
}
namespace rateGroup3 {
enum { WARN = 3, FATAL = 5 };
}
/*!!! This is an example for a flight logic componenet !!!*/
namespace flightLogic {
enum { WARN = 1, FATAL = 2 };
}

/*!!! Insert new namespace matching the component instance name here !!!*/

} // namespace PingEntries
```

## Add component to ping entries copy (Topology.cpp)
A copy of the ping entries list is kept in `<deployment name>/Top/<deployment name>Topology.cpp`. Here the list is stored in an array. This array is what F' uses when sending the error event and as such must be in alphabetical order. If this list is ordered incorrectly then the ping will still work, but if a component fails, the event will claim a different and still working component failed instead. 

This may not happen in every project, but sometimes the Topology.cpp list may include a "chanTlm" ping entry. The "chanTlm" entry apperas to have been replaced by the "tlmSend" component but the autocoder still contains a deprecated reference to "chanTlm". As such you will have to manually remove "chanTlm" and append "tlmSend" to the end of the list.

After making the changes, you list should look similar to this:

```c++
Svc::Health::PingEntry pingEntries[] = {
    {PingEntries::blockDrv::WARN, PingEntries::blockDrv::FATAL, "blockDrv"},
    {PingEntries::cmdDisp::WARN, PingEntries::cmdDisp::FATAL, "cmdDisp"},
    {PingEntries::cmdSeq::WARN, PingEntries::cmdSeq::FATAL, "cmdSeq"},
    {PingEntries::eventLogger::WARN, PingEntries::eventLogger::FATAL, "eventLogger"},
    {PingEntries::fileDownlink::WARN, PingEntries::fileDownlink::FATAL, "fileDownlink"},
    {PingEntries::fileManager::WARN, PingEntries::fileManager::FATAL, "fileManager"},
    {PingEntries::fileUplink::WARN, PingEntries::fileUplink::FATAL, "fileUplink"},
    {PingEntries::flightLogic::WARN, PingEntries::flightLogic::FATAL, "flightLogic"},
    {PingEntries::prmDb::WARN, PingEntries::prmDb::FATAL, "prmDb"},
    {PingEntries::rateGroup1::WARN, PingEntries::rateGroup1::FATAL, "rateGroup1"},
    {PingEntries::rateGroup2::WARN, PingEntries::rateGroup2::FATAL, "rateGroup2"},
    {PingEntries::rateGroup3::WARN, PingEntries::rateGroup3::FATAL, "rateGroup3"},
    {PingEntries::rateGroup3::WARN, PingEntries::rateGroup3::FATAL, "tlmSend"},
};
```

If the order of this list gets messed up, or if the warning and fatal events return the wrong component, the expected order as given by the auto coder can be found at `build-fprime-automatic-native/<deployment name>/Top/<deployment name>TopologyAc.cpp`

## Add pingIn and pingOut ports to your component
The next step is to add ports to accept and reply to pings sent from the health component. First, add these lines of code to your component's fpp file:

```fpp
@ pingIn : receives health pings
async input port pingIn: Svc.Ping

@ pingOut : Returns health ping
output port pingOut: Svc.Ping
```

Run `fprime-util impl` in the component's folder and then copy over the function definitions for pingIn_handler into from the templates into your .hpp and .cpp files.

Finally, in the .cpp file, add this line to your pingIn_handler:

```c++
//This is the code that you should have copied over from implementing the fpp file
void CameraManager ::
    pingIn_handler(
        NATIVE_INT_TYPE portNum,
        U32 key
    )
  {
    this->pingOut_out(0,key); //!!! ADD THIS LINE !!!
  }
```

Run `fprime-util build` in the project directory to check for any errors. An easy way to test the connection is to run health.HLTH_PING_ENABLE command from the gds. If everything is connected correctly, you should be able to enable or disable the ping using that command. Additionally, you can test behavior when the component fails by commenting out `this->pingOut_out(0,key)` in the .cpp file.