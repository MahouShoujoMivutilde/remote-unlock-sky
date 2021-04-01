<!-- This seems to be the place when people discuss different configurations with systemd-tool, what they did to make it work the way they wanted, so i am writing this in case someone will have issues similar to what just i had. -->

If

1. `/home` is on separate encrypted partition
2. `/etc/cryptsetup` doesn't have a mapper named **exactly** `home` (**regardless of if there something else that mounts later on** `/home`, e.g. `crypt-home`)

**TLDR**: if you asked for `/home` password when you know it will be unlocked later with key, rename mapper for `/home` partition exactly to `home`, and it wont ask you anymore, and everything will be unlocked and mounted fine.

<details>

  <summary>Long story and the issue description</summary>

If 1 and 2 are met, the system will boot fine, unlock with your password, but then will ask you for `/home` password (even if you only have key-file for it setup, which actually is on root which you just unlocked).

It will hung there until you ether enter it, or just fail it by pressing <kbd>enter</kbd> few times. Then it will boot, with `/home` mounted just fine (**even if you didn't enter password!**).

For example.

This will work:

`/etc/fstab`
```sh
# ...

/dev/mapper/home /home     	ext4      	rw,relatime	0 2

# ...
```

`/etc/crypttab`
```sh
home UUID=<uuid of encrypted partition> /etc/keys/home.key
```

This will **not** (at least for me):


`/etc/fstab`
```sh
# ...

/dev/mapper/crypt-home /home     	ext4      	rw,relatime	0 2

# ...
```

`/etc/crypttab`
```sh
crypt-home UUID=<uuid of encrypted partition> /etc/keys/home.key
```

Why?

Because (when above conditions are met) `systemd-cryptsetup-generator` generates `systemd-cryptsetup@home.service` even if there is already another service that unlocks `/home` partition.

What i also noticed is that:

1. it tries to mount it not with UUID, but with /ded/sdXY notation, even when crypttab is in UUID notation
2. it doesn't care if encrypted partition even HAS a password
3. you can view `systemd-cryptsetup@home.service` only in emergency shell
4. if you mask it - system fails to boot, since `local-fs.target` also then fails

I haven't tried without systemd-tool, but it this point i am too tired T_T

</details>
