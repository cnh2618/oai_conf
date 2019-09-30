# oai_conf
OAI Auto-start

******************************************************************
**		   Ubuntu 16.04.06 and USRP B210	 		**
**			    EPC+eNB				**
******************************************************************

1. Install USRP Driver

#pacakge install
sudo apt-get install libboost-all-dev libusb-1.0-0-dev python-mako doxygen python-docutils python-requests cmake build-essential

#download usrp driver
git clone git://github.com/EttusResearch/uhd.git

#drive build
cd uhd; mkdir host/build; cd host/build
cmake -DCMAKE_INSTALL_PREFIX=/usr ..
make -j4

#driver install
sudo make install
sudo ldconfig

#firmware download
sudo /usr/lib/uhd/utils/uhd_images_downloader.py



2. Download Patch eNB and others

#download patch zip file
download url: open-cells.com/d5138782a8739209ec5760865b1e53b0/opencells-mods-20170710.tgz 

#extract patch zip file
tar xf opencells-mods-20170710.tgz



3. Download & Compile the eNB

##download 
git clone https://gitlab.eurecom.fr/oai/openairinterface5g.git
cd openairinterface5g
git checkout develop
git checkout 08b8b3142df16831396a5283a015564ff56bf91c

##apply patch
git apply opencells-mods/eNB.patch

##compile eNB
source oaienv  

# install deps SW packages from internet
./cmake_targets/build_oai -I       
#when build this project, we could meet the 'error about /opt/ssh'
#so we copy ssh folder to /opt and then some should be changed in build_helper file. 

#other errors occurs, just we couldn't compile protobuf package
#so i checkout oai_project to 'develop', and retry step 3.


#build eNB and UE (we didn't need ue). 
./cmake_targets/build_oai  -w USRP --eNB --UE # compile eNB




4. Download and patch EPC

##download
git clone https://gitlab.eurecom.fr/oai/openair-cn.git
cd openair-cn
git checkout develop
git checkout 5353b606662a6ae7eaadf6bc39cbf6452d298b0e

##Apply the patch
#git apply opencells-mods/EPC.patch 	#don't use

##Install third party SW for EPC
cd openair-cn; source oaienv; cd scripts
./build_hss -i
    Do you want to install freeDiameter 1.2.0 ?: yes

./build_mme -i
    Do you want to install freeDiameter 1.2.0: y
    Do you want to install asn1c rev 1516 patched? <y/N>: y
    Do you want to install libgtpnl ? <y/N>: yes
    wireshark permissions: as you prefer

./build_spgw -i
    Do you want to install libgtpnl ? <y/N>: no

./build_hss -c
./build_mme -c
./build_spgw -c

5. Our Network setup


    HSS is on localhost: 127.0.0.1
    eNB is on 127.0.0.10
    MME is on 127.0.0.20
    SPGW is on 127.0.0.30

realm for our EPC: “OpenAir5G.Alliance”, so, full distinguish names (FQDN) are: hss.OpenAir5G.Alliance, mme.OpenAir5G.Alliance


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







