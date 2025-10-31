
# Trivial Flag Transfer Protocol

> Figure out how they moved the flag.

## Solution:

The challenge provides a PCAP file containing TFTP (Trivial File Transfer Protocol) traffic. TFTP is an unencrypted file transfer protocol, so any files transferred can be easily extracted and analyzed.

### Step 1: Export files from TFTP
I opened the `tftp.pcapng` file in Wireshark and exported all TFTP objects using **File → Export Objects → TFTP**. This revealed several files:
- `instructions.txt`
- `plan`
- `program.deb`
- `picture1.bmp`, `picture2.bmp`, `picture3.bmp`

### Step 2: Analyze text files
The `instructions.txt` and `plan` files contained encoded text that appeared to be ROT13 cipher. I decoded them using Python:

```
python3 -c "import codecs; print(codecs.decode('GSGCQBRFAGRAPELCGBHEGENSSVPFBJRZHFGQVFTHVFRBHESYNTGENAFSRE.SVTHERBHGNJNLGBUVQRGURSYNTNAQVJVYYPURPXONPXSBEGURCYNA', 'rot_13'))"
```

Output:
```
TFTPDOESNTENCRYPTOURTRAFFICSOWEMUSTDISGUISEOURFLAGTRANSFER.FIGUREOUTAWAYTOHIDETHEFLAGANDIWILLCHECKBACKFORTHEPLAN
```

```
python3 -c "import codecs; print(codecs.decode('VHFRQGURCEBTENZNAQUVQVGJVGU-QHRQVYVTRAPR.PURPXBHGGURCUBGBF', 'rot_13'))"
```

Output:
```
IUSEDTHEPROGRAMANDHIDITWITH-DUEDILIGENCE.CHECKOUTTHEPHOTOS
```
![WhatsApp Image 2025-10-28 at 20 36 50_0b9b8500](https://github.com/user-attachments/assets/f3438ca3-2f7b-4e23-a8cb-23cfa179add3)

The decoded messages revealed that:
- TFTP doesn't encrypt traffic, so they needed to hide the flag
- Someone used a program to hide the flag with the passphrase "DUEDILIGENCE"
- We should check the photos

### Step 3: Extract hidden data from images
Using the passphrase "DUEDILIGENCE" discovered from the decoded messages, I used `steghide` to extract hidden data from the BMP files:

```
steghide extract -sf picture1.bmp -p DUEDILIGENCE
steghide extract -sf picture2.bmp -p DUEDILIGENCE
steghide extract -sf picture3.bmp -p DUEDILIGENCE
```

The extraction from `picture3.bmp` was successful and produced a file called `flag.txt`.

### Step 4: Retrieve the flag
```
cat flag.txt
```

## Flag:
```
picoCTF{h1dd3n_1n_pla1n_51GHT_18375919}
```

## Concepts learnt:
- TFTP (Trivial File Transfer Protocol): A simple, unencrypted file transfer protocol using UDP port 69
- ROT13 Cipher: A simple letter substitution cipher that rotates letters by 13 positions
- Steganography: The practice of hiding data within other files, such as images
- PCAP Analysis: Examining network packet captures to extract transferred files
- Steghide: A steganography tool that can hide data in various file formats

## Notes:
- Initially tried using the `rot13` command but encountered installation issues, so switched to Python's `codecs` module which worked reliably
- The challenge emphasizes that TFTP transfers are unencrypted, making intercepted files easily accessible
- The passphrase "DUEDILIGENCE" was crucial for extracting the hidden flag from the image
- Multiple BMP files were provided, but only `picture3.bmp` contained the actual flag

## Resources:
- Wireshark Documentation: https://www.wireshark.org/docs/
- Steghide Manual: http://steghide.sourceforge.net/documentation.php
- ROT13 Cipher Explanation: https://en.wikipedia.org/wiki/ROT13
- TFTP Protocol Overview: https://en.wikipedia.org/wiki/Trivial_File_Transfer_Protocol


# tunn3l v1s10n

> We found this file. Recover the flag.

## Solution:

The challenge provides a corrupted BMP image file that doesn't display properly. The hint "Weird that it won't display right..." suggests the image header is malformed.

### Step 1: Analyze the file type
First, I checked what type of file we're dealing with:

```
file tunn3l_v1s10n
```

Output:
```
tunn3l_v1s10n: data
```

The file command couldn't identify it properly, confirming it's corrupted.

### Step 2: Examine the hex header
I looked at the file's hex header to understand the structure:

```
xxd tunn3l_v1s10n | head -10
```

This showed the BMP signature was present but other header fields were incorrect.

### Step 3: Create Python script to fix BMP header
I created a Python script to analyze and fix the BMP header issues:

```python
with open('tunn3l_v1s10n', 'rb') as f:
    data = bytearray(f.read())

print("Original header analysis:")
print(f"DIB header size: {int.from_bytes(data[14:18], 'little')}")
print(f"Width: {int.from_bytes(data[18:22], 'little')}")
print(f"Height: {int.from_bytes(data[22:26], 'little')}")

data[14:18] = (124).to_bytes(4, 'little')
data[10:14] = (138).to_bytes(4, 'little')
data[34:38] = (0).to_bytes(4, 'little')

height = int.from_bytes(data[22:26], 'little', signed=True)
if height < 0:
    data[22:26] = abs(height).to_bytes(4, 'little')

with open('fixed.bmp', 'wb') as f:
    f.write(data)

print("Created fixed.bmp")
```

### Step 4: Run the fix script
```
python3 fix_bmp.py
```

Output:
```
Original header analysis:
DIB header size: 40
Width: 1134
Height: -306
Created fixed.bmp
```

### Step 5: View the fixed image
After fixing the BMP header, the image displayed properly and revealed the flag.


## Flag:
```
picoCTF{qu1t3_a_v13w_2020}
```

## Concepts learnt:
- BMP File Structure: Understanding the bitmap file format including headers and data organization
- File Header Analysis: Examining hex headers to identify file corruption
- Hex Editing: Modifying binary files at specific offsets to fix corrupted data
- Image Forensics: Recovering data from damaged or malformed image files

## Notes:
- The challenge required fixing multiple BMP header fields simultaneously
- Key issues included wrong DIB header size, incorrect pixel data offset, and negative height value
- BMP files have a specific structure that must be correct for proper rendering
- Tools like xxd and Python's byte manipulation are essential for binary file analysis

## Resources:
- BMP File Format Specification
- Python Bytes Documentation

