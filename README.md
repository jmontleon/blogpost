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

![Image of Distributions](https://github.com/jmontleon/blogpost/blob/master/distributions.png)


```
    - name: de
      title: Desktop Environment
      type: enum
      enum: ['fvwm', 'i3', 'KDE', 'LXDE', 'LXQt', 'MATE', 'Sugar', 'twm', 'Xfce']
      default: 'Xfce'
      updatable: true
    - name: deshell
      title: Shell
      type: enum
      enum: ['bash', 'csh', 'ksh', 'sh']
      default: 'bash'
      updatable: true
    - name: resolution
      title: Desktop Resolution
      type: enum
      enum: ['800x600', '1024x768', '1280x1024', '1360x768', '1440x900', '1920x1080']
      default: '1360x768'
updatable: true
```


![Image of Parameters](https://github.com/jmontleon/blogpost/blob/master/parameters.png)

Updating Your Choices
---------------------


Code Links
----------
[VNC Desktop APB](https://github.com/ansibleplaybookbundle/vnc-desktop-apb)  
[Fedora 28 Container](https://github.com/fusor/dockerfiles/tree/master/vnc-desktop:f28)  
[Fedora 27 Container](https://github.com/fusor/dockerfiles/tree/master/vnc-desktop:f27)  
[noVNC Container](https://github.com/fusor/dockerfiles/tree/master/vnc-client:latest)  
