# bin-upgrade

This is a try to automate in some ways the FreeBSD binary upgrade (`freebsd-update`).  
It's not a means to do an unatended upgrade.
Indeed, there is just as much or more interactive than the use of `freebsd-update`.  
To be clear, I wrote this to avoid oversight during an upgrade.

**WARNING**: this is a beta version for the moment. Do not use on a system you care for.

---------------------------------
Usage: `bin-upgrade command`  
`command` can be one of:  
* **check**: try to parse the conf file.  
* **com**: comment the speficied lines in the designated files.  
* **uncom**: uncomment the speficied lines in the designated files.  
* **cont**: resume the upgrading process.  
* **clean**: to reset the upgrade process (remove the temp file).  
* **XX.x-RELEASE**: upgrade to this release. By default, it operates
      in "dry run" mode. `freebsd-update` won't run, but files will be
      commented/uncommented and hooks executed.
  
Options:
* -f: don't check "XX.x-RELEASE", instead pass it directly to `freebsd-update`. 
* -r: run in real mode (no dry run). Execute `freebsd-update`.
Both options have no effect for others commands than XX.x-RELEASE.
---------------------------------  

Doing an upgrade may involves several things, among them: 
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

It has three features:
- Comment out selected lines in the config files (and uncomment them once the upgrade is finished).
- Execute some commands of your choice at a given stage of the procedure (also called "hooks").
- In case of major upgrade, execute `pkg-static -f upgrade` and propose to remove old shared files.

It needs a configuration file where you put your actions (see bin-upgrade.conf.example).  

In this configuration file, you introduce the name of a file by ':', like ':/etc/rc.conf'.
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

For these commands, at any stage, you have access to some variables, among them:
- $Release: the target RELEASE for upgrade (e.g. '14.2-RELEASE').
- $Major: true if this is a major upgrade.
- $DryRun: true if it operates in dry run mode.
- $CheckRelease: true if it checked the "XX.x-RELEASE" string.
- $cmd: the command line that it is executing.
- $ScriptName: "bin-upgrade" unless you changed the file name.
- $ScripFullName: as above plus the path.
- $MAX_HOOKS: number of possible hooks.
- $Hook*N*_name: name of the hook number *N*.
- $Hook*N*_cmd: commands in the hook number *N* (delimited by #).
- $nF: number of files to comment.
- $file*N*: name of the file *N*.
- $data*N*: lines to comment out in the file *N* (delimited by a new line).

In addition, starting at the stage 'after-comment':
- $Com: 1 if the files have been commented, 0 otherwise.
- $Fetch: 1 if the upgrade files have been fetched, 0 otherwise.
- $Install: level of installation progress. 0, nothing installed. 1, new kernel installed. 2, new base installed. 3, old shared files removed.

These two shell commands have a special behaviour in a hook:
- exit: `bin-upgrade` is stopped. You can resume it with `bin-upgrade cont`. If there are some others commands after 'exit' in the same hook, they are executed when `bin-upgrade` resumes.
- return: the remaining commands in the hook will be ignored. The process just continue as if 'return' was the last command of the hook.
-----------------
### How to use it?

- Install the script on the machine you want to upgrade.
- Write a suited bin-upgrade.conf file.
- Run: `bin-upgrade check` to see if something is wrong in the conf file. Read carefully what it writes to see if it is what you expect.
- Be sure to not be root and run it in dry mode. It won't be able to change the system configuration files, nor execute potential harmful commands you put in the hooks. You don't need to reboot when asked for. Just type `bin-upgrade cont` to reach the end of the upgrade process. Again, read carefully what it would do. If you encounter an error, you can use `bin-upgrade clean` to reset the process.
- Once the previous stage is validated, you can upgrade your system for real with `bin-upgrade -r XX.x-RELEASE`.

Note: you can upgrade to a non-RELEASE version with the -f option. Example: `bin-upgrade -r -f 13.5-BETA1`.


