**NOTE: this is only if you're trying to upgrade an existing repository from an old version of F'. If you're just updating your tools then you need something else.**

## Updating F'
- Navigate to the fprime directory
- Find the commit for the most recent version of F' and copy the SHA
- In the fprime directory run `git checkout <commit SHA>`
- git pull

## Updating your virtual environment
If it's a big enough update, they will often add new tools or update tools used by fprime-util. To update to the new tools:
- In the repository base directory run `pip install -r fprime/requirements.txt` to update

## Possible Issues
After updating, you may have problems building and even generating. Here are some of the issues we've run into and how to fix them. This section is only a list of suggestions as JPL changes different things with each update.

- ### Fprime Generate Fails Due to New Key Word
	- If the `fprime-util generate` command fails after updating and gives a single variable as the error, then they may have added a new keyword to the fprime source code. Just change the variable name that's throwing the error and you should be able to generate again.
- ### Building throws error about variables in the fprime source code
	- This could mean that they added or renamed variables defined in the config folder
	- Move the current contents of the config folder to a temporary folder named anything other than "config"
	- From the repository home directory, run the command `cp -r fprime/config config`
	- Compare the "AcConstants.fpp" from the old config folder with the "AcConstants.fpp" from the new config folder.
		- Change all constants in the new "AcConstants.fpp" file to match the constants in the old file.
		- There may be constants in the new file that aren't defined in the old one, ignore them.
- ### Fprime doesn't like your strings
	- In version 3.5.1 Fprime added a new type for strings and stopped accepting typical C and C++ strings.
	- Go through and any normal string you find, turn it into a Fw::String
		- This functions similarly to a c++ String type, you can declare and instantiate it like this: `Fw::String <string name>("Some string goes here");`
- ### Fprime doesn't like your deployment
	- Often times they may change the name of telemetry channels or other variables defined in the deployment.
	- Move the current deployment to a temporary holding folder
	- Create a new deployment
	- Compare the topology.fpp, instances.fpp, FSWDeploymentPackets.xml, FSWDeploymentTopology.cpp, and FSWDeploymentTopologyDefs.hpp with their old counterparts
		- Copy and paste in all the code that is specific to your deployment
			- DO NOT copy and paste the entire file as the new file probably has a lot of changes
		- Update syntax to match any recent changes
			- 3.5.1 changed the syntax for connecting components to the Health component
			- 3.5.1 updated how to initialize a GPIO driver