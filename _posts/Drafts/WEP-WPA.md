---
layout: post
title: "WEP"
date: 2022-04-12 20:50:57 +0200
categories: Writeup
tags: Wireless WEP WPA WPA2
image: "assets/images/binary.jpg"
excerpt: "Lorem ipsum dolor sit amet, consectetur adipiscing elit. Cras nulla nisi, gravida eget lacus sed, feugiat rhoncus lectus. Maecenas condimentum rutrum dolor, ut ultrices risus tempor vel. Mauris sed iaculis elit, id efficitur nulla. Morbi vitae purus et eros venenatis hendrerit quis non nibh. Suspendisse est turpis, ultricies et ipsum et, semper tincidunt ex. Phasellus accumsan enim nec arcu mollis ultricies. Suspendisse congue mi diam, ut auctor turpis faucibus ut."
---

## WEP

[Wired Equivalent Privacy](https://en.wikipedia.org/wiki/Wired_Equivalent_Privacy) is a now obsolete security algorithm for 802.11 wireless networks.
**WEP** was introduced as a Wi-Fi security standard in 1999 and had been superseded by **WPA** in 2003, due to its poorly implemented encryption algorithm.

### 64-bit WEP

### Key

Initially, 64-bit WEP used a **40-bit key** (network password), later, 

that is concatenated to the **[Initialization Vector (IV)](https://en.wikipedia.org/wiki/Initialization_vector)**.

The **40-bit key** is entered as a string of 10 [hexadecimal](https://en.wikipedia.org/wiki/Hexadecimal) characters (0-9 & A-F):<br>
Each character represents 4 bits (10 x 4 bits = 40-bit key).

#### Initialization Vector

The **Initialization Vector** is a 24-bit random generated number that should be unique in every packet.<br>
The range of unsigned integers that can be represented in 24 bits is 0 to 16,777,215.

#### Key Stream

The **Initialization Vector** is concatenated to the **key**, to form the **key stream**.

#### Encryption

The encryption mechanism used in WEP is a symmetric cipher; this means that the key that encrypts the data is the same key that will decrypt the data.

The **key stream** is used to encrypt the data that is bing sent through the network.

After the encryption process, the **IV** is then included in the packet in plain text (40 bits + 24 bits = 64 bits) so that the router can decrypt the message.

This allows an attacker to collect two ciphertexts that are encrypted with the same key stream and perform statistical attacks to recover the plaintext.

### 128-bit WEP

128-bit WEP uses a 104-bit key, that is concatenated to the Initialization Vector (IV).

The [IV](https://en.wikipedia.org/wiki/Initialization_vector) is a 24-bit random generated number that should be unique in every packet.

The 128-bit key is entered as a string of 26 [hexadecimal](https://en.wikipedia.org/wiki/Hexadecimal) characters (0-9 & A-F):<br>
Each character represents 4 bits (26 x 4 bits = 104 bits).
The 24-bit IV is then included in the packet in plain text (104 bits + 24 bits = 128 bits).<br>

### Cracking WEP

- WEP generates a random 24-bit AV
- IV is then concatenated to the password of the network to form the key stream
- The key stream is used to encrypt the data (RC4)
- WEP appends the IV to the packet in plain text

## WPA

[Wi-Fi Protected Access](https://en.wikipedia.org/wiki/Wi-Fi_Protected_Access)

## WPA2

## WPA3