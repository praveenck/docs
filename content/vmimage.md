Setting up Magento 2 Extension Model Alpha 1 VM Image
=====================================================

This document describes the steps to download and install a VirtualBox image containing the Magento 2 Extension Model Improvements (Alpha 1). For an overview of the proposed improvements, please refer to the documentation [here][1].

[1]: <http://magento.github.io/magento2-developer-docs/>

Download
--------

Download the ova file by clicking on the link below and save it to the disk. If you do not have virtual box installed, please refer to [this link][3] for download and installation instructions.

[Magento 2 Extension Model Improvements (Alpha 1)][2]

[2]: <https://ebay.box.com/s/a2k6twcsoappu4zv5lsq>
[3]: <https://www.virtualbox.org/wiki/Downloads>

Starting VM
--------

Start the virtual box application. Import the VM image by clicking on File->Import Appliance menu option. Click on "Open Appliance" button and browse to the location where you saved the .ova file. Select the ova file and click open. Continue to the next screen and click on import. Once the import is complete, the virtual appliance is displayed in the list of appliances on the left sidebar as magento2_2.

To start the VM, select the magento2_2 VM and click on the start button. Once it comes up you will be presented with a gui login. You can use this or you can ssh directly into the machine via PuTTY or similar tool.

### Edit your hosts file

Be sure to edit your local hosts file to add an entry for this vm as shown below. You will be able to hit this http://magento.local url after this in your local machine's browser to finish the magento install.

<pre>
192.168.56.101 magento.local
</pre>

Login details
--------

The ip address of the VM is <b>192.168.56.101</b> and below logins will work via ssh.

### Linux User Accounts

| Username | Password | Has Root Access | Login via GUI |
|:--------|:-------|:--------|:--------|
| mage | 123456 | via sudo | yes |
| root | 123456 | yes | no |



### Mysql Accounts

You can use mysql workbench to connect to the mysql server, just specify the ip address of the vm and one of the below accounts.

| Username | Password | Remote Login | Database |
|:--------|:-------|:--------:|:--------|
| magento2 | magento2 | yes | magento2 |
| root | 123456 | yes | All Dbs |



### Magento Admin Account

| Username | Password |
|:--------|:-------|
| magento2 | magento2 |

Magento2 install
--------

Magento is already installed at /var/mage/magento2. A snapshot of magento2 code containing the Extension Model imrpovements is already there. To access Magento Frontend use http://magento.local and for Magento admin (backend), use http://magento.local/index.php/backend/admin. There already is a mysql server running on the vm with a magento2 database.
