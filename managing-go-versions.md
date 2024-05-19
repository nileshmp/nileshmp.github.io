# Managing Multiple Go Versions

Many a times we have the requirement working with multiple go versions, for example we are part of different projects with different repositories which work with different versions of Golang. This blog here tries to address 2 problem statements;

- Ability to work with multiple go versions across repositories.
- Ability to switch to the required version based on the repository, and automating this process.

## How to work with multiple go versions?

Golang ecosystem has many version managers specifically to deal with this task. In this blog we will work with [gvm](https://github.com/moovweb/gvm).  

Installing `gvm` on mac osx is as simple as going to the terminal, paste the below bash script and executing this script

```bash
	bash < <(curl -s -S -L https://raw.githubusercontent.com/moovweb/gvm/master/binscripts/gvm-installer)
```

The installation happens under `USER_HOME` folder and you will see a directory `.gvm` created under the users's home folder. As part of the bash-script execution you will be asked to run the following command

```bash
	source /Users/<user_home_dir>/.gvm/scripts/gvm
```

Based on the shell you are using i.e. `bash` or `Z shell` (you can read about it [here](https://www.geeksforgeeks.org/bash-scripting-difference-between-zsh-and-bash/)), you can create a file under `home_directory` (either `.bashrc` or `.zshrc`)

Once `gvm` is installed you can go ahead and install the required go versions using `gvm`.
You can list the supported go-versions using  `gvm listall`. Post which you can install it using `go install go1.19`.

> [!NOTE]
>  `go install` can fail if there is no go available. In which case one can install the latest go using `brew install go`

> [!NOTE]
>  You can find the go versions installed via `gvm` under directory `/Users/<user-home>/.gvm/gos`

>[!WARNING]
>You will not see the version installed using `brew` under the `.gvm/gos`

All the above instructions are available on the [gvm github](https://github.com/moovweb/gvm)

## Automation

### changing the go version automatically when you `cd` to the repository source.

#### Creating a `.gvmrc` 

`.gvmrc` can be created under each repository or source folder where the go version changes from the default preferred across the systems.

Add the following scrptlet in file `.bashrc` or `.zshrc` depending on your preferred shell.

```bash
	gvm.init()
	{
	   if [ -z "$1" ]
	   then
	       echo "go version not passed..."
	   else
	       echo "Go version passed is ....$1"
	       touch .gvmrc
	       echo "go_version=$1" > .gvmrc
	       # echo -e ".gvmrc created with contents ; " ; cat .gvmrc
	       load.gvmrc
	   fi
	  
	}
	
	load.gvmrc() {
	   # detect if the current directory has a .gvmrc file
	 if [ -e .gvmrc ]; then
	   # load the gvmrc file and read the go.version property file
	   source .gvmrc
	   gvm use $go_version
	 fi
	}

```

The following is the usage; open the terminal and `cd` the desired repository or source directory. Once done you type the following command,

```bash
	gvm.init go1.19
```

This will create a `.gvmrc` file with the following content `go_version=go1.19`

>[!NOTE]
>Makes sure the go version `go1.19` we passed is installed and available. 

``
#### Overriding the default `cd` behaviour

This is pretty advanced scripting and once should be careful and use discretion before venturing this route. However this seems to have worked just fine for me so far.

Copy the following scriptlet in `.bashrc` or `.zshrc` depending on your preferred shell.

```bash
	## advanced scripts
	### running custom logic when cding into a directory
	cd() {
	   if [ -z "$1" ]
	   then
	       builtin cd
	   else
	       builtin cd "$1"
	   fi
	   ## Added to recursively find the .gvmrc file in the ancestry
	   recursively.find.gvmrc.and.load 
	}

    recursively.find.gvmrc.and.load()
	{
	   WORKDIR=$(pwd)
	   HOMEDIR=$(echo $HOME)
	   until test "$WORKDIR" = "$HOMEDIR"
	   do
	       if [ -e "$WORKDIR/.gvmrc" ]; then
	           echo "detected .gvmrc file in $WORKDIR"
	           # # load the gvmrc file and read the go.version property file
	           source "$WORKDIR/.gvmrc"
	           gvm use $go_version
	           break
	       fi
	       # get the full path of the script
	       WORKDIR=$(dirname $WORKDIR)
	   done
	}

```

This will load the configured go version required when you `cd` to the source or repository directory.

>[!NOTE]
>It is a good practice to create a `.gvmrc` under he home folder for the user configured to the default version you need across the system. 
>Having this under `user_home` folder also means that the `bash` or `Z shell` will be able to recursively find it here if not found under a specific source or repository directory. 
>Having `.gvmrc` file in specific folder works like an override to the version configured under the `user_home` directory


#### Loading the right version when opening the terminal

Finally add the following line to the file `.bashrc` or `.zshrc` depending on your preferred shell.

```bash
	# this helps us set golang version based on .gvmrc file when starting a new terminal/cmd session
	recursively.find.gvmrc.and.load
```


#### `.bashrc` ro `.zshrc` files after all the above changes

```bash
	gvm.init()
	{
	   if [ -z "$1" ]
	   then
	       echo "go version not passed..."
	   else
	       echo "Go version passed is ....$1"
	       touch .gvmrc
	       echo "go_version=$1" > .gvmrc
	       # echo -e ".gvmrc created with contents ; " ; cat .gvmrc
	       load.gvmrc
	   fi
	  
	}
	
	load.gvmrc() {
	   # detect if the current directory has a .gvmrc file
	 if [ -e .gvmrc ]; then
	   # load the gvmrc file and read the go.version property file
	   source .gvmrc
	   gvm use $go_version
	 fi
	}
	## advanced scripts
	### running custom logic when cding into a directory
	cd() {
	   if [ -z "$1" ]
	   then
	       builtin cd
	   else
	       builtin cd "$1"
	   fi
	   ## Added to recursively find the .gvmrc file in the ancestry
	   recursively.find.gvmrc.and.load 
	}

    recursively.find.gvmrc.and.load()
	{
	   WORKDIR=$(pwd)
	   HOMEDIR=$(echo $HOME)
	   until test "$WORKDIR" = "$HOMEDIR"
	   do
	       if [ -e "$WORKDIR/.gvmrc" ]; then
	           echo "detected .gvmrc file in $WORKDIR"
	           # # load the gvmrc file and read the go.version property file
	           source "$WORKDIR/.gvmrc"
	           gvm use $go_version
	           break
	       fi
	       # get the full path of the script
	       WORKDIR=$(dirname $WORKDIR)
	   done
	}

	# this helps us set golang version based on .gvmrc file when starting a new terminal/cmd session
	recursively.find.gvmrc.and.load

```

