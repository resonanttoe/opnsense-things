# LTO Tape things

Good info [HERE](https://darkimmortal.com/adventures-with-lto-tape-for-home-casual-use/) (Dump below)

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

## Multi-Tape archives with correct Block Size (Don't set drive block size)
```
tar --label="Name $(date -I)" --one-file-system -b 4096 --xattrs --totals --exclude="Downloads" --totals=USR1 -cf -  /FilesToArchive | mbuffer -m 5G -L -P 95 -f -s 2M -A "echo Insert next tape and press enter; read a < /dev/tty" --tapeaware -o /dev/nst0
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


Adventures with LTO Tape for Home/Casual Use
I've spent the past while buying and experimenting with some old LTO gear as a secondary backup solution besides backing up to both local disks and cloud. I chose LTO because:

My upload speed is trash so the only way I can hope to do a full off-site backup is via physical media
With LTO4, it works out cheaper than buying about 2-3x 2+TB external hard disks, especially if you go SCSI instead of SAS.
I have the option of taking tapes out of rotation and storing them long-term. With hard disks this is risky and expensive.
- Durable for transport
- It's old school cool
- My learnings and notes follow...

## Hardware
- Don't touch any drives on eBay sold as 'powers up, no warning lights', or even as 'tested' without any elaboration on how it was 'tested'. There are so many ways in which the drives can go bad but still accept and read tapes. Once the head gets worn, there will be no obvious errors or warnings, but the drive will operate at half speed or less and can only write about 60-70% of the nominal capacity of a tape. Only buy drives taken from a working environment and/or ones that have explicitly passed diagnostics.

- You should probably buy a cleaning tape unless you feel confident enough to clean the head manually, but either way you need a cleaning tape to clear the light.

- Everything is made by two different OEMs, expect to see rebranding of the same drives.

- Make sure you get a drive that supports hardware encryption if you want to encrypt. Not all LTO4 drives do. Update: potential firmware fix for the ones that don't.

- Have a dedicated Linux PC for your tape drive.
- SCSI or Fibre Channel are probably the cheapest options relative to SAS. Used HBAs are cheap for all three, but SAS SFF-8470 to SFF-8088 cables are extortionately expensive due to their continued relevance in 2018. 

- External drives are just internal drives with a (loud) built-in fan, AC->molex power supply and external to internal port style converter. For home use they importantly have a mains switch power button (i.e. 0 standby current) and can be hot-plugged.

- Internal drives need substantial active cooling or they will overheat. The average silent desktop PC or home server built out of old desktop parts may not have the kind of cooling they expect, or at least not have the feedback loop necessary to start spinning the fans faster when the tape drive overheats. Proper servers use ambient temperature sensors for fan control in addition to CPU temperature etc, so they can respond quicker to overheating drives. If your hard drives run more than a handful of C above room temperature, you don't have enough cooling for an internal drive. The external drives have a thicc ball bearing fan that runs at continuous high RPM and they can still get concerningly hot. Fun fact, you can smell it when the temperature is near exceeding safe tape temperature.

- For at least up until LTO4, Half-Height drives are slower and very compact/over-engineered internally (think modern mobile phone style with special shaped PCBs and tonnes of ribbon cables). You probably want Full-Height unless there is a big cost difference.
Software

- LTO hardware compression is incredible. A tar backup of my Windows C: drive, excluding the Windows folder, and made up of mostly git repos, programs, games, downloads and VM images, came in at 1.45:1 compression ratio. Bzip2 at level 9 came in at 1.49:1.

- If you are determined to use software compression, do not use meme formats like gzip or xz. Stick to formats that can handle bitrot by losing a single block rather than the entire remainder of a tar archive, such as bzip2 and lzip. However you are still best off using hardware compression, as recovering from bitrot is less of a manual process (you just have to split the tar archive in half around the corruption, you don't also have to split the compressed format)

- Hardware encryption is also very useful (and essential if using hw compression). On Linux you can access this using stenc. If you favour future accessibility over super strong encryption, you could feed this a key resulting from the output of sha256sum of a passphrase or similar.

- Don't fall for the proprietary backup software/bacula/amanda meme. You want these backups to be accessible in decades time, and you or the software might not exist then. Use tar. If you want to store multiple backups per tape, use tar labels and/or LTFS.

- Windows 10 WSL is still shit for full-system backups via Linux tools (slow and permissions issues). Use cygwin running as administrator to take a full system tarball.

- When backing up a Windows PC, don't forget to take a registry backup. I use ERUNT.
Pipe through mbuffer when reading from anywhere but a contiguous image on local disk. Ideally the buffer size should be the length of a full tape wrap (i.e. end to end), but a few GB is good enough.

## Conceptual differences to disk
A lot of these might seem like stating the obvious, but it took me a while to get my head around it all.

- Without special tools, it is impossible to know how much space is free without trying to write to all of said space

- Without special tools, it is impossible to know what compression ratio you are achieving without either filling the rest of the tape with incompressible data or writing your tar archive repeatedly until the tape is full, then doing the appropriate maths.

- 'Special tools' referred to in above 2 points are proprietary tools such as HP LTT, IBM ITDT, or from reading the drive SCSI logs (sg_logs -a /dev/nst0)

- The equivalent of HDD sector size, i.e. block size, can be massive in comparison and is dynamically configurable. I saw best performance on my LTO4 drive at a 2MB block size.

- Unless you use consistent blocksizes (dd bs=whatever iflag=fullblock) or special tools as mentioned above, it is impossible to know where you are on the tape in bytes

- Writes to linux tape block device are automatically split into files using EOF markers, which can be seeked between using mt-st

- EOF markers are magic in that they can be found very quickly - does not require reading back the entire tape to find/seek to one. They are a relatively large area of special data that the head can recognise at full rewind/fast-forward speed.

- All data must be contiguous, and you can't insert data at anywhere except the end. Inserting data in the middle of the tape will logically delete all remaining data. (You can recover it with low level commands to skip eof.)
Notes/Processes

## On startup on your tape drive PC:

Either setup and call stinit (see example config near bottom of post), or run 
```
mt-st -f /dev/nst0 stoptions scsi2logical
```

Tar of a Windows PC over SSH to tape: (Important bits are -b 4096and bs=2M, and mbuffer with a generous buffer size to prevent underruns/shoe shining. For Linux it's roughly the same but add --one-file-system and --xattrs)

```
tar \
--exclude='/cygdrive/c/Windows' \
--label="C drive $(date -I)" \
-b 4096 -cf - /cygdrive/c \
| ssh root@blah "mbuffer -m 4G -L -P 95 -f | dd of=/dev/nst0 bs=2M iflag=fullblock"
```

#### Check tape [remaining] capacity / wipe tape:
```
openssl enc -aes-256-ctr -pass pass:"$(dd if=/dev/urandom bs=128 count=1 2>/dev/null | base64)" -nosalt < /dev/zero | dd of=/dev/nst0 bs=2M status=progress iflag=fullblock
```

#### Get tar label:
(Note this uses a raw read command as on my drive using dd count=1 causes the drive to continue uninterruptably until it hits the next file marker! Facepalm, don't use sysv mode in stinit.)
```
sg_raw -n -o - -r 1024 -t 60 -v /dev/nst0 08 00 00 04 00 00 2>/dev/null | perl -e 'read(STDIN, $header, 100); print(unpack("Z100", $header)."\n");'
```

#### Sometimes the perl approach does not work, in which case:
```
sg_raw -n -o - -r 1024 -t 60 -v /dev/nst0 08 00 00 04 00 00 2>/dev/null | strings | grep volume.label | cut -d \= -f2
```

#### Search files:
```
dd if=/dev/nst0 bs=2M status=progress | tar tf - | grep whatever
```

#### Restore selected files:
```
dd if=/dev/nst0 bs=2M status=progress | tar -xvpf - folder/inside/tar
```

#### Rewind tape:
```
mt-st -f /dev/nst0 rewind
```

#### Get tape position in blocks: (only useful if using consistent blocksize without allowing partial blocks)
```
mt-st -f /dev/nst0 tell
```

#### Jump to x file: (where x is 0 indexed file ID)
```
mt-st -f /dev/nst0 asf x
```

#### Jump to end of tape: (do not typo this as eof or you risk data loss!)
```
mt-st -f /dev/nst0 eod
```

#### IBM drives crash from the eod command, but you can run it safely via ITDT instead:
```
./itdt -f `find /dev -group tape | grep /sg` seod
```

#### Erase tape: (potentially more thorough than overwriting)
```
mt-st -f /dev/nst0 erase
```

#### Fast-forward past x files: (where x is number of files to skip)
```
mt-st -f /dev/nst0 fsf x
```

#### Rewind by x files: (where x is the number of files, +1, e.g. use 2 to rewind to start of previous file)
```
mt-st -f /dev/nst0 bsfm x
```

#### Go to start of current file: (e.g. after reading tar label)
```
mt-st -f /dev/nst0 bsfm
```

#### Print index of all tar archives on tape: (requires that they have been suitably labelled)
```
mt-st -f /dev/nst0 rewind
while true; do BLOCK=`mt-st -f /dev/nst0 tell`; echo $BLOCK; sg_raw -n -o - -r 1024 -t 60 -v /dev/nst0 08 00 00 04 00 00 2>/dev/null | perl -e 'read(STDIN, $header, 100); print(unpack("Z100", $header)."\n");'; mt-st -f /dev/nst0 fsf; [[ $BLOCK != `mt-st -f /dev/nst0 tell` ]] || break; done
```

#### Sometimes the perl approach does not work, in which case:
```
mt-st -f /dev/nst0 rewind
while true; do BLOCK=`mt-st -f /dev/nst0 tell`; echo $BLOCK; sg_raw -n -o - -r 1024 -t 60 -v /dev/nst0 08 00 00 04 00 00 2>/dev/null | strings | grep volume.label | cut -d \= -f2 ; mt-st -f /dev/nst0 fsf; [[ $BLOCK != `mt-st -f /dev/nst0 tell` ]] || break; done
```

#### List files:
```
tar -b 4096 -tvf /dev/nst0
```

#### Extract multi-volume:
```
tar -b 4096 --multi-volume -xvpf /dev/nst0
```

#### Tar files newer than:
```
find /dolan/dark-dl -newer "asdf.txt" -print0 -type f | \
xargs -0 tar --label="owo $(date -I)" --no-recursion --multi-volume --one-file-system -b 4096 --xattrs --totals --totals=USR1 -cvf /dev/nst0
```

#### Tar via mbuffer without dd:
(This uses mbuffer to provide volume splitting, which I've found is less hassle to extract than tar --multi-volume, even though theoretically tar is a little more robust due to duplicated header for the last file)
```
tar --label="something $(date -I)" --one-file-system -b 4096 --xattrs --totals --totals=USR1 -cf - /source | mbuffer -m 5G -L -P 95 -f -s 2M -A "echo Insert next tape and press enter; read a < /dev/tty" --tapeaware -o /dev/nst0
```

#### Extract mbuffer multi-volume: (This is not the same as tar multi volume) (-n 0 may need to be the actual number of tapes)
```
mbuffer -i /dev/nst0 -s 2M -m 5G -L -p 5 -f -A "echo Insert next tape and press enter; mt-st -f /dev/nst0 eject; read a < /dev/tty" -n 0 | tar -xvpf -
```

#### Various status commands: (obviously these require extra software)
```
tapeinfo -f /dev/nst0 
stenc -f /dev/nst0 --detail
mt-st -f /dev/nst0 status
smartctl -a /dev/nst0
sg_logs -a /dev/nst0
```

#### Various interesting things in: /sys/class/scsi_tape/nst0
```
/etc/stinit.def: (These settings work reliably for HP and IBM LTO-4 drives, YMMV)

manufacturer="TANDBERG" model = "LTO-4 HH" {
scsi2logical=1
can-bsr=1
auto-lock=1
two-fms=0
drive-buffering=1
buffer-writes
read-ahead=1
async-writes=1
can-partitions=1
fast-mteom=0
sysv=0
noblklimits=0
mode1 blocksize=0 density=0x00 compression=1
mode2 disabled=1
mode3 disabled=1
mode4 disabled=1
}
TAR.BZ2 recovery:
```

See https://wiki.bluelightav.org/pages/viewpage.action?pageId=8126784 (archive link) - you'll need find_tar_headers.pl

```
bzip2recover archive.tar.bz2
bzip2 -tv rec*.bz2 > testoutput.log 2>&1
# split into folders at each broken bit
bzip2 -dc rec*.bz2 > recovery1.tar
bzip2 -dc rec*.bz2 > recovery2_failing.tar
./findtarheader.pl recovery2_failing.tar | head -n 1   # first number from this into tail
tail -c +17185 recovery2_failing.tar > recovery2_working.tar
```

# Backup commands

```
tar --label="LABEL $(date -I)" --one-file-system -b 4096 --xattrs --totals --exclude="Downloads" --totals=USR1 -cf -  /PATH | mbuffer -m 5G -L -P 95 -f -s 2M -A "echo Insert next tape and press enter; read a < /dev/tty" --tapeaware -o /dev/nst0
```
