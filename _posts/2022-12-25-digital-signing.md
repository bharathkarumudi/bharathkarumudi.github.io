---
title: 'Secure Your Documents with Digital Signatures: A Quick Guide'
date: 2022-12-25 15:40:00
featured_image: '/images/posts/digitalSignature.jpg'
excerpt: Learn the basics of digital signatures and how they can be used to secure and verify the authenticity of your documents. This comprehensive guide covers the different use cases for digital signatures, the process of signing and verifying documents digitally, and tips on using the gpg utility to create and verify digital signatures.  
---

![](/images/posts/digitalSignature.jpg){:height="40%" width="40%"}  

A digital signature is a way to verify the authenticity and integrity of a document. It works by attaching a unique code to the document that can be used to verify that the document has not been altered since the signature was applied. This is similar to a handwritten signature, but it has the added advantage of being more secure and difficult to forge. 

#### Use Cases for digital signatures  

- Legal documents: Digital signatures can be used to sign legal documents such as contracts, wills, and deeds. This allows for the documents to be signed and legally binding without the need for physical signatures.  

- Business documents: Digital signatures can be used to sign business documents such as invoices, purchase orders, and employee records. This can improve efficiency and reduce the time and cost associated with physically signing and storing documents.  

- Government documents: Digital signatures are often used by governments to sign and verify important documents such as tax returns, voter registration forms, and passport applications.  

- Educational documents: Digital signatures can be used to sign and verify educational documents such as transcripts, diplomas, and certificates.  

- Healthcare documents: Digital signatures can be used to sign and verify healthcare documents such as consent forms, medical records, and insurance claims.  

### Signing Process   
1. The sender creates a document and a private key.
2. The sender uses the private key to create a digital signature for the document.
3. The sender sends the document and the digital signature to the recipient.
4. The recipient uses the sender's public key to verify the digital signature.
5. If the digital signature is valid, the recipient can trust that the document has not been tampered with and is authentic. 

We can create digital signatures using the `gpg` utilit, and the below examples shows different methods on signing and verifying a `contract.doc` document using `gpg` CLI.  

#### Method 1 - Undetached Signature
This process signs the document `contract.doc` with the senders private key and creates a compressed signed document `contract.doc.sig`.  

```bash  
sender@hostA$ gpg --output contract.doc.sig --sign contract.doc  
```  

The receiver of the signed document `contract.doc.sig` can either check the signature or check the signature and recover the original document. 

```bash 
#Verifying only digital Signature   
receiver@hostB$ gpg --verify contract.doc.sig 
gpg: Signature made Sun 25 Dec 2022 02:25:10 PM EST
gpg:                using RSA key 3A5C5E35CD117C059C874AAFA3B92C14CC147AAA
gpg: Good signature from "test@example.com <test@example.com>" [ultimate]

#Verifying the digital signature and extracting the original document  
receiver@hostB$ gpg --output contract.doc --decrypt contract.doc.sig 
gpg: Signature made Sun 25 Dec 2022 02:25:10 PM EST
gpg:                using RSA key 3A5C5E35CD117C059C874AAFA3B92C14CC147AAA
gpg: Good signature from "test@example.com <test@example.com>" [ultimate] 

receiver@hostB$ ls contract.doc contract.doc.sig 
contract.doc  contract.doc.sig
```  

#### Method 2 - Detached Signature  
This process signs the document `contract.doc` with the senders private key, and creates a separate signature file `contract.doc.sig`, and you need to send both signature file and original file to the receiver. 

```bash 
sender@hostA$ gpg --output contract.doc.sig --detach-sig contract.doc 
``` 
The receiver of the signed document `contract.doc.sig` and `contract.doc` will verify the signature and if it is valid, then it proves the `contract.doc` is unaltered and signed by the sender.   


```bash 
#verifying the signature  
receiver@hostB$ gpg --verify contract.doc.sig contract.doc 
gpg: Signature made Sun 25 Dec 2022 03:01:42 PM EST
gpg:                using RSA key 3A5C5E35CD117C059C874AAFA3B92C14CC147AAA
gpg: Good signature from "test@example.com <test@example.com>" [ultimate]  
```  

#### Method 3 - ClearSigned documents  
This method involves creating a signed message that is combined with the original document in an ASCII-armored signature, where it is not desirable to compress the document during the signing process. This creates a single file that contains both the signature and the content. A common application of this method is to sign and send emails.  

```bash 
sender@hostA$ gpg --clearsign contract.doc

#Signature file  
sender@hostA$ ls contract*
contract.doc		contract.doc.asc

#View the signed message from the signed document.  
sender@hostA$ cat contract.doc.asc
-----BEGIN PGP SIGNED MESSAGE-----
Hash: SHA256

Dear John Doe,

I am pleased to inform you that the contract for Firewall implementation has been approved by Contoso. As a result, we will be proceeding with the project as outlined in the contract.

Best Regards,
Johnny

-----BEGIN PGP SIGNATURE-----

iQIzBAEBCAAdFiEEOlxeNc0RfAWch0qvo7ksFMwUf2oFAmOorjoACgkQo7ksFMwU
f2rRshAAjfD9vxM1qBvQ1riOnJ6FIgKZfX4eW1eVYweT36PHGxjCkKBIAPD810f5
3yyc+4PJEyu0C8bKXUt4eHcwaaxe3//dyZO+G4S9KgG/sLikyAay+DDE9QWwOWzp
bKUVXRD4QAP9/K1lk0WCrvQmTdzxivVPruk1XHy0AVPFiIRP+9k6LSpKheO8Q6lP
Vs5CMvIcgFNmSMpxXazK3saRx/DK4me30NLt0MC85S6DazwRyyH1/w673WVrQGB1
I4QfPgjYqCdVJhDo9CDKG7gBNV6xMKizbnDsJi1/XbFzqb4CrDeYGiSfGaENX35W
wvCEh8qJ587OjO+AvKCkvDU7MprSil0HIBaJr1/yMLrxUF9/jTN2Di6CarEZVwfe
sX7DG3qno/bHEuc7rRDEEjzzlbam35zcUNqpYD0TKT675nlnglH5X7iyO/mLDUbY
udcmCDXbrt+PEfIKYwcAS4DVkwzm3S7O/Gi0WYlto26zRzrywtdXEJ6ySLIYgji0
U8i5sAiHqyUEN7Tlg2Lb4SYG0Obfm69XSqvDpkqP72zHwlDynWGIJBhKVc0bgE+q
yVp+BfW6y14LriDTpwmYe2fdKRErFidqMnOIHauJN5UvZO6s8X966dI/QRd+5nMx
niRHH9Rztge4lkAgxcYXuZaBWVhIURLMC/iudR9jypHIfQcttw8=
=Fw26
-----END PGP SIGNATURE-----
```   

The receiver of `contract.doc.asc` will verify the signature, and reads the message inside the signed file.  

```bash 
receiver@hostB$ gpg --verify contract.doc.asc  
gpg: Signature made Sun 25 Dec 2022 03:01:42 PM EST
gpg:                using RSA key 3A5C5E35CD117C059C874AAFA3B92C14CC147AAA
gpg: Good signature from "test@example.com <test@example.com>" [ultimate]  
```   

Overall, digital signatures are useful for any situation where it is important to confirm the authenticity and integrity of a document.  
