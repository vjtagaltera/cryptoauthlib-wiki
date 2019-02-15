Using the openssl command line tool, here's how you can generate keys, sign, and verify using openssl and verify with the ATECCx08A:

## 1. Create a new key pair
```
openssl ecparam -genkey -out key.pem -name prime256v1
```

## 2. Print out the private and public portions of the key:
```
openssl ec -in key.pem -noout -text
```
This will generate output that looks like the following:
```
read EC key
Private-Key: (256 bit)
priv:
    00:f0:ff:30:ba:ed:4e:e6:5a:3f:3e:8b:be:eb:07:
    e9:a1:1a:65:56:a8:5c:05:9b:af:7d:ce:d9:a5:67:
    2f:2a:29
pub: 
    04:9a:4c:1f:02:7c:7a:57:a8:66:73:ad:62:42:a5:
    3d:c9:ae:0f:f5:93:ff:6c:f0:3d:9e:67:0d:21:b0:
    23:2e:2e:9a:20:37:0b:bc:a8:72:66:15:48:73:b1:
    f0:02:f0:05:f0:a5:bf:f8:67:8e:8a:e0:64:12:4c:
    2e:c8:7c:99:7e
ASN1 OID: prime256v1
NIST CURVE: P-256
```

## 3. Extract the require key components from the output.

The private key is a 32-byte big-endian signed integer found in the priv section. Sometimes the priv section will have 33 bytes, this is because this integer is being expressed in a 2s-complement signed form and requires an extra padding byte to prevent it from being interpreted as negative. In that case, you can disregard the first padding byte (0x00) if you want just the raw 32-byte unsigned integer. For this example, the private doesn’t need to be extracted for any operations with the ATECCx08A, this is just here for completeness.

The public key is in the pub section and is composed of three components. The first is a key compression byte, which should always be 0x04, indicating an uncompressed public key. The second and third components are the public key x and y integers. Both are 32-byte big-endian unsigned integers. Note that because these integers are unsigned, no padding bytes need to be accounted for here like with the private key.
 
When the public keys are passed to the ATECCx08A for the verify command, they are just the x and y integers with no key compression byte. For the example key above, the raw public key would be:
```
uint8_t public_key[64] = {
    0x9a, 0x4c, 0x1f, 0x02, 0x7c, 0x7a, 0x57, 0xa8, 0x66, 0x73, 0xad, 0x62, 0x42, 0xa5, 0x3d, 0xc9,
    0xae, 0x0f, 0xf5, 0x93, 0xff, 0x6c, 0xf0, 0x3d, 0x9e, 0x67, 0x0d, 0x21, 0xb0, 0x23, 0x2e, 0x2e,
    0x9a, 0x20, 0x37, 0x0b, 0xbc, 0xa8, 0x72, 0x66, 0x15, 0x48, 0x73, 0xb1, 0xf0, 0x02, 0xf0, 0x05,
    0xf0, 0xa5, 0xbf, 0xf8, 0x67, 0x8e, 0x8a, 0xe0, 0x64, 0x12, 0x4c, 0x2e, 0xc8, 0x7c, 0x99, 0x7e
};
```

## 4. Create a file with the data you want to sign called message.txt. This example uses the following sample text.
```
The next day, Thursday, August 27, is a well-remembered date in our subterranean journey. It never returns to my memory without sending through me a shudder of horror and a palpitation of the heart. From that hour we had no further occasion for the exercise of reason, or judgment, or skill, or contrivance. We were henceforth to be hurled along, the playthings of the fierce elements of the deep.
```

## 5. The raw P-256 ECDSA sign and verify operations requires a fixed message size of 32-bytes. Since most messages will be arbitrarily sized, we use the SHA256 hash to create a message digest of 32 bytes. This command also signs the message with the private key.
```
openssl dgst -sha256 -sign key.pem -out signature.der message.txt
```

