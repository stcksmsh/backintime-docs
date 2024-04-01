User callback script
====================

## Introduction

During the backup process, `Back In Time` can call a user defined callback script at different steps.
This script is contained in the file `$XDG_CONFIG_HOME/backintime/user-callback` (by default `$XDG_CONFIG_HOME` is `~/.config`).
It can also be configured via:

```
Manage profiles > Options > Edit user-callback
```

## Script arguments

1. The profile id (1=Main Profile, ...).
2. Profile name.
3. Callback reason:

| Value | Reason |
| ----- | ------ |
| **1** | A backup process is about to start. |
| **2** | A backup process has ended. |
| **3** | A new snapshot was taken. <br>The following arguments are snapshot ID and snapshot path. |
| **4** | There was an error. For more information see [below](#errors). |
| **5** | The (graphical) application has started. |
| **6** | The (graphical) application has closed. |
| **7** | Mounting a filesystem for the profile may be necessary. |
| **8** | Unmounting a filesystem for the profile may be necessary. |

### Errors

Possible error codes are:

| Code | Error |
| ---- | ----- |
| **1** | Configuration is either missing or invalid (check configuration file) |
| **2** | A backup process is already running[^1] |
| **3** | Can't find snapshots folder[^2] |
| **4** | A snapshot for "now" already exists <br> The fifth argument is the snapshot ID |
| **5** | Error while taking a snapshot[^3] <br> The fifth argument contains more error information (string) |
| **6** | New snapshot taken but with errors[^4] <br> The fifth argument is the snapshot ID |

## Return value

The script should return $0$ if the backup should continue, any value other than $0$ will cancel the backup

## Implementation

The `UserCallbackPlugin` is a class defined [here](https://github.com/bit-team/backintime/blob/dev/common/plugins/usercallback.plugin.py) which is a 'child' of class `Plugin` which you can find [`here`](https://github.com/bit-team/backintime/blob/dev/common/pluginmanager.py).

## Examples

A simple script to log all calls to the user-callback script to a file in `$HOME/.local/state/backintime_callback_log`

```
#!/bin/bash

# Get current time
current_time=$(date +"%Y-%m-%d %H:%M:%S")

# Check if file exists, if not create it
touch $HOME/.local/state/backintime_callback_log

# Append current time to the file
echo -n "{$current_time}: " >> $HOME/.local/state/backintime_callback_log

# Iterate through all arguments
for arg in "$@"
do
    # Append argument to the file
    echo -n "$arg," >> $HOME/.local/state/backintime_callback_log
done

# Append newline character at the end
echo >> $HOME/.local/state/backintime_callback_log

```

[^1]:Make sure that manual and automatic backups do not run at the same time.
[^2]:For example, if your snapshots folder is on a removable drive, which is either not mounted, or is mounted at a different location
[^3]:introduced Aug. 17. 2023.
[^4]:introduced Aug. 17. 2023.