# Multitech Gateway Installation onto the Cambridge Sensor Network

## General setup

These gateways should all be registered to the main CSN TTN account with gateway IDs like `csn-mtcap-014abe`, `csn-mtcdtip-003c6f` etc (i.e. "csn-" + the product code of the gateway + the last 12 bits of the LoRa node address).
<!-- Currently we're pointing them directly at the `ttn-router-eu` router (not using a local relay).  -->
They use DHCP to obtain their IP addresses wherever they are connected, so the local computer officer needs to be provided with the ethernet MAC address of the gateways to register them in their DHCP server.

## Configuring a new Multitech LoraWAN gateway

Before starting, collect essential settings information from an existing production server
in `acp_prod/acp_lorawan_gateways/secrets`.

### Initial setup - The Things Network

1. Logon at https://eu1.cloud.thethings.network/console (user cambridgesensornetowrk)
2. Select 'Gateways' on top menubar, click '(+) register gateway'.
3. Gateway ID: `csn-mtx-xxxxxx` where `-xxxxxx` is the last 6 digits of the gateway Lora ID (labelled LORA NODE on device) and `-mtx-` will be
    - `mtcap` for a Multitech indoor LoraWAN Access Point (internal antenna)
    - `mtcdt` for a Multitech indoor Conduit LoraWAN Gateway (external whip antenna)
    - `mtcdtip` for a Multitech outdoor IP67 Conduit LoraWAN Gateway (Lora an LTE antennas)
4. Gateway EUI: Available on the gateway label as "LORA NODE".
5. Description: `Cambridge Sensor network xxxx` where xxxx is at least location e.g. `Computer Lab roof`.
6. Gateway Server Address: eu1.cloud.thethings.network
7. Gateway status: Public - check the box
8. Frequency Plan: `Europe 863-870 (SF9 for RX2)`.
9. Schedule downlink rate: Enabled
10. Duty cycle: Enforced
11. Schedule any time delay: 530ms (although not sure what this is)
12. Automatic updates: leave unchecked (make revisit this)
13. Click Add Gateway.
14. Update the location on the map. Save Changes.
15. Download the `global_conf.json` which will be required in TTN setup step below.

<!-- 10. After registration, generate two API Keys (one for LNS and one for CUPS) which will be needed in the Basic Station Setup step below. (Not required for Packet Forwarder) -->

### Initial setup - Multitech Gateway and https://devicehq.com

1. Get a stand-alone ethernet switch, no WAN connectivity needed.
2. Connect the Multitech gateway to the switch and power it up.  The gateway will start with IP 192.168.2.1 and
run a *DHCP server* which will serve addresses in the 192.168.2.X range.
3. Connect a laptop or other workstation configured to accept DHCP addresses to the same switch.  Alternatively
you can configure the workstation to use manual settings and use 192.168.2.100, 255.255.255.0

When making changes to the MultiTech gateways, the changes are only applied once the gateways have been rebooted.

These instructions apply to gateways shipped with >1.6.x firmware.

1. First boot - change admin password and setup IP networking
    - Browse to https://192.168.2.1/ and continue despite the certificate warning. We can configure a real certificate later.
    - On first login, the wizard will ask you the username which is `admin` and ask to set a password. (On some older versions, first time login is required with the default credentials which are admin/admin)
    - Get the password from `acp_prod/acp_lorawan_gateways/secrets`, set the 'admin' user password to this.
    - A wizard will appear. We will accept most of the defaults.
    - On the time zone configuration section, set the time zone to Europe/London
    - Select the default for others and finish.
    - The gateway is configured to run a DHCP **server** by default. Go to `Setup > Network Interfaces`.
    - Edit the ETH0 interface and set Bridge as `--` and Mode to `DHCP Client`. Click Submit.
    - Go to `Setup > DHCP Configuration`. Edit the IPv4 DHCP Server entry to disable the DHCP Server: Uncheck "Enabled". Click Submit.
    - Go to `Setup > Global DNS` and set the DNS Servers as Primary: 8.8.8.8 and Secondary: 4.4.4.4
    - On the top-left of the menu click "Save and Restart" to Reboot.
    - Move the gateway from the temporary switch to a normal ethernet connection on your DHCP-serving network.

