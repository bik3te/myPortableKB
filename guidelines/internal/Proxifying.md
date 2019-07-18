## Burp (VM NAT)
- Host :
```
Listener on the real network interface
User Options / Upstream Proxy Servers
```
- VM, see above

## SSH
- VM :
```
# apt install ssh
# ssh-keygen
# systemctl start sshd.service
```
- Host :
```
Network settings / Port Forwarding
Name --> SSH
Protocol --> TCP
Host Port --> 2022
Guest Port --> 22
Filezilla sftp://127.0.0.1:2022
```

## GIT
```
git config --global http.proxy http://<ip>:<port>
git config --global http.sslVerify false
git config --global https.proxy http://<ip>:<port>
git config --global https.sslVerify false
```

## PIP
- /etc/pip.conf :
```
[global]
proxy = <ip_or_proxy.tld>:<port>
trusted-host = pypi.python.org
               pypi.org
               files.pythonhosted.org
```
- Loading :
```
# export PIP_CONFIG_FILE=/etc/pip.conf
```

## APT
- /etc/apt/apt.conf.d/proxy.conf :
```
Acquire::http::Proxy "http://<user>:<passwd>@proxy.tld:<port>/";
Acquire::https::Proxy "http://<user>:<passwd>@proxy.tld:<port>/";
```

## Curl / Wget
```
# export all_proxy=http://<user>:<passwd>@proxy.tld:<port>
# export http_proxy=http://<user>:<passwd>@proxy.tld:<port>
# export https_proxy=http://<user>:<passwd>@proxy.tld:<port>
# export ftp_proxy=http://<user>:<passwd>@proxy.tld:<port>
```
