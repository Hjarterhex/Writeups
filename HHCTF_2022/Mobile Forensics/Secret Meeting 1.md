# Secret Meeting 1

Solves: 3 <br/> Points: 292

## Challenge description

A meeting was supposed to take place regarding a drug deal. What was the first location that was suggested?

Flag Format: HHCTF{Location}

## Solution

In the "iOS iMessage/SMS/MMS" subcategory under the "COMMUNICATION" category, we find the conversation regarding a drug deal. If we read through the conversation, the buyer asks if the seller "Can deliver here?", and sends a sound file containing morse code.

![Message location](../img/secret_m_1_1.png)

The file is located in the subcategory "Apple Notes" under the category "DOCUMENTS". So we just right click on the file missed_call.m4a, and save the artifact.

![Save file](../img/secret_m_1_2.png)

If we upload the file to a morse code sound decoder such as https://morsecode.world/international/decoder/audio-decoder-adaptive.html we can see that it represent coordinates.

![Coordinates](../img/secret_m_1_3.png)

The coordinates are: </br>
Latitude: 63.34902094017484 </br>
Longitude: 13.4528216509953 </br>

If we use those coordinates on Google maps, the following location is presented:

![Location](../img/secret_m_1_4.png)

**Flag:** `HHCTF{Järpen}`
