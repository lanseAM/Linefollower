# Gebruiksaanwijzing

### opladen / vervangen batterijen
uitleg over het opladen of vervangen van de batterijen
In steken en met micro USB opladen.
![image](https://github.com/lanseAM/Linefollower/assets/114751410/97e39f65-3147-47e0-aa7f-c5c54bca66fe)

https://www.batterijimport.nl/blog/post/10-must-know-veiligheidsissues-over-li-ion-accu-s


### draadloze communicatie
#### verbinding maken
uitleg over het verbinden van de robot met laptop / smartphone


#### commando's
debug [on/off]  
start  
stop  
set cycle [µs]  
set power [0..255]  
set diff [0..1]  
set kp [0..]  
set ki [0..]  
set kd [0..]  
calibrate black  
calibrate white  

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
