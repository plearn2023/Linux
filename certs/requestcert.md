# SSL Certificate for Lighttpd

This page is about how to request a certificate from letsencrypt for the linux server hosting pihole (https://pi-hole.net/)

Bit simpler than I could think

## Request certificate

Let's talk about this.
Below are the preparation steps

```sh
sudo apt update && sudo apt upgrade -y
sudo apt install snapd

# check the latest version of snap is installed again
sudo snap install core ; sudo snap refresh core

# install certbot using snap, so remember to use snap instead apt
sudo snap install --classic certbot

# create symlink to the certbot file using the following command
ln -s /snap/bin/certbot /usr/bin/certbot

# do not know what this command does
sudo snap set certbot trust-plugin-with-root=ok

# install cloudflare plug-in, as I have my domain hosted on Cloudflare
sudo snap install certbot-dns-cloudflare

# create a restricted API on cloudflare with Zone:DNS:Edit permissions only
# create a file example secrets.ini and stored is somewhere. The contents of secret.ini file should be
#----------#
# Cloudflare API token used by Certbot
dns_cloudflare_api_token = 0123456789abcdef0123456789abcdef01234567
#----------#
# then run the following command, replace you certicate with '*.idnspmz.uk'
# mention full file path the cloudflare.api. This file has the api token.
sudo certbot certonly \
--dns-cloudflare \
--dns-cloudflare-credentials cloudflare.api \
-d *.idnspmz.uk

# if something fails in above command repeat below commands
sudo snap set certbot trust-plugin-with-root=ok
sudo snap install certbot-dns-cloudflare
```
### Output
Here is little explaination 

Certificate is saved at: /etc/letsencrypt/live/idnspmz.uk/fullchain.pem
Key is saved at: /etc/letsencrypt/live/idnspmz.uk/privkey.pem
These files will be updated when the certificate renews.
Certbot has set up a scheduled task to automatically renew this certificate in the background. (this not working at yet?)

### Adding certificate to the webserver
Text for SSL enablement is really simple but there is something missing in the inbuilt package of the lighttpd. 
First lets rename the files as per your requirements. I wish to keep things bit simple. I changed the name
Rename `fullchain.pem` to `idnspmzuk.pem` and renamed `privkey.pem` to `idnspmzukpriv.pem`
Second, open the file `/etc/lighttpd/lighttpd.conf`

```sh
# paste the below line at the end
server.modules += ("mod_openssl")
$SERVER["socket"] == ":443" {
    ssl.engine = "enable"
    ssl.pemfile = "/etc/lighttpd/ssl/idnspmzuk.pem"
    ssl.privkey = "/etc/lighttpd/ssl/idnspmzukpriv.pem"
}
```
Please note `ssl` directory might not be there. You should create it.

#### Additional Step
This is not going to work because it needs openssl.module. 

```sh
# Steps to install SSL Module
apt install lighttpd-mod-openssl
systemctl restart lighttpd.service
```

# Renew a wildcard certificate from Letsencrypt.
Renewal is bit easy but i was unsure when i renewed it today. When I ran certbot
```sh
sudo certbot renew
```
But failed for some reason. I think it needs root permissions? cloudflared secret file (cloudflare.api) was not found.
But either ways you have to copy the file from `/etc/letsencrypt/live/idnspmz.uk/` directory and rename them and copy them to `/etc/lighttpd/ssl/`

## Additional Information

How to find which certificates are managed by certbot?

```sh
preetam@nspak:~$ sudo certbot certificates
Saving debug log to /var/log/letsencrypt/letsencrypt.log

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
# Found the following certs:
#  xxxx output is removed xxxx
```
