# LTO Tape things

## Encryption

No real info on the efficacy of LTO native encryption, though it does use AES-256-GCM-128 
That said arguments against it seem to be that its hardware native and "why would you trust it?"

use [STENC](https://github.com/scsitape/stenc) - Apt repos are out of date and have a drive crashing [error](https://serverfault.com/questions/864580/what-could-cause-a-sense-error-when-setting-lto-encryption)

Pre-reqs for compiling on Debian

- build-essentials
- git
- pandoc
- dh-autoreconf

## Set Encryption

Insert a tape and ask the tape drive about its settings:
```
stenc -f /dev/nst0 --detail
```

Generate your 256 bit key and store it:

```
stenc -g 256 -k /root/myaes.key -kd descriptionhere
```

Load the key in the LTO tape drive. With --ckod it will forget the key after tape eject.

```
stenc -f /dev/nst0 -e on -k /root/myaes.key -a 1 --ckod
```

## Large multi-tape archives

```
tar -v --label=ArchiveName -Mcf /dev/nst0 Folder1/ Folder2/
```

Will prompt for new tape on non-library setups.

## Restoring

TDB

## Misc Commands

Using MT-ST (in debian apt)

```
mt-st -f /dev/nst0 rewind
```
```
mt-st -f /dev/nst0 erase
```