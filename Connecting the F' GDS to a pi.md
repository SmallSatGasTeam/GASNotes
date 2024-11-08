## Preparing the code

#### Switching to TcpServer
This is specifically for connecting the GDS to the pi across wifi. The first step is to check the comDriver in instances.fpp. comDriver should be an implementation of Drv.TcpServer NOT Drv.TcpClient. TcpServer will allow you to connect the GDS using ip adresses. TcpClient allows you to run the GDS and the F' code locally on your laptop. After making this change the code detailing the comDriver should look like this:

```fpp
instance comDriver: Drv.TcpServer base id 0x4000
```

This should be the only change in the code needed to connect to the pi.

## Installing the cross-compiler

Note, this section is written assuming the user is running on Linux or WSL. This section only needs to be completed the first time you're cross compiling for the pi.

The easiest way to install the cross-compiler is to run the FirstTimeCompile32.sh bash script. If that has been deleted or lost to time then step by step instructions are detailed below.

#### Create a directory for the cross compiler
Run the command `sudo mkdir -p /opt/toolchains` to create a folder in which to install the cross-compiler. Next, run `sudo chown $USER /opt/toolchains` to set your profile as the owner.

### Set the tool path
Run the following two commands to set the ARM_TOOLS_PATH, if you ever have a problem for compiling for arm-hf-linux you may need to run these commands again.
```bash
export ARM_TOOLS_PATH=/opt/toolchains
# this is to check if environment vairable is correctly set
# it should return `/opt/toolchains` if it's all good
echo $ARM_TOOLS_PATH
```

#### Download and install
After setting permissions, use `curl -Ls https://developer.arm.com/-/media/Files/downloads/gnu-a/10.2-2020.11/binrel/gcc-arm-10.2-2020.11-x86_64-arm-none-linux-gnueabihf.tar.xz | tar -JC /opt/toolchains --strip-components=1 -x` to download and install the cross compiler.

#### Verify installation
You can check if the cross-compiler has been properly installed by running `/opt/toolchains/bin/arm-none-linux-gnueabihf-gcc -v`. If anything other than a not found error, the cross-compiler has been correctly installed.

## Generate and build the code
In the root directory of your F' project run `fprime-util generate arm-hf-linux` to generate auto coded files for the raspberry pi implementation of the code. Then run `fprime-util build arm-hf-linux -j $(nproc)` to build that implementation.

## NOTE
Everything from here below can be done running the CompileFor32.sh bash file.

## Copy the code over to the pi
From the root directory of the project run `scp -r build-artifacts/arm-hf-linux/<Deployment Name>/bin/<Deployment Name> <pi address>:<location on the pi>` to transfer the executable file.

## Start the GDS
Next, start the GDS on your computer by running `fprime-gds -n --dictionary build-artifacts/arm-hf-linux/FSWDeployment/dict/FSWDeploymentTopologyAppDictionary.xml --ip-client --ip-address <pi address>`.

## Run the executable file on the pi
SSH into the pi and navigate to where you copied the project executable. Run the executable using `sudo <File name> -a 0.0.0.0 -p 50000` to start the code on the pi.

## Profit
The marker in the top right of the GDS on your computer should now be a little green dot. You have successfully connected to the pi.