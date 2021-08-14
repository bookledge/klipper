# 비글본 (Beaglebone) 

이 문서는 비글본 PRU 에서 클리퍼를 실행시키는 과정에 대해 설명합니다. 

## OS 이미지를 빌드하기

시작은 다음 이미지를 설치하는 것부터 입니다. 
[Debian 9.9 2019-08-03 4GB SD IoT](https://beagleboard.org/latest-images)

이 이미지는 마이크로SD 카드나 내장된 eMMC 로 부터 실행시킬 수 있습니다. 
만일 eMMC 를 사용한다면, 위 링크로 부터 다음 절차에 따라 eMMC 에 지금 설치하십시오. 

그리고 비글본 장치로 접속하십시오. (ssh debian@beaglebone -- password is "temppwd") 
그다음 아래 명령어를 실행시켜 클리퍼를 설치하시면 됩니다.:

```
git clone https://github.com/KevinOConnor/klipper
./klipper/scripts/install-beaglebone.sh
```

## 옥토프린터 설치하기

이제 옥토프린터를 설치하실 수 있습니다. :

```
git clone https://github.com/foosel/OctoPrint.git
cd OctoPrint/
virtualenv venv
./venv/bin/python setup.py install
```

그리고 옥토프린터를 부팅시 시작하도록 셋업을 해줍니다.:

```
sudo cp ~/OctoPrint/scripts/octoprint.init /etc/init.d/octoprint
sudo chmod +x /etc/init.d/octoprint
sudo cp ~/OctoPrint/scripts/octoprint.default /etc/default/octoprint
sudo update-rc.d octoprint defaults
```

필수적으로 옥토프린터의 **/etc/default/octoprint** configuration 을 수정해줘야할 필요가 있습니다.
OCTOPRINT_USER 는 "debian" 으로
NICE 
One must change the OCTOPRINT_USER user to
"debian", change NICELEVEL to 0, uncomment the BASEDIR, CONFIGFILE,
and DAEMON settings and change the references from "/home/pi/" to
"/home/debian/":
```
sudo nano /etc/default/octoprint
```

Then start the Octoprint service:
```
sudo systemctl start octoprint
```

Make sure the octoprint web server is accessible - it should be at:
[http://beaglebone:5000/](http://beaglebone:5000/)

## Building the micro-controller code

To compile the Klipper micro-controller code, start by configuring it
for the "Beaglebone PRU":
```
cd ~/klipper/
make menuconfig
```

To build and install the new micro-controller code, run:
```
sudo service klipper stop
make flash
sudo service klipper start
```

It is also necessary to compile and install the micro-controller code
for a Linux host process. Run "make menuconfig" a second time and
configure it for a "Linux process":
```
make menuconfig
```

Then install this micro-controller code as well:
```
sudo service klipper stop
make flash
sudo service klipper start
```

## Remaining configuration

Complete the installation by configuring Klipper and Octoprint
following the instructions in
[the main installation document](Installation.md#configuring-klipper).

## Printing on the Beaglebone

Unfortunately, the Beaglebone processor can sometimes struggle to run
OctoPrint well. Print stalls have been known to occur on complex
prints (the printer may move faster than OctoPrint can send movement
commands). If this occurs, consider using the "virtual_sdcard" feature
(see [config reference](Config_Reference.md#virtual_sdcard) for
details) to print directly from Klipper.
