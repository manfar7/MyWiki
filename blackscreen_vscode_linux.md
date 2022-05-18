# Blackscreen vscode linux
Original Post: https://github.com/Microsoft/vscode/issues/6379#issuecomment-219314862

Disclaimer: I am not a Linux dev/regular user. I am in the process of learning Linux myself. I can not be held responsible for this failing or causing you grief.

Now that the disclaimer is out of the way here is what I did.
After installing via the .deb package, I edited the /usr/share/code/bin/code file using a text editor.

```
#!/usr/bin/env bash
#
# Copyright (c) Microsoft Corporation. All rights reserved.
# Licensed under the MIT License. See License.txt in the project root for license information.

ARGS=$@

# If root, ensure that --user-data-dir is specified
if [ "$(id -u)" = "0" ]; then
    while test $# -gt 0
    do
        if [[ $1 == --user-data-dir=* ]]; then
            DATA_DIR_SET=1
        fi
        shift
    done
    if [ -z $DATA_DIR_SET ]; then
        echo "It is recommended to start vscode as a normal user. To run as root, you must specify an alternate user data directory with the --user-data-dir argument." 1>&2
        exit 1
    fi
fi

if [ ! -L $0 ]; then
    # if path is not a symlink, find relatively
    VSCODE_PATH="$(dirname $0)/.."
else
    if which readlink >/dev/null; then
        # if readlink exists, follow the symlink and find relatively
        VSCODE_PATH="$(dirname $(readlink $0))/.."
    else
        # else use the standard install location
        VSCODE_PATH="/usr/share/code"
    fi
fi

ELECTRON="$VSCODE_PATH/code"
CLI="$VSCODE_PATH/resources/app/out/cli.js"
ATOM_SHELL_INTERNAL_RUN_AS_NODE=1 "$ELECTRON" "$CLI" $ARGS
exit $?
```

Above is the original file that is installed via the .deb package. I added 1 line and modified another. The line that I added was DEFAULT_ARGS=--disable-gpu right before the ARGS=$@ line. Then at the bottom of the file I modified ATOM_SHELL_INTERNAL_RUN_AS_NODE=1 "$ELECTRON" "$CLI" $ARGS to be ATOM_SHELL_INTERNAL_RUN_AS_NODE=1 "$ELECTRON" "$CLI" $DEFAULT_ARGS $ARGS

Below is complete copy of the file which I modified.

> Comment @manfar7: I only added ***DEFAULT_ARGS=--disable-gpu***

```
#!/usr/bin/env bash
#
# Copyright (c) Microsoft Corporation. All rights reserved.
# Licensed under the MIT License. See License.txt in the project root for license information.

DEFAULT_ARGS=--disable-gpu
ARGS=$@

# If root, ensure that --user-data-dir is specified
if [ "$(id -u)" = "0" ]; then
    while test $# -gt 0
    do
        if [[ $1 == --user-data-dir=* ]]; then
            DATA_DIR_SET=1
        fi
        shift
    done
    if [ -z $DATA_DIR_SET ]; then
        echo "It is recommended to start vscode as a normal user. To run as root, you must specify an alternate user data directory with the --user-data-dir argument." 1>&2
        exit 1
    fi
fi

if [ ! -L $0 ]; then
    # if path is not a symlink, find relatively
    VSCODE_PATH="$(dirname $0)/.."
else
    if which readlink >/dev/null; then
        # if readlink exists, follow the symlink and find relatively
        VSCODE_PATH="$(dirname $(readlink $0))/.."
    else
        # else use the standard install location
        VSCODE_PATH="/usr/share/code"
    fi
fi

ELECTRON="$VSCODE_PATH/code"
CLI="$VSCODE_PATH/resources/app/out/cli.js"
ATOM_SHELL_INTERNAL_RUN_AS_NODE=1 "$ELECTRON" "$CLI" $DEFAULT_ARGS  $ARGS
exit $?
```
