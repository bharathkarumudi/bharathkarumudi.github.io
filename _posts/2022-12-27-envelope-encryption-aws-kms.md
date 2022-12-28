---
title: 'Unlock the Power of Envelope Encryption for Improved Data Security'
date: 2022-12-28 11:30:00
featured_image: '/images/posts/envelope-enc-featured.jpg'  
excerpt: 'Envelope encryption is a widely used technique for protecting sensitive data as it moves between parties. By combining the speed of symmetric key encryption with the security of asymmetric key encryption, envelope encryption allows you to transfer large amounts of data quickly and securely. In this article, we delve into the details of envelope encryption and provide a Python code example for implementing it using AWS KMS.' 
--- 

![](/images/posts/envelope-enc-process.png){:height="40%" width="40%"}  

**Envelope encryption** is a method of encrypting data where the data is encrypted using a symmetric key, and the symmetric key is then encrypted using an asymmetric key. The encrypted data and encrypted symmetric key are referred to as the "envelope." 

The advantage of this approach is that the asymmetric key, which is typically used to encrypt the symmetric key, is much slower to use for encrypting and decrypting large amounts of data compared to the symmetric key. This means that the data can be quickly encrypted and decrypted using the symmetric key, while the slower asymmetric key is used only to secure the symmetric key. This makes envelope encryption particularly useful for protecting sensitive data, as it allows for fast and efficient encryption and decryption of large amounts of data without sacrificing security.

Envelope encryption is often used to secure sensitive data in cloud environments, where the data may need to be accessed by multiple parties but the security of the data key cannot be guaranteed. It provides an additional layer of security by ensuring that only authorized parties can decrypt the data, even if the data key itself is compromised. 

#### Envelope Encryption Process  

##### Sequence Diagram for Envelope Encryption
```  
         +--------+                  +--------+
         |        |                  |        |
         |  User  |                  |  KMS   |
         |        |                  |        |
         +--------+                  +--------+
              |                            |
(Encryption)  |  Generate data key (DEK)   |
              |---------------------------->
              |                            |
              |  DEK (plaintext and        |
              |  ciphertext)               |
              |<----------------------------
              |                            |
              |  Encrypt data with DEK     |
              |---------------------------->
              |                            |
              |  Encrypted data and        |
              |  ciphertext DEK            |
              |<----------------------------
              |                            |
              |  Store encrypted data and  |
              |  ciphertext DEK            |
              |---------------------------->
              |                            |
              |  Discard plaintext DEK     |
              |<----------------------------
              |                            |
 (Decryption) |  Retrieve ciphertext DEK   |
              |  and encrypted data        |
              |---------------------------->
              |                            |
              |  Obtain plaintext DEK by   |
              |  decrypting ciphertext DEK |
              |---------------------------->
              |                            |
              |  Plaintext DEK             |
              |<----------------------------
              |                            |
              |  Decrypt data with         |
              |  plaintext DEK             |
              |---------------------------->
              |                            |
              |  Decrypted data            |
              |<----------------------------
``` 

##### Encryption
1. Create a Customer Managed, Customer Master Key (CMK) in KMS.  
2. Using the CMK, generate a data key (DEK).  
3. KMS returns the DEK in both plaintext and ciphertext (which is encrypted with CMK).     
4. Using the plaintext data key, encrypt the data.  
5. Store the ciphertext data key and encrypted data in a persistent storage.  
6. Discard the plaintext data key.  

##### Decryption 
1. Retrieve the ciphertext data key and encrypted files from the persistent storage service.  
2. Obtain the plaintext data key by decrypting the ciphertext data key using the CMK.  
3. Use the plaintext data key to decrypt the files.

#### Advantages of Envelope Encryption    

1.  **Improved security:** Envelope encryption provides an additional layer of security by encrypting the data using symmetric key and then encrypting the symmetric key using an asymmetric key. This means that even if the symmetric key is compromised, the data is still protected by the asymmetric key.  
2.  **Efficiency:** The use of a symmetric key for encrypting and decrypting the data allows for fast and efficient encryption and decryption of large amounts of data.  
3.  **Flexibility:** Envelope encryption allows you to choose different keys for encrypting and decrypting the data, depending on your security needs. For example, you can use a stronger key for encrypting the data and a weaker key for decrypting it.  
4.  **Key management:** Envelope encryption allows you to separate the management of the symmetric key from the management of the data. This can make it easier to manage the keys and rotate them as needed.  
5.  **Compatibility:** Envelope encryption is compatible with a wide range of encryption algorithms and protocols, making it a flexible and versatile option for protecting data.  

