# oai_lab

OAI-R14 version
 
Configuration:

/usr/local/etc/oai/spgw.conf

/usr/local/etc/oai/mme.conf

/usr/local/etc/oai/hss.conf

# OAI(vR14) Installation
### Following
https://open-cells.com/index.php/2019/09/22/all-in-one-openairinterface/

This document explains how to install and configure OAI EPC+eNB on one single Ubuntu 18.04 64 bits machine connected with a regular UE, routing the UE traffic to internet.

The description uses a USRP B210 board.

* install git and configure your identification in git:
```shell=
sudo apt install git 
git config --global user.name "cly1213"
git config --global user.email "cly901213@gmail"
```
* Add the OAI repository as authorized remote system
```shell=
echo -n | openssl s_client -showcerts -connect gitlab.eurecom.fr:443 2>/dev/null | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' | sudo tee -a /etc/ssl/certs/ca-certificates.crt
```
