!pip install pyserial

import serial
import time
import matplotlib.pyplot as plt


configurer le port serie

serialPort = serial.Serial() # créer une instance, création et configuration port série
serialPort.baudrate = 115200 # vitesse
serialPort.parity = serial.PARITY_NONE # aucun bit de parité
serialPort.bytesize = serial.EIGHTBITS # taille d'un caractère : 8 bits
serialPort.stopbits = serial.STOPBITS_ONE # 1 bit de stop
serialPort.port = "/dev/cu.usbmodem34B7DA6494902" # nom/chemin du port série à utiliser

ouvir le port serie

serialPort.open() # active la liaison série avec la carte

envoyer le code 'r' soit 114

cmd = 'r' # caractère à envoyer pour démarrer la carte
serialPort.write(cmd.encode(encoding = "ascii")) # conversion en octer ASCII puis envoi sur le port série
serialPort.flush() # Etape 4 - s'arruer que la commande 'r' est bien arrivé
on laisse tourner l'essai pendant envrion 10sec

tStart = time.perf_counter() # instant de référence
tEcoule = 0 # initalise à 0 seconde
times = [] # endroit ou se stocker le temps
bruts = [] # step 5 - stockage valeurs brutes FSR
while tEcoule < 10: # si le temps tEcoule est inférieur à 10, cela continu dans la boucle
tEcoule = time.perf_counter() - tStart # Quand tEcoule est = à 10, cela s'arrete
ligne = serialPort.readline() # Etape 4 - lire la ligne envoyé par l'Arduino
if ligne :
try :
# valeur_us = int(ligne.decode(encoding="ascii").strip())
# times.append(valeur_us / 1000000) # Etape 4 - accumlulation en seconde car en micro sur Arduino
morceaux = ligne.decode(encoding="ascii").strip().split(";") # Step 5 - "1234567;400" -> ["1234567", "400"]
valeur_us = int(morceaux[0]) # Step 5 - le temps en µs
brut = int(morceaux[1]) # Step 5 - la valeur brute du FSR
times.append(valeur_us / 1000000) # Step 5 - temps en secondes
bruts.append(brut) # Step 5
except (ValueError, IndexError): # Step 5 - IndexError si une ligne est incomplète
pass
envoyer le code 's' soit 115 : déclanche le software reset côté Arduino

cmd = 's' # caractère à envoyer pour déclancher le reset
serialPort.write(cmd.encode(encoding = "ascii", errors="ignore")) # conversion en octet ASCII puis envoi sur le port série
serialPort.flush() # Etape 4 - s'arruer que la commande 's' est bien arrivé
fermer le port serie

serialPort.close() # libère le port séries




Step 5 - tracer l'acquisition

plt.plot(times, bruts)
plt.xlabel("Temps (s)")
plt.ylabel("Valeur brute FSR (0-1023)")
plt.title("Acquisition capteur FSR")
plt.grid(True)
plt.show()
