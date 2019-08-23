# Encrypting Disks

- [Encrypting Disks](#encrypting-disks)
  - [Why Use Encryption?](#why-use-encryption)
  - [LUKS](#luks)
    - [cryptsetup](#cryptsetup)
  - [Using an Encrypted Partition](#using-an-encrypted-partition)
  - [Mounting at boot](#mounting-at-boot)

## Why Use Encryption?

Encryption should be used wherever there's sensitive data. Modern linux distros offers the choice of encryption all or some of the disks partitions during installation. 

## LUKS

Modern Linux distros provice block device level encryption mainly through the use of LUKS (Linux Unified Key Setup). Using block device encryption is highly recommended for portable systems such as laptops, tablet and smart phone. It's installed on top of cryptsetup, a powerful utility that can also use other methods such as plain dm-crypt volumes, loop-AES and TrueCrypt-comptaible format.

### cryptsetup

Everything is done with the swiss-army knife program cryptsetup. Once encrypted volumes have been set up, they can be mounted and unmounted with normal disk utilities.

## Using an Encrypted Partition

```bash
# First we need to give the partition to LUKS
sudo cryptsetup luksFormat /dev/sda1

# Examine /proc/crypto to see the methods your system supports and then supply one such as
sudo cryptsetup luksFormat --cipher aes /dev/sdc12

# make the volume avail by doing
sudo cryptsetup --verbose luksOpen /dev/sda1 SECRET

# format the partition
sudo mkfs.ext4 /dev/sda/SECRET

# mount it
sudo mount /dev/mapper/SECRET /mnt

# use it, and when you are dnone
sudo umount /mnt

# remove the association for now
sudo cyptsetup --verbose luksClose SECRET
```

## Mounting at boot

To mount an encrypted partition at boot, two conditions ahve to be satisfied

* Make an appropriate entry in /etc/fstab. Can be simple as `/dev/mapper/SECRET /mnt ext4 defaults 0 0`
* Add an entry to /etc/crypttab. Simple as `SECRET /dev/sdc12` 