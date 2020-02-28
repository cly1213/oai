# oai_lab

OAI-R14 version
 
Configuration:

/usr/local/etc/oai/spgw.conf

/usr/local/etc/oai/mme.conf

/usr/local/etc/oai/hss.conf

# OAI-R14 version Installation
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
## Install Ubuntu
Download Ubuntu 18.04 64 bits version iso file
```shell=
$ lsb_release -a
```
```shell=
$ uname -a
```
### Using your package manager (recommand)
Most distributions provide UHD as part of their package management. On Debian and Ubuntu systems, this will install the base UHD library, all headers and build-specific files, as well as utilities:
```shell=
sudo apt-get install libuhd-dev libuhd003 uhd-host
```
Copy and paste these commands into your terminal. This will install UHD software as well as allow you to receive package updates.
```shell=
sudo add-apt-repository ppa:ettusresearch/uhd
sudo apt-get update
sudo apt-get install libuhd-dev libuhd003 uhd-host
```
```shell=
root@e3464901f7b4:~# uhd_find_devices
[INFO] [UHD] linux; GNU C++ version 7.4.0; Boost_106501; UHD_3.14.1.1-release
[WARNING] [B200] EnvironmentError: IOError: Could not find path for image: usrp_b200_fw.hex

Using images directory: <no images directory located>

Set the environment variable 'UHD_IMAGES_DIR' appropriately or follow the below instructions to download the images package.

Please run:

 "/usr/lib/uhd/utils/uhd_images_downloader.py"
No UHD Devices Found
root@e3464901f7b4:~#
```
```shell=
$ sudo /usr/lib/uhd/utils/uhd_images_downloader.py
```
## Download and patch EPC
Clone OAI EPC:
```shell=
Download and extract the data:
#go back to home directory
cd ~ 
git clone https://gitlab.eurecom.fr/oai/openair-cn.git
cd openair-cn
git checkout develop
```

```shell=
cd ~
wget https://raw.githubusercontent.com/cly1213/oai/master/opencells-mods-20190923.tgz
tar xf opencells-mods-20190923.tgz
```

or

```shell=
cd ~
wget https://raw.githubusercontent.com/cly1213/oai/master/openair-cn.tgz
tar xf openair-cn.tgz
cd openair-cn
git checkout develop
```
* We tested with commit: 724542d0b59797b010af8c5df15af7f669c1e838

Apply the patch:
```shell=
git apply ~/opencells-mods/EPC.patch
```
## Install third party SW for EPC
* Install hss
```shell=
cd openair-cn; source oaienv; cd scripts
./build_hss -i
```
For ubuntu 18.04, we set back the legacy mysql security level=>
因為MySQL會有帳號權限有安全上的限制，若一開始使用會出現以下訊息
:::danger
’Access denied for user ‘root’@‘localhost’ (using password: YES)’

or

Access denied for user 'leo'@'localhost' (using password: YES)’
:::

:::success
[solved 1]:

$ sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf

加入這行
skip-grant-tables

$ /etc/init.d/mysql restart

重啟後任何用戶名就可以以root的身份登錄MySQL了。

$ mysql -u root -p
mysql> USE mysql;
mysql> UPDATE user SET plugin='mysql_native_password' WHERE User='root';
mysql> FLUSH PRIVILEGES;
mysql> quit

結束後再刪除在mysqld.cnf所添加的內容，並重啟服務。
$ sudo systemctl restart mysql.service
:::

The last command will ask a few questions:
:::info
password: set your password (linux is set in our default config files)
VALIDATE PASSWORD PLUGIN: no
Remove anonymous users: yes
Disallow root login remotely: yes
Remove test database and access to it: yes
Reload privilege tables now: yes
:::





* Answer yes to install: freeDiameter 1.2.0
  * phpmyadmin:
    * We don’t use phpmyadmin later in this procedure to update the MySQL database
    * We removed the installation of phpmyadmin (of course you can use it if you prefer)




* Install 3GPP SW for mme and spgw
```shell=
./build_mme -i
```
Do you want to install freeDiameter 1.2.0: no
Do you want to install asn1c rev 1516 patched? <y/N>: yes
Do you want to install libgtpnl ? <y/N>: yes
wireshark permissions: as you prefer
```shell=
./build_spgw -i
```
Do you want to install libgtpnl ? <y/N>: no

## Compile the EPC nodes
```shell=
cd openair-cn; source oaienv; cd scripts
./build_hss
./build_mme
./build_spgw
```
If you face compilation issues, the log files are in openair-cn/build/log
In there files, look for “error:” string.

## Download & Compile the eNB on 18.04
```shell=
git clone https://gitlab.eurecom.fr/oai/openairinterface5g.git
cd openairinterface5g
git checkout develop
```

* We tested with commit edb74831dabf79686eb5a92fbf8fc06e6b267d35
此版本為R14，若直接clone會是最新的目前已更新到R15，會編譯不出所要的module(待觀察解決更新前後的差異)。 [date]:20200215

* Build in two steps

```shell=
$ source oaienv  
$ ./cmake_targets/build_oai -I  # install SW packages from internet
$ ./cmake_targets/build_oai -w USRP --eNB --UE # compile eNB and UE
```
可不用編UE(optional)

## Our Network setup description

