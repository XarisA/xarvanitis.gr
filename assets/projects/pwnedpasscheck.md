# Pwned Password Check Tool

***pwnedpasscheck***s for pwned passwords from [haveibeenpwned.com](https://haveibeenpwned.com/API/v2#PwnedPasswords) v2 API by using the [pwnedpasswords](https://github.com/lionheart/pwnedpasswords) library.

*[View on GitHub](https://github.com/XarisA/pwnedpasscheck)*

How to use:
---

```bash
python pwnedpasscheck.py -h
  
usage: pwnedpasscheck.py [-h] [-p PASSWORD] [-f FILE]

optional arguments:
  -h, --help            show this help message and exit
  -p PASSWORD, --password PASSWORD
                        The password you want to test
  -f FILE, --file FILE  Load a file with multiple passwords to check
```

Installation
---

```bash
# Install pwnedpasswords library
$ pip install pwnedpasswords

# Download the file (without git installed)
$ wget https://github.com/XarisA/pwnedpasscheck/archive/master.zip -O pwnedpasscheck.zip
# Extract it and clean up
$ unzip pwnedpasscheck.zip -d pwnedpasscheck
$ mv pwnedpasscheck/pwnedpasscheck-master/* pwnedpasscheck && rm -rf pwnedpasscheck/pwnedpasscheck-master && rm pwnedpasscheck.zip

# Download the file (with git)
$ git clone https://github.com/XarisA/pwnedpasscheck.git

# Make it executable
$ cd pwnedpasscheck
$ chmod +x pwnedpasscheck.py

# Run the program
$ ./pwnedpasscheck.py
```


Examples 
---

#### Option 1: Call the interpreter

![Call The interpreter](https://user-images.githubusercontent.com/3985557/50558943-c05a2000-0cfa-11e9-823d-56e55ec08aa1.png)

#### Option 2: Let the script call the interpreter (linux only)

![Script1](https://user-images.githubusercontent.com/3985557/50558940-bfc18980-0cfa-11e9-84ba-34284e7241bc.png)

![Script2](https://user-images.githubusercontent.com/3985557/50558941-bfc18980-0cfa-11e9-8005-03450017e3c2.png)

![Script3](https://user-images.githubusercontent.com/3985557/50558942-c05a2000-0cfa-11e9-9509-0eff14123e47.png)


Security Note from [lionheart/pwnedpasswords](https://github.com/lionheart/pwnedpasswords)
---

*No plaintext passwords ever leave your machine using pwnedpasswords.
How does that work? Well, the Pwned Passwords v2 API has a pretty cool k-anonymity implementation.*

*From https://blog.cloudflare.com/validating-leaked-passwords-with-k-anonymity/:*

>Formally, a data set can be said to hold the property of k-anonymity, if for every record in a released table, there are k − 1 other records identical to it.

*This allows us to only provide the first 5 characters of the SHA-1 hash of the password in question. The API then responds with a list of SHA-1 hash suffixes with that prefix. On average, that list contains 478 results.
People smarter than I am have used math to prove that 5-character prefixes are sufficient to maintain k-anonymity for this database.*

In short: your plaintext passwords are protected if you use this library. You won't leak enough data to identity which passwords you're searching for.
