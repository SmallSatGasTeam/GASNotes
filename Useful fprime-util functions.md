# fprime-util
## Format is: frpime-util "command"
### generate
- Generates the build files
- Must be used at the beginning of every project before running build
- Does not need to be re-run unless dependent libraries are changed
- `fprime-util generate -j $(nproc)` - uses all cores to generate, takes much less time to build
### build
- Must be run at the beginning of the project in order to use intelli sense
- Compiles the current folder
	-  If run within Deployment folder, it will compile the code to create the executable file
	-  If run within Components folder, will create the "component name"Ai cpp, hpp, and xml files
### new
-   Accepts 1 argument for what new item to create then gives a series of prompts to help you create said item
- Options include: (remove the quotation marks when entering in bash)
	- "--component" 
	- "--deployment"
	- "--project"
	- "--module"
- Note, the fprime documentation mentions being able to use fprime-util new --port, but this is not actually supported. Instead, use --module and create the ports inside that module
### purge
- Deletes the build files
- Often used if you messed something up and are getting weird errors that you don't understand
### impl
- Run within a specific component's directory
- Creates .hpp-template and .cpp-template files based on the fpp file
### fpp-check -u `filename`
- Run inside Top folder of a deployment
- Creates a file with a list of all unconnected ports
### deploy
 - run `fprime-gds` to deploy
 - go to `http://localhost:5000` to access the webpage
### generate 
 - run `fprime-util impl` in the component's local directory
### error
 - `[WARNING] Failed to open port with status -9 and errno 98 fprime` - just reboot the pi

 
