# INSTALLATION DE TRAPPETTE  

<small>
__NOTE 1 :__ LES EXEMPLES SONT DONNÉS POUR LE RASPBERRY PI AVEC CLÉ SDR ET LA SONDE M10  
__NOTE 2 :__ NE PAS CONNECTER VOTRE CLÉ SDR AVANT D’AVOIR INSTALLÉ RTLSDR !  ( MAIS CE N’EST PAS GRAVE SI C’EST DÉJÀ FAIT )</small>

## 1.	INSTALLATION DES OUTILS NÉCESSAIRES  

Avant toute chose, pour être sûr de partir sur de bonnes bases, mettez à jour votre système :  


>~$ sudo apt-get update  
~$ sudo apt-get upgrade  
~$ sudo apt-get autoremove  

	
Dans votre console, exécutez la commande suivante :  

	~$ sudo aptget install cmake git libusb-1.0-0-dev

Elle permet de'installer les outils nécessaires :   

* git ( pour récupérer les sources de rtlsdr)
* cmake ( pour compiler rtlsdr)
* libusb-1.0-0-dev ( lien entre les programmes de rtlsdr et le hardware )  


## 2.	RÉCUPÉRER RTL-SDR  

(source : http://sdr.osmocom.org/trac/wiki/rtlsdr])  


	~$ git clone git://git.osmocom.org/rtl-sdr.git
	~$ cd rtl-sdr
	~/rtlsdr $ mkdir build
	~/rtlsdr $ cd build/
	~/rtlsdr/build $ cmake ../ -DINSTALL_UDEV_RULES=ON
	~/rtlsdr/build $ make
	~/rtlsdr/build $ sudo make install
	~/rtlsdr/build $ sudo ldconfig

    
## 3.	EMPÊCHER LE KERNEL DE JOUER AVEC LA CLÉ RTL2832U À LA PLACE DE RTLSDR  

( source: https://opendesignengine.net/news/53 )  

	~$ sudo bash                        (le mot de passe par defaut est "raspberry")  
	root@raspberrypi:/home/pi # echo "blacklist dvb_usb_rtl28xxu" > /etc/modprobe.d/blacklist.conf  
	root@raspberrypi:/home/pi # exit

Eteignez votre Raspberry Pi  
Connectez votre clé RTL2832U dans un des port USB  
Rebranchez votre Raspberry Pi et relancer un terminal.  

## 4.	VÉRIFIER QUE LA CLÉ RTL2832U FONCTIONNE  

	~$ rtl_test

ça fonctionne? Bravo! On a fait le plus dur…  
Ctrl+C pour arrêter rtl_test  

ça ne fonctionne pas?  
lisez calmement les sources cités ci-dessus pour chercher le soucis.

## 5.	MESURER LA DÉVIATION PPM DE VOTRE CLÉ RTL2832U  

Lancez **rtl_test** avec l’option -p, et laissez-le tourner **30 min à 1h** :

	~$ rtl_test p
	[...]
	real sample rate: 2048140 current PPM: 69 cumulative PPM: 79
	real sample rate: 2048147 current PPM: 72 cumulative PPM: 79

Ctrl+C pour arrêter rtl_test  
La déviation est par exemple ici pour moi de 79 ppm ( à mémoriser )  

## 6.	VÉRIFIER QU’ON REÇOIT BIEN QUELQUE CHOSE AVEC LA CLÉ  

	~$ rtl_fm -p 79 -f 402M -M fm -s 48k -E dc - | hexdump -Cv
	 ou 
	~$ rtl_fm -p 79 -f 402M -M fm -s 48k -E dc - | aplay -r 48k -f S16_LE
Ctrl+C pour arrêter rtl_fm 

## 7.	RÉCUPÉRER ET COMPILER LE PROGRAMME “TRAPPETTE”  

Si c'est la première fois que vous récupérez les sources :  

	~$ git clone http://github.com/Quadricopter/trappette.git
	~$ cd trappette
	~/trappette $ make

Pour mettre à jour une installation existante :  

	~$ cd trappette
	~/trappette$> git pull
	~/trappette$> make clean
	~/trappette$> make

## 8.	FICHIER DE CONFIGURATION  

Pour créer le fichier de configuration nous allons commencer par copier l'exemple fourni :  

	~/trappette $ cp trappette.cfg.sample trappette.cfg

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
 
 ## XX. Fichier READ.ME  
 
    # M10 Radiosonde Demodulation

    # -----------------------------------------------------
    # Retrieve sources and build "trappette" ( first time )
    # -----------------------------------------------------

    ~ $ git clone http://88.127.147.114:8019/git/trappette.git
    ~ $ cd trappette
    ~/trappette $ make

    # -----
    # Usage
    # -----

    # Decode from soundcard ( alsa )
    ~/trappette $ arecord -f S16_LE -c 1 -r 48k -t raw | ./trappette

    # Decode from gqrx server
    ~/trappette $ nc -l -u 7355 | ./trappette

    # Decode from WAVE file (48kHz only!)
    ~/trappette $ ./trappette -w file.wav


    # ------------------------
    # Decode from SDR RTL2832U
    # ------------------------

    # Prerequest tools
    ~ $ sudo apt-get install cmake git libusb-1.0-0-dev

    # rtl-sdr ( http://sdr.osmocom.org/trac/wiki/rtl-sdr )
    ~ $ git clone git://git.osmocom.org/rtl-sdr.git
    ~ $ cd rtl-sdr
    ~/rtl-sdr $ mkdir build
    ~/rtl-sdr $ cd build/
    ~/rtl-sdr/build $ cmake ../ -DINSTALL_UDEV_RULES=ON
    ~/rtl-sdr/build $ make
    ~/rtl-sdr/build $ sudo make install
    ~/rtl-sdr/build $ sudo ldconfig

    # rtl-sdr driver management ( https://opendesignengine.net/news/53 )
    ~ $ sudo bash ( Raspbian default password: "raspberry" )
    /home/pi# echo "blacklist dvb_usb_rtl28xxu" > /etc/modprobe.d/blacklist.conf
    /home/pi# exit

    # measure PPM derivation
    ~ $ rtl_test -p
    [...]
    real sample rate: 2048140 current PPM: 69 cumulative PPM: 79
    real sample rate: 2048147 current PPM: 72 cumulative PPM: 79

    # Decode from rtl_fm
    ~/trappette $ rtl_fm -p 75 -f 402M -M fm -s 48k -E dc - | ./trappette


    # ------------------------------------------------
    # Retrieve updated sources and rebuild "trappette"
    # ------------------------------------------------

    ~ $ cd trappette
    ~/trappette $ git pull
    ~/trappette $ make


    # -------------------------
    # Some interesting commands
    # -------------------------

    # Read SoundCard and pipe RAW stream
    ~ $ arecord -f S16_LE -c 1 -r 48k -t raw | aplay -r 48k -f S16_LE
    ~ $ arecord -f S16_LE -c 1 -r 48k -t raw | baudline -stdin -channels 1 -format le16
    ~ $ arecord -f S16_LE -c 1 -r 48k -t raw > output.raw

    # Convert RAW file to WAV file
    ~ $ sox -b 16 -e signed-integer -c 1 -r 48k -t raw infile.raw outfile.wav

    # Convert WAV file to 48k WAV file
    ~ $ sox infile.wav -r 48k -b 16 -e signed-integer -c 1 outfile.wav

    # Pipe wav file to trappette
    ~/trappette $ sox infile.wav -b 16 -e signed-integer -c 1 -r 48k -t raw - | ./trappette

    # Read RAW stream from rtl_fm
    ~/trappette $ rtl_fm -p 75 -f 402M -M fm -s 48k -E dc - | aplay -r 48k -f S16_LE
