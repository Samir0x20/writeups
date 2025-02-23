## Description of the challenge

Zeroes, ones, dots and dashes
Data streams in bright flashes

111 0000 10 111 1000 00 10 01 010 1011 11 111 010 000 0

NOTE: Wrap the capitalized flag in KashiCTF{}

## Solution

Dots and dashes is a reference to morse.  
1 = **-**
0 = **.**

Morse code: ````--- .... -. --- -... .. -. .- .-. -.-- -- --- .-. ... .  ````

Then I used https://morsecode.world/international/translator.html to decode the morse code.

**Decoded morse**: OHNOBINARYMORSE

**Flag**: KashiCTF{OHNOBINARYMORSE}