2. Second boot - Update firmware
    - The gateway should reboot as a DHCP client, so now find IP address. If you have more than on Multitech gateway on your network, check the
    Lora EUI on the Gateway Home page to make sure you are configuring the correct device.
    - Browse to [MultiTech Downloads](http://www.multitech.net/developer/downloads/) pages. Note these are FTP downloads of zip files,
    if you use the Windows FTP command you must set 'bin' before the download or Windows will corrupt the file. The zip file contains a '.bin'
    file, which is what you will upload to the gateway.
    - For the metal gateways, use "Conduit: AEP Model w/Node-RED", for the plastic Conduit AP use "Conduit Access Point: AEP Model (MTCAP)"
    - Browse to IP address of gateway, login as 'admin' now with `acp_prod/acp_lorawan_gateways/secrets` password. From menu select `Administration > Firmware Upgrade`.
    - Note that every time the firmware on the devices is updated, the packet forwarder will need to be reinstalled (if a custom packet forwarder is being used).
    - You may need to upgrade through several different version of firmware one at a time. If you want to upgrade to 5.x (CSN default) you must be on at least 1.6.4 before you begin. For more information, see: http://www.multitech.net/developer/software/aep/aep-firmware-changelog/
    - Once upgraded, the gateway will reboot by itself. If the upgrade has no effect the first time, a second attempt may work.

3. Third boot - set up 'Remote Management' in gateway using Multitech key
    - From the gateway's web interface, select `Administration` -> `Remote Management`
    - Enter 'Account Key' as obtained from Device HQ web site (top-right account name, drop-down, Account Info)
    - Switch auto update Check-In Interval to 240, check the (top-left) 'Enabled' checkbox and click 'Submit' button.
    - On gateway menu, click `Save and Apply`.
    - Browse to https://devicehq.com, logon as `admin@smartcambridge.org`, pwd as `acp_prod/acp_lorawan_gateways/secrets`.
    - Click `Devices` (which means gateways). In due course the new gateway will appear, matching the
    `Serial` written on the device (or you might note the gateway with a ~2min uptime).
    - Click the right-side 'pen' edit symbol for the new gateway, update the `Description` to exactly match the
    `Gateway ID` set on TTN, e.g. `csn-mtcdtip-00de8c` (this isn't technically required, but visually helpful). Also
    set the location, and save.

4. Fourth boot - set gateway to use EU868 LoraWAN frequencies
    - By default, the gateway is configured to use US915 frequencies. To switch it back to EU868 log in to the web interface.
    - Select `LoRaWAN` -> `Network Settings`. You may now get an error message `Detected card (868) does not match selected    channel plan (US915)` which we're about to fix.
    - Select `LoRa mode` -> `Mode` -> `Network Server` (i.e. change from `DISABLED`).
    - In `LoRaWAN Network Server Configuration` set `Channel Plan` to EU868, and click `Submit` at bottom of page.
    - Back at the top drop-down select `Packet Forwarder`, again set `Channel Plan` to EU868, and click `Submit`.
    - At the top `LoRa Mode` -> `Mode` drop-down, select `DISABLED` and `Submit`. To check your settings you can select
    `Network Server` and `Packet Forwarder` and confirm the `EU868` settings are there ok. Make sure you finish with
    the mode set to `DISABLED`.
    - On the left-menu, click `Save and Apply`.

<!-- 5. Fifth boot - set up custom packet forwarder via SSH terminal session to gateway
    - SSH to the IP address of the LoRaWAN gateway and log in with admin credentials.
    - We will switch to using Jac Kersing's packet forwarder. Instructions for doing this may be found on
    [TTN's web site about AEP MultiTech Conduits](https://www.thethingsnetwork.org/docs/gateways/multitech/aep.html).
    - `wget https://github.com/kersing/multitech-installer/raw/master/installer.sh --no-check-certificate`
    - `chmod +x installer.sh`
    - `sudo ./installer.sh`
    - `... time zone and network?` hit '1 <Enter>'
    - `Gateway ID:` as entered in TTN console, e.g. 'csn-mtcdtip-012345 <Enter>'
    - `Gateway Key:` <copy/paste from TTN gateway 'Overview' page>, i.e. begins `ttn-account-...`. Hit '1 <Enter>' to confirm.
    - `Email...`: `admin@smartcambridge.org` <Enter>, confirm all with '1 <Enter>'
    - Visit the TTN console and view `Gateways` and in a few mins you should see new gateway as 'connected'. -->

5. Fifth boot - Set up Packet Forwarder mode for TTN
    - Go to `LoRaWAN > Network Settings` and select the LoRa Mode as `PACKET FORWARDER`
    - Ensure Channel Plan is EU868.
    - Under `Server`, set Network to `The Things Network` and the server address from `gateway_conf` in the `global_conf.json` file downloaded earlier.
    - Click `Submit` and the `Save and Apply`. The gateway should appear on the TTN console.

6. Sixth boot - save configuration to survive factory resets (if desired)
    - Save default config via gateway menu Administration - Save/Restore.

<!-- PRIOR INFO FOR V2 TTN: 5. Fifth boot - set up Basic Station to connect the gateway to TTN (**Not working as of now**)
    - Go to `LoRaWAN > Network Settings` and select the LoRa Mode as Basic Station.
    - Set the `Credentials` as CUPS.
    - You'll need the the Things Stack CLI to get the CUPS keys. So install the CLI using snap on your laptop/workstation as;
    ```
    sudo snap install ttn-lw-stack
    sudo snap alias ttn-lw-stack.ttn-lw-cli ttn-lw-cli
    ```
    - Run `ttn-lw-cli login`. This will open a browser window or provide you with a link to open in a browser for authorization.
    - CUPS also provides support for LNS, so run the following to set the LNS credentials for CUPS. If succesful you'll receive a response output with the gateway_id.
    ```
    export GTW_ID="your-gateway-id"
    export LNS_KEY="your-lns-api-key"
    export SECRET=$(echo -n $LNS_KEY | xxd -ps -u -c 8192)
    ttn-lw-cli gateways update $GTW_ID --lbs-lns-secret.value $SECRET
    ```
    - Set the URI as `https://eu1.cloud.thethings.network:443`.
    - Copy the contents of `global_conf.json` to Station Config.
    - Get the complete certificate from *https://www.thethingsindustries.com/docs/reference/root-certificates/* and paste the contents of the file in Server Cert.
    - Generate the CUPS key file with the following steps and copy the contents of the `cups.key` file into Gateway Key.
    ```
    export CUPS_KEY="your-cups-api-key"
    echo "Authorization: Bearer $CUPS_KEY" | perl -p -e 's/\r\n|\n|\r/\r\n/g'  > cups.key
    ```
    - Click `Submit` and the `Save and Apply`. The gateway should appear on the TTN console. -->
