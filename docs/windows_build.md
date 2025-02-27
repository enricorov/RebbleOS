# Building on Windows

## Windows Subsystem for Linux

First set up WSL2 and get RebbleOS building on it, for which you can use the [Building on Debian](docs/debian_build.md) steps.

### Visual Studio Code integration

To automate the process of building and debugging with VS Code on WSL2, you can use these template `tasks.json` and `launch.json`:

```json
{
    // tasks.json

    "version": "2.0.0",
    "tasks": [
        {
            "label": "Build RebbleOS",
            "type": "shell",
            "command": "make ${input:buildTargets}",
            "group": "build",
            "problemMatcher": "$gcc",
        },
    ],
    "inputs": [
        {
            "id": "buildTargets",
            "type": "pickString",
            "description": "Selects the build target from the list",
            "options": [
                "snowy",
                "snowy_qemu",
                "snowy_qemu QEMUFLAGS=-S",
                "snowy_gdb",
                "tintin"
            ]
        }
    ]
}
```

```json
{
    // launch.json

    "version": "0.2.0",
    "configurations": [
        {
        "type": "gdb",
        "request": "attach",
        "name": "Attach to gdbserver",
        "executable": "${workspaceFolder}/build/snowy/tintin_fw.elf",
        "target": "localhost:63770",
        "remote": true,
        "cwd": "${workspaceRoot}", 
        "gdbpath": "/usr/bin/arm-none-eabi-gdb",
        "autorun": [
                // "any other gdb commands to initiate your environment, if it is needed"
            ]
        }
    ]
}
```

To build and step through RebbleOS, you need to:

1. Set a breakpoint somewhere in the code, selecting the line and pressing `F9`.
1. Invoke the build task with `CTRL+Shift+B` and select `make snowy_qemu QEMUFLAGS=-S`. The output from `qemu` will be displayed in the console. 
1. Press `F5` to attach VSCode to `gdb`. The emulation will continue and stop at the breakpoint you set.


## Linux VM
### Setting up a virtual machine 

- Install [VirtualBox](http://www.virtualbox.org/)
- Download the Ubuntu's ISO from [their website](https://www.ubuntu.com/download/desktop)
- Inside virtualbox install Ubuntu
- You propably want to install the guest additions from VirtualBox
- Once inside the booted Ubuntu, open a terminal for further install steps

### Install pebble SDK

For installing the pebble sdk and building on debian/ubuntu, please see the instructions in the [debian build documentation](https://github.com/pebble-dev/RebbleOS/blob/master/docs/debian_build.md)


## Setting up a development environment on Windows

If you are a more experienced, you can follow these steps to gradually move some of the development process back to Windows. Please notice that this is not an alternative to the solution above, but rather an extension which should make life easier. As such setting it up can be rather complicated.

### First steps

- Download MSYS from [this website](https://sourceforge.net/projects/mingwbuilds/files/external-binary-packages/)
- Extract it to some directory
- Download the arm-none-eabi `Windows ZIP` package from [the ARM website](https://developer.arm.com/open-source/gnu-toolchain/gnu-rm/downloads)
- Extract it over MSYS (both archives have a bin folder with various *.exe files, these should be in the same folder after this step)

You can now compile the firmware by first cloning this repository and then running `make` (explicitly the one you downloaded) in the root folder, e.g.

```batch
C:\msys\bin\make.exe snowy_qemu
```

To prevent the copying of the compiled firmware, you can set up a shared folder in VirtualBox (in the menu tab "Devices"). However you can only have build files from either Windows or Linux present, which basically refrains you from using `make`. To start the emulator you have to run the command yourself (or put it in a bash script nearby):

```sh
cat Resources/snowy_boot.bin build/snowy/tintin_fw.bin > build/snowy/fw.qemu_flash.bin
qemu-pebble -rtc base=localtime -serial null -serial null -serial stdio -gdb tcp::63770,server -machine pebble-snowy-bb -cpu cortex-m4 -pflash build/snowy/fw.qemu_flash.bin -pflash Resources/snowy_spi.bin
```

and to attach gdb to it:

```sh
arm-none-eabi-gdb -ex "target remote localhost:63770" -ex "sym build/snowy/tintin_fw.elf"
```

### Remote Debugging

For remote debugging to work you first have to enable the communication to your VM. The easiest way to do this (if you followed the steps above) is to open VirtualBox and select Your VM->Change->Networking->Advanced->Port Forwarding and add a new rule, which should like this:

|  Name  | Protocol | Host-IP | Host-Port | Guest-IP | Guest-Port |
|--------|----------|---------|:---------:|----------|:----------:|
| Rule 1 | TCP      |         |   63770   |          |    63770   |

Then you can start the emulator on the virtual machine by running `make snowy_qemu` and then in windows:

```batch
C:\msys\bin\arm-none-eabi-gdb.exe -ex "target remote localhost:63770" -ex "sym build/snowy/tintin_fw.elf"
```