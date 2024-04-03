---
description: https://cryptohack.org/courses/intro
---

# 👀 Introduction to CryptoHack

## ASCII Encoding in Cryptography 🌐

ASCII (American Standard Code for Information Interchange) is a character encoding standard widely used in cryptography and computer systems. It represents text using integers in the range of 0-127.

### Understanding ASCII

ASCII assigns unique integers to characters, allowing for their representation in binary form. In cryptography, these integers are often manipulated for various purposes.

#### Conversion Process

To convert ASCII values to characters, Python provides the `chr()` function. Let's explore this using an example:

```python
# Given integer array
integer_array = [99, 114, 121, 112, 116, 111, 123, 65, 83, 67, 73, 73, 95, 112, 114, 49, 110, 116, 52, 98, 108, 51, 125]

# Convert integers to ASCII characters
flag = ''.join(chr(num) for num in integer_array)

# Print the obtained flag
print("Flag:", flag)
```

## Understanding Hexadecimal Encoding in Cryptography 🌐

Hexadecimal encoding plays a crucial role in cryptography when representing binary data in a more user-friendly and portable format. In this challenge, we'll explore how hexadecimal encoding is used to represent ASCII strings.

### Hexadecimal Basics

Hexadecimal, often abbreviated as "hex," is a base-16 number system. It uses the digits 0-9 and the letters A-F to represent values from 0 to 15. Hexadecimal is commonly used to represent binary data in a more compact and human-readable form.

#### ASCII to Hex Conversion

When encrypting data, the resulting ciphertext may contain bytes that are not printable ASCII characters. To share encrypted data, it's common to encode it into a more user-friendly format. One way to achieve this is by converting each ASCII character to its ordinal number and then representing these numbers in hexadecimal.

```python
# Example: Convert ASCII to Hex in Python
text = "Hello"
hex_representation = ''.join(format(ord(char), 'x') for char in text)
print("Hex Representation:", hex_representation)
```

### Challenge: Decode the Hex String

Now, let's dive into a challenge where a flag is encoded as a hex string. The task is to decode this hex string back into bytes to reveal the flag.

```
# Given hex string
hex_string = "63727970746f7b596f755f77696c6c5f62655f776f726b696e675f776974685f6865785f737472696e67735f615f6c6f747d"

# Decode hex string to bytes
decoded_bytes = bytes.fromhex(hex_string)

# Print the obtained flag
print("Flag:", decoded_bytes.decode('utf-8'))

```

## Base64 Encoding in Cryptography 🌐

Base64 is a common encoding scheme used to represent binary data as an ASCII string. It utilizes an alphabet of 64 characters (A-Z, a-z, 0-9, '+' and '/') to encode binary information. Each character in a Base64 string represents 6 binary digits (bits), and every 4 characters encode three 8-bit bytes.

### How Base64 Works

* **Character Set:** A-Z, a-z, 0-9, '+', and '/'
* **Encoding Rule:** 6 bits of binary data are represented by each character in the set.

#### Use Cases

Base64 is frequently employed online, especially when including binary data like images into HTML or CSS files. Its ASCII representation ensures compatibility across various systems and platforms.

### Challenge: Decode Hex to Bytes, Encode to Base64

Let's dive into a practical example. Take the given hex string, decode it into bytes, and then encode it into Base64.

```python
# Given hex string
hex_string = "72bca9b68fc16ac7beeb8f849dca1d8a783e8acf9679bf9269f7bf"

# Decode hex string to bytes
decoded_bytes = bytes.fromhex(hex_string)

# Encode bytes to Base64
base64_encoded = decoded_bytes.b64encode().decode('utf-8')

# Print the obtained Base64 encoding
print("Base64 Encoding:", base64_encoded)
```

### Bytes and Numbers in Cryptography 📊

When we're dealing with cryptography, we often need to perform mathematical operations on messages, which are essentially sequences of characters. However, cryptographic algorithms, such as RSA, work with numbers. So, how do we bridge this gap?

#### Bytes to Numbers

The most common way is to convert the characters in a message into numbers. This conversion starts with something called "ordinal bytes." Each character has a corresponding number, and these numbers are represented in bytes.

