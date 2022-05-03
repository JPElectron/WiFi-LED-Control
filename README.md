# WiFi-LED-Control

EDIT: Years after I made this "documentation" I have discovered newer/better LED controllers...

Pixelblaze controller: https://www.bhencke.com/pixelblaze

QuinLED-Dig-Uno: https://quinled.info/pre-assembled-quinled-dig-uno/

QuinLED-Dig-Quad:  https://quinled.info/pre-assembled-quinled-dig-quad/

To understand the differences between digital LED strips (that have a DATA pin, and use the LED controllers linked above) versus those that are just plain RGB (using the LED controllers explained below) see <a href="https://www.youtube.com/watch?v=K4H2y51LKok">this video</a>

A) <a href="https://www.amazon.com/gp/product/B01DY56N8U">LEDENET Smart WiFi LED Controller 5 Channels</a>

B) <a href="https://www.amazon.com/gp/product/B01JZ2SI6Q">SUPERNIGHT WiFi Wireless LED Smart Controller</a><br>

Both use the Magic Home WiFi app, or send data directly on TCP port 5577 (more info on that below)

http://www.ledmagical.com/Apps/MgcHome/AppDown.aspx

https://play.google.com/store/apps/details?id=com.Zengge.LEDWifiMagicHome

https://play.google.com/store/apps/details?id=com.zengge.wifi

First the LEDENET unit, I prefer this one for a variety of reasons...

    - completely silent operation even when dimming
    - nice cross-fade when changing colors  
    - solid screw terminals for connections
    - runs cool (advertised as 4Amp switching per channel)
    - extra 4th and 5th channel intended for Warm White/Cool White LEDs but could also be used for relays, status light, nightlight, or otherwise
    - internal WiFi module is marked HF-LPB100-1
    - runs a webserver (u: admin p: nimda)

Default IP: 10.10.123.3 (when broadcasting its own wireless SSID and not connected to your own wireless network)

Note link at the very top right to make the page English

HTTP page on mine indicates MID: HF-LPB100-ZJ200 and Software v1.0.06 and Web v1.0.14

Packaging label reads "X000ZYJ6KH LEDNET Smart WiFi Controller" and includes door screws and a reset button push-pin

After sniffing out the network traffic, I learned that it continually tries to update the time from a NTP server located in China, 
this is likely for the timer functions to stay accurate, but disappointingly there is no way to change the NTP server to something else 
or disable it entirely. I really hope this is changed in a future firmware update, but since I don't need this I just block it at the firewall.

![ntp_on_boot](ntp_on_boot.png?raw=true "ntp_on_boot")
![ntp_to_china](ntp_to_china.png?raw=true "ntp_to_china")

From the factory it will broadcast its own wireless SSID which you can connect to directly, there is also the much more useful option to have it connect to your own home router/AP's SSID. 

Here's what the traffic looks like between the smartphone app (on IP .50) and the WiFi controller (on IP .54)

![on_cmd](on_cmd.png?raw=true "on_cmd")
![off_cmd](off_cmd.png?raw=true "off_cmd")

And here is everything I captured: <a href="a_box_prot.txt">a_box_prot.txt</a>

So if you wanted to send from Windows command-line or batch...

<a href="https://packetsender.com/download">packetsender.com</a> -q [IP of module] [port] [data in quotes]

For example, send "on"

    packetsender.com -q 192.168.1.54 5577 "71 23 0f a3"
    
For example, send "red" at full brightness

    packetsender.com -q 192.168.1.54 5577 "31 ff 00 00 00 00 f0 0f 2f"

A checksum of sorts is always the last octet, for example when sending "lt blue"

    31 00 cc cc 00 00 f0 0f c8
    
c8 is the checksum, which you can find yourself by adding up all the values:

31+00+cc+cc+00+00+f0+0f AND 255 like this... Here's a link <a href="https://defuse.ca/big-number-calculator.htm">defuse.ca/big-number-calculator.htm</a>

![checksum_calc](checksum_calc.png?raw=true "checksum_calc")

A handful of other people have "documented" the protocol for similar/older WiFi LED controllers and made scripts 
dependant on some 3rd party IoT service. I don't care for any of that risk or complexity just to control an LED strip on my LAN, but if you insist, here you go...

 - <a href="http://steve.zazeski.com/using-node-red-to-send-commands-to-wifi-led-controllers">node-red</a>
 - <a href="https://gist.github.com/linuxkidd/63013efd73e198d035e6">python</a>
 - <a href="https://github.com/vikstrous/zengge-lightcontrol">zengge-lightcontrol</a>
 - <a href="http://forum.universal-devices.com/topic/12977-fibaro-rgbw-controller">fibaro</a>
 - <a href="http://g.chasefox.net/zentyal/unifi/rgb-led">node.js</a>
 - <a href="http://domoticz.com/forum/viewtopic.php?f=38&t=7957#p54379">ESP8266 commercial H801</a>
 - <a href="http://stephenradford.me/cheap-homekit-led-strips-lighting">Stephen Radford</a>
 - <a href="http://www.emessaging.biz/blog/?p=629">Shauna Ruyle</a>

Now for the SUPERNIGHT unit, while it certainly works and has the added benefit of an IR remote (incase you lost your smartphone? or don't trust your guests on your WiFi?) here are some things I noticed:

 - when dimming any color LED (or mixing colors) it makes a high-pitched whine that annoys me
 - no cross-fade when changing colors
 - it gets VERY warm
 - there is a solder pad on the board (and the accompanying transistor) marked W, as if there was a 4th channel for White LEDs,
    but I found no way to turn it on so perhaps this is only enabled in a different model firmware
 - internal WiFi module is marked ESP-12S
 - doesn't seem to run any webserver
 - it still works if you de-solder the IR receiver

For these reasons I decided to only trust this unit to trigger 3 relays (one attached to each of the R, G, B pins and with a flyback diode to automate a children's toy, it runs cooler in this way.<br>

The capture for this one: <a href="b_box_prot.txt">b_box_prot.txt</a>

Conclusions:

If you wanted to use these to switch bigger loads, perhaps by connecting a relay directly to the channel output, 
then the SUPERNIGHT unit is easier because sending the command for red at 100% brightness will immediately turn red on full, 
whereas the LEDNET unit insists on doing a nice "fade in" or "ramp up" from 0% to 100% which makes a relay scream prior to picking. 
Controlling something like a DC motor, since any channel output is already PWM, would be much easier. 
Both modules are affordable and an easy way to add WiFi control options (or an IR remote) to any project, not just LED strips. 
Besides controlling these from Windows/command-line, the protocol is easy enough to control from a network connected Arduino or RasPi board.

[End of Line]
