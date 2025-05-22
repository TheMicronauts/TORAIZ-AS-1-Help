# Pioneer DJ Dave Smith Instruments TORAIZ AS-1 (work in progress)
## MIDI Cheat Sheet &amp; Preset Packs

The TORAIZ AS-1 is an analog monosynth with full digital control, designed by Dave Smith Instruments (now [Sequential](https://sequential.com/)) for [Pioneer DJ](https://www.pioneerdj.com/). 

The synthesis engine and signal path are entirely analog, except for an optional digital multi FX. In essence, it offers a single voice from the Prophet-6, housed in an enclosure with a form factor reminiscent of a classic silver box.

It was apparently designed for live use, but didn’t really catch on because of [sync](https://gearspace.com/board/electronic-music-instruments-and-electronic-music-production/1244592-pioneer-toraiz-as1-midi-out-issues.html) [issues](https://gearspace.com/board/showpost.php?p=15940161&postcount=1). In the studio, where it can be played and automated as a sound module via the DAW, it’s excellent.

It was reviewed by [*Sound On Sound*](https://www.soundonsound.com/reviews/pioneer-dj-toraiz-1) in November 2017.

![2017-10-25 17 13 47](https://github.com/user-attachments/assets/67185c02-7175-4bcf-a8d5-be0f94a32024)
*TORAIZ AS-1*

![2022-11-07 15 16 13](https://github.com/user-attachments/assets/7d2e4497-e7b0-4cc4-be8c-169a956db544)
*A classic silver box*

### MIDI Cheat Sheet

The TORAIZ AS-1 MIDI implementation is comprehensive. All sound parameters can be set and automated directly from the DAW. They are listed in this Google Sheets table along with their corresponding controls and value ranges:

https://docs.google.com/spreadsheets/d/1XDerLaoKoy6zsbu0w4pXwNQaluc6XFW1W9XfXDOYYyM

### Reset And Control MIDI Parts For Cubendo

This XML file contains two one-bar MIDI Parts (patterns), ready to be imported into Cubase or Nuendo (via *File > Import > Track Archive…*); it includes all the MIDI messages that control the TORAIZ AS-1’s sound parameters, with names and comments entered as Text / Score Events: 

I usually place this type of pattern at the beginning of the main MIDI track controlling a synth. Playing it resets the performance parameters, the synth configuration, and restores the base sound for the song—without needing a SysEx dump. 

Thanks to the Acoustic Feedback function in Cubendo’s List Editor, I also use it to remotely program the patch, without having to navigate through the synth’s menus and submenus.

To avoid accidentally losing an important setting, the MIDI messages that directly control the patch are initially muted; they must therefore be unmuted in order to modify the synth’s settings.

<img width="1726" alt="AS-1 rà0" src="https://github.com/user-attachments/assets/c7efe647-aec5-462a-95ca-e7ffa7f0f65a" />     

*Cubendo’s List Editor (best in class – other DAWs, seriously, take note, copy it, improve it if you can)*

### Converting NRPNs Into Single CCs

All of the parameters of this synth can be automated via MIDI using NRPNs (right half of the table), and a good number of them can also be automated using a simple CC (left half of the table).

Now, a single CC is quicker to visualise and edit than its NRPN equivalent (which uses a group of four consecutive CCs). Moreover, the CC#s of a specific NRPN can conflict on the MIDI bus with the same CC#s used by other NRPNs, preventing the synth from recalling the intended sound.

Fortunately, when the parameter can be controlled by both methods, it is possible, if needed, to convert NRPNs into simple CCs in the MIDI sequence.

#### A) If the parameter’s value range is ≤ 127 (column L of the table):

1. delete CC#99, 98, and 6

2. convert CC#38 into the CC whose number matches the target parameter (column E)

The conversion is straightforward and doesn’t cause any resolution loss.

#### B) If the parameter’s value range is > 127 (column L):

1. split into two distinct lanes or patterns the values ≤ 127 (accessed via NRPN with CC#6 = 0), and the values > 127 (accessed via NRPN with CC#6 = 1)

2. delete CC#99, 98, and 6

3. convert CC#38 into the CC whose number matches the target parameter (see column E)

4. modify the values sent by the new CCs using the formulas below (where x is the old value and y the new one):

   - When the value range is 0–254 or 0–255 and the target value is ≤ 127 (accessed with CC#6 = 0)
     
     `y = x / 2`
     
   - When the value range is 0–254 or 0–255 and the target value is > 127 (accessed with CC#6 = 1)
     
     `y = (x / 2) + 64`
     
   - When the value range is 0–164 (LOW-PASS FILTER Cutoff) and the target value is ≤ 127 (accessed with CC#6 = 0)
     
     `y = x / (165 / 128)` = `x * 0.7812`
     
   - When the value range is 0–164 (LOW–PASS FILTER Cutoff) and the target value is > 127 (accessed with CC#6 = 1)
     
     `y = (x * 0.7812) + 100`

Some resolution loss is unavoidable: it’s halved when the value range is 0–254 or 0–255, and reduced by a factor of 1.3 for the Low-Pass Filter Cutoff. 

### Sequential Prophet-6

It seems that the [Prophet-6](https://sequential.com/product/prophet-6/) shares the same MIDI implementation, although I can’t confirm it. If that’s the case, these resources could be useful for that synth as well.

# Traduction en français

C'est un synthé analogique entièrement à commande numérique

Apparemment conçu pour le live, il n'a pas rencontré un franc succès en raison de problèmes de synchronisation. En studio où il peut être joué et automatisé en tant que module sonore via le DAW, il est excellent.

Tous les paramètres de ce synthé peuvent être automatisés via MIDI en utilisant des NRPN (moitié droite de ce tableau). Une bonne moitié peut aussi l’être en utilisant un simple CC (moitié gauche du tableau). 

Or, un CC unique est plus rapide à visualiser et à modifier que son équivalent en NRPN (un groupe de quatre CC qui se suivent). De plus, les CC# d’un NRPN en particulier peuvent entrer en collision dans le bus MIDI avec les mêmes CC# utilisés par les autres NRPN, empêchant ainsi le synthé de retrouver le son souhaité.

Heureusement, lorsque le paramètre est contrôlable par les deux méthodes, il est possible si besoin de convertir les NRPN en CC simples dans la séquence MIDI.



A) Si la plage de valeurs du paramètre est ≤ 127 (colonne L), la conversion est directe et n’entraîne pas de perte de résolution ; il suffit de :

1. supprimer les CC#99, 98 et 6

2. convertir les CC#38 en CC dont le nº correspond au paramètre considéré (colonne E)



B) Si la plage de valeurs du paramètre est > 127 (colonne L), la perte de résolution est inévitable et il faut :

1. séparer en deux portées ou deux patterns distincts les valeurs ≤ 127 (accédées en NRPN via CC#6 = 0) et les valeurs > 127 (accédées en NRPN via CC#6 = 1)

2. supprimer les CC#99, 98 et 6

3. convertir les CC#38 en CC dont le nº correspond au paramètre considéré (colonne E)

4. modifier les valeurs envoyées par les nouveaux CC en appliquant les formules ci-dessous (où x est l’ancienne valeur et y la nouvelle)

a) quand la plage des valeurs est 0–254 ou 0–255 et que la valeur souhaitée est ≤ 127 (accédées via CC#6 = 0)
y = x / 2

b) quand la plage des valeurs est 0–254 ou 0–255 et que la valeur souhaitée est > 127 (accédées via CC#6 = 1)
y = (x / 2) + 64

c) quand la plage des valeurs est 0–164 (LOW-PASS FILTER Cutoff) et que la valeur souhaitée est ≤ 127 (accédées via CC#6 = 0)
y = x / (165 / 128) = x * 0.7812

d) quand la plage des valeurs est 0–164 (LOW-PASS FILTER Cutoff) et que la valeur souhaitée est > 127 (accédées via CC#6 = 1)
y = (x * 0.7812) + 100

J'ai pour habitude de placer ce genre de pattern au début de la principale piste MIDI pilotant un synthé. La jouer permet de remettre à zéro les paramètres de jeu et de retrouver le son de base pour le morceau considéré. Grâce à la fonction Acoustic Feedback de Cubendo, je m’en sers aussi pour programmer à distance le patch, sans devoir naviguer dans les menus et sous-menus de l’interface du synthé.

Pour ne pas risquer de perdre par inadvertence un réglage important, les messages MIDI pilotant directement le patch sont tous mutés. Ils doivent donc être démutés avant de pouvoir modifier les réglages du synthé.

Il semblerait que le Prophet-6 partage la même implémentation MIDI, bien que je ne puisse pas le vérifier. Si c'est le cas, ces ressources peuvent aussi servir à ce synthé.
