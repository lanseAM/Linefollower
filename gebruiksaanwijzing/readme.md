# Gebruiksaanwijzing

### opladen / vervangen batterijen
uitleg over het opladen of vervangen van de batterijen
In steken en met micro USB opladen.
![image](https://github.com/lanseAM/Linefollower/assets/114751410/97e39f65-3147-47e0-aa7f-c5c54bca66fe)

https://www.batterijimport.nl/blog/post/10-must-know-veiligheidsissues-over-li-ion-accu-s


### draadloze communicatie
#### verbinding maken
uitleg over het verbinden van de robot met laptop / smartphone

1) Open de app op je phone   

![image](https://github.com/lanseAM/Linefollower/assets/114751410/f5babd2b-af57-4305-b93a-ec7b08a40322)

2) Bij "Devices" zoek je naar de HC-05 kijk zeker of je bluetooth aanstaat en moest deze er niet bij staan probeer er eerst zonder app verbinding mee te maken dan vraagt die een code vul "1234" of "0000" in. Ga vervolgens terug in de app.
   
![image](https://github.com/lanseAM/Linefollower/assets/114751410/6eeb92fe-5e6c-4ab2-948a-d89bfd89e407)

3) Klik op het apparaat

![image](https://github.com/lanseAM/Linefollower/assets/114751410/48c071fe-e5d5-46f3-ada2-a7e69641bd46)

4) Groene lijn naast om te weten of deze aan staat en verbinding kan maken.
Nu zou je vanzelf in de monitor moeten komen en krijg je "ready".
Probeer een commando uit "debug" om te zien of je response krijgt.

#### commando's
debug [on/off]  --> Je krijgt de onderstaande parameters te zien die laatst waren ingesteld, de positie van de sensor tov de zwarte lijn en de calculation time

start  --> LF begint te rijden

stop  --> LF stopt met rijden

set cycle [µs]  --> Bepalen hoelang er min aan 1 procedure mag gewerkt worden

set power [0..255]  --> Snelheid, hoe hoger hoe sneller LF zal rijden

set diff [0..1]  --> Nauwkeurigheid van het volgen van de lijn in een bocht, hoe hoger des te meer toegeeflijk behandelt hij de kromming van de lijn in de bocht.

set kp [0..]  --> Hierbij mag die sterk bijsturen in het geval van een grote fout of dus afwijking. Mist de LF vaak zijn bocht dan mag dit hoger gezet worden.

set ki [0..]  --> Kijkt hoelang een fout zich blijft voordoen tijdens een afwijking, hoe langer de duur hoe sterker of dus sneller LF zal bijregelen.

set kd [0..]  --> In dit geval zal de lijnvolger analyseren hoe snel de fout is veranderd ten opzichte van de vorige cyclus. Indien het verschil met de fout van de vorige cyclus aanzienlijk is, zal er een krachtige bijsturing plaatsvinden. Als daarentegen het verschil relatief klein is, zal de bijsturing milder zijn.

calibrate black  --> Leert de sensor zwart herkennen

calibrate white  --> Leert de sensor wit herkennen

### kalibratie
uitleg kalibratie  

Houdt de QTR-8A sensor boven het zwarte oppervlak en eens verbonden in de bluetooth monitor geef je het commando "calibrate black" door.
Experimenteer met de hoogte van de sensor boven zwart en controleer met debug of de waarden bij "Black" richting de 1000 gelegen zijn.
Nu doe je hetzelfde voor wit met commando "calibrate white" en controleer je opnieuw met gewenste waarden onder 50 richting 20.
![image](https://github.com/lanseAM/Linefollower/assets/114751410/9b1003ad-43df-4e05-a908-0fdfc0de33e8)



### settings
De robot rijdt stabiel met volgende parameters:  
power 96; kp 13.25; ki 2.5; kd 3; diff 0.02

### start/stop button
uitleg locatie + werking start/stop button

De knop zit naast de schakelaar om de batterijspanning door het circuit te laten stromen. Bij een eerste klik zal LF starten mits batterijspanning aanstaat en 2de klik om te stoppen.
