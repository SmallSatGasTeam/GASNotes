### Opening a new repository
- `git clone <Address>`
	- Creates a new folder and clones a new repository into it
### Updating your repository (Pulling)
- `git pull origin`
	- Pulls down the updates from github
### Uploading your changes (Pushing)
- `git add .`
	- Adds all new files and changes to the git repository
- `git add <file>`
	- Adds a specific file or files to the repository
- `git commit`
	- Commits your changes to the log
	- Will create a pop up that asks for a commit message
		- **LEAVE A GOOD COMMIT MESSAGE**
- `git commit -m "<message>"`
	- Commits your changes to the log
	- Uses the text in the quotations for the commit message
- `git push origin`
	- Uploads your changes to the repository
- Order of operations
	- `git add .`
	- `git commit`
	- `git push origin`

### Branch commands
- `git branch`
	- Lists what branch you are currently in
- `git branch -m <new branch name>`
	- Creates a new branch
	- This branch must be pushed after creation to be seen on GitHub and become available to others
- `git checkout <branch>`
	- Allows you to switch to a different branch

### Checking out Old Commits
- `git log`
	- Prints a log of all previous commits and their SHA (commit number)
- `git checkout <SHA>`
	- Detaches the HEAD and switches it to the commit specified by the SHA
