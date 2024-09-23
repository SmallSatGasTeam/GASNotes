## Activate WSL

- Search power shell or command line in your applications (press the windows thingy then search them)
- Right click one of them and click `Run as administrator`
- In the terminal type `wsl --install`
## Install Ubuntu

- Open the Microsoft store
- Search ubuntu
- Install the Ubuntu app that doesn't have a bunch of fancy letters next to it
- Open the ubuntu app, create a username and password (REMEMBER THESE)
## Update applications

- In the Ubuntu terminal enter the following commands
	- `sudo apt update`
	- `sudo apt upgrade`
## Install needed applications

- Run the following command in the Ubuntu terminal
	- `sudo apt install python3 pip python3-venv cmake`
## Install F'

- Run the following command in the terminal
	- `pip install fprime-bootstrap`

## Adding to the path

- If you receive the warning, "the script fprime-bootstrap is installed in `folder` which is not on PATH..." you need to add `folder` to your path
- Run the command `nano ~/.bashrc` in your terminal
	- This should open a text editor on the command line
	- Use the arrow keys to move down to the very bottom and then add:
	- `export PATH=$PATH:<folder>` 
		- Replace `<folder>` with the file path fprime bootstrap gave you earlier
- Use ctrl + x to save and exit

- Run this command to ensure you completed it correctly
	- `fprime-bootstrap --help`
	- If this shows a usage you've successfully installed F' on your computer!
