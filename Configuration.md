Cryptoauthlib uses CMake for configuration for Windows, Linux, and MacOS. The
CMake configuration files for cryptoauthlib do not currently natively support cross-compilation.

There are many options available that can be provided to CMake through the command line
or GUI interface.

For example to configure:
```
cd cryptoauthlib
mkdir build
cd build
cmake -DATCA_ATECC608A_SUPPORT=ON -DATCA_TFLEX_SUPPORT=on -DATCA_OPENSSL=ON -DATCA_HAL_KIT_HID=ON ../
```

To build:
```
cd cryptoauthlib/build
cmake --build .
```

### CMake Configuration Options

#### Global Options
* BUILD_TESTS (default off) - Create the Test Application
* ATCA_PRINTF (default off) - Enable Debug print statements in library
* ATCA_BUILD_SHARED_LIBS (default on) - Builds cryptoauthlib as a shared library (.dll/.so)

#### HAL Options
* ATCA_HAL_KIT_HID (default off) - Include the HID HAL Driver
* ATCA_HAL_I2C (default off) - Include the I2C Hal Driver - Linux & MCU only
* ATCA_HAL_SPI (default off) - Include the SPI HAL Driver - Linux & MCU only
* ATCA_HAL_CUSTOM (default off) - Support for Custom/Plug-in Hal Driver

#### External Crypto Library
* ATCA_MBEDTLS (default off) - Integrate with mbedtls
* ATCA_WOLFSSL (default off) - Integrate with WolfSSL
* ATCA_OPENSSL (default off) - Integration with OpenSSL

#### PKCS11 Options
* ATCA_PKCS11 (default off) - Build the library as a PKCS#11 Provider

#### Trust & Go Options 
* ATCA_TNGTLS_SUPPORT (default off) - Include Trust & Go TLS Certificates
* ATCA_TNGLORA_SUPPORT (default off) - Include Trust & Go LORA Certificates
* ATCA_TFLEX_SUPPORT (default off) - Include Trust Flex Certificates
* ATCA_TNG_LEGACY_SUPPORT (default off) - Include previous version of Trust & Go Certificates

#### Device Selection Options 
* ATCA_ATSHA204A_SUPPORT (default on) - Include support for ATSHA204A device
* ATCA_ATECC508A_SUPPORT (default on) - Include support for ATECC508A device
* ATCA_ATECC608A_SUPPORT (default on) - Include support for ATECC608A device
* ATCA_TA100_SUPPORT (default off) - Include support for TA100 device

#### TA Specific Options
* ATCA_TA100_AES_AUTH_SUPPORT - Include Encrypted (GCM) and CMAC Auth session support