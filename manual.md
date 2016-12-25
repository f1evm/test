# INSTALLATION DE TRAPPETTE  

<small>
__NOTE 1 :__ Les exemples sont donnés pour le Raspberry Pi avec clé SDR et la sonde M10.  
__NOTE 2 :__ Ne pas connecter votre clé SDR avant d'avoir installé rtl-sdr  ! 
</small>

## 1.	INSTALLATION DES OUTILS NÉCESSAIRES  

Avant toute chose, pour être sûr de partir sur de bonnes bases, mettez à jour votre système :  

	sudo apt-get update  <br>
	sudo apt-get upgrade  
	sudo apt-get autoremove  
	
Dans votre console, exécutez la commande suivante :  

	sudo aptget install cmake git libusb-1.0-0-dev

Elle permet de'installer les outils nécessaires :   

* git ( pour récupérer les sources de rtlsdr)
* cmake ( pour compiler rtlsdr)
* libusb-1.0-0-dev ( lien entre les programmes de rtlsdr et le hardware )  


## 2.	RÉCUPÉRER RTL-SDR  

(source : http://sdr.osmocom.org/trac/wiki/rtlsdr])  

	git clone git://git.osmocom.org/rtl-sdr.git
	cd rtl-sdr
	mkdir build
	cd build/
	cmake ../ -DINSTALL_UDEV_RULES=ON
	make
	sudo make install
	sudo ldconfig
    
## 3.	EMPÊCHER LE KERNEL DE JOUER AVEC LA CLÉ RTL2832U À LA PLACE DE RTLSDR  

( source: https://opendesignengine.net/news/53 )  

	sudo bash                        (le mot de passe par defaut est "raspberry")  
	echo "blacklist dvb_usb_rtl28xxu" > /etc/modprobe.d/blacklist.conf  
	exit

Eteignez votre Raspberry Pi  
Connectez votre clé RTL2832U dans un des port USB  
Rebranchez votre Raspberry Pi et relancer un terminal.  

## 4.	VÉRIFIER QUE LA CLÉ RTL2832U FONCTIONNE  
	rtl_test

ça fonctionne? Bravo! On a fait le plus dur…  
Ctrl+C pour arrêter rtl_test  

ça ne fonctionne pas?  
lisez calmement les sources cités ci-dessus pour chercher le soucis.

## 5.	MESURER LA DÉVIATION PPM DE VOTRE CLÉ RTL2832U  

Lancez **rtl_test** avec l’option -p, et laissez-le tourner **30 min à 1h** :  

	rtl_test -p
	[...]
	real sample rate: 2048140 current PPM: 69 cumulative PPM: 79
	real sample rate: 2048147 current PPM: 72 cumulative PPM: 79

	
Ctrl+C pour arrêter rtl_test  
La déviation est par exemple ici pour moi de 79 ppm ( à mémoriser )  

## 6.	VÉRIFIER QU’ON REÇOIT BIEN QUELQUE CHOSE AVEC LA CLÉ  
	rtl_fm -p 79 -f 402M -M fm -s 48k -E dc - | hexdump -Cv
	 ou 
	rtl_fm -p 79 -f 402M -M fm -s 48k -E dc - | aplay -r 48k -f S16_LE

Ctrl+C pour arrêter rtl_fm 

## 7.	RÉCUPÉRER ET COMPILER LE PROGRAMME “TRAPPETTE”  

Si c'est la première fois que vous récupérez les sources :  

	git clone http://github.com/Quadricopter/trappette.git
	cd trappette
	make

Pour mettre à jour une installation existante :  

	cd trappette
	git pull
	make clean
	make

## 8.	FICHIER DE CONFIGURATION  

Pour créer le fichier de configuration nous allons commencer par copier l'exemple fourni : 

	cp trappette.cfg.sample trappette.cfg

Avec votre éditeur favori, examinez alors les paramètres et modifiez ceux qui doivent l'être (notamment vos coordonnées et les diverses options).  


	# Your QRA coordinates, Eiffel Tower follow:
	qra.latitude = 48.858256
	qra.longitude = 2.294544
	qra.altitude  = 334
	
	# Output
	time.offset = 3600
	earth.ellipsoid = 49
	#header.repeat = 22
	
	# GPS output
    	gps.out.port = /dev/ttyUSB0
    	gps.out.baud = 4800

	# Rotor
	rotor.port = /dev/ttyACM0
	rotor.baud = 9600
	rotor.azimuth.init = 246 # Eiffel Tower -> Trappes
	rotor.elevation.init = 0

	rotor.azimuth.min = 0
	rotor.azimuth.max = 360
	rotor.elevation.min = 0
	rotor.elevation.max = 90
 
