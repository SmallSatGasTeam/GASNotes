# Connecting the Endurosat EPS 1 to the pi
## References
*  This file refers to the Endurosat EPS 1 & EPS 1 Plus data sheet.
*  It will explain how to connect on hardware first, then go into the software involving F-Prime and the I2C communication protocol.
## Hardware
//TODO
## Software
### F-Prime
*  Make sure that in your component you have established connection with the I2C Linux Driver that F-Prime provides.
*  Without this, you will not be able to connect to the EPS and see data when you open your GDS.
*  Information on how to do this can be found in "Connecting the I2C Linux Driver to your component.md"
### I2C Connection
*  In your component, make sure that your I2C device address is correct.
*  Once you have found your I2C address on the pi using the command `i2cdetect -y 1`, confirm that is the same address used in software.
#### Reading From Device
*  When reading from the I2C device, there are some things that you have to keep in mind:
    *  The EPS address should be left-shifted. 
    *  The result of a READ command being sent will always be a buffer of 2 bytes.
*  There is a command provided by F-Prime that handles most of the I2C bit-shifting process. The command goes as follows:
    *  `i2cReadWrite_out(0, reg, writeBuffer, readBuffer);`
    *  Argument 1 is the port number
    *  Argument 2 is the device address
    *  Argument 3 is the writeBuffer (The buffer with the command number)
    *  Argument 4 is the readBuffer (The buffer that the READ data will go into, 2 bytes long)
*  You can use this method when you have the device address and command number that you want to send, and it will return a readBuffer that is 2 bytes long containing READ data.
*  To get the raw data from the readBuffer. you can use something like the following method:
```
   I16 getSensorReadData(Fw::Buffer readBuffer){
    //! Get data out of the buffer
    U8* data = reinterpret_cast<U8*>(readBuffer.getData());
    U8 msb = data[0];
    U8 lsb = data[1];

    //! Combine the elements of the buffer
    I16 rawData = (static_cast<U16>(msb) << 8) | lsb;
    
    //! Deallocate read buffer
    this->deallocate_out(0, readBuffer);

    return rawData;
  }
```
#### Writing To The Device
*  The process of writing to the I2C device is very similar to Reading:
    *  The EPS adress should be left-shifted.
    *  The result of a WRITE command will be void (no returned buffer)
*  There is a command provided by F-Prime that handles the I2C bit-shifting process. The command goes as follows:
    *  `i2cWrite_out(0, reg, writeBuffer);`
    *  Argument 1 is the port number
    *  Argument 2 is the device address
    *  Argument 3 is the writeBuffer
*  The writeBuffer in argument 3 should contain (1)the command number and (2)the state (ON, OFF, AUTO, etc.)
*  Use this method when you have the device address, command number, and state of the comman.
*  It is simply a set of instructions, so there will be no returned buffer.

