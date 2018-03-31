# A Mini-ScienceDMZ For Scientific Instruments

## Introduction
Modern scientific instruments such as telescopes, electron microscopes, and DNA sequencers, capture and process data using on-board computers. These on-board computers transform what the instrument "sees" as digital data. In case of a telescope, the on-board computer converts the lens' focused image into a digital jpeg file. Like most computers, these instruments are vulnerable to network-based attacks, which risk compromising the data as well as the instrument. When an instrument is compromised via a network-based attack, the science it supports grinds to a halt. To ensure that researchers can control the integrity of their instruments and data, this project develops a device that works with the instruments' on-board computers to prevent unauthorized access and manipulation of data. The device serves as both a firewall for the instrument and as a data transfer system that ensures data collected by instrument is not altered. Technology developed through this project will have broader applications like protecting medical devices, industrial machines, and aircraft components. 

The project designs, develops, and tests the deployment of a small-scale device that functions as a firewall, a large file transfer facility, and a network performance monitor. The device consists of a small, low-power box positioned between the instrument and the campus network. Firewall policies are configurable by the administrator or a designated opoerator via a cloud-based portal. The file transfer facility will transfer large data files over long distances, while maintaining reliable performance. This facility will also digitally sign files, so that data integrity can be verified throughout the science workflow. In turn, the network performance monitor will interface with associated online systems, so performance bottlenecks can be easily identified. Ultimately, this architecture and implementation will ensure optimal data processing while safeguarding the integrity of the instrument and its data throughout the scientific workflow.

## Installation Instructions

### Raspberry Pi Setup For Mini-ScienceDMZ

1. Follow the instructions on the official Raspberry Pi page to install Raspbian on a SD card.
   - The Advanced Setup option is recommended for installation. Download the **Stretch-Lite** image and flash it using _Etcher_ tool.
   - Click [here](https://www.raspberrypi.org/documentation/installation/installing-images/) for the official instructions.

2. Download the Raspberry Pi setup files using [this](/pi_setup_files) link. 

3. Once step 1 and 2 are complete, the SD card would be partitioned into two parts. One of those partitions would be named as **boot**. Place the files from step 2 in the boot partition of the SD card.

4. Create an account in [Dynv6](https://dynv6.com/) and register a **DOMAIN_NAME** (to create a new host).
   - This account will allow you to create a DNS record which can be used for [Dynamic DNS](https://tools.ietf.org/html/rfc2136) purposes

5. Go to [Dynv6 APIs](https://dynv6.com/docs/apis) and obtain *Your API token* which is the **token** parameter in the HTTP Update API section. 

6. Edit the **dynv6_token.txt** file in the boot directory and replace _TOKEN_WILL_REPLACE_THIS_ with *Your API token*. 

7. Edit the **pi_settings.py** and store the **DOMAIN_NAME** you registered in step 4 in the variable **DOMAIN_NAME**.

8. Insert the SD card in the Raspberry Pi and connect power cable, HDMI and keyboard to it.

9. Connect the Pi to the RDP enabled equipment using a LAN cable.
   - NOTE :  This **MUST** be done by connecting the Pi to the default Ethernet/LAN port in the Raspberry Pi. It will be the eth0 interface. Do **NOT** connect the RDP enabled equipment to a USB port using a LAN cable + USB Ethernet adapter combination. This connection will be assigned eth1+ interface and would interfere with the configuration scripts.

10. Login to the Raspberry Pi with default credentials. (Username : pi, Password : raspberry)

11. Modify the file permissions and launch the initial setup script as follows. 
    ```
    sudo chmod 700 /boot/*.py
    sudo /boot/pi_initial_setup.py
    ```
    - Follow the prompts to finish the initial configuration of Raspberry Pi. The Pi will reboot after this configuration completes.
     - Answer the prompt if the pi will be using a wireless internet connection. 
       If answered with yes, you will be prompted to answer if the WiFi network is WPA-Enterprise.
       If answered with yes, you will be prompted for username. Provide the username.
       Enter the WiFi password when you are prompted for it.
       Change the default Raspberry Pi password to a **NEW_PASSWORD** when prompted for it.
     - NOTE : This script has two optional arguments that user can use to modify the automated configuration of dynamic DNS and dynamic IP configuration. Use `sudo /boot/pi_initial_setup.py -h` further information.

12. Log into the Raspberry Pi with the username **pi** and the **NEW_PASSWORD**.

13. Launch the final setup script using the following command and wait for the Raspberry Pi to reboot.
    ```
    sudo /boot/pi_final_setup.py -e YOUR_EMAIL_ADDRESS
    ```
    - This script will by default try to use [certbot](https://certbot.eff.org/about/) to configure HTTPS using a production grade certificate from [Let's Encrypt](https://letsencrypt.org/about/) Certificate Authority (CA). If that fails, it falls back to a self-signed certificate generation to ensure an HTTPS connection between user and our minidmz setup. 
    - We can obtain [test certificates](https://letsencrypt.org/docs/staging-environment/) from Let's Encrypt by issuing the following command. `sudo /boot/pi_final_setup.py -t -e YOUR_EMAIL_ADDRESS`
    - We can force the script to use self-signed certificates or use HTTP configuration (**Not Recommended**) by using appropriate arguments. Use `sudo /boot/pi_final_setup.py -h` further information.
    - NOTE : Let’s Encrypt CA issues short-lived certificates (valid for 90 days).The *YOUR_EMAIL_ADDRESS* is used by the CA to warn you if the certificate is about to expire or for revocation purposes. If email address is used for self-signed certificate, it is also used in the certificate generation process. 
 

### Guacamole Setup For Mini-ScienceDMZ

1. Login to Pi with username **pi** and the password set by you (**NEW_PASSWORD**).

2. Edit the file **/home/pi/minidmz/guacamole_setup_files/settings.py** and change the DOMAIN_NAME to the one obtained in step 4 of **Raspberry Pi Setup for Mini-ScienceDMZ**.

3. Enter the following command to launch the Guacamole setup. 
   ```
   /home/pi/minidmz/guacamole_setup_files/setup.py -u USERNAME
   ```
   - Replace the USERNAME with the username used to authenticate with CAS server. This username (user) will be the administrator for the application. The administrator would be able to add other users to the application granting them access to the scientific device.
   - Users added by the administrator persists even when the *setup.py* is launched multiple times. To completely delete all the user information and create a fresh instance, use the following command. `/home/pi/minidmz/guacamole_setup_files/setup.py -f -u USERNAME`

4. The Guacamole page can be visited at https://DOMAIN_NAME/guacamole/ if configured with HTTPS and at http://DOMAIN_NAME/guacamole/ if configured as HTTP.

## Important Points of Consideration

**HTTP configuration is discouraged** as the password for connecting to the remote desktop machine (the scientific instrument in this case) will be transferred un-encrypted from the browser on the user machine to the Raspberry Pi. 

Ensure that the SSH server on the Raspberry Pi is secure. Disable Password logins and move to SSH-Keys. Check [official debian documentation](https://wiki.debian.org/SSH#Good_practices_with_SSH_Server) for futher information.

A WPA-Enterprise WiFi network (like a corporate network or university network) requires a username and password for connection. A WPA-Personal WiFi network (like a home WiFi network) requires only password for connection.

The script tries to configure wireless/wired connection using standard configuration. If the wireless is not configured, manual configuration may be required. Check wpa_supplicant.conf and interfaces files for details on modifications.

