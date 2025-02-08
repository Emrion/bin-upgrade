# bin-upgrade

This is a try to automate in some ways the FreeBSD binary upgrade (`freebsd-update`).  
But, it's not a means to do an unatended upgrade.
Indeed, there is just as much or more interaction than the use of `freebsd-update`.  

**WARNING**: this is a beta version for the moment. Do not use on a system you care for.

---------------------------------
Usage: `bin-upgrade command`  
`command` can be one of:  
* **check**: try to parse the conf file.  
* **com**: comment the speficied lines in the designated files.  
* **uncom**: uncomment the speficied lines in the designated files.  
* **cont**: to use after kernel upgrade and reboot.  
* **clean**: to reset the upgrade process (remove temp file).  
* **XX.x-RELEASE**: upgrade to this release.
  * Prefix by 'n-' to do a dry run. Hooks will be executed.
---------------------------------  

Doing an upgrade involves several things, among them: 
- Make a backup of the system (if not automatic).
- Disable some module loading at startup to avoid crash.
- Disable the graphical login manager if any.
- Doing the upgrade itself (including reboot).
- Replace some third-party modules by the versions compiled on the new release.
- Test the new installed modules.
- Enable anew the loading of these modules at startup.
- Upgrade all pkg if this is a major upgrade.
- Update the loaders.

This tool proposes to do for you some of these things instead of doing them manually.  

It has two features:
- Comment out selected lines in the config files (and uncomment them once the upgrade is finished).
- Execute some commands of your choice at a given stage of the procedure (also called "hooks").

It needs a configuration file where you put your actions (see bin-upgrade.conf.example).  

Inside, you introduce the name of a file by ':', like ':/etc/rc.conf'.
The lines that follow are searched in this file and commented near the beginning of the upgrade. You don't need to write the whole line, but only the beginning. `bin-upgrade` will find the appropriate(s) line(s).  

The commands to execute are CLI, including any installed software. It must begin by '@' followed by the name of the stage. Then, each following line is executed in the writted order at the selected stage.  

The different stages are:  
- **prelude**: just before to comment out the target lines.
- **after-comment** : just after.
- **before-reboot**: the new kernel is in place, but not active until a reboot.
- **kernel-ok**: the new kernel is active, but the base isn't yet installed.
- **base-installed**: as the name, but some pieces may not be active until a reboot. This is the point where you need to install new third-party kernel modules.
- **after-uncomment**: the target lines of the configuration files are uncommented. This is the end if the upgrade is minor.
- **major-upgrade**: we are just before the automatic run of `pkg-static -f upgrade`.
- **pkg-upgraded**: you are asked if it's ok to remove old shared files (libs). After this possibly last `freebsd-update install`, this is the end of the major upgrade.

For these commands, at any stage, you have access to two special variables:
- $Release: the target RELEASE for upgrade (e.g. '14.2-RELEASE').
- $Major: true if this is a major upgrade.

Note that the command `exit` is allowed in a stage hook. The script is then stopped. You can resume it with `bin-upgrade cont`.



