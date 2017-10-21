## prerequisties
1. Enable user access on ubuntu environment 
1. Enable RDP port on Security Group(3389)

## Using gnome for Ubuntu Version > 12.4LTS

 Install GNome Desktop

```
sudo apt-get update
sudo apt-get install ubuntu-desktop
```
 Install xrdp 
```
sudo apt-get install xrdp
```

 Start xrdp and set xdrp service at boot-up
```
sudo systemctl start xrdp
sudo systemctl enable xrdp
```

## Using xfce for Ubuntu Version > 12.4LTS

 Install xfce using this command
```
sudo apt-get install xubuntu-desktop
```

Then enable xfce using this command:

```
echo xfce4-session >~/.xsession
```

Edit the config file /etc/xrdp/startwm.sh:
```
sudo vi /etc/xrdp/startwm.sh 
```


Add the line xfce4-session before the line /etc/X11/Xsession.2
To restart the xrdp service, use this:
```
sudo service xrdp restart
```