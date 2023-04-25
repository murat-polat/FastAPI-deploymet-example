### This  is a simple example  for deploying a FastAPI application, to production environment. And probably the easiest way to run an application with secure SSL/HTTPS, and with Caddy server magic. So, you can focus just your application and code, not deployment process :)




### What we need ?

- Linux machine/VM 
- Domainname
- FastAPI application
- Caddy server and reverse proxy configuration for HTTPS/SSL

### Linux VM :
You can order a Linux VM from the Digital Ocean, Contabo, Vultr, Linode etc. For this tutorial, we choose an Debian instance on Contabo.
After  first login, run:

`sudo apt-get update`

If sudo not working on Debian:

`apt install sudo`

It's optional add a new user, or continue with "root" user. But we choose and recommend a new user, and give it admin privileges.

`sudo adduser <newuser>`

`sudo usermod -aG  <newuser>`

Change root user to new user

`su <newuser>`


`python3 -m pip install fastapi[all] gunicorn uvicorn`

If "pip" not working, so install python3 pip

`sudo apt install python3-pip`


### Domain configuration

We need a domain name for publishing our application in World Wide Web. We choose a domainname from the https://www.namecheap.com. And we must tell the domain provider which server/IP provider will be used for publishing. Our FastAPI application will be served on Contabo. From  domain list => management => Nameservers => Custom DNS add three nameservers (ns1.contabo.net, ns2.contabo.net, ns3.contabo.net) and save.

![](/src/custom_DNS.png)

From Contabo "DNS Zone Management", add your domainname to "Domain", and  "Target IP address" select your VM instance/IP from dropdown menu => then click "create zone".

![](/src/DNS_Zone_mgmt.png)

### FastAPI application
 As we mentioned above that, this is just a demo application.  For demonstration perposes

 `nano main.py`

 ```
from fastapi import FastAPI

app = FastAPI()


@app.get("/items/{item_id}")
async def read_item(item_id: int):
    return {"item_id": item_id}
 
  
 ```
For testing just run:

`uvicorn main:app --reload`

If it's running without error, so vi need make a "service" or "supervisor" for our FastAPI application. The reason for that, to do application more robust and relable for pruduction. After that you can start, stop and see status just from the command line.

To do that run:

`sudo nano /etc/systemd/system/fastapi.service`

and copy-paste the code block below

```

[Unit]
Description=fastapi service
After=network.target

[Service]
User=<newuser>
Group=www-data
WorkingDirectory=/home/<newuser>/fastapi
ExecStart=gunicorn main:app --workers 4 --worker-class uvicorn.workers.UvicornWorker --bind 0.0.0.0:8000

Restart=always

[Install]
WantedBy=multi-user.target

```

Hit "Ctrl+X" then "Y" for save the "fastapi.service" (For Nano editor)

Now our FastAPI application more robust and easy to control. For start and enable the service, run:

` sudo systemctl start  fastapi`

To enable the service:

` sudo systemctl enable  fastapi`

To stop the service:

` sudo systemctl stop  fastapi`

To see status of the service:


` sudo systemctl status  fastapi`

```
● fastapi.service - fastapi service
     Loaded: loaded (/etc/systemd/system/fastapi.service; enabled; vendor preset: enabled)
     Active: active (running) since Sat 2023-04-22 21:52:48 CEST; 15h ago
   Main PID: 333920 (gunicorn)
      Tasks: 5 (limit: 9507)
     Memory: 82.6M
        CPU: 8min 26.889s
     CGroup: /system.slice/fastapi.service
             ├─333920 /usr/bin/python3 /usr/bin/gunicorn main:app --workers 4 --worker-class uvicorn.workers.UvicornWorker --bind 0.0.0.0:8000
             ├─333921 /usr/bin/python3 /usr/bin/gunicorn main:app --workers 4 --worker-class uvicorn.workers.UvicornWorker --bind 0.0.0.0:8000
             ├─333922 /usr/bin/python3 /usr/bin/gunicorn main:app --workers 4 --worker-class uvicorn.workers.UvicornWorker --bind 0.0.0.0:8000
             ├─333923 /usr/bin/python3 /usr/bin/gunicorn main:app --workers 4 --worker-class uvicorn.workers.UvicornWorker --bind 0.0.0.0:8000
             └─333924 /usr/bin/python3 /usr/bin/gunicorn main:app --workers 4 --worker-class uvicorn.workers.UvicornWorker --bind 0.0.0.0:8000

Apr 22 21:52:48 vmi1180878.contaboserver.net gunicorn[333924]: [2023-04-22 21:52:48 +0200] [333924] [INFO] Application startup complete.
Apr 22 22:03:18 vmi1180878.contaboserver.net gunicorn[333923]: [2023-04-22 22:03:18 +0200] [333923] [WARNING] Invalid HTTP request received.

```

Now everything is working properly with HTTP on port 8000. But we need SSL/HTTPS,and configure Caddy server for reverse proxy.

### Caddy Server configuration
#### Installation:

https://caddyserver.com/docs/install#debian-ubuntu-raspbian 

`sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https`


`curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg`

`curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list`

`sudo apt update`

`sudo apt install caddy`

#### Configuration reverse proxy with Caddyfile

`sudo nano /etc/caddy/Caddyfile`

```
yourdomainname.net {

        reverse_proxy  localhost:8000

}

```

`sudo systemctl reload caddy`


Done ! 

Happy coding with awasome FastAPI :)

![](/src/browser.png)



