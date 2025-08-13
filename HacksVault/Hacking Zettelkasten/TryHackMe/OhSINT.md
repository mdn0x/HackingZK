- - - 
created : 17-07-2025 

Tags : #easy  
- - - 
# Objective

Practice [OSINT](../../3%20-%20Tags/Hacking%20Concepts/OSINT.md) techniques .
# Scenario

What information can you possible get with just one image file?

**Note:** This challenge was updated on 2024-02-01. If you are following any older walkthroughs, expect a small change. Additionally, the file is also available on the AttackBox, under the `/Rooms/OhSINT` directory.

# Download

We have the background image for the windowsXP Desktop :' ) :

<img src="../../Flameshots/76c34a71ef3a1cf190acd8c38a2e6c56.png" alt="76c34a71ef3a1cf190acd8c38a2e6c56.png" width="366" height="189" class="jop-noMdConv">

## Exiftool

We use [Exiftool](../../3%20-%20Tags/Hacking%20Tools/Exiftool.md) to find metadata info on the image :

```
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Easy/OhSINT]
└─$ exiftool WindowsXP_1551719014755.jpg       
ExifTool Version Number         : 13.25
File Name                       : WindowsXP_1551719014755.jpg
Directory                       : .
File Size                       : 234 kB
File Modification Date/Time     : 2025:07:10 11:34:25+02:00
File Access Date/Time           : 2025:07:10 11:36:59+02:00
File Inode Change Date/Time     : 2025:07:10 11:34:25+02:00
File Permissions                : -rw-rw-r--
File Type                       : JPEG
File Type Extension             : jpg
MIME Type                       : image/jpeg
XMP Toolkit                     : Image::ExifTool 11.27
GPS Latitude                    : 54 deg 17' 41.27" N
GPS Longitude                   : 2 deg 15' 1.33" W
Copyright                       : OWoodflint
Image Width                     : 1920
Image Height                    : 1080
Encoding Process                : Baseline DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:2:0 (2 2)
Image Size                      : 1920x1080
Megapixels                      : 2.1
GPS Latitude Ref                : North
GPS Longitude Ref               : West
GPS Position                    : 54 deg 17' 41.27" N, 2 deg 15' 1.33" W

```

<span style="color: rgb(236, 240, 241);">So we have a GPS Location, and a <span style="color: rgb(45, 194, 107);">username</span> : **OWoodflint .**</span>

# Google Searching

We can search for the name :

<img src="../../Flameshots/e2a897573d45c5a4c75f6a7c49fab33b.png" alt="e2a897573d45c5a4c75f6a7c49fab33b.png" width="423" height="233" class="jop-noMdConv">

We have a Twitter :

<img src="../../Flameshots/7cddbc0c9fa2a7ef99050b0b6e1be2ec.png" alt="7cddbc0c9fa2a7ef99050b0b6e1be2ec.png" width="294" height="337" class="jop-noMdConv">

We can answer first question: What is this user's avatar of?  **cat**

# Wigle.net

Now we search for the BSSID in the Twitter post on [Wigle.net ](Wigle.net.md):

<img src="../../Flameshots/10ce7a7da9417750331a9211fb9dc691.png" alt="10ce7a7da9417750331a9211fb9dc691.png" width="420" height="302" class="jop-noMdConv">

And answer the second and third questions on Location and Name of WAP (zoom on wigle) : **London** , **UnileverWiFi**

And we have a blog :

<img src="../../Flameshots/5468f5a21a782b1e7bb01e858913639c.png" alt="5468f5a21a782b1e7bb01e858913639c.png" width="336" height="145" class="jop-noMdConv"> <img src="../../Flameshots/7d1a436d4051a04fbeccb92b58ff7f7c.png" alt="7d1a436d4051a04fbeccb92b58ff7f7c.png" width="375" height="281" class="jop-noMdConv">

This is the answer to question number 6 : Where has he gone on holiday? **New York**

If we look into the source code of the blog page we can also find a strange string similar to a passwd :

<img src="../../Flameshots/3bdb23f1787ea692a677bf6979da51c4.png" alt="3bdb23f1787ea692a677bf6979da51c4.png" width="482" height="105" class="jop-noMdConv">

We can try answer to the last question on thm and it is .

<img src="../../Flameshots/2e2f9be09a3be33359d51c4ea4a1e36d.png" alt="2e2f9be09a3be33359d51c4ea4a1e36d.png" width="175" height="86" class="jop-noMdConv">

We can try also to get the email for answer 7 :

<img src="../../Flameshots/9888f28dac6d52521f1969f083308e05.png" alt="9888f28dac6d52521f1969f083308e05.png" width="217" height="72" class="jop-noMdConv">    <img src="../../Flameshots/20b9b48313a5cc94795101abeec3bcf6.png" alt="20b9b48313a5cc94795101abeec3bcf6.png" width="238" height="194" class="jop-noMdConv">

But if we look on Google we have a Github account :

<img src="../../Flameshots/7cae868a08e48413e2781382e0f716fa.png" alt="7cae868a08e48413e2781382e0f716fa.png" width="443" height="96" class="jop-noMdConv">               <img src="../../Flameshots/f63e8ef528ddcb03ccbfd732162a955a.png" alt="f63e8ef528ddcb03ccbfd732162a955a.png" width="307" height="136">

And these are the two last answers ; **Github** , **OWoodfint@gmail.com .**
