# RG353-WiFi-Penetration-Tool
Hacking WiFi Networks with the Anbernic RG353 line of hardware and other Retro Gaming Handhelds

![IMG_4443](https://github.com/ZeroDayArcade/RG353-WiFi-Penetration-Tool/assets/141867962/292c51b3-1f78-4382-a247-f172a24ea28b)

This repo contains modified versions of my <a href="https://github.com/search?q=owner%3AZeroDayArcade+wpa&type=repositories">WiFi hacking python scripts</a>, now meant to run on Anbernic RG353 series hardware using the RG353's built in WiFi module. This code was specifically tested with the RG353PS, using the <a href="https://github.com/RetroGFX/UnofficialOS">UnofficialOS</a> operating system with the RG353P image from <a href="https://github.com/RetroGFX/UnofficialOS/releases/tag/20230427">release 20230427</a>. The scripts should work with UnofficialOS on other varients of the RG353 without modification assuming the same basic file structure and can easily be modified to run on many other Linux-based Retro Handhelds that have WiFi capabilites.

Although there is no easy way to put the built-in WiFi chip into monitor mode, it is still capable of reading packets in managed mode that are necessary for PMKID attacks. This means that an RG353 running this code can capture a PMKID and potentially crack the password of a WiFi network from access points that are vulnerable to the <a href="https://hashcat.net/forum/thread-7717.html">PMKID exploit</a> found by <a href="https://hashcat.net/forum/user-1.html">atom</a> and the hashcat team in 2018. 

The capture script should be run first which will extract the PMKID from the first EAPoL frame that the access point sends back to the RG353 when the RG353 attemps to connect to it with a random password. Although the random password will obviously fail to connect, the PMKID, MAC addresses, and SSID will be stored by the script in an hashcat hc22000 format hash line and saved to a file `hashline.txt`. The hash line can then be used later for an offline attack using hashcat on a more powerful computer ***OR*** the user can immediately try to crack it with the cracking script that comes with this codebase on the RG353 itself without any extra hardware. 

The cracking script is more for demonstration purposes than anything, but for low hanging fruit (weak passwords) and with relatively small password lists, it can crack passwords without the user having to leave the RG353. I've added a very small sample `passlist.txt` file which can be used for testing, but you can also supply your own `passlist.txt` file containing larger and/or more sophisticated password dictionaries. This makes hacking on the go possible, and even if the password can't be cracked by the on-board script, the hash line that is saved to `hashline.txt` on the TF2 slot Micro SD card can be cracked later with more powerful cracking tools as previously mentioned. 

I successfully tested this on WPA2 with a TP-Link Archer C1200 v2.0 Router (Firmware Version 2.0.0) with both 2.4Ghz and 5Ghz networks. I found capturing PMKID on 2.4Ghz to work almost 100% of the time and within a couple of seconds, whereas capturing from 5Ghz was less reliable only working ~50% of the time and often taking longer. Make sure in testing that networks are visible (not hidden). You may be able to get more reliability out of 5Ghz captures by playing around with when WiFi gets restarted on the device (see `capture.sh` and potentially move WiFi restart into `capture_pmkid.py` with `os.system`) and/or by removing timeout code in `capture_pmkid.py`. Although they are very common, only WPA/WPA2 networks that use PMKID are vulnerable to this attack. 

Also, and I hope this goes without saying, only ever hack a network you own and have legal permission to hack. This is for educational purposes only and to help you advance your penetration testing skills and knowledge.

## Steps to using the tool

*TL;DR: Use UnofficialOS, copy "RG353-WiFi-Pen/" to "ports/" on ROMS MicroSD, run scripts from "PORTS" Menu*

1. ***Install UnofficialOS*** on your RG353P/PS/M/etc (Backup your original OS or use a seperate Micro SD in slot TF1)  

   ![IMG_4436](https://github.com/ZeroDayArcade/RG353-WiFi-Penetration-Tool/assets/141867962/653041d7-0f8c-4074-bfa3-cf51e018d8e8)
2. Clone this repo and ***copy the RG353-WiFi-Pen directory*** to the "ports/" directory of the slot TF2 (Roms) MicroSD card
3. With the TF2 MicroSD back in the RG353, press the 'START' button and ***go to Network Settings*** and make sure WiFi is enabled
4. Select the target WiFi network and ***enter a random password for the network***
5. Exit the menus and ***navigate to "PORTS"*** on the main interface  

![IMG_4440](https://github.com/ZeroDayArcade/RG353-WiFi-Penetration-Tool/assets/141867962/dbd82ccd-b9fd-4d0b-a6ad-9edff2188816)

6. Enter "PORTS" and ***select the "RG353-WiFi-Pen" directory***, you will see two options: ***capture*** and ***crack***  

![IMG_4441](https://github.com/ZeroDayArcade/RG353-WiFi-Penetration-Tool/assets/141867962/2ff95edd-189d-49ad-8341-036e85e3c738)

7. ***Select "capture"*** to run the PMKID capture script on you target network. If it is unable to capture PMKID it will timeout after a minute  

![IMG_4442](https://github.com/ZeroDayArcade/RG353-WiFi-Penetration-Tool/assets/141867962/383554bf-e82b-4204-a974-487636eac724)

8. If the PMKID capture was successfull you will see the PMKID, MAC addresses and SSID printed to the screen along the a hashcat format hash line which has been saved to the MicroSD card in `ports/RG353-WiFi-Pen/hashline.txt`. If the PMKID is all 0's, that likely means the access point you are targeting does not append a PMKID to the end of the first EAPol frame and is not vulnerable to this type of attack. Otherwise you're ready to crack  

![IMG_4446](https://github.com/ZeroDayArcade/RG353-WiFi-Penetration-Tool/assets/141867962/618305cd-7283-4714-a80f-aff746f46513)

9. Now ***run the "crack" script*** from the same place you ran the capture script. If the password of the network is part of the `passlist.txt` dictionary, than the password will be cracked. Otherwise you can take the hash line saved to `hashline.txt` for later cracking with hashcat or other cracking tools (you won't see it from the main menu but it's on the MicroSD if you open the MicroSD on your computer).  

![IMG_4447](https://github.com/ZeroDayArcade/RG353-WiFi-Penetration-Tool/assets/141867962/f0982460-94f5-4507-abb7-fa19a88df0c9)

In the example above you can see that my test network "ZDA_TP_LINK" was cracked and the password was "minecraft". If you are testing with the on board cracking script, use an 8+ character password that is in your `passlist.txt` file (Note some passwords in the sample `passlist.txt` are too short, pick one that's at least 8) to ensure that everything is working correctly.

# More ZDA Code and Resources:
**Learn Reverse Engineering, Assembly, Code Injection and More:**  
🎓  <a href="https://zerodayarcade.com/tutorials">zerodayarcade.com/tutorials</a> 

**More WiFi Hacking with Simple Python Scripts:**  
<a href="https://github.com/ZeroDayArcade/capture-pmkid-wpa-wifi-hacking">Capturing PMKID from WiFi Networks</a>  
<a href="https://github.com/ZeroDayArcade/wpa-password-cracking-with-pmkid/">Cracking WiFi Passwords with PMKID</a>  
<a href="https://github.com/ZeroDayArcade/capture-handshake-wpa-wifi-hacking">Capturing 4-Way Handshake from WPA/WPA2 Networks</a>  
<a href="https://github.com/ZeroDayArcade/cracking-wpa-with-handshake">Cracking WPA/WPA2 Passwords with 4-Way Handshake</a>  

**More Retro Gaming Handhelds:**  
<a href="https://www.youtube.com/shorts/auvxesBrZwU">Connecting a Game Boy Advance SP to the Internet</a>  
<a href="https://www.youtube.com/shorts/94pTU2rXiVE">Multiplayer Quake 1 on a Nintendo DS vs PC Player: Crossplay</a>  
<a href="https://zerodayarcade.com/tutorials/anbernic-rg353-quake-multiplayer">Tutorial - Crossplay Quake on Anbernic RG353PS vs PC</a>

# Find Hacking Bounties in Gaming:
🎮  <a href="https://zerodayarcade.com/bounties">zerodayarcade.com/bounties</a>



