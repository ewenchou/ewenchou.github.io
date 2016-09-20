---
layout: post
title: Bluetooth Proximity Detection
categories: blog
tags: raspberry-pi bluetooth iphone ios apple-watch watch-os
---
Since getting my Raspberry Pi 2 back in February, I've wanted to build a gadget that could detect when I'm nearby and take some action. This was the underlying idea for my [morning greeter project]({{ site.base_url }}/blog/2016/03/25/good-morning-alexa/).

> *TLDR; I used Bluetooth to check if my iPhone/Apple Watch was nearby. Here's the [code](#sample-code)*.

A quick search on the Internet shows that the usual way of doing something like this with a Raspberry Pi is through a PIR (Passive Infra Red) sensor. There are many tutorials available online, including a cool ["Parent Detector"](https://www.raspberrypi.org/learning/parent-detector/worksheet/) project from the official Raspberry Pi website that is geared towards kids learning to hack with the Pi.

However, there is one major drawback to this approach -- the PIR sensor cannot differentiate between different people (or people versus pets for that matter). For my morning greeter, I wanted it to be able to tell if it was me or my wife who had walked downstairs into the kitchen.

Fortunately (Unfortunately?), like many people these days, we have our smartphones with us all the time -- especially if we're getting ready to leave the house. So I figured a good way to detect who was nearby would be to check which phone was nearby.

Just about every smartphone is equipped with WiFi and Bluetooth, each with their own unique addresses. While WiFi is probably the easiest and most reliable way to detect if the device is connected and at home, I couldn't find an easy way to detect the distance (i.e. proximity) of the WiFi enabled device. So I focused on looking for ways to do this using Bluetooth. 

This proved to be more difficult with our iPhones due to the security policies in iOS. The iPhones do not connect to (non-Apple) Bluetooth devices unless the user opens up the Settings app and navigates to the Bluetooth page. This would totally defeat the purpose of the morning greeter, since I may as well just lookup the information on my phone after I've pulled it out.

So I needed a way to detect if the phone was nearby without any human interaction with the phone itself. I found bits and pieces of information and code snippets online for doing this. I needed to weave them together and do quite a bit of trial-and-error before getting something that worked.

Initially, I tried using a combination of the `rfcomm` and `hcitool` shell commands but this was very hacky. The commands would sometimes hang for upwards of 10-15 seconds before returning, and the return values were inconsistent.

Eventually I found some [code on Github](https://github.com/dagar/bluetooth-proximity) by *Daniel Agar* that worked really well. I adapted it into the sample code below.

## Sample Code

{% highlight python %}
import bluetooth
import bluetooth._bluetooth as bt
import struct
import array
import fcntl


class BluetoothRSSI(object):
    """Object class for getting the RSSI value of a Bluetooth address.
    Reference: https://github.com/dagar/bluetooth-proximity
    """
    def __init__(self, addr):
        self.addr = addr
        self.hci_sock = bt.hci_open_dev()
        self.hci_fd = self.hci_sock.fileno()
        self.bt_sock = bluetooth.BluetoothSocket(bluetooth.L2CAP)
        self.bt_sock.settimeout(10)
        self.connected = False
        self.cmd_pkt = None

    def prep_cmd_pkt(self):
        """Prepares the command packet for requesting RSSI"""
        reqstr = struct.pack(
            "6sB17s", bt.str2ba(self.addr), bt.ACL_LINK, "\0" * 17)
        request = array.array("c", reqstr)
        handle = fcntl.ioctl(self.hci_fd, bt.HCIGETCONNINFO, request, 1)
        handle = struct.unpack("8xH14x", request.tostring())[0]
        self.cmd_pkt = struct.pack('H', handle)

    def connect(self):
        """Connects to the Bluetooth address"""
        self.bt_sock.connect_ex((self.addr, 1))  # PSM 1 - Service Discovery
        self.connected = True

    def get_rssi(self):
        """Gets the current RSSI value.

        @return: The RSSI value (float) or None if the device connection fails
                 (i.e. the device is nowhere nearby).
        """
        try:
            # Only do connection if not already connected
            if not self.connected:
                self.connect()
            if self.cmd_pkt is None:
                self.prep_cmd_pkt()
            # Send command to request RSSI
            rssi = bt.hci_send_req(
                self.hci_sock, bt.OGF_STATUS_PARAM,
                bt.OCF_READ_RSSI, bt.EVT_CMD_COMPLETE, 4, self.cmd_pkt)
            rssi = struct.unpack('b', rssi[3])[0]
            return rssi
        except IOError:
            # Happens if connection fails (e.g. device is not in range)
            self.connected = False
            return None
{% endhighlight %}

A bonus for using Bluetooth, it works for both our iPhones and Apple Watches because they each have a unique Bluetooth address. I think this method should work for any smartphone/smartwatch/device equipped with Bluetooth.

You can find the full code on my [Github](https://github.com/ewenchou/bluetooth-proximity)
