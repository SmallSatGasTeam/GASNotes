## Required Ports
To connect to the I2C Driver your component needs the following 2 ports:
```fpp
@ Reads data from the i2c line
output port i2cRead: Drv.I2c  

@ Writes data on the i2c line
output port i2cWrite: Drv.I2c
```
and or the following 1 port:
```fpp
@ Port for synchronous writing and reading from I2C
guarded input port writeRead: Drv.I2cWriteRead
```

The ports can be named whatever you want but ensure the use the Drv.I2c port definition for the first kind or the Drv.I2cWriteRead for the second option.

## Adding the I2c driver to the Deployment

### Instances.fpp

Add the following line to the instances file in your deployment right below the other passive component declarations.

```fpp
instance i2cDriver: Drv.LinuxI2cDriver base id 0x4E00
```

Once again feel free to name the component whatever, but if you change its name from i2cDriver, be sure to use the new name in the following steps. !!! Don't forget to edit the base id so that it does not overlap with any other components in your deployment.

### topology.fpp

Near the top of the topology file add this line below the other instance declarations (the lines that look similar to this one).

```fpp
instance i2cDriver
```

Next, add these lines of code within your deployment connections (right after the comment saying "# Add here connections to user-defined components").

```fpp
myComponent.i2cRead -> i2cDriver.read

myComponent.i2cWrite -> i2cDriver.write
```

Replace `myComponent` with the name of the component sending i2c commands.

### DeploymentTopology.cpp

In the DeploymentTopology.cpp file add the following lines at the end of the `setupTopology` function.

```c++
//Configure i2c Device
const char* device = "/dev/i2c-1";
i2cDriver.open(device);
```

## Connecting to hardware

Finally, connect pi pin GPIO2 to the serial data port (SDA) on your peripheral and pi pin GPIO3 to the serial clock port (SCLK) on your peripheral.

## Compile For and Run on the Pi

See [[Connecting the F' GDS to a pi]].