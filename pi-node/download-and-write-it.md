---
description: Flash image
---

# Download & Flash

## Flash Image

Download, install & open [Raspberry Pi Imager](https://github.com/raspberrypi/rpi-imager/releases/latest). Plug in your target USB drive.

{% tabs %}
{% tab title="Local Machine (Ubuntu)" %}
```bash
# Ubuntu users can download and install with snapd
sudo apt update
sudo apt install snapd
sudo snap install rpi-imager
```
{% endtab %}
{% endtabs %}

{% hint style="danger" %}
Older models of the Pi4B 8GB need to have their boot loader updated to boot from USB. If your image won't boot remove the USB3 drive and use rpi-imager to flash Pi 4 EEPROM boot recovery to an sd card.

Plug the Pi into a monitor, insert the sd card and power up. Once you see a green screen you should be good to boot from your USB3 drive. Newer versions are shipping with a USB boot capable boot loader. **Feeling lucky?**

**Choose OS -> Misc utility images -> Raspberry Pi 4 EEPROM boot recovery** [https://www.raspberrypi.org/documentation/hardware/raspberrypi/booteeprom.md](https://www.raspberrypi.org/documentation/hardware/raspberrypi/booteeprom.md)
{% endhint %}

![](https://github.com/wcatz/pi-pool/tree/7f765c0a2a01dae79d14feaa1cb2d3400ee7e08a/.gitbook/assets/otgpoltut.png)

{% tabs %}
{% tab title="Pre configured Pi-Node.img.gz" %}
### Obtain Pi-Pool .img.gz file

| [Pi-Node](https://db.adamantium.online/Pi-Node.img.gz) |
| ------------------------------------------------------ |

### Within Raspberry Pi Imager

**Choose OS -> Use custom**

Locate the .img.gz file you downloaded & wish to flash.

Locate your target drive & write it to disk.

![](https://github.com/wcatz/pi-pool/tree/7f765c0a2a01dae79d14feaa1cb2d3400ee7e08a/.gitbook/assets/image-2-.png)
{% endtab %}

{% tab title="Fresh Ubuntu 22.04 installation" %}
### Within Raspberry Pi Imager

### Select  Ubuntu Server 22.04 (RPI 3/4/400)

**Choose OS -> Other general purpose OS -> Ubuntu -> Ubuntu Server 22.04 (RPI 3/4/400)**. The 64 bit server option.

[https://cdimage.ubuntu.com/releases/22.04/release/ubuntu-22.04-preinstalled-server-arm64+raspi.img.xz](https://cdimage.ubuntu.com/releases/22.04/release/ubuntu-22.04-preinstalled-server-arm64+raspi.img.xz)

Locate your target drive & write it to disk.

![](https://github.com/wcatz/pi-pool/tree/7f765c0a2a01dae79d14feaa1cb2d3400ee7e08a/.gitbook/assets/21.04-rpi-imager.png)
{% endtab %}
{% endtabs %}
