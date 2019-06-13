# ie4k-iox-custom-top-process
Demonstrate how you can run your own top level process on IOx on Cisco IE4000

## Build

Do this on a Linux.  You will get package.tar for IE4000.
See note on the bottom of this README if your Docker environment
is behind proxy.

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

## Run

Navigate to `app` directory and do following.

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

## Proxy for Docker container

The build process of Docker image involves Internet access from
inside of container to be built.  If your Docker is sitting behind
a proxy, you have to tell container proxy configuration to be used
by processes of inside container.  This must be done separately
from proxy setting for Docker engine itself.

To tell your proxy settings to container you run, create
`~/.docker/config.json` and write proxy configuration like this.

```json
{
 "proxies":
 {
   "default":
   {
     "httpProxy": "http://your-proxy-server:port",
     "httpsProxy": "http://your-proxy-server:port"
   }
 }
}
```