# automatic-octo-telegram
Number guess game for beginner Python 
import random
import time
import json
import os
import sys


# --- PFAD FINDEN ---
# 1. Wo liegt diese Python-Datei gerade?
skript_ordner = os.path.dirname(os.path.abspath(__file__))
# 2. Wir basteln den Pfad für die Highscore-Datei zusammen
filename = os.path.join(skript_ordner, "highscores.json")

print(f"Speichere Highscores in: {filename}") # Zur Kontrolle anzeigen

try:
        with open(filename, "r") as datei:
            geladene_daten = json.load(datei)
            
            # CHECK: Ist das überhaupt eine Liste?
            if isinstance(geladene_daten, list):
                highscores = geladene_daten
                
                # CHECK 24: Prüfen, ob die Einträge "gesund" sind
                # Wir schauen uns den ersten Eintrag an (falls es einen gibt)
                if len(highscores) > 0:
                    erster_eintrag = highscores[0]
                    # Ein Eintrag muss eine Liste sein, z.B. [3, 12]
                    # Wenn es nur eine Zahl ist (z.B. 3), ist die Datei "kaputt"
                    if not isinstance(erster_eintrag, list):
                        print(" Altes Speicherformat erkannt! Liste wird zurückgesetzt.")
                        highscores = []
            else:
                print(" Datei war beschädigt. Starte neue Liste.")
                highscores = []
                
except FileNotFoundError:
    highscores = []
except json.JSONDecodeError:
        print(" Datei war leer oder kaputt. Starte neue Liste.")
        highscores = []
    



if os.path.exists(filename):
    print("DEBUG:  Datei wurde gefunden!")
    try:
        with open(filename, "r") as datei:
            # Wir lesen ALLES roh ein, um zu sehen, was drin steht
            raw_data = datei.read()
            print(f"DEBUG: Roher Inhalt der Datei: '{raw_data}'")
            
            if not raw_data:
                print("DEBUG:  Die Datei ist leer!")
                highscores = []
            else:
                # Jetzt versuchen wir es zu verstehen
                geladene_daten = json.loads(raw_data)
                print(f"DEBUG: Als Liste erkannt: {geladene_daten}")
                highscores = geladene_daten
                
    except Exception as e:
        print(f"DEBUG:  Fehler beim Lesen: {e}")
        highscores = []
else:
    print("DEBUG:  Keine Datei gefunden. Es wird eine neue angelegt.")


# --- ÄUSSERE SCHLEIFE (Das Programm) ---
while True:
    
    # 1. Alles für eine neue Runde vorbereiten
    geheime_zahl = random.randint(1, 100)
    max_versuche = 7 # Versuche
    zeit_limit = 60 # Sekunden
    versuche = 0   # Zähler für die Versuche

    # NEU: Hier merken wir uns den vorherigen Tipp
    letzter_tipp = None
    
    print("\n" + "="*40)
    print("Willkommen zum Zahlen-Rate-Spiel!")
    print(f"Ich habe mir eine Zahl zwischen 1 und 100 ausgedacht.")
    print(f"Du hast {max_versuche} Versuche und {zeit_limit} Sekunden Zeit, um sie zu erraten.")
    print("\n" + "="*40) # Macht eine schöne Trennlinie
    print("       HALL OF FAME (TOP 5)       ")
    print("-" * 40)

    # --- NEU: Schöne Ausgabe der Liste ---
    if not highscores:
        print("Noch keine Einträge. Sei der Erste!")
    else:
        # Wir gehen durch die ersten 5 Einträge
        for i, eintrag in enumerate(highscores):
            if i >= 5: break
            # Kleiner Check, damit es nicht crasht
            if isinstance(eintrag, list):
                print(f"   Platz {i+1}: {eintrag[0]} Versuche ({eintrag[1]}s)")
    # -------------------------------------


    print("\n" + "="*40)
    if highscores:
        print(f" BESTENLISTE (Versuche | Sekunden): {highscores}")
    else:
        print(" Noch keine Rekorde.")
    
    print("="*40)
    
    
    input("Drücke ENTER zum Starten...")
    start_zeit = time.time() # Zeit startet erst hier neu!


    # --- INNERE SCHLEIFE (Die Raterunde) ---
    while True:
        eingabe = input("Dein Tipp: ")
        
        # Zeit prüfenj

        aktuelle_zeit = time.time()
        vergangene_zeit = aktuelle_zeit - start_zeit
        verbrauchte_sekunden = int(vergangene_zeit)

        if vergangene_zeit > zeit_limit:
            print(f"Zeit abgelaufen! Du hast {verbrauchte_sekunden}s gebraucht.")
            print(f"Die Zahl war {geheime_zahl}.")
            break # Bricht die INNERE Schleife ab (Runde vorbei)

        # Fehler abfangen
        try:
            tipp = int(eingabe)
        except ValueError:
            print("Bitte eine Zahl eingeben!")
            continue

        versuche = versuche + 1
        
        # Gewonnen?
        if tipp == geheime_zahl:
            print(f" GEWONNEN! Du hast {versuche} Versuche und {verbrauchte_sekunden}s gebraucht.")

            # Highscore speichern
            highscores.append(verbrauchte_sekunden) # Zeit speichern
            highscores.append(versuche) # Versuche speichern
            highscores.sort() # Sortieren, damit der beste Score vorne ist
            
            

             # --- TEIL 2: Speichern (Auf Festplatte schreiben) ---
            # Wir öffnen die Datei zum Schreiben ("w" = write)
            try:

                with open(filename, "w") as file:
                    json.dump(highscores, file, indent=4)
                    print(f"Highscore gespeichert in {filename}.")
            except Exception as e:
                # Hier fangen wir den Absturz ab und zeigen den Fehler an
                print(f"FEHLER beim Speichern: {e}")

            break # Bricht die INNERE Schleife ab

             
        
        # Game Over (Versuche)?
        if versuche >= max_versuche:
            print(f" Keine Versuche mehr! Die Zahl war {geheime_zahl}.")
            break # Bricht die INNERE Schleife ab


        richtung = ""
        if tipp < geheime_zahl:
            richtung = "Zu niedrig "
        else:
            richtung = "Zu hoch"

        # 2. Warm/Kalt Logik (Vergleich zum letzten Mal)
        temperatur = "Aber nahe dran"
        if letzter_tipp is not None:
            # Wir berechnen die Abstände (absolute Differenz)
            abstand_alt = abs(geheime_zahl - letzter_tipp)
            abstand_neu = abs(geheime_zahl - tipp)

            if abstand_neu < abstand_alt:
                temperatur = "aber WÄRMER"
            elif abstand_neu > abstand_alt:
                temperatur = "und KÄLTER"
            else:
                temperatur = "Aber sehr nah dran"
        
        print(f"{richtung} {temperatur}")

        # WICHTIG: Den jetzigen Tipp für die nächste Runde merken
        letzter_tipp = tipp

        # Tipps geben
       

    # --- HIER SIND WIR WIEDER IN DER ÄUSSEREN SCHLEIFE ---
    # Die Runde ist vorbei. Jetzt fragen wir nach einer neuen Runde.
    
    nochmal = input("\nMöchtest du nochNochmal spielen? (j/n): ")
    
    # Wir machen die Eingabe klein (.lower), damit "J" und "j" funktionieren
    if nochmal.lower() != "j":
        print(f" Deine Bestscores: {highscores}")
        print("Danke fürs Spielen! Bis zum nächsten Mal.")
        break # Bricht die ÄUSSERE Schleife ab -> Programm endet

