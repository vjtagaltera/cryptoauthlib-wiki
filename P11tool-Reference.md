## P11tool Reference

### Installation

To use p11tool it has to be installed:
```Shell
    # Debian like systems
    $ sudo apt-get install gnutls-bin
```
```Shell
    # RPM based systems
    $ yum install gnutls-utils
```

__Note__: If not using p11-kit-proxy then the provider has to be specified in p11tool calls:
```Shell
    $ p11tool --provider /usr/lib/libcryptoauth.so
```

### Get the public key for a private key:

* With p11-kit set up
    ```Shell
    $ p11tool --export-pubkey "pkcs11:token=0123EE;object=device;type=private"
    warning: --login was not specified and it may be required for this operation.
    warning: no --outfile was specified and the public key will be printed on screen.
    -----BEGIN PUBLIC KEY-----
    MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAE9wzUq1EUAoNrG01rXYjNd35mxKuA
    Ojw/klIrNEBciSLLOTLjs/gvFS7N8AFXDK18vpxxu6ykzF2LRd7RY8yEFw==
    -----END PUBLIC KEY-----
    ```

* Without p11-kit
    ```Shell
    $ p11tool --export-pubkey --provider /usr/lib/libcryptoauth.so "pkcs11:token=0123EE;object=device;type=private"
    warning: --login was not specified and it may be required for this operation.
    warning: no --outfile was specified and the public key will be printed on screen.
    -----BEGIN PUBLIC KEY-----
    MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAE9wzUq1EUAoNrG01rXYjNd35mxKuA
    Ojw/klIrNEBciSLLOTLjs/gvFS7N8AFXDK18vpxxu6ykzF2LRd7RY8yEFw==
    -----END PUBLIC KEY-----
    ```