### Envelope Encryption in Python using AWS KMS Service  

```python 
'''
Usage: Python code to demonstrate the implementation on Envelope Encryption using AWS KMS
Before running this code, configure AWS Credentials in your environment:

$ export AWS_ACCESS_KEY_ID=<Your Access KEY ID> 
$ export AWS_SECRET_ACCESS_KEY=<Your Secret Access Key> 

Written on Dec/28/2022
Permalink: https://gist.github.com/bharathkarumudi/6a6b8836c827d846167381d3ba42974d 
'''

# Importing required libraries  
import boto3 
from Cryptodome.Random import get_random_bytes
from Cryptodome.Cipher import AES 
from Cryptodome.Util.Padding import pad, unpad
import base64 

# Setting the region  
region = 'us-east-1'

# Create the KMS client
kms_client = boto3.client('kms', region_name=region)


def encrypt_data(plaintext, key_id):
    '''
    Method to encrypt the data using envelope encryption 
    The method returns encrypted data, encrypted data key, and also respective base64 encoded values. 
    '''  
    # Generate a data key using AWS CMK
    data_key = kms_client.generate_data_key(KeyId=key_id, KeySpec='AES_256')
    plaintext_data_key = data_key['Plaintext']
    encrypted_data_key = data_key['CiphertextBlob']

    # Generate a random IV
    iv = get_random_bytes(16)

    # Encrypt the data with AES CBC mode  
    cipher = AES.new(plaintext_data_key, AES.MODE_CBC, iv)
    padded_data = pad(plaintext, 16)
    ciphertext_blob = cipher.encrypt(padded_data)

    # Encode the encrypted data and data key  with base64  
    encoded_ciphertext_blob = base64.b64encode(ciphertext_blob) 
    encoded_encrypted_data_key = base64.b64encode(encrypted_data_key)

    # Return the encrypted data and the encrypted data key
    return ciphertext_blob, encoded_ciphertext_blob, encrypted_data_key, encoded_encrypted_data_key, iv

def decrypt_data(encoded_ciphertext_blob, encoded_encrypted_data_key, iv):
    '''
    Method to decrypt the data 
    The method returns the decrypted data  
    '''  
    # Base64 decode of encrypted data and encrypted data key 
    decoded_ciphertext_blob = base64.b64decode(encoded_ciphertext_blob)
    decoded_encrypted_data_key = base64.b64decode(encoded_encrypted_data_key)

    # Decrypt the data key
    data_key = kms_client.decrypt(CiphertextBlob=decoded_encrypted_data_key)
    plaintext_data_key = data_key['Plaintext']

    # Decrypt the data
    cipher = AES.new(plaintext_data_key, AES.MODE_CBC, iv)
    decrypted_padded_data = cipher.decrypt(decoded_ciphertext_blob)
    plaintext = unpad(decrypted_padded_data, 16)

    # Return the decrypted data
    return plaintext

def main(): 
    ''' 
    Main method  
    '''
    #AWS CMK Key ARN 
    key_id = "<Your Key ARN here>" 

    # Setting the plain text, which needs to be encrypted  
    input_plaintext = b"This is a confidential message which needs to be encrypted." 

    # Encrypting the data 
    ciphertext_blob, encoded_ciphertext_blob, encrypted_data_key, encoded_encrypted_data_key, iv = encrypt_data(input_plaintext, key_id)
    
    # Decrypt the data
    plaintext = decrypt_data(encoded_ciphertext_blob, encoded_encrypted_data_key, iv)

    # Store the encrypted string and the encrypted data key in a file (optional)
    with open('datastore.csv', 'w') as f:
        f.writelines([str(encoded_ciphertext_blob) + ',', str(encoded_encrypted_data_key) + ',', str(iv)])

if __name__ == "__main__":
    main()

```

Envelope encryption is a valuable tool for securing sensitive data in a variety of contexts, offering improved security, efficiency, and key management. By combining the speed of symmetric key encryption with the security of asymmetric key encryption, envelope encryption allows you to transfer large amounts of data quickly and securely.  
  




   