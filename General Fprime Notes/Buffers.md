 - To send data via a buffer 
	 - make sure your component has two output ports `allocate` and `deallocate` that are connected to `bufferManager.bufferGetCallee` and `bufferManager.bufferSendIn`
		 - ex (in your components' fpp file):
			 - `output port allocate: Fw.BufferGet` and `output port deallocate: Fw.BufferGet`
			 - note: you don't need to run `fprime-util impl` after adding these since these are just output ports
		 - ex (in your projects' topology file):
			 - `myComponent.allocate -> bufferManager.bufferGetCallee` and `myComponent.deallocate -> bufferManager.bufferSendIn`
	 - then once you have this you need to assign the buffer to a pointer
		 - `Fw::SerializeBufferBase& myBufferPointer = myBuffer.getSerializeRepr();`
	 - then you can serialize your data into the buffer
		 - ex with a U8: `U8 myData = 0x01;`
			  ```c++
			  myBufferPointer.resetSer();
			  myBufferPointer.serialize(myData);
				```
	 - then you send that buffer out through a port of type `Fw.BufferSend` to whatever component it needs to go to
 - To receive data via a buffer
	 - assign the buffer being passed in to a pointer
		 - `Fw:SerializeBufferBase& sb = fwBuffer.getSerializeRepr();`
	 - set the size of the buffer based on its size (it's weird ik)
		 - `sb.setBuffLen(fwBuffer.getSize());`
	 - then create a variable with the size that you expect to be receiving, and deserialize 
		 - ex: `U8 myData;`
		 - the deserialize function returns the data via pass-by-reference
			 - ex: `sb.deserialize(myData);`
			 - `myData` will now hold the data that was contained in the buffer
- Full example
```c++
// Putting data into buffer
Fw::Buffer myBuffer = this->allocate_out(0, 1);
Fw::SerializeBufferBase& myBufferPointer = myBuffer.getSerializeRepr();

U8 myData = 0x01;
myBufferPointer.resetSer();
myBufferPointer.serialize(myData);
// at this point you would send the buffer out

// Reading data from buffer (fwBuffer is given to the function if it's an input port handler of type `Fw.BufferSend` )
Fw::SerializeBufferBase& sb = fwBuffer.getSerializeRepr();
sb.setBuffLen(fwBuffer.getSize());

U8 myDeserializedData;
sb.deserialize(myDeserializedData);
// then you'd possibly format the data and output it to an event or telemetry. 

```

# Buffer Behaviors
 - Clumps data together
	 - for example if you serialize four `U8's` into the same buffer, all that data can be received and interpreted at a `U32`
 - Won't get mad at you
	 - if you give the buffer more data than you specified it can take, it will output as much of the data as it has size for, and probably run into overflow issues and give you odd looking data
		 - tldr: if you supply too much data, it won't error it'll just output unexpected data
 - Buffers are read and creating with Big-endian (as far as we can tell)