# Remote-control-car

# **Objectif du projet**

L’objectif est de concevoir une voiture télécommandée par Bluetooth avec certaines fonctionnalités supplémentaires comme un petit laser ou une caméra amovible.

# I- **Assemblage de la voiture**

## a) Assemblage du kit

Utilisation d’un kit avec deux supports plastiques, quatre roues et moteurs, un support de piles et de la visserie.

Assemblage selon la figure ci-dessous.

![Image2](https://user-images.githubusercontent.com/92324336/139718660-0ff360cb-2888-408b-87ac-ae516c1c5b0f.png)

## b) Assemblage supplémentaire

Placement du support de piles sur le devant de la voiture et d’un MotorShield pour Nodemcu à l’arrière. Ajout d’un support Lego pour déposer la caméra et d’autres éléments comme le laser.

Assemblage selon la figure ci-dessous.

![Image1](https://user-images.githubusercontent.com/92324336/139718817-14d26911-e2a1-4050-9552-0c2f32952d2a.jpg)

# II- **Conception électronique**

## a) Module Bluetooth 

L’objectif n’est pas de connecter la voiture par wifi et de la contrôler sur Blynk, mais plutôt de privilégier une connexion par Bluetooth et de contrôler le véhicule avec une manette. La communication entre le module et la carte se fait par le protocole UART.

Pour cela, nous avons besoin d’un module HC-05 (maitre et esclave) et d’un module HC-06 (esclave). Il est aussi possible d’utiliser deux modules HC-05.

### Configuration du module

Nous allons mettre en place la configuration AT avec Arduino en envoyant des commandes spécifiques à l'appareil Bluetooth pour configurer ses paramètres, tels que le nom de l'appareil, le mot de passe, le taux de bauds, et d'autres options de connectivité.

**Les branchements :**

Pour configurer le module, les branchements ne sont pas les mêmes qu’en communication : 

Le module HC-05 présente 6 broches et le pin Key ou EN doit être alimentée pour entrer dans le mode de configuration AT, et ne doit pas être connecté pour être en mode communication. Pour le HC-05, le baudrate par défaut en mode AT est 38400 bauds

Le HC-06 n'a que 4 broches et le pin KEY n’y est pas. C’est normal car ses fonctionnalités s’arrête à être esclave. Le baudrate par défaut pour la communication avec le HC-06 est 9600 bauds.

Nous commençons par la configuration de l'esclave dans un terminal série:

- AT+NAME=HC05-Slave

- AT+ROLE=0 // On ne peut pas modifier le rôle du HC-06

- AT+ADDR pour obtenir l’adresse du module esclave

Pour le maitre :

- AT+NAME=HC05-Master

- AT+ROLE=1

- Nous enregistrons l’adresse du module esclave pour que le module maître puisse s’appairer : AT+BIND=adresse esclave

### Entrer en communication

Lorsque les modules sont configurés, on peut débrancher le pin KEY. Nous utilisons la bibliothèque SoftwareSerial pour ajouter une communication série.  L’esclave reçoit des informations du maitre donc dès que celui-ci reçoit une donnée, il la lit et la traite. Le maître envoie des données selon certaines conditions :

La manette connectée à l'Arduino envoie des informations au module Bluetooth "maitre" par UART. Les deux modules étant configurés pour communiquer, le module "esclave" reçoit l’information et la partage à son tour à la Nodemcu grâce aux ports RX TX.

## b) Connexion MotorShield Nodemcu

### Les moteurs

Nous possédons 4 moteurs mais le MotorShield uniquement deux ponts en H pour contrôler la vitesse et le sens des moteurs. Nous allons donc devoir brancher dans un emplacement du MotorShield deux moteurs à la fois.

Pour alimenter les moteurs et la carte, on utilise un boitier de piles 6V que l’on branche selon l’image suivante :

![Image3](https://user-images.githubusercontent.com/92324336/139718881-2b7898d0-e0a1-4e5e-aad1-11aacb46a1c8.jpg)

### Les modules connectés

- Le module Bluetooth lié au MotorShield (l’esclave) est branché sur le pin VIN pour avoir suffisamment de puissance, au ground et à un pin digital.

- Le laser est branché au 3.3V, au ground et à un pin digital.

- Les servomoteurs sont branchés chacun à un pin digital PWM, au Ground et au 3.3V.

## c) Connexion Shield manette Arduino

La manette est un Shield connecté directement à un Arduino et permet d’avoir directement des joysticks et des boutons utilisables. Un module Bluetooth maitre est branché dessus.

Pour en savoir plus : https://www.google.com/search?q=joystick+shield+pinout&rlz=1C1CHBF_frFR911FR911&sxsrf=ALeKk02XzmC-Yngf9aDbZE6Xqd2bFflECQ%3A1623847269479&ei=ZfHJYN3gHISMa8f1gagO&oq=joystick+shield+&gs_lcp=Cgdnd3Mtd2l6EAEYBzIECCMQJzIECCMQJzIECCMQJzICCAAyAggAMgUIABDLATIFCAAQywEyBQgAEMsBMgUIABDLATIFCAAQywFQripYripg0EdoAHABeACAAWmIAYwCkgEDMi4xmAEAoAEBqgEHZ3dzLXdpesABAQ&sclient=gws-wiz

![Image4](https://user-images.githubusercontent.com/92324336/139719004-9611d1cf-ad13-4df8-b5bd-ea5e005d1385.jpg)

# III- **Conception informatique**

## a) Envoie des données 

En fonction de la position du joystick, on envoie un caractère au module esclave.

```arduino
if(x >= 150 && x <=570 && y>=650 && y<= 689) 
		ArduinoSlave.print('A');
```

De même pour l’appui d’un bouton :

```arduino
if(valeurA==LOW)
	ArduinoSlave.print('1');
```

Le laser et les servomoteurs sont contrôlés par l’appui de boutons

## b) Réception des données 

Tous les éléments sont contrôlés par le module esclave.

On regarde tout d’abord si le port reçoit une information :
``` arduino
if (ArduinoMaster.available() >0) 
    char c = ArduinoMaster.read();
```
On exécute ensuite une action comme l'activation des moteurs, du laser ou des servomoteurs.

```arduino
case 'A':

digitalWrite(RightMotorDir,HIGH);
digitalWrite(LeftMotorDir,HIGH);
digitalWrite(RightMotorSpeed,HIGH);
digitalWrite(LeftMotorSpeed,HIGH);
break;
```

![Image5](https://user-images.githubusercontent.com/92324336/139719094-c4b0a5d5-fff3-431a-8349-de97b54ccbb1.gif)


# IV- **Mise en place caméra**

La caméra vient se rajouter devant la voiture. C’est un module qui ne se connecte pas directement à l’Arduino mais dont les images sont retransmises sur une ip locale et que l’on peut donc visualiser sur un téléphone.

On souhaite obtenir un support permettant à la caméra de tourner à gauche et à droite ainsi que de se lever ou baisser.

![Image6](https://user-images.githubusercontent.com/92324336/139719058-3ed2ca72-dc3b-4ebb-b729-651a8f5118da.png)

- Le 1er support sert à placer la caméra à l’intérieur. Un servomoteur est collé sur le côté pour pouvoir la lever ou la baisser.

- Le trou du 2nd support permet de maintenir en place ce servomoteur. Ce support sert également à faire tourner la caméra grâce à un autre servomoteur placé en dessous.

![Image7 (1)](https://user-images.githubusercontent.com/92324336/139719375-81103dfe-122c-49fb-989f-257e91e0d35a.gif)
