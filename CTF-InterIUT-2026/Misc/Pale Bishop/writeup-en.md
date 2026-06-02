# Pale Bishop - Misc

(Auto translated using mistral, do tell me if anything isn't clear and / or straight up wrong)

## Challenge Description

Null-Syndicate managed to intercept recordings of chess games. While these games seem harmless at first glance, the organization is convinced that secret documents were exchanged during these matches. The goal is to recover the document(s) that were shared.

Note: The title **Pale Bishop** directly references chess (a Bishop being the piece, aka "Fou" in French), and—go figure—the analysis revolves around a `.pgn` file, which corresponds to games exported from Lichess. Who would’ve thought?

---

## Initial Analysis

### Format Confirmation

As a certified genius, I’ve developed the habit of hex-viewing files instead of just running `strings` or opening them as text. So, in about 5 seconds, I recognized the chess game notation format—thanks to some random 45-minute video I watched at 3 AM.

![Hex view of the file](./hexview.png)

---

## Extracting Hidden Data

### Using the Chess_Steganography Tool

The [Chess_Steganography](https://github.com/KKlockenbrink/Chess_Steganography/blob/master/chessDecoder.py) tool was used to decode the moves. After adapting and running the script, an SVG file was generated. This file contains a graphical representation with hidden elements.

For the record, the "adaptation" involved the titanic effort of copy-pasting the code and modifying the import file (don’t forget the **`r`** before the string, or Python will explode with Windows paths).

### SVG Content

The generated SVG file contains a structured document in the form of a classified report, as well as a hidden section (`<g id="archive" aria-hidden="true">`) with a series of circles. Each circle has an `r` attribute (radius) with seemingly random values.

```xml
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 600 420" width="600" height="420" font-family="monospace">
    <rect width="600" height="420" fill="#0a0a0f"/>
    <rect x="20" y="20" width="560" height="380" fill="none" stroke="#00ffe0" stroke-width="1.5" opacity="0.6"/>
    ...
    <g id="archive" aria-hidden="true">
        <circle cx="1" cy="-500" r="105" fill="none" stroke="none"/>
        <circle cx="2" cy="-500" r="110" fill="none" stroke="none"/>
        ...
        <circle cx="36" cy="-500" r="10" fill="none" stroke="none"/>
    </g>
</svg>
```

![The flag](./exported.svg)

---
## Decoding the Data

### ASCII Hypothesis

While skimming through, the hidden section immediately caught my eye. The circle radius values varied quite randomly, so I figured it was probably some kind of encoding—the first that came to mind being ASCII.

Here are the radius values for each circle:
- 105, 110, 116, 101, 114, 105, 117, 116, 123, 97, 95, 118, 101, 114, 121, 95, 115, 101, 99, 114, 101, 116, 95, 105, 110, 102, 111, 114, 109, 97, 116, 105, 111, 110, 125

Converting these values to ASCII characters gives us:

**Flag:** `interiut{a_very_secret_information}`

![Flag obtained](./flag.png)

---
## Conclusion

I found this challenge terribly interesting. Even though the solution is *ridiculously simple* (I mean—the biggest hurdle was finding the right tool), it’s the first time I’ve seen something like this.

So, honestly, I tip my hat to the effort of creating something unique, and I’ve discovered a new way to hide my `.env` secrets on GitHub for the lulz.