## 6. To verify the message again with openssl, we first need just the public key extracted from the key.pem file:
```
openssl ec -in key.pem -pubout -out pub-key.pem
```

## 7. Now we verify the signature using openssl:
```
openssl dgst -sha256 -verify pub-key.pem -signature signature.der message.txt
Verified OK
```

## 8. To verify using the ATECCx08A, we first need the message digest:
```
openssl dgst -sha256 -out message-digest.txt message.txt
```
In the message-digest.txt file, you’ll find the 32-byte digest in hex:
```
SHA256(message.txt)= 6c3e2cad0722aad472e1244a4237f55e7bb34e3235370cfc0b9acb648c1df24e
```
```
uint8_t digest = {
    0x6C, 0x3E, 0x2C, 0xAD, 0x07, 0x22, 0xAA, 0xD4, 0x72, 0xE1, 0x24, 0x4A, 0x42, 0x37, 0xF5, 0x5E,
    0x7B, 0xB3, 0x4E, 0x32, 0x35, 0x37, 0x0C, 0xFC, 0x0B, 0x9A, 0xCB, 0x64, 0x8C, 0x1D, 0xF2, 0x4E
};
```

## 9. We also need the raw signature R and S integers. We can use the openssl ASN.1 parser to pull them out of the signature der file:
```
openssl asn1parse -inform DER -in signature.der
    0:d=0  hl=2 l=  67 cons: SEQUENCE          
    2:d=1  hl=2 l=  31 prim: INTEGER           :65CBF987E61747C91BA02B622F99775D17C09E42B80854C9275BA9B642AACF
   35:d=1  hl=2 l=  32 prim: INTEGER           :7775DB68D0CEEBDC6429540CF1759DCE3B1CA849E97CC12B85A276ACEEB8C0F3
```

The first integer is R, the second is S. We want these values as raw 32-byte unsigned integers. If they’re longer than 32 bytes, they are padded in the DER encoding and you want to remove the padding (0x00) byte at the beginning. However, openssl's asn1parse output seems to strip that padding byte automatically. If they’re shorter than 32 bytes, add padding bytes (0x00) at the beginning to make it 32 bytes long (which is the case with R in the above example).
 
When signatures are passed to the ATECCx08A for the verify command, they are just the R and S integers. For the example signature above, the raw signature would be:
```
uint8_t signature[64] = {
    0x00, 0x65, 0xCB, 0xF9, 0x87, 0xE6, 0x17, 0x47, 0xC9, 0x1B, 0xA0, 0x2B, 0x62, 0x2F, 0x99, 0x77,
    0x5D, 0x17, 0xC0, 0x9E, 0x42, 0xB8, 0x08, 0x54, 0xC9, 0x27, 0x5B, 0xA9, 0xB6, 0x42, 0xAA, 0xCF,
    0x77, 0x75, 0xDB, 0x68, 0xD0, 0xCE, 0xEB, 0xDC, 0x64, 0x29, 0x54, 0x0C, 0xF1, 0x75, 0x9D, 0xCE,
    0x3B, 0x1C, 0xA8, 0x49, 0xE9, 0x7C, 0xC1, 0x2B, 0x85, 0xA2, 0x76, 0xAC, 0xEE, 0xB8, 0xC0, 0xF3
};
```

Note that elliptic curve signatures typically have a random component, so every signature will be different.

## 10. Now we have all the pieces needed to verify the signature on the ATECCx08A. This can be done with the atcab_verify_extern() function.
```
bool is_verified = false;
status = atcab_verify_extern(digest, signature, public_key, &is_verified);
if (status == ATCA_SUCCESS)
{
    if (is_verified)
        printf("ECDSA Verify OK");
    else
        printf("ECDSA Verify FAILED");
}
else
{
    printf("atcab_verify_extern() failed");
}
```

Note that the verify command in external mode is available at any time regardless of config or data lock status. This function doesn’t require any stored data on the ATECCx08A and just acts as a crypto accelerator.