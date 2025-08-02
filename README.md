# bar-notes
Tools and reverse-engineering documentation related to Beetle Adventure Racing

## Intro

Beetle Adventure Racing! is one of my favorite racers for the N64.  Unfortunately, the game currently suffers from a lack of accurate emulation.  Timing issues can be clearly seen in the intro cutscenes, where the NPC cars reach their pre-recorded end points and go beyond them by mindlessly slamming into a wall or respawning too soon after crashing on purpose.

One day, I would like to port this game to the PC in a effort to properly preserve this game and make it playable on other platforms but I'm not sure if I want to lead a decomp or recomp project just yet.  In the meantime, I'll do what I can to reverse engineer and document as much as possible.

## UltraVision NU64

The engine behind Beetle Adventure Racing! is an updated version of the "UltraVision NU64 1.5" game engine developed by Paradigm Simulations and originally developed for Pilotwings 64.  All the game files and assets are stored inside numerous "FORM" chunks.  The chunks are sorted into different types, each of them starting with "UV" for UltraVision.  Each header should be followed by a 4-byte length, but you'll see shortly that this is not always the case.  Thanks to the debug strings that contain the original filenames for the compiled object files, we have a strong idea of what these abbrevations mean.

### FORM Chunk Types

Header|Description|Notes
| --- | --- | --- |
UVMO|Module|
UVTT|Track Terrain|
UVDS|Dynamic Set??|
UVTX|Textures|
UVTR|Terrain|
UVCT|Contour|
UVVL|Volume (like water?)|
UVMD|Model|
UVAN|Animation|
UVEN|Environment|
UVRW|Save Game?|
UVPX|Particle Effect|
UVBT|Blitter?|
UVFT|Font|
UVSX|Sound Effect|
UVMS|Music Sequence|
UVMB|Music Bank|
UVTS|Texture Sequence|

### FORM #0

At 0x237D0 in the US ROM, we have the initial FORM chunk, which contains the pointers to all the FORM chunks in the ROM.  The pointers are offset by the start of the NEXT 'FORM' header (0x25FD0).
You can think of FORM #0 as a table of contents, while FORM #1 is the start of actual game data.

#### Weird UVFT header

In FORM #0 at 0x237D8, there's a "UVFT" header that's immediately followed by 'UVMO'. I'm not sure why this is because 'UVFT' is related to fonts.  There's no length dword after the "UVFT" header so I think this might be a mistake.

### FORM #1

0x25FD0 marks the start of the common "FORM" chunk.  Inside these chunks we have several different headers that tell us what kind of data is being stored.  The string "PAD " is used immediately after the header instead of the 4-byte length to denote that we have a file entry.

### PAD Types

Header|Description|Notes
| --- | --- | --- |
MODU|Module|
COMM|Metadata|
CODE|Object Code|
PNTS|??|
MDBG|Debug Message|
RELA|Relative?|
LNKS|??|
VOBJ|??|
VMSK|Related to save game stuff|
SEQS|MIDI Music Sequence Data|
.CTL|N64 Soundbank format|
.TBL|N64 ADPCM Audio Wavetable|
UVSQ|Texture Sequence data|
STRG|String|
BITM|Font Bitmap|
FRMT|Font Format?|
IMAG|Font Image|
GZIP|Fake MIO0 header|followed by COMM, then MIO0
MIO0|Real MIO0 header|
TEXT|Particle Effect related|
PIID|Particle Effect related|
PHDR|Particle Effect related|
PDAT|Particle Effect related|


