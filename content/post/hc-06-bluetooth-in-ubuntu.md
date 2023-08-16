---
title: "HC 06 Bluetooth in Ubuntu"
tags: ["arduino", "bluetooth", "hc-06"]
date: 2018-08-02T12:09:56+01:00
comments: true
draft: false
---

## TL;DR

In this article we are going see how to connect HC-06 Bluetooth in Ubuntu, which is commonly used in embedded systems such as arduino. But this time we will see how to use it with linux in order to interact with bluetooth channels.

## Configuration

So first thing first we need to pair this device with ubuntu using the Bluetooth GUI in Ubuntu .

{{< figure src="/img/hc-06-bluetooth-in-ubuntu/hc-06.png" width="50%" >}}

Select the device and make sure you set up the Pin Option to '1234'.

Then, Open the Terminal and make sure that you can see the device !

```bash
$ hcitool scan
```

Now go head and edit this file :  `/etc/bluetooth/rfcomm.conf`

```bash
$ sudo vi /etc/bluetooth/rfcomm.conf
```

uncomment and change it to :

```yaml

    rfcomm0 {
        # Automatically bind the device at startup
        bind no;

        # Bluetooth address of the device
        device 98:D3:31:80:51:48;

        # RFCOMM channel for the connection
        channel    1;

        # Description of the connection
        comment "HC-06";
    }
```

Make sure you change the device to your HC-06 Module Address !.

Finally, bind the device with :

```bash
$ sudo rfcomm bind rfcomm0
```

Then use minicom to communicate with the modue in serial !

```bash
$ sudo minicom -D /dev/rfcomm0 -b 9600 -8
```

and don't forget to release the device when you finished

```bash
$ sudo rfcomm release rfcomm0
```
