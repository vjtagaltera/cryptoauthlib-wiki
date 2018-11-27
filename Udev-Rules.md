## Using HID Kits with Linux

By default when using the Cryptoauthentication kits that communicate via USB-HID they will be 
unavailable to cryptoauthlib when not run with sudo. To resolve this it is recommended to update
the udev rules by adding the [90-cryptohid.rules](https://github.com/MicrochipTech/cryptoauthlib/blob/master/lib/hal/90-cryptohid.rules)
file to the `/etc/udev/rules.d` folder

```
$ sudo cp cryptoauthlib/lib/hal/90-cryptohid.rules /etc/udev/rules.d
```

