# Set up Environment Mac M1

Run the project in docker dev-container

## steps
1. download visual studios code & docker

2. install extensions(remote container)

3. click the button [reopen in a contain]

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
