1. Add the ports to whatever component you will be using as the data product producer
	1. The data product producer is what will call all the other components. It collects data then calls the manager to receive a data buffer, places the data into the data buffer and then sends it to the writer.
	2. Ports:
			```
		@ A port for getting a data product container
        product get port productGetOut

        @ A port for requesting a data product container
        product request port productRequestOut

        @ An async port for receiving a requested data product container
        async product recv port productRecvIn priority 10 assert

        @ A port for sending a filled data product container
        product send port productSendOut
			```
2. Add the DpCatalog, DpManager, and DpWriter components to the instances and topology
```fpp
  instance dpManager: Svc.DpManager base id 0x0E00 \
    queue size Default.QUEUE_SIZE \
    stack size Default.STACK_SIZE \
    priority 95
  
  instance dpWriter: Svc.DpWriter base id 0x0F00 \
    queue size Default.QUEUE_SIZE \
    stack size Default.STACK_SIZE \
    priority 94

  instance dpCatalog: Svc.DpCatalog base id 0x1000 \
    queue size Default.QUEUE_SIZE \
    stack size Default.STACK_SIZE \
    priority 93
```
3. Add the telemetry channels for each component into their own packet in the deployment packets file
```xml
<packet name="DataProduct" id="8" level="1">
        <channel name="dpCatalog.CatalogDps"/>
        <channel name="dpCatalog.DpsSent"/>
        <channel name="dpManager.NumSuccessfulAllocations"/>
        <channel name="dpManager.NumFailedAllocations"/>
        <channel name="dpManager.NumDataProducts"/>
        <channel name="dpManager.NumBytes"/>
        <channel name="dpWriter.NumBuffersReceived"/>
        <channel name="dpWriter.NumBytesWritten"/>
        <channel name="dpWriter.NumSuccessfulWrites"/>
        <channel name="dpWriter.NumFailedWrites"/>
        <channel name="dpWriter.NumErrors"/>
    </packet>
```
4. Add the Catalog's ping entries
	1. In `<Deployment name>Topology.cpp` copy and paste one of the already existing lines then change its name to add the dpCatalog to the ping entries array. Make sure to place it in the list alphabetically.
	2. In `<DeploymentTopologyDefs.hpp` add the dpCatalog namespace enum next to all the others. You can copy and paste one of the ones already there and then just change the name.
5. FIX THE FREAKING DEPLOYMENT
	1. In `<Deployment Name>Topology.cpp` 
		1. Switch `startSocketTask(..)` to `start(...)`
			1. They renamed the task but have yet to change the deployment autocoder
		2. Switch `stopSocketTask(..)` to `stop(...)`
		3. Switch `joinSocketTask(...)` to `join()` and don't pass in any arguments or parameters
		4. Pass in a Fw::Time object instead of a different object.
```c++
Fw::Time milli; 
milli.set(milliseconds/1000,(milliseconds%1000)*1000);

while (cycling) {
	Deployment::blockDrv.callIsr();
	Os::Task::delay(milli);
}
```
6. Connect all the ports
	1. Catalog
		1. ```dpCatalog.fileOut -> fileDownlink.SendFile
			fileDownlink.FileComplete[0] -> dpCatalog.fileDone```
	2. Manager
		1. `rateGroup3.RateGroupMemberOut[3] -> dpManager.schedIn`
		2. ```producer.productGetOut -> dpManager.productGetIn[0]
			producer.productRequestOut -> dpManager.productRequestIn[0]
			producer.productSendOut -> dpManager.productSendIn[0]
			dpManager.productResponseOut[0] -> producer.productRecvIn
			dpManager.bufferGetOut -> bufferManager.bufferGetCallee
			dpManager.productSendOut[0] -> dpWriter.bufferSendIn```
	1. Writer
		1. `rateGroup3.RateGroupMemberOut[4] -> dpWriter.schedIn`
		2. `dpWriter.deallocBufferSendOut -> bufferManager.bufferSendIn`
7. Build a dummy DpProcessor and connect it to DpWriter
	1. Add ports for procBufferSendIn of type Fw.bufferSend length DpWriterNumProcPorts
	2. Add port for dpWrittenIn type Svc.DpWritten
```fpp
sync input port procBufferSendIn: [DpWriterNumProcPorts] Fw.BufferSend

sync input port dpWrittenIn: Svc.DpWritten
```
8. Configure DpWriter and DpCatalog in the `configureTopology` function in the `<deployment name>Topology.cpp` file. 
	1. DpWriter needs the file path
	2. DpCatalog needs mallocator passed to it.
```fpp
	//Configure the place where everything is going to be saved in DpWriter
    Fw::String stringy;
    stringy = "insert file path here";
    dpWriter.configure(stringy);

    //Configure DpCatalog
    Fw::FileNameString string [2];
    string[0] = stringy;
    stringy = "";
    string[1] = stringy;
    dpCatalog.configure(string,1,0,mallocator);
```

