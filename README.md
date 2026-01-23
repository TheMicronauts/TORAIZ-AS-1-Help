# Work in progress: Pioneer DJ-Dave Smith Instruments TORAIZ AS-1 MIDI Toolkit

_([Version française ci-dessous](#vf))_

## Introduction

The TORAIZ AS-1 is an analog monosynth with full digital control, designed by Dave Smith Instruments (now [Sequential](https://sequential.com/)) for [Pioneer DJ](https://www.pioneerdj.com/). 

The synthesis engine and signal path are entirely analog, except for an optional digital multi-FX. In essence, it offers a single voice from the Prophet-6, housed in an enclosure with a form factor reminiscent of a classic silver box (sound-wise, it really doesn’t have as much character as the latter, but it is much more versatile).

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

Placed anywhere on a MIDI track, this _Part_ allows for quick adjustments and then, on playback, ensures the instrument recalls the settings at that specific moment in time—for instance, at the start of a song to restore its initial state (where it can also act as a panic button). Not only does it spare you from menu-diving, but it greatly reduces the need for managing presets, internal memory, Program Changes, or SysEx dumps.

This XML file allows you to import it into your Cubase or Nuendo _Project_, along with two other _Parts_ (via _File > Import > Track Archive…_):

- The first _Part_ is the one described above and contains the patch settings (_Program_ in DSI-speak). Its screen grab is also shown below. To avoid accidentally losing important settings, the MIDI messages that directly control the patch are muted; they must, therefore, be unmuted in order to work. On the other hand, CCs that cut the sound, the arpeggiator, or reset performance parameters are left unmuted. This is desirable for the _Part_’s primary use case: being positioned at the start of the timeline. These should probably be muted, however, if the _Part_ is used mid-song.

- The second _Part_ contains the unit’s _Global_ settings, with default values that I find sensible (those shown in the first table), but which you should tweak to your liking.

- The third _Part_ contains the SysEx message that will trigger the unit to dump the patch currently residing in its working memory (DSI calls this message _Request Program Edit Buffer Dump_). It allows you to record the dump (also a SysEx message) into a MIDI _Part_ and keep it within the _Project_ it is related to.

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
     
   - When the value range is 0–164 (_LOW–PASS FILTER Cutoff_) and the target value is ≤ 127 (accessed with CC#6 = 0)
     
     `y = x * 0.78`
     
   - When the value range is 0–164 (_LOW–PASS FILTER Cutoff_) and the target value is > 127 (accessed with CC#6 = 1)
     
     `y = (x * 0.78) + 100`

Some resolution loss is unavoidable: it’s halved when the value range is 0–254 and divided by a factor of 1.3 for the _LOW–PASS FILTER Cutoff_.

<a name="vf"></a>
# Pioneer DJ-Dave Smith Instruments TORAIZ AS-1 MIDI Toolkit (Version française)

## Introduction

Le TORAIZ AS-1 est un monosynthé analogique entièrement à commande numérique, conçu par Dave Smith Instruments (désormais [Sequential](https://sequential.com/)) pour [Pioneer DJ](https://www.pioneerdj.com/). Oui, le nom craint et ressemble à celui d’une voiture électrique, mais bon, Pioneer DJ n’a jamais été le temple de la discrétion et de la classe. 

Le moteur de synthèse et le trajet du signal sont entièrement analogiques, à l’exception d’un multi-effet numérique optionnel. En substance, il offre une unique voix issue du Prophet-6, intégrée dans un boîtier dont le format rappelle celui d’une célèbre « boîte argentée » (côté son, il n’a clairement pas autant de caractère que cette dernière, mais il est bien plus polyvalent).

Apparemment conçu pour le live, il n’a pas rencontré un franc succès en raison de [problèmes](https://gearspace.com/board/electronic-music-instruments-and-electronic-music-production/1244592-pioneer-toraiz-as1-midi-out-issues.html) de [synchronisation](https://gearspace.com/board/showpost.php?p=15940161&postcount=1) (Oui… Le jour où les codeurs retireront leurs moufles, les poules auront des dents). En studio, où il peut être joué et automatisé comme un module sonore via MIDI, il est excellent.

[*Sound On Sound*](https://www.soundonsound.com/reviews/pioneer-dj-toraiz-1) l‘a testé en novembre 2017.

Il semblerait qu’il partage la même implémentation MIDI que le [Prophet-6](https://sequential.com/product/prophet-6/), bien que je ne puisse pas le confirmer. Si c’est le cas, ces ressources pourront également servir à ce synthé.

![2017-10-25 17 13 47](https://github.com/user-attachments/assets/67185c02-7175-4bcf-a8d5-be0f94a32024)
_TORAIZ AS-1_

![2022-11-07 15 16 13](https://github.com/user-attachments/assets/7d2e4497-e7b0-4cc4-be8c-169a956db544)
_A classic silver box_

## Pense-bête MIDI

L’implémentation MIDI du TORAIZ AS-1 est très complète. Ce n’est pas un hasard, puisque Dave Smith faisait partie du petit groupe de luthiers électroniques de génie qui ont établi la norme MIDI. Tous les paramètres sonores peuvent être réglés et automatisés directement depuis un séquenceur via des [messages MIDI 1.0](https://github.com/TheMicronauts/MIDI-1.0-Messages-Demystified). Ceux-ci sont répertoriés dans ce document Google Sheets, accompagnés de leurs commandes correspondantes et des plages de valeurs :

https://docs.google.com/spreadsheets/d/1XDerLaoKoy6zsbu0w4pXwNQaluc6XFW1W9XfXDOYYyM

- Le premier tableau liste l’ensemble des paramètres sonores en regard de leurs commandes physiques, des CC et des NRPN qui les pilotent, et de leurs plages de valeurs, réelles et MIDI.

- Le second tableau, accessible via un onglet en bas à gauche, propose des fiches de patch vierges placées côte à côte, à compléter avec vos propres valeurs (après avoir copié ou téléchargé la feuille). Il est fait pour noter rapidement des réglages et les comparer.

## Parts MIDI de réinitialisation et de contrôle pour Cubendo

À l’arrêt, le moyen le plus direct d’ajuster à distance depuis Cubase ou Nuendo l’état et le timbre d’un instrument MIDI consiste à activer _Acoustic Feedback_ dans le _List Editor_, puis à saisir manuellement les CC et à faire défiler leurs valeurs à la souris (à condition, bien sûr, qu’aucune _Preferences_ n’interfère avec cette fonctionnalité).

J’ai donc créé un pattern MIDI dédié (_Part_, en jargon Cubendo) contenant l’ensemble des CC utilisés par l’AS-1, avec une courte description sous forme de _Score_ or _Text Events_ (nommés différemment à différents endroits du logiciel, fatigue).

Placée n’importe où sur une piste MIDI, cette _Part_ permet des ajustements rapides puis garantit qu’à la relecture, l’instrument retrouve ces réglages à ce moment précis – par exemple, au début d’un morceau pour restaurer son état initial (où elle peut également servir de bouton « Panic »). Non seulement cela évite de se farcir les menus du synthé, mais cela simplifie considérablement la gestion des presets, de la mémoire interne, des Program Changes et des dumps SysEx.

Ce fichier XML vous permet de l’importer dans votre projet Cubase ou Nuendo, accompagnée de deux autres _Parts_ (via _File > Import > Track Archive…_) :

- La première _Part_ est celle décrite ci-dessus et contient les réglages de patch (_Program_ en jargon DSI). Sa capture d’écran se trouve également en illustration ci-dessous. Afin d’éviter perdre accidentellement un réglage important, les messages MIDI qui contrôlent directement le patch sont mutés ; ils doivent donc être réactivés pour fonctionner. En revanche, les CC qui coupent le son, l’arpégiateur ou réinitialisent les paramètres de jeu sont démutés. C’est ce qu’on veut a priori lorsque cette Part se trouve à son endroit de prédilection, avant le début du morceau. Ils devront probablement être mutés si la Part est utilisée au milieu du morceau.

- La deuxième _Part_ contient les réglages _Global_ de l’unité, avec des valeurs par défaut que je juge pertinentes (celles indiquées dans le premier tableau) et que vous pourrez ajuster à votre convenance.

- La troisième _Part_ contient le message SysEx qui déclenche l’envoi par l’unité des réglages de patch actuellement présent dans sa mémoire vive (DSI appelle ce message _Request Program Edit Buffer Dump_). Elle permettra de sauver ce dump (également un SysEx) dans une _Part_ MIDI, afin de le conserver au sein du projet auquel il se rapporte.

Une fois que vous vous êtes forgé un modèle mental de l’instrument, cette manière de faire perturbe beaucoup moins le processus créatif. Elle réduit la charge cognitive en supprimant la nécessité d’apprendre et de mémoriser de nouvelles interactions. Elle est de manière générale plus simple, rapide et flexible qu’en recourant à des couches supplémentaires d’abstraction et des éditeurs dédiés – presque toujours buggés, incomplets et idiosyncratiques, hélas.

<img width="1726" alt="AS-1 rà0" src="https://github.com/user-attachments/assets/c7efe647-aec5-462a-95ca-e7ffa7f0f65a" />     

_Le List Editor de Cubendo (le meilleur du genre — sérieusement les autres, prenez-en de la graine, copiez-le ou améliorez-le si vous pouvez…)_

## Conversion des NRPN en CC simples

Comme nous venons de le voir, tous les paramètres de ce synthé peuvent être automatisés via MIDI à l’aide de NRPN (moitié droite du premier tableau). Un bon nombre d’entre eux peuvent également l’être à l’aide de CC simples (moitié gauche du premier tableau).

Bien sûr, un CC unique est plus facile à visualiser et à éditer que son équivalent NRPN (qui repose sur un groupe de quatre CC consécutifs). De plus, les CC constituant un NRPN donné peuvent entrer en conflit dans le flux MIDI avec les mêmes CC utilisés par d’autres NRPN, empêchant ainsi le synthé de restituer le son escompté.

Heureusement, lorsqu’un paramètre peut être contrôlé par les deux méthodes, il est possible en cas de besoin de convertir les NRPN en CC simples au sein d’une séquence MIDI, en s’appuyant dur les diverses fonctionnalités de Cubendo, notamment le _Logical Editor_.

### Si la plage de valeurs du paramètre (colonne L du premier tableau) est ≤ 127 :

1. supprimer les CC#99, 98 et 6

2. convertir les CC#38 en CC dont le numéro correspond au paramètre considéré (colonne E)

La conversion est directe et n’entraîne aucune perte de résolution.

### Si la plage de valeurs du paramètre (colonne L du premier tableau) est > 127 :

1. séparer en Parts distinctes les valeurs ≤ 127 (accédées par les NRPN où CC#6 = 0) et les valeurs > 127 (accédées par les NRPN où CC#6 = 1).

2. supprimer les CC#99, 98 et 6

3. convertir les CC#38 en CC dont le numéro correspond au paramètre considéré (colonne E)

4. modifier les valeurs envoyées par les nouveaux CC en appliquant les formules ci-dessous (où x est l’ancienne valeur et y la nouvelle) :

   - Lorsque la plage de valeurs est 0–254 ou 0–255 et que la valeur cible est ≤ 127 (les Parts venant des CC#6 = 0)
     
     `y = x / 2`
     
   - Lorsque la plage de valeurs est 0–254 ou 0–255 et que la valeur cible est > 127 (les Parts venant des CC#6 = 1)
     
     `y = (x / 2) + 64`
     
   - Lorsque la plage de valeurs est 0–164 (_LOW-PASS FILTER Cutoff_) et que la valeur cible est ≤ 127 (les Parts venant des CC#6 = 0)
     
     `y = x * 0.78`
     
   -  Lorsque la plage de valeurs est 0–164 (_LOW-PASS FILTER Cutoff_) et que la valeur cible est > 127 (les Parts venant des CC#6 = 1)
     
     `y = (x * 0.78) + 100`

Une perte de résolution est inévitable : elle sera divisée par deux lorsque la plage de valeurs est 0–254 et par 1,3 pour _LOW–PASS FILTER Cutoff_.
