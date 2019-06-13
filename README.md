# ie4k-iox-custom-top-process
Demonstrate how you can run your own top level process on IOx on Cisco IE4000

## Build

Do this on a Linux.  You will get package.tar for IE4000.

```sh
cd docker
sudo docker build -t topproc --no-cache .
cd ../app
sudo ioxclient docker package topproc .
```

Note that you cannot run your custom top process on Docker.
Following must fail.

```shell-session
$ sudo docker run -ti topproc
standard_init_linux.go:207: exec user process caused "exec format error"
$
```

It is because all executable inside this docker image except `/bin/sh` are
for PowerPC.  Docker cannot run PowerPC executables directly.
Instead, you can run `/bin/sh` and then invoke any executable in the image
from the shell.  The `/bin/sh` actually kicks QEMU emulator and all
subsequent shell sessions run on top of QEMU emulated environment.

So, you can do this.

```shell-session
$ sudo docker run -ti topproc /bin/sh
/ # topproc
.....^C
/ #
```

# Run

Navigate to app directory and do following.

```console
ioxclient application run topproc package.tar
```

You can determine if `topproc` is actually running as PID 1 process of IOx container with following command.

```shell-session
$ ioxclient application console topproc
Currently active profile :  default
Command Name:  application-console
Console setup is complete..
Running command : [ssh -p 22 -i topproc.pem appconsole@10.10.10.10]
/ # ps
  PID USER       VSZ STAT COMMAND
    1 root      1856 S    /usr/local/bin/topproc
   36 root      3624 S    /bin/sh
   37 root      3628 R    ps
/ # 
```
## Tested Software Versions

- IOS 15.2(7)E
- IOx 1.7.1.7 (bundled with IOS 15.2(7)E)
- ioxclient version 1.8.1.0
