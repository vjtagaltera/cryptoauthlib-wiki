## Using OpenSSL with PKCS11 



### OpenSSL Configuration without using p11-kit-proxy

OpenSSL (via libp11) supports p11-kit-proxy natively and does not require additional set up. 
If p11-kit-proxy is not being used then OpenSSL will have to be manually configured to use
libp11 and cryptoauthlib

This requires editing the default openssl.cnf file. To locate the file being used by the system run
the following command:

```Shell
    $ openssl version -a | grep OPENSSLDIR:
    
    OPENSSLDIR: "/usr/lib/ssl"
```

This gives the default path where openssl is compiled to find the openssl.cnf file

In this case the file to edit will be /usr/lib/ssl/openssl.cnf

This line must be placed at the top, before any sections are defined:
```
    openssl_conf = openssl_init
```

This should be added to the bottom of the file:
```
    [openssl_init]
    engines=engine_section

    [engine_section]
    pkcs11 = pkcs11_section

    [pkcs11_section]
    engine_id = pkcs11
    # Wherever the engine installed by libp11 is. For example it could be:
    # /usr/lib/arm-linux-gnueabihf/engines-1.1/libpkcs11.so
    dynamic_path = /usr/lib/ssl/engines/libpkcs11.so 
    MODULE_PATH = /usr/lib/libcryptoauth.so
    init = 0
```

### Create a CSR for a PKCS11 Private Key
```Shell
    $ openssl req -engine pkcs11 -key "pkcs11:token=0123EE;object=device;type=private" -keyform engine -new -out new_device.csr -subj "/CN=NEW CSR EXAMPLE"
    engine "pkcs11" set.
    
    $ cat new_device.csr
    -----BEGIN CERTIFICATE REQUEST-----
    MIHVMHwCAQAwGjEYMBYGA1UEAwwPTkVXIENTUiBFWEFNUExFMFkwEwYHKoZIzj0C
    AQYIKoZIzj0DAQcDQgAE9wzUq1EUAoNrG01rXYjNd35mxKuAOjw/klIrNEBciSLL
    OTLjs/gvFS7N8AFXDK18vpxxu6ykzF2LRd7RY8yEF6AAMAoGCCqGSM49BAMCA0kA
    MEYCIQDUPeLfPcOwtZxYJDYXPdl2UhpReVn6kK2lKCCX6byM8QIhAIfqfnggtcCi
    W21xLAzabr8A4mHyfIIQ1ofYBg8QO9jZ
    -----END CERTIFICATE REQUEST-----
```

### Verify a newly created csr
```Shell
    $ openssl req -in new_device.csr -verify -text -noout
    verify OK
    Certificate Request:
        Data:
            Version: 1 (0x0)
            Subject: CN = NEW CSR EXAMPLE
            Subject Public Key Info:
                Public Key Algorithm: id-ecPublicKey
                    Public-Key: (256 bit)
                    pub:
                        04:f7:0c:d4:ab:51:14:02:83:6b:1b:4d:6b:5d:88:
                        cd:77:7e:66:c4:ab:80:3a:3c:3f:92:52:2b:34:40:
                        5c:89:22:cb:39:32:e3:b3:f8:2f:15:2e:cd:f0:01:
                        57:0c:ad:7c:be:9c:71:bb:ac:a4:cc:5d:8b:45:de:
                        d1:63:cc:84:17
                    ASN1 OID: prime256v1
                    NIST CURVE: P-256
            Attributes:
                a0:00
        Signature Algorithm: ecdsa-with-SHA256
             30:46:02:21:00:d4:3d:e2:df:3d:c3:b0:b5:9c:58:24:36:17:
             3d:d9:76:52:1a:51:79:59:fa:90:ad:a5:28:20:97:e9:bc:8c:
             f1:02:21:00:87:ea:7e:78:20:b5:c0:a2:5b:6d:71:2c:0c:da:
             6e:bf:00:e2:61:f2:7c:82:10:d6:87:d8:06:0f:10:3b:d8:d9
```
