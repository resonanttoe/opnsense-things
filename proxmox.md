# Random Notes
- Failing backup with
```
Permission denied at /usr/share/perl5/PVE/Storage/Plugin.pm line 952. (500)
```
Is a result of manually changing an LXC priviledge level. Don't do that, backup and restore to change privs.
