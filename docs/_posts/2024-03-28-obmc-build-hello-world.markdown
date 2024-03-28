---
layout: post
title:  "[OpenBMC] Build First Image and Modify Repo Using devtool"
date:   2024-03-28
categories: openbmc
tags: openbmc, bmc
---

# [OpenBMC] Build First Image and Modify Repo Using devtool
---

## Build Image

```bash
git clone https://github.com/openbmc/openbmc.git
cd openbmc
. setup romulus
bitbake obmc-phosphor-image
```

## [Download and Start QEMU Session](https://github.com/openbmc/docs/blob/master/development/devtool-hello-world.md)

### Get QEMU
**Note**
- The 'user' networking backend is provided by the 'slirp' library; you get this message when the QEMU binary was built without slirp support compiled in. (Maydell)

- As noted in the 7.2 changelog, QEMU no longer ships a copy of the slirp module with its sources. Instead you need to make sure you have installed your distro's libslirp development package (which is probably called libslirp-devel or libslirp-dev or something similar) before configuring and building QEMU. You need at least libslirp 4.7 or better. (Maydell)
- So we use the QEMU provided by openbmc and don't use the upstream QEMU. If you use the QEMU installed with `apt get`, you will get the error `network backend 'user' is not compiled into this binary`.

```bash
wget https://jenkins.openbmc.org/job/latest-qemu-x86/lastSuccessfulBuild/artifact/qemu/build/qemu-system-arm
chmod u+x qemu-system-arm
```

### Copy Image
```bash
cp ./tmp/deploy/images/romulus/obmc-phosphor-image-romulus.static.mtd ./
```

### Start QEMU session with Romulus Image
**Note** 
- For REST, SSH and IPMI to work into your QEMU session, you must connect up some host ports to the REST, SSH and IPMI ports in your QEMU session. In this example, it just uses 2222, 2443, 2623. You can use whatever you prefer.
```bash
./qemu-system-arm -m 256 -M romulus-bmc -nographic \
    -drive file=./obmc-phosphor-image-romulus.static.mtd,format=raw,if=mtd \
    -net nic \
    -net user,hostfwd=:127.0.0.1:2222-:22,hostfwd=:127.0.0.1:2443-:443,hostfwd=udp:127.0.0.1:2623-:623,hostname=qemu
```

### Check Status
```
# In obmc
obmcutil state
```

### Check ssh
```bash
sshpass -p 0penBmc ssh root@172.0.0.1 -p 2222
```

### Check webui

Open the browser and browse `http://127.0.0.1:2443`

### Check IPMI
```bash
ipmitool -I lanplus -p 2623 -H 127.0.0.1 -U root -P 0penBmc -C17 mc info
```
**Note** 
- You need to designate ciphersuite 17 by adding option `-C17` for openbmc, which is assigned in [cipher_list.json](https://github.com/openbmc/openbmc/blob/f11e65566a8e3ce4f413d954c9df2c03b58ca4e6/meta-phosphor/recipes-phosphor/ipmi/phosphor-ipmi-config/cipher_list.json). The default for ipmitool is 3 which specifies RAKP-HMAC-SHA1 authentication, HMAC-SHA1-96 integrity, and AES-CBC-128 encryption algorightms. ciphersuite 3 is removed by [this commit](https://github.com/openbmc/openbmc/commit/a95e4a952c182380b98edcd8d4f615faabb8af95) because HMAC-SHA1 is deprecated. 

Table 22-19 in the [IPMIv2 spec](https://www.intel.com.tw/content/www/tw/zh/products/docs/servers/ipmi/ipmi-second-gen-interface-spec-v2-rev1-1.html)
![ipmi_table_22_19](https://hackmd.io/_uploads/B1X0IifJC.jpg) 

### Exit QEMU Session
press `Ctrl+a` and `x`

## [Clone a Repo and modify it Locally using devtool](https://github.com/openbmc/docs/blob/master/development/devtool-hello-world.md)

### Method One: Clone, Modify, and Rebuild the Whole Image

#### 1. Use devtool to Extract Source Code 
```bash
. setup romulus
devtool modify phosphor-state-manager
```

#### 2. Add Hello World
```bash
vi workspace/sources/phosphor-state-manager/bmc_state_manager_main.cpp
```

diff should look like this
```clike
+#include <iostream>

int main(int argc, char**)
{
@@ -17,6 +18,8 @@ int main(int argc, char**)

     bus.request_name(BMC_BUSNAME);

+    std::cout<<"Hello World" <<std::endl;
+
     while (true)
     {
```

#### 3. Rebuild the Image
```bash
bitbake obmc-phosphor-image
```

#### 4. Verify
Start the QEMU session and use journalctl to check the hello world is added.
```bash
journalctl | grep "Hello World"
```

Should see something like this
```bash
<date> romulus phosphor-bmc-state-manager[1089]: Hello World
```

### Method Two: Rebuild the Repo Only and Load it Directly into a Running QEMU

#### 1. Repeat Step 1, 2 of Method One

#### 2. Bitbake only the phosphor-state-manger Repo
```bash
bitbake phosphor-state-manager
```
Your new binary will be located at the following location relative to your bitbake directory: ./workspace/sources/phosphor-state-manager/oe-workdir/package/usr/bin/phosphor-bmc-state-manager

#### 3. Create a Safe File System for Your Application
```bash
# in obmc
mkdir -p /tmp/persist/usr
mkdir -p /tmp/persist/work/usr
mount -t overlay -o lowerdir=/usr,upperdir=/tmp/persist/usr,workdir=/tmp/persist/work/usr overlay /usr
```

#### 4. Scp the New Binary onto QEMU Instance
```bash
scp -P 2222 ./workspace/sources/phosphor-state-manager/oe-workdir/package/usr/bin/phosphor-bmc-state-manager root@127.0.0.1:/usr/bin/
```

#### 5. Verify
```bash
# in obmc
systemctl restart xyz.openbmc_project.State.BMC.service
journalctl | tail
```

## References
[OpenBMC Development Environment](https://github.com/openbmc/docs/blob/master/development/dev-environment.md)
[OpenBMC Hello World using devtool](https://github.com/openbmc/docs/blob/master/development/devtool-hello-world.md)
[IPMIv2 spec](https://www.intel.com.tw/content/www/tw/zh/products/docs/servers/ipmi/ipmi-second-gen-interface-spec-v2-rev1-1.html)
[stack overflow: network backend 'user' is not compiled into this binary](https://stackoverflow.com/questions/75641274/network-backend-user-is-not-compiled-into-this-binary)