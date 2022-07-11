# Set up Environment Mac M1

Run the project in docker dev-container

## steps
1. download visual studios code & docker

2. install extensions(remote container)

3. click the button [reopen in a contain]

*Load the project*

*There are two ways of doing this:*

1)Load VS Code, and it should ask you to re-open the project in a container, if it finds the .devcontainer folder.

2)If not, you can open the Command Palette, and run “Remote-Containers: Rebuild and Reopen in Container”.

The first time you do this, it may take a little while as it needs to build the Docker image. After the first load, each time you open the project, it should be much quicker (Unless you change the Dockerfile).

You are now using the Dev Container in VS Code. When you open the terminal, you are inside the container. Plugins are running inside the container too.

### Error  :  Java gateway process exited before sending its port number 



Usually happens when java doesn't exist.

**Ways to check:(for Mac m1 if java installed)**
```bash
# Install Homebrew
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install OpenJDK
brew install openjdk
```
Verify it's installed:
```bash
$(brew --prefix openjdk)/bin/java --version
```
Verify it's for the arm64 hardware:
```bash
file $(brew --prefix openjdk)/bin/java     
# /opt/homebrew/opt/openjdk/bin/java: Mach-O 64-bit executable arm64
```

**Ways to set up java home**


check the version
```bash
$ java -version
java version "11.0.2" 2019-01-15 LTS
Java(TM) SE Runtime Environment 18.9 (build 11.0.2+9-LTS)
Java HotSpot(TM) 64-Bit Server VM 18.9 (build 11.0.2+9-LTS, mixed mode)
```
find where jdk is installed
```bash
$ cd /usr/lib/jvm/
$ ls
java-1.8.0-openjdk-arm64 java-8-openjdk-arm64
```
set environment variable
```bash
$ export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-arm64
```
check the environment variable
```bash
$ echo JAVA_HOME
/usr/lib/jvm/java-8-openjdk-arm64
```



## License

# set up Anaconda for Mac M1

1.download Anaconda for mac m1 from offical website

2.zsh: Command Not Found Conda

first open the .zshrc file and then write the following code there
```bash
$ export PATH="$PATH:/Users/Dz/anaconda/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/opt/X11/bin:/Users/Dz/.rvm/bin"
```


Then, you can run the following command to check it.
```bash
$ source ~/.zshrc
$ echo $HOME
$ echo $PATH
```


3.Problems installing python packages on Mac M1

using conda to use python in a virtual environment

First Enable the open with rosetta in your zsh.

```bash
# create environment in conda
$ conda create -n venv python=3.8 # with your python version

# activate
$ conda activate venv
```

You need to enable that specific channel to get that package with this command:
```bash
# config channel
conda config --append channels conda-forge # available channel name

# then install
conda install --yes --file requirements.txt

```


# Git
```bash
git -version

git init

git status

#create .gitignore files to hide files

git add .

git commit -m "message"

#pushing an existing repo from the command line
git remote add origin //https:github.com...

#fix problems for difference between remote and local 
git pull

git push 

```
