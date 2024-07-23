1. Add the ports to whatever component you will be using as the data product producer
2. Add the DpCatalog, DpManager, and DpWriter components to the instances and topology
3. Add the telemetry channels for each component into their own packet in the deployment packets file
4. Add the Catalog's ping entries 
5. FIX THE FREAKING DEPLOYMENT
	1. Switch startSocketTask to just start
		1. They renamed the task but have yet to change the deployment autocoder
	2. Switch stopSocketTask to stop
	3. Switch joinSocketTask to join and don't pass in any arguments or parameters
	4. Pass in a Fw::Time object instead of a different object.
		1. Use `Fw::Time milli; milli.set(milliseconds/1000,(milliseconds%1000)*1000);` to do it.
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
8. Configure DpWriter and DpCatalog
	1. DpWriter needs the file path
	2. DpCatalog needs to mallocator passed to it.

