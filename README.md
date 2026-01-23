# Work in progress: Pioneer DJ-Dave Smith Instruments TORAIZ AS-1 MIDI Toolkit

_(version française ci-dessous)_

## Introduction

The TORAIZ AS-1 is an analog monosynth with full digital control, designed by Dave Smith Instruments (now [Sequential](https://sequential.com/)) for [Pioneer DJ](https://www.pioneerdj.com/). 

The synthesis engine and signal path are entirely analog, except for an optional digital multi-FX. In essence, it offers a single voice from the Prophet-6, housed in an enclosure with a form factor reminiscent of a classic silver box (sound-wise, it really doesn’t have as much character, but it is much more versatile).

It was apparently designed for live use, but didn’t really catch on because of [sync](https://gearspace.com/board/electronic-music-instruments-and-electronic-music-production/1244592-pioneer-toraiz-as1-midi-out-issues.html) [issues](https://gearspace.com/board/showpost.php?p=15940161&postcount=1). In the studio, where it can be played and automated as a sound module via MIDI, it’s great.

It was reviewed by [*Sound On Sound*](https://www.soundonsound.com/reviews/pioneer-dj-toraiz-1) in November 2017.

It seems that the [Prophet-6](https://sequential.com/product/prophet-6/) shares the same MIDI implementation, although I can’t confirm. If that’s the case, these resources could be useful for that synth as well.

![2017-10-25 17 13 47](https://github.com/user-attachments/assets/67185c02-7175-4bcf-a8d5-be0f94a32024)
_TORAIZ AS-1_

![2022-11-07 15 16 13](https://github.com/user-attachments/assets/7d2e4497-e7b0-4cc4-be8c-169a956db544)
_A classic silver box_

## MIDI Cheat Sheet

The TORAIZ AS-1’s MIDI implementation is great. It is no coincidence, as Dave Smith was among the small group of genius electronic luthiers who established the MIDI standard. All sound parameters can be set and automated directly from the DAW using [MIDI 1.0 messages](https://github.com/TheMicronauts/MIDI-1.0-Messages-Demystified). They are listed in this Google Sheets document along with their corresponding controls and value ranges:

https://docs.google.com/spreadsheets/d/1XDerLaoKoy6zsbu0w4pXwNQaluc6XFW1W9XfXDOYYyM

- The first table lists all sound parameters alongside their physical controls, the CC and NRPN numbers that control them, and their range of both real-world and MIDI values.

- The second table, accessible via a tab at the bottom left, provides side-by-side blank patch sheets to be filled in with your own values (after copying or downloading the spreadsheet). It is designed for quickly noting down settings and comparing patches.

## Reset And Control MIDI Parts For Cubendo

While playback is stopped, the most direct way to remotely adjust the state and timbre of a MIDI instrument from Cubase and Nuendo is to enable _Acoustic Feedback_ in the _List Editor_, then enter CCs manually and scroll through their values (assuming, of course, that no _Preferences_ interfere with this functionality).

Therefore, I have created a dedicated MIDI pattern (_Part_ in Cubendo-speak) containing every CC used by the AS-1, along with descriptive text (in the form of _Score_ or _Text Events_—named differently depending on where you are within the software, sigh). 

Placed anywhere on a MIDI track, this _Part_ allows for quick adjustments and then, on playback, ensures the instrument recalls the settings at that specific moment in time—for instance, at the start of a song to restore its initial state. Not only does it spare you from menu-diving, but it greatly reduces the need for managing presets, internal memory, Program Changes, or SysEx dumps.

This XML file allows you to import it into your Cubase or Nuendo _Project_, along with two other _Parts_ (via _File > Import > Track Archive…_):

- The first _Part_ is the one described above and contains the patch settings (_Program_ in DSI-speak). It’s also shown below. To avoid accidentally losing important settings, the MIDI messages that directly control the patch are muted; they must, therefore, be unmuted in order to work. On the other hand, CCs that cut the sound, the arpeggiator, or reset performance parameters are left unmuted. This is desirable for the _Part_’s primary use case: being positioned at the start of the timeline. These should probably be muted, however, if the _Part_ is used mid-song.

- The second _Part_ contains the unit’s _Global_ settings, with default values that I find sensible (those shown in the first table), but which you should tweak to your liking.

- The third _Part_ contains the SysEx message that will trigger the unit to dump the patch currently residing in its working memory. It allows you to record the dump (also a SysEx message) into a MIDI _Part_ and keep it within the _Project_ it is related to.

Once you have established a mental model of the instrument, this workflow is far less disruptive to the creative process. It reduces cognitive load by removing the need to learn and memorise new interactions. Generally, it is simpler, faster, and more flexible than going through additional layers of abstraction or dedicated editors—which are, alas, almost always buggy, incomplete, and idiosyncratic.

<img width="1726" alt="AS-1 rà0" src="https://github.com/user-attachments/assets/c7efe647-aec5-462a-95ca-e7ffa7f0f65a" />     

_Cubendo’s List Editor (best in class – other DAWs, seriously, take note, copy it, improve it if you can)_

## Converting NRPNs Into Single CCs

As we’ve just seen, all parameters of this synth can be automated via MIDI using NRPNs (right half of the first table), and a good number of them can also be automated using single CCs (left half of the first table).

Now, a single CC is quicker to visualise and edit than its NRPN equivalent (which uses a group of four consecutive CCs). Moreover, CC#s constituting a specific NRPN can conflict on the MIDI stream with the same CC#s used by other NRPNs, preventing the synth from recalling the intended sound.

Fortunately, when the parameter can be controlled by both methods, it is possible—if needed—to convert NRPNs into single CCs within the MIDI sequence, using Cubendo’s various functionalities, including the _Logical Editor_.

### If the parameter’s value range (column L of the first table) is ≤ 127:

1. delete CC#99, 98, and 6

2. convert CC#38 into the CC whose number matches the target parameter (column E)

The conversion is straightforward and doesn’t cause any resolution loss.

### If the parameter’s value range (column L of the first table) is > 127:

1. split into two distinct lanes or patterns the values ≤ 127 (accessed via NRPN with CC#6 = 0), and the values > 127 (accessed via NRPN with CC#6 = 1)

2. delete CC#99, 98, and 6

3. convert CC#38 into the CC whose number matches the target parameter (see column E)

4. modify the values set for the new CC using the formulas below (where x is the old value and y the new one):

   - When the value range is 0–254 or 0–255 and the target value is ≤ 127 (accessed with CC#6 = 0)
     
     `y = x / 2`
     
   - When the value range is 0–254 or 0–255 and the target value is > 127 (accessed with CC#6 = 1)
     
     `y = (x / 2) + 64`
     
   - When the value range is 0–164 (LOW-PASS FILTER Cutoff) and the target value is ≤ 127 (accessed with CC#6 = 0)
     
     `y = `x * 0.78`
     
   - When the value range is 0–164 (LOW–PASS FILTER Cutoff) and the target value is > 127 (accessed with CC#6 = 1)
     
     `y = (x * 0.78) + 100`

Some resolution loss is unavoidable: it’s halved when the value range is 0–254 and reduced by a factor of 1.3 for the Low-Pass Filter Cutoff. 

# Pioneer DJ-Dave Smith Instruments TORAIZ AS-1 MIDI Toolkit (version française)

C’est un synthé analogique entièrement à commande numérique. Oui le nom est débile et pourrait être celui d’une voiture électrique, mais bon, Pioneer DJ n’a jamais été le temple du bon goût et de la classe.

Le verdict ? Il n’a vraiment pas autant de personnalité mais il est beaucoup plus versatile.

Apparemment conçu pour le live, il n’a pas rencontré un franc succès en raison de problèmes de synchronisation. En studio où il peut être joué et automatisé en tant que module sonore via le DAW, il est excellent.

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

J’ai pour habitude de placer ce genre de pattern au début de la principale piste MIDI pilotant un synthé. La jouer permet de remettre à zéro les paramètres de jeu et de retrouver le son de base pour le morceau considéré. Grâce à la fonction Acoustic Feedback de Cubendo, je m’en sers aussi pour programmer à distance le patch, sans devoir naviguer dans les menus et sous-menus de l’interface du synthé.

Pour ne pas risquer de perdre par inadvertence un réglage important, les messages MIDI pilotant directement le patch sont tous mutés. Ils doivent donc être démutés avant de pouvoir modifier les réglages du synthé.

Il semblerait que le Prophet-6 partage la même implémentation MIDI, bien que je ne puisse pas le vérifier. Si c’est le cas, ces ressources peuvent aussi servir à ce synthé.
