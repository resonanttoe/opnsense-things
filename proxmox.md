# Random Notes
## Failing backups

### LXC Privs
```
Permission denied at /usr/share/perl5/PVE/Storage/Plugin.pm line 952. (500)
```
Is a result of manually changing an LXC priviledge level. Don't do that, backup and restore to change privs.

### SMB noserverino

From [here][def]

On host, in /etc/fstab add CIFS/SMB mount maunally
```
//server/Mount /media/localdir cifs username=<pass>,password=<pass>,noserverino 0 0
```
Then add as directory storage in Proxmox PVE

[def]: https://forum.proxmox.com/threads/backup-vm-fehler-fehlende-vzdump-quemu-ordner.115271/

## Performance

### Multicast snooping on Bridge Devices messes with IPV6

```
echo 0 > /sys/devices/virtual/net/vmbr0/bridge/multicast_snooping
```