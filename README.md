# HOWTO-Nectar-GPU

*BETA*

The University of Melbourne Nectar node has a small number of machines with NVIDIA K1 GPUs installed. These are intended for VDI (Virtual Desktop) applications, rather than general purpose (GPGPU) usage. Check out our Hybrid HPC cluster [Spartan](https://dashboard.hpc.unimelb.edu.au/) if you're interested in that, as it has some GPU nodes.

At the moment only Ubuntu 14.04 and Windows Server 2012 has been thoroughly tested,  let us know if you're interested in other operating systems. This guide assumes you're already familiar with the Nectar dashboard.

# Windows
### Background
You can run Windows on our GPU flavour, the only limitation being that Windows Server is required as it includes support for RemoteFX which ensures graphics are rendered on the remote GPU, rather than your client computer. Windows Server 2012 R2 is equivalent to Windows 8.1, and so most applications should run without issue.

### Steps

1\. Send a request to support (support@ehelp.edu.au), letting us know you'd like to run a GPU instance, and require the following:

* GPU flavour *mel.gpu-k1.large*
* Access to the *melbourne-qh2-uom* availability zone.
* Access to the Windows Server 2012 R2 install image.

2\. Install windows as per the steps at https://github.com/resbaz/HOWTO-Nectar-Windows-VM The install and configuration interface is slightly different in Windows Server, but the steps are much the same.

3\. Make sure you can connect to your instance via Remote Desktop, the GPU won't work via the browser based console.

4\. Install the NVIDIA drivers on the Windows instance: http://www.nvidia.com/download/driverResults.aspx/111408/en-us

5\. Reboot, and test using a benchmark like Unigine Heaven, we've been getting scores of around 440 using the default settings.


# Linux

### Background
You'll need to install the following on your instance to take advantage of the GPU.

* NVIDIA Binary Drivers
* VirtualGL - The redirects OpenGL and 3D data to the remote GPU, rather than attemping to render it on your local machine.
* TurboVNC - A VNC implementation that is compatible with VirtualGL, and optimised for 3D graphics.

You then run remote OpenGL applications via the `vglrun` command so that any 3D commands are picked up, rendered remotely, and the result forwarded to your client machine via TurboVNC.

### Steps

1\. Send a request to support (support@ehelp.edu.au), letting us know you'd like to run a GPU instance, and require the following:

* GPU flavour *mel.gpu-k1.large*
* Access to the *melbourne-qh2-uom* availability zone.

2\. Start an instance based on the mel.gpu-k1.large flavour. Make sure you select the *mel.gpu-k1.large* flavour, and in the post-creation tab post in the following shell script:

```
#!/bin/bash

export DEBIAN_FRONTEND=noninteractive
apt-get update
apt-add-repository -y ppa:ubuntu-mate-dev/ppa
apt-add-repository -y ppa:ubuntu-mate-dev/trusty-mate
apt-get update
apt-get -y -o Dpkg::Options::="--force-confdef" -o Dpkg::Options::="--force-confold" dist-upgrade
apt-get -y install linux-headers-$(uname -r) linux-image-extra-$(uname -r) build-essential
apt-get -y install --no-install-recommends ubuntu-mate-core ubuntu-mate-desktop
apt-get -y install xorg nvidia-352 mesa-utils

wget -P /tmp/ http://internode.dl.sourceforge.net/project/virtualgl/2.5.1/virtualgl_2.5.1_amd64.deb
wget -P /tmp/ http://internode.dl.sourceforge.net/project/turbovnc/2.1/turbovnc_2.1_amd64.deb
dpkg -i /tmp/turbovnc_2.1_amd64.deb
dpkg -i /tmp/virtualgl_2.5.1_amd64.deb
nvidia-xconfig
nvidia-xconfig --device="Device0" --busid="$(nvidia-xconfig --query-gpu-info | grep BusID | cut -f2- -d":" | xargs)" --use-display-device=none
/opt/VirtualGL/bin/vglserver_config -config +s +f +t

echo '#!/bin/sh' > /home/ubuntu/.Xclients
echo '. /etc/X11/xinit/xinitrc' >> /home/ubuntu/.Xclients
echo "startx" >> /home/ubuntu/.Xclients

reboot
```


3\. Once the instance has completed the scripted installs and rebooted, you can login to the instance, start the VNC server and set the VNC password:
```
/opt/TurboVNC/bin/vncserver
```

You'll need to create a VNC security group to allow incoming VNC connections, these are generally on port 5900+N where N is the display number. An alternative is to open an SSH tunnel to the instance.

4\. Connect to the instance using the TurboVNC client. https://sourceforge.net/projects/turbovnc/
Specify the instance IP address and include the display suffix (usually ':1')
E.g. 'VNC Server: 115.146.84.100:1'
Once connected you should be able to open a Terminal and run:

```
/opt/VirtualGL/bin/vglrun glxinfo
/opt/VirtualGL/bin/vglrun glxinfo | grep "OpenGL\|render"
/opt/VirtualGL/bin/vglrun glxgears
```


These instructions are with thanks to Dylan Mcculloch.