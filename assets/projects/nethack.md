# Nethack: A Suite for Testing Various Networking Tools build in Python.

A suite for testing various networking tools build in python.
*[View on GitHub](https://github.com/XarisA/nethack)*

## Installation

```shell
# Download the file
$ wget https://github.com/XarisA/nethack/archive/master.zip -O nethack.zip

# Extract it and clean up
$ unzip nethack.zip -d nethack
$ mv nethack/nethack-master/* nethack && rm -rf nethack/nethack-master && rm nethack.zip
$ cd nethack

# Make it executable
$ chmod +x toolsname.py

# Run the program
$ ./toolsname.py
```

---

## BruteForceFTP


Dictionary Attack on FTP server

### How to use:

```shell
python BruteForceFTP.py -h

usage: FTP Dictionary Attack [-h] -a ADDRESS -d DICTIONARY
                             (-s SINGLEUSER | -u USERLIST)

arguments:
  -h, --help            show this help message and exit
  -a ADDRESS, --address ADDRESS
                        The target IP address
  -d DICTIONARY, --dictionary DICTIONARY
                        The path of the dictionary file
  -s SINGLEUSER, --singleuser SINGLEUSER
                        The username to attack (single user mode)
  -u USERLIST, --userlist USERLIST
                        The path of the file with the usernames to attack (do
                        not use with -s)
```

```shell
# Make it executable
$ chmod +x BruteForceFTP.py

# Run the program
$ ./BruteForceFTP.py
```

### Examples:


```shell
# Let the script call the interpreter (linux only)
$ ./BruteForceFTP.py -a 192.168.1.15 -d data/passwords.lst -u data/usernames.lst
$ ./python BruteForceFTP.py -a 192.168.1.15 -d data/passwords.lst -s ftpuser

# Calling the interpreter
$ python BruteForceFTP.py -a 10.0.14.87 -d data/passwords.lst -u data/usernames.lst
$ python BruteForceFTP.py -a 192.168.2.65 -d data/passwords.lst -s ftpuser
```

## PortChecker

A python tool to check for open ports of a remote host.
PortChecker works with ip as well as with domain name.
You can use a single port or multiple ones seperated with comma.

### Example:

![PortChecker](https://user-images.githubusercontent.com/3985557/99880912-071ecf00-2c1f-11eb-9e3c-70f51fbb81fe.png)

## GetInfo

GetInfo is a python tool that gathers information about a specific host.
More specifically it checks if the host is up and finds it's mx records.

dnspython must be installed in order to use this tool.
You can install it with the following command.

```shell
pip install dnspython
```

### Example:

![GetInfo](https://user-images.githubusercontent.com/3985557/99881117-54e80700-2c20-11eb-957a-404c4232b0be.png)
