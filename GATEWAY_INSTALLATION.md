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
3. Gateway ID: `csn-mtx-xxxxxx` where `-xxxxxx` is the last 6 digits of the gateway Lora ID and `-mtx-` will be
    - `mtcap` for a Multitech indoor LoraWAN Access Point (internal antenna)
    - `mtcdt` for a Multitech indoor Conduit LoraWAN Gateway (external whip antenna)
    - `mtcdtip` for a Multitech outdoor IP67 Conduit LoraWAN Gateway (Lora an LTE antennas)
4. Gateway EUI: Available on the gateway label.
4. Description: `Cambridge Sensor network xxxx` where xxxx is at least location e.g. `Computer Lab roof`.
5. Frequency Plan: `Europe 863-870`.
<!-- 6. Router: `ttn-router-eu` (will be suggested automatically after selecting Frequency Plan). -->
6. Location: click on the map.
<!-- 8. Antenna Placement: `indoor`|`outdoor` as appropriate. -->
9. Click `Create Gateway` bottom right of page.
10. After registration, generate an API Key which will be needed in the Multitech 'setup packet forwarder' step below.

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
    - The default credentials to log in to the web interface are admin/admin
    - A wizard will appear. We will accept most of the defaults.
    - Get the password from SmartCambridge tfc_prod/secrets.sh, reset the 'admin' user password to this.
    - On the time zone configuration section, set the time zone to Europe/London
    - On the network settings page, add the correct information for the gateway address and DNS servers. For the Huawei network this should be 192.168.2.254 for the gateway, and Google's DNS servers 8.8.8.8 and 4.4.4.4
    - The gateway is configured to run a DHCP **server** by default. Disable this by choosing the neworking 'DHCP Client' option
    - Reboot
    
2. Second boot - Update firmware
    - The gateway should reboot as a DHCP client, so now find IP address.
    - Browse to [MultiTech Downloads](http://www.multitech.net/developer/downloads/) pages. For the metal gateways, use "Conduit: AEP Model w/Node-RED", for the plastic Conduit AP use "Conduit Access Point: AEP Model (MTCAP)"
    - Browse to IP address of gateway, login as 'admin' now with tfc_prod/secrets.sh password. From menu select Administration - Update Firmware.
    - Note that every time the firmware on the devices is updated, the packet forwarder will need to be reinstalled (if a custom packet forwarder is being used).
    - You may need to upgrade through several different version of firmware. If you want to upgrade to 5.x (CSN default) you must be on at least 1.6.4 before you begin. For more information, see: http://www.multitech.net/developer/software/aep/aep-firmware-changelog/
    - Reboot
    
3. Third boot - set up 'Remote Management' in gateway using Multitech key
    - From the gateway's web interface, select `Administration` -> `Remote Management`
    - Enter 'Account Key' as obtained from Device HQ web site (top-right account name, drop-down, Account Info)
    - Switch auto update Check-In Interval to 240, check the 'Enabled' checkbox and click 'Submit' button.
    - On gateway menu, click 'Save and restart' and gateway should appear 
    - Configure the gateway in [Device HQ](https://www.devicehq.com/), email admin@smartcambridge.org, pwd as tfc_prod/secrets.sh
    - Browse to https://devicehq.com, logon as admin@smartcambridge.org.
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
    - Do the same for the `Packet Forwarder`, again clicking `Submit`.
    - At the top `LoRa Mode` -> `Mode` drop-down, select `DISABLED` and `Submit`. To check your settings you can select
    `Network Server` and `Packet Forwarder` and confirm the `EU868` settings are there ok. Make sure you finish with
    the mode set to `DISABLED`.
    - On the left-menu, click `Save and Restart`.
    
5. Fifth boot - set up custom packet forwarder via SSH terminal session to gateway
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
    - Visit the TTN console and view `Gateways` and in a few mins you should see new gateway as 'connected'.
    
6. Sixth boot - save configuration to survive factory resets (if desired)
    - Save default config via gateway menu Administration - Save/Restore.
