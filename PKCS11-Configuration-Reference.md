When building the PKCS11 library several configuration options are available in [pkcs11_config.h](https://github.com/MicrochipTech/cryptoauthlib/blob/master/lib/pkcs11/pkcs11_config.h.in)

- **PKCS11_DEBUG_ENABLE** (Default: disabled)

   Enabling debug statements to be logged (on most platforms this will be stderr)

- **PKCS11_USE_STATIC_MEMORY** (Default: enabled)

   If set uses static library context and object stores.

- **PKCS11_USE_STATIC_CONFIG** (Default: disabled)

   If set uses a static configuration defined in pks11_config.c where all objects are defined in code rather than
configuration files. Enable for embedded applications without filesystems.

- **PKCS11_MAX_SLOTS_ALLOWED** (Default: 1)

   The number of PKCS11 slots (which are different than device slots) the library will support. Currently cryptoauthlib
(via atcab_init) only supports one device context at a time.

- **PKCS11_MAX_SESSIONS_ALLOWED** (Default: 10)

   The maximum number of PKCS11 sessions allowed. Memory will be allocated upon library init for these sessions (especially if PKCS11_USE_STATIC_MEMORY is set so this should be configured to account for the use case. Embedded applications should reduce this value to the minimum required by the application.

- **PKCS11_MAX_OBJECTS_ALLOWED** (Default: 16)

   The maximum number of PKCS11 objects to be supported by the library.

- **PKCS11_MAX_LABEL_SIZE** (Default: 10)

   The maximum number of characters allowed in an object label. Suitable for embedded applications.

- **PKCS11_EXTERNAL_FUNCTION_LIST** (Default: disabled)

   An optimization for embedded applications to allow for externally defined functions lists. This will allow compilers to prune pkcs11 APIs that are unused by the embedded application. 

- **PKCS11_SEARCH_CACHE_SIZE** (Default: 128)

   This configures the memory allocated for caching the attribute search structure that is provided to the library when an application is performing an object find api call.

- **PKCS11_508_SUPPORT** (Default: enabled)

   Compile in ATECC508A support into pkcs11 initialization. If both 508A & 608A support is enabled the library will attempt to detect which device is attached and reinitialize cryptoauthlib properly for the detected device. If only one device is expected to be supported then only the specific define needs to be enabled and will therefore disable this detection logic.

- **PKCS11_608_SUPPORT** (Default: enabled)

   Compile in ATECC608A support into pkcs11 initialization. If both 508A & 608A support is enabled the library will attempt to detect which device is attached and reinitialize cryptoauthlib properly for the detected device. If only one device is expected to be supported then only the specific define needs to be enabled and will therefore disable this detection logic.
