## Stegcracker Overview

Stegcracker is a specialized tool designed for cracking passwords embedded in steganographic files, particularly those that hide data within images. It automates the process of extracting hidden information from various image formats and is particularly useful for security professionals and researchers working in the field of digital forensics and information security.

### Key Features

- **Password Cracking**: Stegcracker uses wordlists to attempt to crack passwords protecting hidden data in images, making it effective for recovering concealed information.

- **Support for Multiple Formats**: The tool supports various image formats, including JPEG, PNG, and BMP, which are commonly used for steganography.

- **Integration with Other Tools**: Stegcracker can work alongside other steganography tools, such as `steghide`, to extract hidden data once the password is cracked.

- **Customizable Wordlists**: Users can specify their own wordlists or use built-in lists to enhance the cracking process.

- **User-Friendly Interface**: While primarily a command-line tool, Stegcracker is designed to be straightforward, making it accessible for users with varying levels of technical expertise.

### Installation

To install Stegcracker, you can clone the repository from GitHub and install the required dependencies. Here’s how to do it:

```bash
git clone https://github.com/Paradoxis/Stegcracker.git
cd Stegcracker
pip install -r requirements.txt
```

### Basic Commands

Here are some common commands and options for using Stegcracker:

#### 1. Cracking a Password

To crack a password from a steganographic image file, use the following command:

```bash
python stegcracker.py <image_file> <wordlist>
```

For example:

```bash
python stegcracker.py secret_image.jpg wordlist.txt
```

#### 2. Specifying Output Directory

To specify an output directory for the extracted data, use the `-o` option:

```bash
python stegcracker.py secret_image.jpg wordlist.txt -o output_directory
```

#### 3. Using Built-in Wordlists

Stegcracker may come with built-in wordlists. You can use them directly by specifying the path:

```bash
python stegcracker.py secret_image.jpg /path/to/built_in_wordlist.txt
```

### Example Usage

Here’s an example of cracking a password from an image file named `hidden_data.png` using a custom wordlist:

```bash
python stegcracker.py hidden_data.png my_wordlist.txt
```

### Conclusion

Stegcracker is a valuable tool for anyone involved in digital forensics or security research, providing an efficient way to recover hidden data from steganographic files. Its ability to automate the password cracking process and support for multiple image formats make it an essential resource for uncovering concealed information. By utilizing the commands and options outlined, users can effectively leverage Stegcracker in their investigations.