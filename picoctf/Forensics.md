
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

***
```
