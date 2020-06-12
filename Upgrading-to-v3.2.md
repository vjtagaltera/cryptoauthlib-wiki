Upgrading to v3.2.x from a previous version of cryptoauthlib will take some effort and this page will document the items that will need to evaluated and potentially adjusted in the application. The biggest change is the HAL function signatures and implementations requiring a modification to adjust behavior to match the expected behavior of the interface in v3.2 

## API changes between v3.1.x and v3.2.x
-----------------------------------------------

### HAL changes

The signature for the send and receive functions have changed to incorporate an
additional parameter (word_address)

Old Signatures:
```
ATCA_STATUS (*halsend)(ATCAIface iface, uint8_t *txdata, int txlength);
ATCA_STATUS (*halreceive)(ATCAIface iface, uint8_t* rxdata, uint16_t* rxlength);
```

New Signatures:
```
ATCA_STATUS (*halsend)(ATCAIface iface, uint8_t word_address, uint8_t *txdata, int txlength);
ATCA_STATUS (*halreceive)(ATCAIface iface, uint8_t word_address, uint8_t* rxdata, uint16_t* rxlength);
```

This will necessitate a change to a ported HAL to accommodate the above. Internally those
functions will need to also be changed to support the purpose of this field. The word_address
is written to a device before data is written or read from it. Previously a fixed value was
used by the HAL implementations. Now the the HAL must accept this parameter and transmit it
according to the datasheet. Some HAL implementations may benefit from this change as some
I2C hardware have the option to read from a device register addresses.

See `hal_i2c_harmony.c` for more details


#### atcab (basic)

This API remains largely the same between versions however a few important
elements exist:

1) If atca_config.h only contains support for a classic cryptoauth device such
   as the ATECC608A then atcab_* functions actually reference a macro that redirects
   to the equivalent calib_* function with the first parameter being the global
   device context.
2) Similarly to the above if only support for a TA device is selected then
   atcab_* functions are redirected to compatible talib_* functions using the global
   device context
3) If both classic CryptoAuth and TA devices are selected then atcab_* functions
   are abstraction functions that will determine the proper calib_* or talib_* api
   to call based on the global device context

#### calib (basic)

This API is the actual implementation of cryptoauth commands that formerly were
defined as the atcab_ API.

These functions no longer act upon a single global device context but the one provided
as an input parameter.

This means that it is now possible to maintain application specific cryptoauth contexts
for multiple devices and busses.

#### talib (basic)

This is a new API that is the actual implementation for Trust Anchor (TA) commands.
Similarly to the calib_ API these functions work with a provided device context.

#### atcac (software/host crypto)

This API has been expanded to support additional cryptographic functions for
* HMAC
* AES-CMAC (requires an external crypto library)
* AES-GCM (requires an external crypto library)

Additionally if the build is configured to include or link to an external
cryptographic library (MbedTLS, OpenSSL, WolfSSL) the cryptoauthlib software
implementations for SHA1 & SHA2 are replaced by the version in the external
library. This will obviously reduce overall code size but moreover will mean
that cryptoauthlib can be configured to use a qualified library for host side
cryptographic operations.

Limitations:
* Cryptoauthlib does not have an AES software implementation so any feature
  that requires AES (CMAC, GCM, etc) will require the use of an external library