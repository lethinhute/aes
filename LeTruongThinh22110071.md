# Lê Trường Thịnh 22110071

# 4.1. Encrypt and Decrypt Text file

For this lab, we will use WSL running Ubuntu that have installed openssl.

First, we create a txt file in linux user folder named plain.txt.

## a. ecb encryption:

We will use the following key: 00112233445566778899AABBCCDDEEFF00112233445566778899AABBCCDDEEFF

Encrypting plain.txt using the following command on linux terminal to encrypt the text by aes encryption ecb mode:

```
openssl enc -aes-256-ecb -nosalt -in plain.txt -out ecb_encrypted.txt -K 00112233445566778899AABBCCDDEEFF00112233445566778899AABBCCDDEEFF
```

The option -nosalt will make the command not requiring extra input on salt. This is not recommended in normal encryption, but since we are just watching what aes will do, salt isn't neccessary.

After encrypting plain.txt, we will decrypt the previous output using the following command:

```
openssl enc -d -aes-256-ecb -nosalt -in ecb_encrypted.txt -out ecb_decrypted.txt -K 00112233445566778899AABBCCDDEEFF00112233445566778899AABBCCDDEEFF
```

By adding the extra -d, we have successfully the decrypt the text file.

## b. cbc encryption

We will use the key from the last part, but this time it will come with the initialization vector -iv 0102030405060708090A0B0C0D0E0F10

cbc mode will XOR the current encryption block with the previous one, but since the first block wont have any previous one to XOR, it will XOR with the initialization vector.

We will encrypt the previous plain.txt with aes cbc using the following command:

```
openssl enc -aes-256-cbc -nosalt -in plain.txt -out cbc_encrypted.txt -K 00112233445566778899AABBCCDDEEFF00112233445566778899AABBCCDDEEFF -iv 0102030405060708090A0B0C0D0E0F10
```

We can decrypt the text by adding -d to the command:

```
openssl enc -d -aes-256-cbc -nosalt -in cbc_encrypted.txt -out cbc_decrypted.txt -K 00112233445566778899AABBCCDDEEFF00112233445566778899AABBCCDDEEFF -iv 0102030405060708090A0B0C0D0E0F10
```

# 4.2 Encryption Mode – ECB vs. CBC 

We download this picture and rename it to origin.bmp.

Since we only want the data of the bitmap to be encrypted and not the header that determine what the file is, we need to keep the first 54 of origin.bmp using the following command:

```
dd if=origin.bmp of=header.bin bs=1 count=54
dd if=origin.bmp of=body.bin bs=1 skip=54
```

## a. ecb encryption

After the setup, we encrypt body.bin using ecb and key from the previous part:

```
openssl enc -aes-256-ecb -nosalt -in body.bin -out encrypted_body.bin -K 00112233445566778899AABBCCDDEEFF00112233445566778899AABBCCDDEEFF
cat header.bin encrypted_body.bin > partially_encrypted.bmp
```

The created bitmap while completely lost it color, still contain the original overall shape. 

## b. cbc encryption

We will do the exact same thing as ecb, but replace it with cbc and adding initialization vector:

```
openssl enc -aes-256-cbc -nosalt -in body.bin -out encryptedcbc_body.bin -K 00112233445566778899AABBCCDDEEFF00112233445566778899AABBCCDDEEFF -iv 0102030405060708090A0B0C0D0E0F10
cat header.bin encryptedcbc_body.bin > partially_encryptedcbc.bmp
```
The result is a completely unrecognizable picture with random pixel colors.

## Observation and conclusion

As we can see from 2 picture, ecb encryption bmp retain it original structure while cbc make it completely unrecognizable.

Using ecb, hacker while still hard to crack what is in the data, they will still able to see the remain structure, repetition and get useful information from there. This is because ecb only decrypt block by block repeatedly without any changes, so same data will have same output, hence the struture will remain.

cbc on the other hand, make the bitmap unrecognizable, so hacker will barely get anything from the encrypted data itself. This is because the data of each block will be XORed with the previous block, making this a chain of changes that make the whole thing random and hard to get anything useful.

# 4.3 Encryption Mode – Corrupted Cipher Text

First we create a text file called text1.txt that's more than 64 bytes long.

Then we will encrypt the file using different aes mode (text).

After that, we will corrupt the 5th byte of the message

```
dd if=(text).txt of=(name).txt bs=1 count=4
dd if=(text).txt of=(name2).txt bs=1 skip=5
dd if=(text).txt of=(name3).txt bs=1 count=1
cat (name).txt (name3).txt (name2).txt > (name4).txt
```

This command will copy the first byte of the file and insert it into the 5th byte. 

## a. Guessing what each mode will affect the decrypted message if the encrypted one is corrupted

As what we know, ecb only affect the encrypted data block by block without any further changes, therefore the decrypted message will remain what it is for the most part except the block that was corrupted

cbc and probably the other one might have decrypted into unrecognizable message due to how those chain block by block together, hence the small change from first block will affect the entire message

## b. Encrypting/decrypting

We will use the set of key and iv from previous part: -K 00112233445566778899AABBCCDDEEFF00112233445566778899AABBCCDDEEFF -iv 0102030405060708090A0B0C0D0E0F10

### Using ecb

Encryption:
```
openssl enc -aes-256-ecb -nosalt -in text1.txt -out ecbtext1.txt -K 00112233445566778899AABBCCDDEEFF00112233445566778899AABBCCDDEEFF
dd if=ecbtext1.txt of=ecb1.txt bs=1 count=4
dd if=ecbtext1.txt of=ecb2.txt bs=1 skip=5
dd if=ecbtext1.txt of=ecb3.txt bs=1 count=1
cat ecb1.txt ecb3.txt ecb2.txt > corruptecb.txt
```

Decryption:

```
openssl enc -d -aes-256-ecb -nosalt -in corruptecb.txt -out ecbout.txt -K 00112233445566778899AABBCCDDEEFF00112233445566778899AABBCCDDEEFF
```

### Using cbc

Encryption:
```
openssl enc -aes-256-cbc -nosalt -in text1.txt -out cbctext1.txt -K 00112233445566778899AABBCCDDEEFF00112233445566778899AABBCCDDEEFF -iv 0102030405060708090A0B0C0D0E0F10
dd if=cbctext1.txt of=cbc1.txt bs=1 count=4
dd if=cbctext1.txt of=cbc2.txt bs=1 skip=5
dd if=cbctext1.txt of=cbc3.txt bs=1 count=1
cat cbc1.txt cbc3.txt cbc2.txt > corrupcbc.txt
```

Decryption:

```
openssl enc -d -aes-256-cbc -nosalt -in corrupcbc.txt -out cbcout.txt -K 00112233445566778899AABBCCDDEEFF00112233445566778899AABBCCDDEEFF -iv 0102030405060708090A0B0C0D0E0F10
```

## c. Conclusion

Suprisingly, cbc and ecb decrypt only corrupt a small part of the data. In case of ecb, this is because the decryption only affect that block. And in the case of cbc, the corrupted file is located in the first part of the data, which is the last part that cbc will be decrypting, hence a small change in what happened.
