Singularity is similar to a virtual machine: encapsulation of an entire root file system along with an application. Container can be package all of this into a unique file. Reproducible independently on the computer you are using (but I guess OS matters). So, you can have Manjaro host with a Ubuntu host, double check with `/etc/os-release`.

Think about your computer has set of stacks:
1. hardware
2. kernel: operates the hardware
3. OS
4. file system
5. applications

If you use a VM, on top of the file system you have another another kernel, with another virtualization and OS and file system and application. 

Remember that containers share the kernel with the "host", so you cannot use singularity Linux on Windows. 

Containers are Linux beasts: you can create a container at runtime, your are using namespaces and you put them all together, which is a Linux thing (apparently).

Singularity is a single find (that why the name). Singularity was created with the HPC in mind: if you are not root on the system, you cannot be root on the container (different compared to docker). 

Singularity provides (I think) the glibc. The Linux kernel and GNU C Library together form the Linux API.

Singularity can read and write from host. So you can pipe into the container `cat cowsaid | singularity exec lolcow_latest.sif cowsay -n`. But to pipe inside the container you need to do that `singularity exec lolcow_latest.sif sh -c "fortune | cowsay | lolcat"`.

## Useful commands

**exec:** execute some programs in the container from the host with arguments also `singularity exec lolcow_latest.sif cowsay bla`

**shell:** enter the container `singularity shell lolcow_latest.sif`. To make changes to the container you need to enter it as root and add the option `--writable`: `sudo singularity shell --writable lolcow`. 

**run:** when you build you can assign a runscript, metadata, to the container. Run that script. Note that you can execute the container directly, without the `run` command: `./lolcow_latest.sif`. 

**inspect:** inspect some programs in the container, such as the `runscript`: `singularity inspect --runscript lolcow_latest.sif`

**build:** takes the options (such as `--sandbox` see below) the name of the container and the definition file. 

# Building a container

To build a container you need a definition file: with header, . The this file is just a set of instructions telling Singularity what software to install in the container.

Use the Singularity flow: create a `sandbox` a build stuff progressively and record your commands until you break your container. Then, recreate a new `sandbox` starting from your recorded commands, and so on.

Examples are located in the folder `examples` of the singularity pkg installed from the AUR. Taking the debian example definition file:

```
BootStrap: debootstrap
OSVersion: stable
MirrorURL: http://ftp.us.debian.org/debian/
```
this is the header. `deboostrap` is a program that a debian like file system, this program will be used by the host to create the image, **therefore you must have it installed in your machine in order to create the container**. The value associated to the `Boostrap` field, in this case `deboostrap`, will determine the other fields downstream, because the downstream fields are the argument that `deboostrap` will need to construct the debian-like file system. The value associated to the `MirrorURL` key is a debian mirror.

```
%post
    echo "Hello from inside the container"
    yum -y install vim-minimal
```
`post` instructions that will be run after the container is created, so after the header kind of.

When you look at the container just created, you can notice that it really looks like a file system with some symlinks that "store" the metadata. Read also [this](https://github.com/fraterenz/Singularity-Tutorial/tree/master/03-building#developing-a-new-container).

Always good to update the system once you build your container. Remember to add to the path the software that you install. To find the path you can do `cd /` and then `find -name fortune` where `fortune` is the software you have just installed. Sometimes `export LC_ALL=C`, see [here](https://youtu.be/3Yg7XI39H4U?t=7469), perl do not care about different locals, just use the default one.
# Questions
Can we use Slurm?
