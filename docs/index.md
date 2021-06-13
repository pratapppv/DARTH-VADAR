## Welcome to Darth VADAR- a simple 2.4GHz ISM band FMCW RADAR

Sometime back on early 2019, I took the plunge into RF circuits and the first RF design I've always wanted to do was a simple RADAR. [Hforsten](https://hforsten.com/6-ghz-frequency-modulated-radar.html) was a big inspiration for this and he frankly made it seem simple enough.
This post is supposed to be a mix of a tutorial and documentation for whatever RADAR me and a couple of my friends built.

### Design of the RADAR
As I wasn't too picky about the range or the resolution, I decided to choose to operate the RADAR at a center frequency of $$f_0 = 2.45GHz$$ and a maximum bandwidth of $$\Delta f = 100MHz$$ setting the RADAR to work completely within the allocated ISM band. As this band is also where WiFi and bluetooth reside, finding chips which were'nt exorbidantly priced was easy. Another advantage of choosing this frequency was that the dielectric loss of FR4 at this frequency was not too high and could be easily used. This ment that no exotic substrates would be required and any standard PCB fab house could make the PCB's.

A 

