Cryptoauthlib comes with a testing application and a number of integration test suites. These tests are used to validate use cases internally and are provided to demonstrate usage. These tests are not able to completely cover every configuration combination so a "fail" is most often an indication of a configuration incompatibility with the test.

### Test Application Notes

The provided test application is interactive and allows for direct execution of many
commands

#### Safe Commands

These commands do not alter the state of the device and thus would be considered
safe to use

* info - Read the device revision register
* sernum - Read the device serial number
* lockstat - Read the device lock status (config/data, config/setup)
* rand - Generate random numbers (cryptoauth devices SHA204, ECC608, etc return a fixed pattern if the configuration is not locked)

#### Commands that write to the device
* basic - Run atcab test commands that exercise the atcab_ api
* cio - Run compressed certificate IO tests
* cd - Run compressed certificate data tests

__Do not issue lock commands until the configuration is satisfactory__