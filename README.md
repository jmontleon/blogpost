VNC Desktop with Automation Broker
==================================

Introduction
------------



VNC Server
----------



noVNC
-----
This container was very easy to create thanks to a launch script that was recently added. Aside from installing dependency packages all we needed to do is clone the git repository, link the vnc.html file to index.html and set the entrypoint to start the script.

```
RUN git clone https://github.com/novnc/novnc
RUN ln -sf /novnc/vnc.html /novnc/index.html
ENTRYPOINT /novnc/utils/launch.sh --vnc vnc-desktop:5901
```

APB
---




Code Links
----------
[VNC Desktop APB](https://github.com/ansibleplaybookbundle/vnc-desktop-apb)  
[Fedora 28 Container](https://github.com/fusor/dockerfiles/tree/master/vnc-desktop:f28)  
[Fedora 27 Container](https://github.com/fusor/dockerfiles/tree/master/vnc-desktop:f27)  
[noVNC Container](https://github.com/fusor/dockerfiles/tree/master/vnc-client:latest)  
