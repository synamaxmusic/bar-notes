<!-- vim-markdown-toc GFM -->

* [Intro](#intro)
* [UltraVision NU64](#ultravision-nu64)
	* [FORM Chunk Types](#form-chunk-types)
	* [FORM #0](#form-0)
	* [FORM #1](#form-1)
	* [PAD Types](#pad-types)

<!-- vim-markdown-toc -->

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

## Important ROM addresses

Name|Address|Notes
| --- | --- | --- |
FORM #0|0x237D0|Also start of BSS and start of FORM in physical ROM
FORM #1|0x25FD0|First FORM with actual game data
D_ROMEND|0xF16160|Last address of used ROM space

## Important VRAM addresses (very WIP)

Address|Name|otes
| --- | --- | --- |
8001F790|START OF RAM
80022BD0|bss_start|
80025BD8||table of addresses (points tables of jump tables)
80025E80||counter for AI car movements events during cutscenes (controller buffer count in-game)
80025E84||total number of AI car movements events during cutscenes
80025E88||counter for cutscenes length
80025E8C||total number of cutscenes length
80025E90||ring buffer for logging all event durations during gameplay for replay (value ranges from 0 to FE, next byte starts incrementing upon overflow)
80025E7C|D_debugEnable|
80033770|D_formheader|
80033778|D_formTOC|(looks like absolute addresses)
80036010|D_formLoader|(looks like this loads FORM data in and out of memory)
8008E340||start of executable code for uvstring_rom.o (ROM location at 16E468)
8008E404||grabs first byte of "Start with Saved Data" string at 801C6F20
8008F094||pointer for start of uvstring_rom.o (8008E340...this is not in ROM)
8008F0A8||start of new executable code
8992CC85|Game Loaded flag|non-zero means game data has been loaded


