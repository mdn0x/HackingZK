**What is base64**  

Base64 is a group of binary-to-text encoding schemes that represent binary data in an [ASCII](./ASCII.md) string format. In summary, it is just another way in which data can be represented; some systems rely on a base64 encoding of data for processing while others may not. Head over to: [https://www.base64encode.net/](https://www.base64encode.net/)  and input any text and encode it. You should end up with something similar below:

As you can see, the string 'example' gets coverted to the base64 ' ZXhhbXBsZQ ' encoded string. This would allow certain systems to now be able to read and process the data properly.  

![](https://i.imgur.com/RR3uS3d.png)  

**Why should I care?**  

There may be times when you encounter base64 converted data in files on a system and needed to convert it to a human readable format. Therefore, we can use the tool 'base64' with a pipe to translate it back to something that is human readable.

cat filename | base64 -d  

![](https://i.imgur.com/JiGYvDY.png)

to transfer the output to a new text file simply use he > operator followed by the new filename.

![](https://i.imgur.com/ahBMfSy.png)  
  
  
Once again, encoding/decoding is changing data format. The data itself does not change, just how it reads.  
  
I encourage you to spend some time reading about encoding. At least enough so you understand the difference between encoding/decoding vs encryption/decryption as beginners sometimes confuse them with each other.