Kiln Controller
==========

Transforme un Raspberry Pi en un contrôleur de four à poterie et connecté à Internet.

## Caractéristiques

  * Compatible avec de [nombreuses cartes](https://github.com/greenxav/kiln-controller/blob/main/docs/supported-boards.md) en plus du Raspberry Pi
  * Compatible avec les cartes thermocouples Adafruit MAX31856 et MAX31855
  * Compatible avec les thermocouples de type K, J, N, R, S, T, E et B
  * Création et modification simplifiées des programmes de cuisson
  * 
  * Durée de fonctionnement illimitée : cuisson pendant plusieurs jours
  * Visualisation simultanée de l'état du programme depuis plusieurs appareils (ordinateur, tablette, etc.)
  * Estimation du coût de cuisson en temps réel
  * Affichage de la vitesse de chauffe en temps réel (degrés par heure)
  * Compatible avec les paramètres PID personnalisables pour votre four
  * Surveillance de la température du four après la fin du programme
  * API pour démarrer et arrêter un programme à tout moment
  * Simulation précise
  * Possibilité de décaler le programme si le four ne chauffe pas assez rapidement
  * Possibilité de sauter la première partie du profil pour correspondre à la température actuelle du four
  * Empêche l'emballement du programme lorsque les températures ne sont pas suffisantes À proximité du point de consigne
  * Redémarrages automatiques en cas de coupure de courant ou autre incident
  * Possibilité de vous alerter via Slack en cas de dysfonctionnement du four
  * Programmation simplifiée des cycles de cuisson futurs


**Programme d'exécution du four**

![Image](https://github.com/greenxav/kiln-controller/blob/main/public/assets/images/kiln-running.png)

**Edition des courbes de chauffe**

![Image](https://github.com/greenxav/kiln-controller/blob/main/public/assets/images/kiln-schedule.png)

## Hardware

### Parties

| Image | Hardware | Description |
| ------| -------- | ----------- |
| ![Image](https://github.com/greenxav/kiln-controller/blob/main/public/assets/images/rpi.png) | [Raspberry Pi](https://www.adafruit.com/category/105) | Pratiquement n'importe quel Raspberry Pi fonctionnera, car seules quelques broches GPIO sont utilisées. Toute carte prise en charge par [blinka](https://circuitpython.org/blinka) et SPI devrait fonctionner. Vous voudrez également vous assurer que la carte dispose du wifi. Si vous utilisez autre chose qu'un Raspberry PI et que vous le faites fonctionner, faites-le moi savoir. |
| ![Image](https://github.com/greenxav/kiln-controller/blob/main/public/assets/images/max31855.png) | [Adafruit MAX31855](https://www.adafruit.com/product/269) or [Adafruit MAX31856](https://www.adafruit.com/product/3263) | Carte pour mesure du Thermocouple |
| ![Image](https://github.com/jbruce12000/kiln-controller/blob/main/public/assets/images/k-type-thermocouple.png) | [Thermocouple](https://www.auberins.com/index.php?main_page=product_info&cPath=20_3&products_id=39) | Investissez dans un thermocouple céramique haute performance conçu pour les fours. Assurez-vous de sa compatibilité avec votre carte de thermocouple. La carte Adafruit-MAX31855 fonctionne uniquement avec les thermocouples de type K. La carte Adafruit-MAX31856 est plus polyvalente et compatible avec de nombreux types, mais le type S est généralement privilégié. |
| ![Image](https://github.com/jbruce12000/kiln-controller/blob/main/public/assets/images/breadboard.png) | Plaque d'essai | Plaque d'essai, câble, connecteur pour broches GPIO du Raspberry Pi et fils de connexion |
| ![Image](https://github.com/jbruce12000/kiln-controller/blob/main/public/assets/images/ssr.png) | Relais statique | Passage par zéro : assurez-vous qu'il supporte le courant maximal de votre four. Même pour un four 220 V, un [relais statique triphasé suffit](https://www.auberins.com/index.php?main_page=product_info&cPath=2_30&products_id=331). C'est comme avoir 3 relais statiques en un. Les relais de cette taille nécessitent toujours un dissipateur thermique. |
| ![Image](https://github.com/jbruce12000/kiln-controller/blob/main/public/assets/images/ks-1018.png) | Four électrique | On trouve encore sur le marché de nombreux fours électriques anciens sans commande numérique. Vous pouvez en trouver d'occasion à petit prix. Ce contrôleur fonctionne en 110 V ou 220 V (choisissez un relais statique adapté). |

### Schématique

Le Raspberry Pi possède trois broches GPIO connectées à la puce MAX31855. La broche D0 est configurée en entrée, tandis que les broches CS et CLK sont des sorties. Le signal commandant le relais statique est initialement une sortie GPIO qui pilote un transistor faisant office d'interrupteur. Ce transistor fournit une tension de 5 V et un courant suffisant pour commander le relais statique. Comme seules quatre broches GPIO sont utilisées, n'importe quel Raspberry Pi peut convenir pour ce projet. Voir [le fichier configuration](https://github.com/jbruce12000/kiln-controller/blob/main/config.py) pour la configuration des broches GPIO.

Mon contrôleur se branche sur la prise murale, et le four se branche sur le contrôleur.

**AVERTISSEMENT** Ce projet implique des hautes tensions et de forts courants. Veuillez vous assurer que tout ce que vous construisez est conforme aux codes électriques locaux et respecte les meilleures pratiques de l'industrie.

**Remarque :** La configuration GPIO dans ce schéma ne correspond pas aux valeurs par défaut, vérifiez la configuration et assurez-vous que la configuration des broches GPIO correspond à vos connexions réelles.

![Image](https://github.com/jbruce12000/kiln-controller/blob/main/public/assets/images/schematic.png)

*Remarque : j'ai essayé d'alimenter mon SSR directement en utilisant une broche GPIO, mais cela n'a pas fonctionné. Mon SSR nécessitait 25 mA pour commuter et le GPIO du RPi ne pouvait fournir que 16 mA. Votre expérience peut varier.*

## Logiciel

### Raspberry PI OS

Téléchargez [Raspberry PI OS](https://www.raspberrypi.org/software/). Utilisez l'outil d'imagerie Raspberry PI pour installer le système d'exploitation sur une carte SD. Pour moi, c'est Bookworm 64 ou 32 bits sur Raspberry Pi 3 B+.
Démarrez le système d'exploitation, ouvrez un terminal et... 

    sudo apt-get update
    sudo apt-get install build-essential python3-dev
    sudo apt-get install git
    sudo apt-get dist-upgrade
    git clone https://github.com/greenxav/kiln-controller
    cd kiln-controller
    python3 -m venv venv
    source venv/bin/activate
    pip install -r requirements.txt

*Remarque : Les étapes ci-dessus fonctionnent sur Ubuntu si vous préférez*

### Déploiement Raspberry Pi

Si vous avez fini de vous amuser avec les simulations et que vous voulez déployer le code sur un Raspberry PI pour contrôler un four, vous devrez faire cela en plus des éléments listés ci-dessus :

    sudo raspi-config
    
    interfacing options -> SPI -> Select Yes to enable
    select reboot

## Configuration

All parameters are defined in config.py. You need to read through config.py carefully to understand each setting. Here are some of the most important settings:

| Variable | Default | Description |
| -------- | ------- | ----------- |
| sensor_time_wait | 2 seconds | It's the duty cycle for the entire system.  It's set to two seconds by default which means that a decision is made every 2s about whether to turn on relay[s] and for how long. If you use mechanical relays, you may want to increase this. At 2s, my SSR switches 11,000 times in 13 hours. |
| temp_scale | f | f for farenheit, c for celcius |
| pid parameters | | Used to tune your kiln. See PID Tuning. |
| simulate | True | Simulate a kiln. Used to test the software by new users so they can check out the features. |
 

## Testing

After you've completed connecting all the hardware together, there are scripts to test the thermocouple and to test the output to the solid state relay. Read the scripts below and then start your testing. First, activate the virtual environment like so...

     source venv/bin/activate

then test the thermocouple with:

     ./test-thermocouple.py

then test the output with:

     ./test-output.py

and you can use this script to examine each pin's state including input/output/voltage on your board:

     ./gpioreadall.py

## PID Tuning

Run the [autotuner](https://github.com/jbruce12000/kiln-controller/blob/main/docs/ziegler_tuning.md). It will heat your kiln to 400F, pass that, and then once it cools back down to 400F, it will calculate PID values which you must copy into config.py. No tuning is perfect across a wide temperature range. Here is a [PID Tuning Guide](https://github.com/jbruce12000/kiln-controller/blob/main/docs/pid_tuning.md) if you end up having to manually tune.

There is a state view that can help with tuning. It shows the P,I, and D parameters over time plus allows for a csv dump of data collected. It also shows lots of other details that might help with troubleshooting issues. Go to /state.

## Usage

### Server Startup

    source venv/bin/activate; ./kiln-controller.py

### Autostart Server onBoot
If you want the server to autostart on boot, run the following command:

    /home/xavier/kiln-controller/start-on-boot

### Client Access

Click http://127.0.0.1:8081 for local development or the IP
of your PI and the port defined in config.py (default 8081).

### Simulation

In config.py, set **simulate=True**. Start the server and select a profile and click Start. Simulations run at near real time.

### Scheduling a Kiln run

If you want to schedule a kiln run to start in the future. Here are [examples](https://github.com/jbruce12000/kiln-controller/blob/main/docs/scheduling.md).

### Watcher

If you're busy and do not want to sit around watching the web interface for problems, there is a watcher.py script which you can run on any machine in your local network or even on the raspberry pi which will watch the kiln-controller process to make sure it is running a schedule, and staying within a pre-defined temperature range. When things go bad, it sends messages to a slack channel you define. I have alerts set on my android phone for that specific slack channel. Here are detailed [instructions](https://github.com/jbruce12000/kiln-controller/blob/main/docs/watcher.md).

## License

This program is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with this program.  If not, see <http://www.gnu.org/licenses/>.

## Support & Contact

Please use the issue tracker for project related issues.
If you're having trouble with hardware, I did too.  Here is a [troubleshooting guide](https://github.com/jbruce12000/kiln-controller/blob/main/docs/troubleshooting.md) I created for testing RPi gpio pins.

## Origin
This project was originally forked from https://github.com/apollo-ng/picoReflow but has diverged a large amount.
