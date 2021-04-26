# u2f-token project

Origin repo https://github.com/gl-sergei/u2f-token.git

## Preparing environment

* Install build tools

```sh
sudo apt update
sudo apt upgrade
sudo apt autoremove
sudo apt install build-essential git python3-pip openssl gcc-arm-none-eabi
sudo apt install openocd libhidapi-hidraw0 python3-hid
sudo ./VBoxLinuxAdditions.run
pip3 install --user -u asn1crypto
pip3 install --user asn1crypto
pip3 install --user easyhid
pip3 install --user pyu2f
```

* Allow access to new token usb device

```sh
sudo nano /etc/udev/rules.d/10-u2f-token.rules
ACTION=="add|change", KERNEL=="hidraw*", SUBSYSTEM=="hidraw", ATTRS{idVendor}=="16d0", ATTRS{idProduct}=="0e90", TAG+="uaccess"
ACTION=="add|change", SUBSYSTEM=="usb", ATTRS{idVendor}=="16d0", ATTRS{idProduct}=="0e90", TAG+="uaccess"
```

## Original instruction for cloning from git

```
mkdir token
cd token
git clone https://github.com/gl-sergei/u2f-token.git
cd u2f-token
git submodule update --init
cd src
```

## My build instruction

* Get clone from git

```
cd ~
git clone https://github.com/zanac86/u2f-token.git
cd u2f-token/src/cert
sudo chmod +x gen.sh
./gen.sh
cd ..
make TARGET=BLUE_PILL
```

## Programming stm32 flash

* stm32f103c8t6 on board with Micro-USB has id=0x1ba01477
* stm32f103c8t6 on board with USB-C has 0x2ba01477

### Erase full flash

```sh
openocd -c 'set CPUTAPID 0x1ba01477' -f interface/stlink-v2.cfg -f target/stm32f3x.cfg -c "init" -c "halt" -c "wait_halt" -c "stm32f1x mass_erase 0" -c "sleep 200" -c "reset run" -c "shutdown"
                             or
openocd -c 'set CPUTAPID 0x2ba01477' -f interface/stlink-v2.cfg -f target/stm32f3x.cfg -c "init" -c "halt" -c "wait_halt" -c "stm32f1x mass_erase 0" -c "sleep 200" -c "reset run" -c "shutdown"
```

### Programming flash

```sh
openocd -c 'set CPUTAPID 0x1ba01477' -f interface/stlink-v2.cfg -f target/stm32f1x.cfg -c 'init' -c 'halt' -c 'flash write_image erase unlock build/u2f.bin 0x08000000' -c 'exit'
                             or
openocd -c 'set CPUTAPID 0x2ba01477' -f interface/stlink-v2.cfg -f target/stm32f1x.cfg -c 'init' -c 'halt' -c 'flash write_image erase unlock build/u2f.bin 0x08000000' -c 'exit'
```

### Initializing key

After flashing mcu connect board to USB and run tool

```sh
cd cert
python3 certtool init
```

## Clearing all generated files

```sh
make certclean && make distclean && make clean
```

## Generate custom key

```sh
cd cert
openssl ecparam -name prime256v1 -genkey -noout -outform der -out key.der
cd ..
python3 ./inject_key_bin.py --key cert/key.der
```
