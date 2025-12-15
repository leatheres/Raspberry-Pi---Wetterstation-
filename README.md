# Raspberry-Pi---Wetterstation-
Code zur wissenschaftlichen Arbeit
 

Code: 

import adafruit_dht
import board
import matplotlib.pyplot as plt
import matplotlib.animation as animation 
import time
import csv
import os 

# sensor initialisieren 
dht = adafruit_dht.DHT11(board.D4)

#stratzeit Merken 
startzeit = time.time()

#Listen für die Messwerte 
zeiten  = []
temperaturen = []
feuchten = []

#CSV Datei definieren 
csv_datei = "wetterdaten.csv"

if not os.path.exists(csv_datei):
    with open(csv_datei, "w", newline="") as f:
        writer =csv.writer(f)
        writer.writerow(["sekunden_seite_start", "temperatur", "luftfeuchtigkeit"])
        
#Diagramm einrichten 
plt.style.use("ggplot")
fig, ax1 = plt.subplots()

#zweite y-Achse für Luftfeuchtigkeit
ax2 = ax1.twinx()

line_temp, = ax1.plot([],[], label="Temperatur (°C)", color="red", linewidth=2)
line_hum, = ax2.plot([],[], label="Luftfeuchtigkeit(%)", color="blue", linewidth=2)

ax1.set_xlabel("Zeit(Sekunden seit Messbeginn)")
ax1.set_ylabel("Temperatur (°C)")
ax2.set_ylabel("Luftfeuchtigkeit (%)")
plt.title("Live Wetterstation - Temperatur& Luftfeuchtigkeit(2 Stunden Ansicht)")

#x-Achse fest auf zwei stunden setzen (7200s)
ax1.set_xlim(0, 7200)

def update(frame):
    try: 
        temp = dht.temperaturen
        hum = dht.humidity
        jetzt = time.time()
        
        # Zeit relativ zu Messbeginn statt absolute Uhrzeit
        zeit_relativ = jetzt - startzeit
        
        if temp is not None and hum is not None:
            print(f"(time.strftime('%H:%M:%S')}-> {temp}°C, {hum}% ({zeit_relativ:.1f}s)")
            
            #Messerte Speichern
            zeiten.append(zeit_relativ)
            temperaturen.apped(temp)
            feuchten.append(hum)
            
            #CSV Speicherung integrieren
            with open (csv_datei, "a", newline="") as f:
                writer = csv.writer(f)
                writer.writerow([zeit_relativ, tem, hum])
                
            #linien aktualisieren 
            line_temp.set_data(zeiten, temperaturen)
            line_hum.set_data(zeiten, feuchten)
            
            #y-Achsen automatisch anpassen 
            ax1.set_ylim(min(temperaturen) - 1, max(temperaturen) +1)
            ax2.set_ylim(min(feuchten) -5, max(feuchten) + 5)
    
    except Exception as e:
        print("Fehler beim Auslesen:", e)
        
    return line_temp, line_hum
   
#Animation starten
ani = animatiion.FuncAnimation(fig, update, interval=1000)
plt.show()
                
