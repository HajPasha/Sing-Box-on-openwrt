# Setting Up **Sing-Box** on OpenWRT

Welcome to this guide on how to set up **Sing-Box** on OpenWRT. My goal is to make this tutorial as simple and readable as possible, enabling users of all technical backgrounds to enjoy free internet access effortlessly.

---

## Prerequisites: Installing Necessary Packages

Start by installing the required packages:

```bash
opkg update
```
```bash
opkg install kmod-inet-diag kmod-netlink-diag kmod-tun iptables-nft nano
```
Next, install **Sing-Box**:

```bash
opkg install sing-box
```

> **Note**: At the time of writing, the default installation via `opkg` installs version **1.9.7**. However, the latest stable version is **1.10.1**.

---

## (Optional) Installing the Latest Version of Sing-Box

To install the latest version, you can use one of the following methods:

### **Method 1: Manual Installation via Passwall Builds**

1. Visit the [Passwall builds SourceForge page](https://sourceforge.net/projects/openwrt-passwall-build/files/releases/).
2. Locate your OpenWRT version (e.g., **OpenWrt 23.05.5**) and click on **Packages 23.05**.  
3. Select your hardware type (e.g., **x86_64** for an old computer).  
4. Click on **passwall_packages**, then download the latest **Sing-Box IPK file**.  
5. Upload the downloaded file to OpenWRT:
   - Go to **System > Software** in the OpenWRT web interface.
   - Click **Upload Package...**, choose the downloaded file, and hit **Upload**.

Verify the installation by connecting to OpenWRT via SSH and running:

```bash
sing-box version
```
---

### **Method 2: Automatic Updates via Passwall Feeds**

1. Install the required tools:

```bash
opkg update
```
```bash
opkg install wget-ssl
```

2. Add Passwall's public key:

```bash
wget -O passwall.pub https://master.dl.sourceforge.net/project/openwrt-passwall-build/passwall.pub
opkg-key add passwall.pub
```

3. Add the Passwall feeds (Copy all of the code below and paste them:

```bash
read release arch << EOF
$(. /etc/openwrt_release ; echo ${DISTRIB_RELEASE%.*} $DISTRIB_ARCH)
EOF
for feed in passwall_luci passwall_packages passwall2; do
  echo "src/gz $feed https://master.dl.sourceforge.net/project/openwrt-passwall-build/releases/packages-$release/$arch/$feed" >> /etc/opkg/customfeeds.conf
done
```

4. Update and install **Sing-Box**:

```bash
opkg update
opkg install sing-box
```

> **Pro Tip**: Avoid installing the latest version immediately upon release, as it may contain bugs. Test on a non-critical system first.

---

## Configuring Sing-Box

The default configuration file is located at `/etc/sing-box/config.json`. Use the following commands to navigate and edit it:

```bash
cd /etc/sing-box
nano config.json
```

You’ll find a sample configuration file. For optimized settings tailored to **1.9.7** or **1.10.1**, use the provided configurations in this repository. Replace the **outbound** section with your Config.

> **caution**: I have created two configs for version `1.9.7` use the `config197.json` and for version `1.10.1` use the `config1101.json`
> **Tip**: Use tools like **VS Code** for easier configuration edits.

### Validate the Configuration

Run the following command to check the validity of your configuration:

```bash
sing-box check
```

If no errors appear, your configuration is valid. 

For configurations with custom filenames or locations:

```bash
sing-box check -c yourconfig.json
``` 
> **Tip**: -c is used when your config file is not "config.json" or it's in another directory rather than the current one you are right now!
---

## Running Sing-Box

To test your setup:

```bash
sing-box run -c yourconfig.json
```

> On the first run, Sing-Box may download necessary resources like routing files (e.g., `geosite-ir`) or UI components (e.g., **MetaCube**).

---

## Advanced Tips

### Routing Internal Sites via Sing-Box

By using the **routing** feature, you can access local websites (e.g., banking apps) without disabling the VPN. This is particularly useful when deploying Sing-Box on a network-wide scale.

### Using MetaCube for Management

MetaCube enables you to manage proxies, view logs, and switch servers without SSH access to OpenWRT.
simply open your browser and type `http://OpenWRT IP:9090` eg `http://192.168.2.1:9090` 
It may ask you to insert an EndPoint URL you just simply insert `http://OpenWRT IP:9090` eg `http://192.168.2.1:9090` then click add

---

---
### Setting Up and Managing Sing-Box as a Service

To automate **Sing-Box** as a service, we’ll edit the `sing-box` file in  `init.d` directory to manage its start, stop, and reload operations effectively. Below are the complete steps:

---

#### Step 1: Edit the `/etc/init.d/sing-box` File

Open the file using a text editor:

```bash
nano /etc/init.d/sing-box
```

Then, add  the content with the following code (be sure that you add this below the pre-existing code):

```bash
#!/bin/sh /etc/rc.common

START=99
USE_PROCD=1

#####  ONLY EDIT THIS BLOCK #####
PROG=/usr/bin/sing-box 
RES_DIR=/etc/sing-box/ # resource directory / working directory / where you store IP/domain lists
CONF=./config.json   # path to the config file, can be relative to $RES_DIR
#####  ONLY EDIT THIS BLOCK #####

start_service() {
  sleep 10 
  procd_open_instance
  procd_set_param command $PROG run -D $RES_DIR -c $CONF

  procd_set_param user root
  procd_set_param limits core="unlimited"
  procd_set_param limits nofile="1000000 1000000"
  procd_set_param stdout 1
  procd_set_param stderr 1
  procd_set_param respawn "${respawn_threshold:-3600}" "${respawn_timeout:-5}" "${respawn_retry:-5}"
  procd_close_instance
  iptables -I FORWARD -o tun+ -j ACCEPT
  echo "sing-box is started!"
}

stop_service() {
  service_stop $PROG
  iptables -D FORWARD -o tun+ -j ACCEPT
  echo "sing-box is stopped!"
}

reload_service() {
  stop
  sleep 5s
  echo "sing-box is restarted!"
  start
}
```

---

#### Step 2: Make the File Executable

Run the following command to set the file as executable:

```bash
chmod +x /etc/init.d/sing-box
```

---

#### Step 3: Add the Service to Startup

To enable the service at startup and ensure it runs automatically on system boot:

```bash
/etc/init.d/sing-box enable
```

To start the service:

```bash
/etc/init.d/sing-box start
```

---

### **Recommendations:**
1. It’s advisable to add this code below the existing content in the file instead of replacing it entirely. This minimizes potential conflicts.
2. Double-check **iptables** rules when stopping or restarting the service to avoid connectivity issues.
---

## Setting Up OpenWRT for Sing-Box

### Configure the Interface

1. Navigate to **Network > Interfaces** in OpenWRT.  
2. Create a new interface:
   - **Name**: e.g., `SingBoxTUN`
   - **Protocol**: `Unmanaged`
   - **Device**: Select the device name (e.g., `sing-tun`) specified in your Sing-Box Inbound configuration.

### Configure the Firewall

1. Go to **Network > Firewall**.  
2. Add a new firewall zone:
   - **Name**: e.g., `SingBox`
   - **Input/Output/Forward**: `Accept`
   - **Masquerading**: Enabled
   - **Covered Networks**: The interface you created earlier (e.g., `SingBoxTUN`).
   - **Allow Forward From Source Zones**: `LAN`.


---
### Configuring Devices to Use the Sing-Box and OpenWRT Setup

After setting up **Sing-Box** and configuring it on **OpenWRT**, the next step is to configure your devices (like phones and laptops) to connect to the network. Here’s how to do it:

---

#### On Android:

1. **Open Network Settings**  
   - Navigate to **Settings** > **Network and Internet**.
   ![Wi-Fi Settings Gear Icon](https://github.com/user-attachments/assets/22b639f3-1a2d-4792-bae3-cf624237d87e)

2. **Locate Current Wi-Fi Network**  
   - Tap on the gear icon next to the Wi-Fi network you are connected to.
   ![IP Address Display](https://github.com/user-attachments/assets/1c97790a-588b-45fd-b0a5-dc4ba0741ca9)

3. **Find Your Device’s IP Address**  
   - On the Wi-Fi settings page, look for an option named **IP Address**.  
     Take note of this IP address; you’ll need it in the next step.



4. **Edit Network Settings**  
   - Tap on the **pencil icon** to edit the network.

   ![Edit Wi-Fi Network](https://github.com/user-attachments/assets/3c6c1b9a-6c20-407c-b7c8-9d0ba4ce5337)

5. **Enable Advanced Options**  
   - Scroll down and tap **Advanced Options**.
   - Change **IP Settings** from **DHCP** to **Static**.

6. **Enter Static Network Settings**  
   Fill in the following details:
   - **IP Address**: The IP address you noted in step 3.  
   - **Gateway**: The IP address of your **OpenWRT server**.  
   - **Network Prefix Length**: Leave this unchanged.  
   - **DNS 1**: The IP address of your **OpenWRT server**.

   Example configuration:  
   ![Static Network Settings Example](https://github.com/user-attachments/assets/f0e39e29-0149-442b-a6bd-6f8f3b17a6e1)

7. **Apply the Settings**  
   - Turn off your Wi-Fi connection and turn it back on for the new settings to take effect.

---

#### Notes:
- The **Gateway** and **DNS 1** values must point to the IP address of the **OpenWRT server** where Sing-Box is running.
- Ensure that the static IP you assign to your device is within the same subnet as your network.

By following these steps, your Android device will route its traffic through the **Sing-Box** setup on your OpenWRT router. Repeat similar steps for other devices to connect them to this network.
---
## Conclusion

You’ve now set up and configured **Sing-Box** on OpenWRT. Whether you’re optimizing for personal use or a network-wide deployment, this guide should serve as a robust foundation. 

Feel free to explore and adapt the settings further to suit your needs!

--- 

Let me know if you need further modifications or additional sections!