I’ve made a simple configuration for this all-in-one setup.

Each node is on a separate IP address, this address is used for all it’s interfaces. In our case of all-in-one, we take addresses on the loopback: this will be fine on all your machines.

    HSS is on localhost: 127.0.0.1
    eNB is on 127.0.0.10
    MME is on 127.0.0.20
    SPGW is on 127.0.0.30

The LTE diameter configuration is now isolated from Linux hostname.

realm for our EPC: “OpenAir5G.Alliance”, so, full distinguish names (FQDN) are: hss.OpenAir5G.Alliance, mme.OpenAir5G.Alliance

## Install this configuration for eNB

In your eNB configuration file, the network is now fixed, as lo interface always exists and our computer internal addresses also:
```c=
////////// MME parameters:
 mme_ip_address = ( { ipv4 = "127.0.0.20";
 ipv6 = "192:168:30::17";
 active = "yes";
 preference = "ipv4";
 }
 );

NETWORK_INTERFACES : 
 {
 ENB_INTERFACE_NAME_FOR_S1_MME = "lo";
 ENB_IPV4_ADDRESS_FOR_S1_MME = "127.0.0.10/8";

 ENB_INTERFACE_NAME_FOR_S1U = "lo";
 ENB_IPV4_ADDRESS_FOR_S1U = "127.0.0.10/8";
 ENB_PORT_FOR_S1U = 2152; # Spec 2152
 };
```
 
 In the eNB config file, you need also to set the MCC and MNC as per your SIM card:
:::info
tracking_area_code = “1”;
mobile_country_code = “466”;
mobile_network_code = “07”;
:::

And obviously, your radio parameters.

We tested with USRP B210 20MHz band, Google pixel 4 UE, a cavity duplexer a simple antenna, about 1 meter distance UE/eNB antenna with this file: ~/opencells-mods/enb.10MHz.b200

if you use the OpenAir UE, a sim card file that match our hss database example: opencells-mods/sim.conf. We will make another tutorial to use together OpenAir UE and rf board simulation

## Install this configuration for EPC
For the EPC, we install in OAI default directory: /usr/local/etc/oai
```shell=
sudo mkdir -p /usr/local/etc/oai
sudo cp -rp ~/opencells-mods/config_epc/* /usr/local/etc/oai
```
```shell=
cd openair-cn; source oaienv; cd scripts
./check_hss_s6a_certificate /usr/local/etc/oai/freeDiameter hss.OpenAir5G.Alliance
./check_mme_s6a_certificate /usr/local/etc/oai/freeDiameter mme.OpenAir5G.Alliance
```

Only the SGi output to internet need to be configured.
In /usr/local/etc/oai/spgw.conf,
your should set the Ethernet interface that is connected to Internet, and,
to tell to the PGW to implement NAPT for the UE traffic
```c=
PGW_INTERFACE_NAME_FOR_SGI = "eno2"; 
PGW_MASQUERADE_SGI = "yes";
```

竟量避免ip過於相近干擾（實驗過程有遇過，會可能會導致routing出錯）
```c=
#Pool of UE assigned IP 
IP_ADDRESS_POOL :
{
    IPV4_LIST = (
        "192.168.0.0/24" # STRING, CIDR, YOUR NETWORK CONFIG HERE.
                );
};
```
For the SIM card, you’ll have more to do:

* SIM MCC/MNC should be duplicated in a couple of files
  * eNB: See above in eNB configuration chapter
  * MME file: /usr/local/etc/oai/mme.conf to update

:::info
GUMMEI_LIST = ( MCC="466" ; MNC="07"; MME_GID="4" ; MME_CODE="1"; } );
TAI_LIST = ({MCC="466" ; MNC="07"; TAC = "1"; } );
:::

#### HSS
* Configure the password for MySQL
  * in /usr/local/etc/oai/hss.conf, set password as the password you created during MySQL installation
* A HSS database in text is in: ~/opencells-mods/opencells_db.sql
  * We don’t use phpmyadmin: we load the database from a ascii file
  * It is pre-configured with the
    * mme id
    * 10 users is network 466/07 (a Taiwan test network) are also created (don’t use 3GPP test network: 001/01: the mme fails when MCC starts by “0”)
  * Each time you import this db, it erases the entire database
        (example: you set mysql password to “linux”)

        ~/opencells-mods/hss_import 127.0.0.1 root linux oai_db ~/opencells-mods/opencells_db.sql



## Final test and verification
open 4 terminal windows

```shell=
$ cd openair-cn; source oaienv; cd scripts; ./run_hss
```

```shell=
$ cd openair-cn; source oaienv; cd scripts; ./run_mme
```

```shell=
$ cd openair-cn; source oaienv; cd scripts; sudo -E ./run_spgw
```

```shell=
$ sudo bash
$ cd ~/openairinterface5g; source oaienv
$ cd cmake_targets/lte_build_oai/build
$ ./lte-softmodem -O ~/opencells-mods/enb.10MHz.b200
```

Connect the UE, it should attach to network and be able to reach internet through OAI network.

If the UE attaches, but you don’t have internet access, verify phone configuration: enable data in config->sim and verify the APN value

記得到手機設定APN就可以實驗上網的功能服務了～ Enjoy it! 

#### 20200228  Chen Li Ying edited/uploaded.
