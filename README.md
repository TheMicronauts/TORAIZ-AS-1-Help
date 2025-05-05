# TORAIZ AS-1 Help (work in progress)
## Pioneer DJ Dave Smith Instruments TORAIZ AS-1 MIDI cheat sheet &amp; sound packs

The TORAIZ AS-1 is an analog monosynth with full digital control, designed by Dave Smith Instruments (now [Sequential](https://sequential.com/)) for [Pioneer DJ](https://www.pioneerdj.com/). 

The synthesis engine and signal path are entirely analog, except for an optional digital multi FX. In essence, it offers a single voice from the Prophet-6, housed in an enclosure with a form factor reminiscent of a classic silver box.

It was reviewed by [*Sound On Sound*](https://www.soundonsound.com/reviews/pioneer-dj-toraiz-1) in November 2017.

![2017-10-25 17 13 47](https://github.com/user-attachments/assets/67185c02-7175-4bcf-a8d5-be0f94a32024)
*TORAIZ AS-1*

![2022-11-07 15 16 13](https://github.com/user-attachments/assets/7d2e4497-e7b0-4cc4-be8c-169a956db544)
*A classic silver box*

### MIDI cheat sheet

The TORAIZ AS-1 MIDI implementation is comprehensive. All sound parameters can be set and automated directly from the DAW.

I've listed them in this Google Sheets table along with their value ranges and corresponding controls:

https://docs.google.com/spreadsheets/d/1XDerLaoKoy6zsbu0w4pXwNQaluc6XFW1W9XfXDOYYyM

You’ll notice that all of this synth’s parameters can be automated via MIDI using NRPNs (right half of the table), and a good number of them can also be automated using a simple CC (left half of the table).

Now, a single CC is quicker to visualise and edit than its NRPN equivalent (which uses a group of four consecutive CCs). Moreover, the CC#s of a specific NRPN can conflict on the MIDI bus with the same CC#s used by other NRPNs, preventing the synth from recalling the intended sound.

Fortunately, when the parameter can be controlled by both methods, it is possible, if needed, to convert NRPNs into simple CCs in the MIDI sequence. Here's how.

A) If the parameter’s value range is ≤ 127 (column L—the conversion is straightforward and doesn’t cause any resolution loss):

1. delete CC#99, 98, and 6

2. convert CC#38 into the CC whose number matches the target parameter (column E)

B) If the parameter’s value range is > 127 (column L—resolution loss is unavoidable):

1. split into two distinct lanes or patterns values ≤ 127 (accessed via NRPN with CC#6 = 0), and values > 127 (accessed via NRPN with CC#6 = 1)

2. delete CC#99, 98, and 6

3. convert CC#38 into the CC whose number matches the target parameter (see column E)

4. modify the values sent by the new CCs using the formulas below (where x is the old value and y the new one):

   - When the value range is 0–254 or 0–255 and the target value is ≤ 127 (accessed with CC#6 = 0)
     
     `y = x / 2`
     
   - When the value range is 0–254 or 0–255 and the target value is > 127 (accessed with CC#6 = 1)
     
     `y = (x / 2) + 64`
     
   - When the value range is 0–164 (LOW-PASS FILTER Cutoff) and the target value is ≤ 127 (accessed with CC#6 = 0)
     
     `y = x / (165 / 128)` = `x * 0.7812`
     
   - When the value range is 0–164 (LOW-PASS FILTER Cutoff) and the target value is > 127 (accessed with CC#6 = 1)
     
     `y = (x * 0.7812) + 100`
