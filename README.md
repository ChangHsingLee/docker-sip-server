# docker-sip-server
The example for setup SIP server in docker container

## Initial Date
	2020/10/20

## Version
	Alpine Linux: 3.14 
	Asterisk: 18.2.2

## Documentation
> <https://wiki.asterisk.org/wiki/display/AST/Home>

## Docker Hub
> <https://hub.docker.com/r/mlan/asterisk>

## GitHub (source)
> <https://github.com/mlan/docker-asterisk>
> <https://github.com/asterisk/asterisk>

## Installation
1. You can install recent version of Asterisk from docker store, there are four types of docker images,\
if you want to know what's difference between these images, you can check [source code](https://github.com/mlan/docker-asterisk/blob/master/Dockerfile).
    - docker pull mlan/asterisk:xtra (large size,  ~132MB)
    - docker pull mlan/asterisk:full (medium size, ~76.1MB)
    - docker pull mlan/asterisk:base (small size,  ~56.9MB)
    - docker pull mlan/asterisk:mini (tiny size,   ~42.1MB)

	OR<br>
	You can load the docker image "[mlan-asterisk-mini-dockerImg.tar.bz2](https://github.com/ChangHsingLee/backup-dockerImg/blob/main/mlan-asterisk-mini-dockerImg.tar.bz2)" which be saved/tested.
    ```shell
    bzip2 -dcv mlan-asterisk-mini-dockerImg.tar.bz2 | docker load
    ```
	The image is saved/compressed by below command
    ```shell
	docker save mlan/asterisk:mini | bzip2 -9vz > mlan-asterisk-mini-dockerImg.tar.bz2
	```
2. Prepare docker volume to store [configuration files, logs and others](https://github.com/ChangHsingLee/docker-sip-server/srv).
	```shell
	VOLUME_TOPDIR=$HOME/workspace/dockerVolumes/asterisk; \
    cp -a ./srv $VOLUME_TOPDIR/
	```
## Default [configuration](https://github.com/ChangHsingLee/docker-sip-server/srv/etc/asterisk/) for Asterisk SIP server
For this setting, you can check configuration files 'pjsip.conf' and 'extensions.conf'
	If you want to know syntax of configuration file, you can check [source code](https://github.com/asterisk/asterisk/tree/master/configs/samples). 
1. Two accounts:\
User Name: sip-user1\
Password: sip1234\
Extension No: 1001\
\
User Name: sip-user2\
Password: sip1234\
Extension No: 1002
2. To dial 9999, it will connect to SIP server and you will hear the voice that say "hello world".
3. Use UDP protocol

## Start Container
	VOLUME_TOPDIR=$HOME/workspace/dockerVolumes/asterisk; \
    ASTERISK_IMG_TYPE=mini; \
    CONTAINER_NAME=sip-server; \
    HOST_NAME=AsterikSIPserver; \
    docker run -d --restart always -p 5060:5060/udp -p 5060:5060 -p 5061:5061 -p 10000-10099:10000-10099/udp -p 8080:80 --cap-add SYS_PTRACE --cap-add=NET_ADMIN --cap-add=NET_RAW --name $CONTAINER_NAME -h $HOST_NAME -e SYSLOG_LEVEL=8 -v $VOLUME_TOPDIR/srv:/srv -v /etc/localtime:/etc/localtime:ro mlan/asterisk:$ASTERISK_IMG_TYPE

## Start Asterisk CLI
	CONTAINER_NAME=sip-server; \
    docker exec -it $CONTAINER_NAME asterisk -rvvvddd

## Some of useful CLI commands
- add log file\
	logger add channel <log file> <levels>\
	logger add channel test_log.txt DEBUG,NOTICE,WARNING,ERROR,VERBOSE,DTMF\
	/* see log file in docker volume $VOLUME_TOPDIR/srv/var/log/asterisk/ */

- del log file\
	logger remove channel <log file>\
	logger remove channel test_log.txt

- show log setting\
	logger show channels

- reload the PJSIP module to pick up the changes\
	pjsip reload

- reload the dialplan module to pick up the changes\
	dialplan reload

- show dailplan\
	dialplan show [*context name*]\
	dialplan show sip-test

## Softphone Application(MicroSIP, use for testing)
web site: <https://www.microsip.org/> \
MicroSIP is open source portable SIP softphone based on PJSIP stack for Windows OS.\
To check picture 'microsip-setting-example.png' for setting example.\
If you have more then one NIC card, suggest you don't select 'auto' for option 'Public Address'.

## Note
Asterisk 11 and previous: chan_sip is the primary option.\
Asterisk 12 and beyond: You'll probably want to use chan_pjsip (the newest driver), but you still have the option of using chan_sip as well

