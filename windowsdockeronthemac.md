Objective
=========
Run Windows Containers on the Mac. Not just the docker client.

Ingredients
===========
* [VMware Fusion 8.0.2](http://www.vmware.com/products/fusion/features.html)
* [Windows Server 2016 TP 4](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-technical-preview)
* [Docker Toolbox for Mac](https://www.docker.com/docker-toolbox)

What Success Looks Like
=======================

![Docker client](/clientconfigured.png)
![Running cmd.exe](/cmdexe.png)

Approach
========
![Architecture](/arch.png)

Steps
=====

1.  Install VMware Fusion and Docker Toolbox on Mac.

    In Fusion, Preferences > Network > Uncheck "Require authentication to enter promiscuous mode"
    
1. Create new Virtual Machine in Fusion with the following settings

    * Settings > Processors and Memory > 2 cores
    * Settings > Processors and Memory > 4096 MB
    * Settings > Processors and Memory > Advanced > Enable hypervisor applications
    * Settings > Network > Private to my Mac
    
1. Install Windows Server 2016 TP4 in the VM.

1. Install the Docker container feature using the [instructions and script](https://msdn.microsoft.com/en-us/virtualization/windowscontainers/quick_start/container_setup).

1. Check the inner VM (Hyper-V guest) has started and is running

![Hyper-V](/hyperv.png)

1. Enable the inner VM for [Remote powershell](https://technet.microsoft.com/en-us/magazine/ff700227.aspx):

    ```winrm s winrm/config/client '@{TrustedHosts="192.168.177.129"}'```

7. On the inner container host VM:

  * Stop the docker service using PS:
  
      ```Stop-Service docker```
      
  * [Open port](http://superuser.com/questions/842698/how-to-open-a-firewall-port-in-windows-using-power-shell) 2375 in the Hyper-V internal network
  
    ```netsh advfirewall firewall add rule name="Open Port 2375" dir=in action=allow protocol=TCP localport=2375```
  
  * Run the docker daemon manually:
  
    ```docker daemon -b "Virtual Switch" -H 0.0.0.0:2375```

8.1. On the Mac:

* Set remote docker daemon:

    ```export DOCKER_HOST=tcp://<innerVMip>:2375```
    ```docker version```

* Launch a container from the built-in windowsservercore image:

    ```docker run --name test -it windowsservercore cmd```

Shortcomings
============
1. Performance is not ideal.
1. Nesting hypervisors from different vendors is definitely #unsupported
1. Insecure - TLS not enabled on host (but limited to host only network)
1. Need to explicitly open firewall ports using PowerShell

PRs and feedback welcome!