**Example:**

Consider the word "HELLO."

* ASCII Bytes: \[72, 69, 76, 76, 79]

To make these numbers more manageable, we often convert them into hexadecimal. Hexadecimal is a base-16 number system that uses the digits 0-9 and the letters A-F to represent values. When we concatenate these hexadecimal values, we get a large number.

* Hex Bytes: \[0x48, 0x45, 0x4c, 0x4c, 0x4f]
* Base-16: 0x48454c4c4f
* Base-10: 310400273487

#### Using Python's PyCryptodome Library

In Python, there's a library called PyCryptodome that makes these conversions easy. It provides functions like `bytes_to_long()` to convert bytes to a big integer (a very large number) and `long_to_bytes()` to convert a big integer back to bytes.

Here's a challenge to illustrate:

### Challenge: Convert a Big Integer to Message

Now, let's reverse the process. Given the big integer:

```python
given_integer = 11515195063862318899931685488813747395775516287289682636499965282714637259206269
```

We can use PyCryptodome to convert it back into a message.

```python
from Crypto.Util.number import *

# Given integer
given_integer = 11515195063862318899931685488813747395775516287289682636499965282714637259206269

# Convert integer to bytes
message_bytes = long_to_bytes(given_integer)

# Convert bytes to string
message = message_bytes.decode('utf-8')

# Print the obtained message
print("Message:", message)
```

This Python script uses `long_to_bytes()` to convert the big integer back into bytes and then decodes the bytes into a string. The result is the decoded message.

## Understanding XOR in Cryptography 🌐

XOR (Exclusive OR) is a bitwise operator widely used in cryptography. It returns 0 if the bits are the same and 1 otherwise. While textbooks often denote the XOR operator as ⊕, in most challenges and programming languages, you'll see the caret (^) used.

When XORing longer binary numbers, we XOR bit by bit. For example, `0110 ^ 1010 = 1100`. In programming, we can XOR integers by first converting them from decimal to binary. Similarly, we can XOR strings by converting each character to the integer representing the Unicode character.

<figure><img src="https://www.build-electronic-circuits.com/wp-content/uploads/2022/09/Truth-table-XOR-gate-417x500.png" alt=""><figcaption></figcaption></figure>

### Challenge: XOR and Convert

Let's apply XOR in a practical scenario. Given the string "label," XOR each character with the integer 13. Convert these integers back to a string and submit the flag as `crypto{new_string}`.

#### Manual XOR Implementation

In Python, you can implement your own XOR function:

```python
# Given string
given_string = "label"

# XOR each character with the integer 13
xored_integers = [ord(char) ^ 13 for char in given_string]

# Convert integers back to a string
result_string = ''.join(chr(x) for x in xored_integers)

# Print the flag
print("Flag: crypto{" + result_string + "}")
```

**Explanation:**

1.  **Given String:**

    ```python
    given_string = "label"
    ```

    This line initializes a string variable named `given_string` with the value "label."
2.  **XOR each character with the integer 13:**

    ```python
    xored_integers = [ord(char) ^ 13 for char in given_string]
    ```

    This line creates a list called `xored_integers` using a technique called list comprehension. It XORs (Exclusive OR) each character in the `given_string` with the integer 13. The `ord(char)` function converts each character to its Unicode integer representation.

    * For example, for the first character 'l', `ord('l')` is 108. XORing 108 with 13 gives a new integer.
3.  **Convert integers back to a string:**

    ```python
    result_string = ''.join(chr(x) for x in xored_integers)
    ```

    This line uses another list comprehension to convert each integer in `xored_integers` back to its corresponding character using the `chr(x)` function. The characters are then joined together into a single string.
4.  **Print the flag:**

    ```python
    print("Flag: crypto{" + result_string + "}")
    ```

    Finally, this line prints the obtained string as part of a flag format: "crypto{new\_string}".

So, in summary, the code takes the original string "label," performs XOR with the integer 13 on each character, converts the resulting integers back to characters, and prints the flag in a specified format. The result will be something like "crypto{new\_string}" where "new\_string" is the transformed version of "label" after XOR operations.
