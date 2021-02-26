# Scid File Format Specification Version 4

Scid is a popular free chess database program. It stores games in it's
own binary format. Whereas Scid is open source, there is no single clear
file format specification, and one has to resort to deep study of it's
C++ source code to understand the format. This document attempts to
provide a technical specification of the format in order to facilitate
the implementation of the format by programs other than direct forks of
the SCID source base.

An SCID database consists of three mandatory files:

  - An Index File (.si4) containing meta-information for each game
  - A file containing the actual game data (.sg4)
  - A file containing player names, tournament names and other textual
    information (.sn4)

# The Index File (.si4)

> The Index File contains a description for the database and a small fixed-size entry for each game. Each game entry includes essential information such as the result, date, player/event/site name IDs (the actual names are in the Name File), and some redundant but useful information that is used to speed up searches

The Index file consists of a header 182 bytes long, followed by one 47 byte index record for each stored game.

**Consecutive bytes are big endian unless otherwise stated**

## The Header

| Name | Length (Bytes) | Data Type | Description |
| --- | --- | --- | --- |
| Magic Header      | 8   | String | 8-byte identifier for Scid index files. 53 63 69 64 2E 73 69 00 in Hex i.e. Scid.si\NUL in ASCII |
| Version Number    | 2   | Uint    | Scid version number that wrote the file. 400 for si4. |
| Database Type     | 4   | uint    | A byte sequence that describes the type of the database, e.g. tournament, theory, etc. |
| Number of Games   | 3   | uint    | Number of games stored in the database. |
| Auto Load Game    | 3   | uint    | Game number to autoload: 0=none, 1=1st, >numGames=last |
| Description       | 108 | string  | A fixed-length null-terminated string describing the database |
| Custom Flags      | 54  | custom  | There are six custom flag entries. Each entry is a nine byte null-terminated ASCII string. Within each entry, any byte following a zero byte must be ignored. |

**Total Length of Header: 182 bytes**

## Index Entries

The index file contains one entry per game in the database. It contains more than just the location of the game data
in the main data file. For fast searching, it also store some other important values: players, event, site, date,
result, eco, game length. Each entry takes up 47 bytes on-disk in version 4, and 46 bytes in some older versions.

| Name | Length (Bytes) | Data Type | Description |
| --- | --- | --- | --- |
| Offset | 4 | uint | Location of entry starting byte in the GameBase |
| Length Low | 2 | custom | Two least significant bytes (Big Endian) of the entry length in the GameBase |
| Length High | 1 | custom | Most significant byte of the entry length in the GameBase. On Scid versions < 4 this value did not exist, so should be skipped if trying to read an older version file. |
| Flags | 2 | custom | Custom Index Flags (see below). |
| White/Black High | 1 | custom | Most significant bits of the White and Black Player IDs in the NameBase. |
| White ID Low | 2 | custom | Two least significant bytes of the White Player ID in the NameBase. |
| Black ID Low | 2 | custom | Two least significant bytes of the Black Player ID in the NameBase. |
| Event/Site/Round High | 1 | custom | Most significant bits of Event, Site, and Round IDs in the NameBase. |
| Event ID Low | 2 | custom | Least significant bytes of the Event ID in the NameBase. |
| Site ID Low | 2 | custom | Least significant bytes of the Site ID in the NameBase. |
| Round ID Low | 2 | custom | Least significant bytes of the Round ID in the NameBase. |
| Result/Variation Counts | 2 | custom | Counters for comments, variations, etc. Also stores the result of the Game. |
| ECO Code | 2 | custom | Encyclopedia of Chess Openings Code for the game. |
| Dates | 4 | custom | The date of the Game and date of the Event. |
| White Elo | 2 | uint | Elo rating of the White Player, limited to 4000 since Elo ratings are stored interally in 12 bits. |
| Black Elo | 2 | uint | Elo rating of the Black Player, limited to 4000 since Elo ratings are stored interally in 12 bits. |
| Stored Line Code | 1 | custom | Index of the opening line played in the Game stored in a lookup table. Used for searching. |
| Final Material | 3 | custom | Material of the final position in the game. |
| Number of Plies Low | 1 | uint | Least signficant byte of the number of plies (half moves) played in the Game. |
| Number of Plies High / Home Pawn Data | 9 | custom | Most significant bits of number of plies and Home Pawn Data. |

**Total Length of Index Entry: 47 bytes**

### Description of Custom Data Types

#### Length
| **Bit**   | 23-16 | 15-0 |
| --- | --- | --- |
| **Name**  | Length High | Length Low |

#### Flags

| **Bit**   | 15 | 14 | 13 | 12 | 11 | 10 | 9 | 8 | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| **Name**  | User Defined | Blunder | Brilliancy | Queenside Play | Kingside Play | Tactics | Pawn Structure | Novelty | Endgame | Middlegame | Black Opening | White Opening | Marked for Deletion | Underpromotion(s) | Promotion(s) | Has Custom Starting Position |

#### White/Black ID

##### White/Black High
| **Bit**   | 7-4 | 3-0 |
| --- | --- | --- |
| **Name**  | White ID High | Black ID High |

##### White ID
| **Bit**   | 19-16 | 15-0 |
| --- | --- | --- |
| **Name**  | White ID High | White ID Low |

##### Black ID
| **Bit**   | 19-16 | 15-0 |
| --- | --- | --- |
| **Name**  | Black ID High | Black ID Low |


#### Event/Site/Round ID

##### Event/Site/Round ID High
| **Bit**   | 7-5 | 4-2 | 1-0 |
| --- | --- | --- | --- |
| **Name**  | Event ID High | Site ID High | Round ID High |

##### Event ID
| **Bit**   | 18-16 | 15-0 |
| --- | --- | --- |
| **Name**  | Event ID High | Event ID Low |

##### Site ID
| **Bit**   | 18-16 | 15-0 |
| --- | --- | --- |
| **Name**  | Site ID High | Site ID Low |

##### Round ID
| **Bit**   | 17-16 | 15-0 |
| --- | --- | --- |
| **Name**  | Round ID High | Round ID Low |

#### Result/Variation Counts
| **Bit**   | 15-12 | 11-0 |
| --- | --- | --- |
| **Name**  | Result | Variation Counts |

##### Result
| **Bit**   | 3-2 | 1 | 0 |
| --- | --- | --- | --- |
| **Name**  | Unused | Black | White |

| Result | Score | Description |
| --- | --- | --- |
| 0x00 | * | None |
| 0x01 | 1-0 | White Wins |
| 0x02 | 0-1 | Black Wins |
| 0x03 | 1/2-1/2 | Draw |

##### Variation Counts
Variation Counts stores three values: Variations, Comments, and NAGs (Numeric Annotation Glyphs).

| **Bit**   | 11-8 | 7-4 | 3-0 |
| --- | --- | --- | --- |
| **Name**  | NAGs Code | Comments Code | Variations Code |

###### Code Values
| Code | Count Range |
| --- | --- |
| 0-10 | Exact Value |
| 11 | 13-17 |
| 12 | 18-24 |
| 13 | 25-34 | 
| 14 | 35-44 |
| 15 | >= 50 |


#### ECO Code
| ECO Code | String Value |
| ---      | ---          |
| 0x0000 | None |
| 0x0001 | A00 |
| 0x0002 | A00a |
| 0x0003 | A00a1 |
| 0x0004 | A00a2 |
| 0x0005 | A00a3 |
| 0x0006 | A00a4 |
| 0x0007 | A00b |
| 0x0008 | A00b1 |
| ... | ... |
| 0x0083 | A00z4 |
| 0x0084 | A01 |
| 0x0085 | A01a |
| 0x0086 | A01a1 |
| ... | ... |
| 0x332C | A99z4 |
| 0x332D | B00 |
| 0x332E | B00a |
| ... | ... |
| 0xFFDC | E99z4 |

*Note: Values 0xFFDD up to 0xFFFF are possible, but are not valid ECO Codes*

[Reference Decoder Source](https://github.com/xmcpam/scid/blob/b21bf33983b21d37fc916a5edaed89897240f91d/src/misc.cpp#L112)

#### Dates
| **Bit**   | 31-20 | 19-0 |
| --- | --- | --- |
| **Name**  | Event Date | Game Date |

##### Game Date
| **Bit**   | 19-9 | 8-5 | 4-0 |
| --- | --- | --- | --- |
| **Name**  | Game Year | Game Month | Game Day |

##### Event Date
| **Bit**   | 11-9 | 8-5 | 4-0 |
| --- | --- | --- | --- |
| **Name**  | Year Mod | Event Month | Event Day |

##### Year Mod
If Year Mod == 0 then event date is unknown
Else Event Date = Game Date + Year Mod - 4

Common values:
| Year Mod | Description |
| --- | --- |
| 0b000 | Undefined Event Date |
| 0b011 | Event date is in the year before the game |
| 0b100 | Event date is in the same year as the game |
| 0b101 | Event date is in the year after the game |

#### Stored Line Code
##### Stored Lines
| Stored Line Code | Moves | SAN |
| --- | --- | --- |
| 0x01 |  0x6000251, | 1.b3 |
| 0x02 |  0x600029a, | 1.c4 |
| 0x03 |  0x600029a, 0xe000ca2, | 1.c4 c5 |
| 0x04 |  0x600029a, 0xe000ca2, 0x5000195, | 1.c4 c5 2.Nf3 |
| 0x05 |  0x600029a, 0xe000d24, | 1.c4 e5 |
| 0x06 |  0x600029a, 0xe000d24, 0x5000052, | 1.c4 e5 2.Nc3 |
| 0x07 |  0x600029a, 0xe000d24, 0x5000052, 0xd000fad, | 1.c4 e5 2.Nc3 Nf6 |
| 0x08 |  0x600029a, 0xe000d2c, | 1.c4 e6 |
| 0x09 |  0x600029a, 0xe000d2c, 0x5000195, | 1.c4 e6 2.Nf3 |
| 0x0A |  0x600029a, 0xe000dae, | 1.c4 g6 |
| 0x0B |  0x600029a, 0xd000fad, | 1.c4 Nf6 |
| 0x0C |  0x600029a, 0xd000fad, 0x5000052, | 1.c4 Nf6 2.Nc3 |
| 0x0D |  0x600029a, 0xd000fad, 0x5000052, 0xe000d2c, | 1.c4 Nf6 2.Nc3 e6 |
| 0x0E |  0x600029a, 0xd000fad, 0x5000052, 0xe000dae, | 1.c4 Nf6 2.Nc3 g6 |
| 0x0F |  0x60002db, | 1.d4 |
| 0x10 |  0x60002db, 0xe000ce3, | 1.d4 d5 |
| 0x11 |  0x60002db, 0xe000ce3, 0x600029a, | 1.d4 d5 2.c4 |
| 0x12 |  0x60002db, 0xe000ce3, 0x600029a, 0xe000caa, | 1.d4 d5 2.c4 c6 |
| 0x13 |  0x60002db, 0xe000ce3, 0x600029a, 0xe000caa, 0x5000052, | 1.d4 d5 2.c4 c6 3.Nc3 |
| 0x14 |  0x60002db, 0xe000ce3, 0x600029a, 0xe000caa, 0x5000052, 0xd000fad, | 1.d4 d5 2.c4 c6 3.Nc3 Nf6 |
| 0x15 |  0x60002db, 0xe000ce3, 0x600029a, 0xe000caa, 0x5000052, 0xd000fad, 0x5000195, | 1.d4 d5 2.c4 c6 3.Nc3 Nf6 4.Nf3 |
| 0x16 |  0x60002db, 0xe000ce3, 0x600029a, 0xe000caa, 0x5000195, | 1.d4 d5 2.c4 c6 3.Nf3 |
| 0x17 |  0x60002db, 0xe000ce3, 0x600029a, 0xe000caa, 0x5000195, 0xd000fad, | 1.d4 d5 2.c4 c6 3.Nf3 Nf6 |
| 0x18 |  0x60002db, 0xe000ce3, 0x600029a, 0xe000caa, 0x5000195, 0xd000fad, 0x5000052, | 1.d4 d5 2.c4 c6 3.Nf3 Nf6 4.Nc3 |
| 0x19 |  0x60002db, 0xe000ce3, 0x600029a, 0xe000caa, 0x5000195, 0xd000fad, 0x5000052, 0xe000d2c, | 1.d4 d5 2.c4 c6 3.Nf3 Nf6 4.Nc3 e6 |
| 0x1A |  0x60002db, 0xe000ce3, 0x600029a, 0xec008da, | 1.d4 d5 2.c4 dxc4 |
| 0x1B |  0x60002db, 0xe000ce3, 0x600029a, 0xec008da, 0x5000195, | 1.d4 d5 2.c4 dxc4 3.Nf3 |
| 0x1C |  0x60002db, 0xe000ce3, 0x600029a, 0xec008da, 0x5000195, 0xd000fad, | 1.d4 d5 2.c4 dxc4 3.Nf3 Nf6 |
| 0x1D |  0x60002db, 0xe000ce3, 0x600029a, 0xe000d2c, | 1.d4 d5 2.c4 e6 |
| 0x1E |  0x60002db, 0xe000ce3, 0x600029a, 0xe000d2c, 0x5000052, | 1.d4 d5 2.c4 e6 3.Nc3 |
| 0x1F |  0x60002db, 0xe000ce3, 0x600029a, 0xe000d2c, 0x5000052, 0xe000caa, | 1.d4 d5 2.c4 e6 3.Nc3 c6 |
| 0x20 |  0x60002db, 0xe000ce3, 0x600029a, 0xe000d2c, 0x5000052, 0xd000fad, | 1.d4 d5 2.c4 e6 3.Nc3 Nf6 |
| 0x21 |  0x60002db, 0xe000ce3, 0x600029a, 0xe000d2c, 0x5000195, | 1.d4 d5 2.c4 e6 3.Nf3 |
| 0x22 |  0x60002db, 0xe000ce3, 0x5000195, | 1.d4 d5 2.Nf3 |
| 0x23 |  0x60002db, 0xe000ce3, 0x5000195, 0xd000fad, | 1.d4 d5 2.Nf3 Nf6 |
| 0x24 |  0x60002db, 0xe000ce3, 0x5000195, 0xd000fad, 0x600029a, | 1.d4 d5 2.Nf3 Nf6 3.c4 |
| 0x25 |  0x60002db, 0xe000ce3, 0x5000195, 0xd000fad, 0x600029a, 0xe000caa, | 1.d4 d5 2.Nf3 Nf6 3.c4 c6 |
| 0x26 |  0x60002db, 0xe000ce3, 0x5000195, 0xd000fad, 0x600029a, 0xe000d2c, | 1.d4 d5 2.Nf3 Nf6 3.c4 e6 |
| 0x27 |  0x60002db, 0xe000ceb, | 1.d4 d6 |
| 0x28 |  0x60002db, 0xe000ceb, 0x5000195, | 1.d4 d6 2.Nf3 |
| 0x29 |  0x60002db, 0xe000d2c, | 1.d4 e6 |
| 0x2A |  0x60002db, 0xe000d2c, 0x600029a, | 1.d4 e6 2.c4 |
| 0x2B |  0x60002db, 0xe000d2c, 0x600029a, 0xd000fad, | 1.d4 e6 2.c4 Nf6 |
| 0x2C |  0x60002db, 0xe000d65, | 1.d4 f5 |
| 0x2D |  0x60002db, 0xe000d65, 0x6000396, 0xd000fad, 0x400014e, | 1.d4 f5 2.g3 Nf6 3.Bg2 |
| 0x2E |  0x60002db, 0xe000dae, | 1.d4 g6 |
| 0x2F |  0x60002db, 0xe000dae, 0x600029a, 0xc000f76, | 1.d4 g6 2.c4 Bg7 |
| 0x30 |  0x60002db, 0xd000fad, | 1.d4 Nf6 |
| 0x31 |  0x60002db, 0xd000fad, 0x40000a6, | 1.d4 Nf6 2.Bg5 |
| 0x32 |  0x60002db, 0xd000fad, 0x40000a6, 0xd000b5c, | 1.d4 Nf6 2.Bg5 Ne4 |
| 0x33 |  0x60002db, 0xd000fad, 0x600029a, | 1.d4 Nf6 2.c4 |
| 0x34 |  0x60002db, 0xd000fad, 0x600029a, 0xe000ca2, | 1.d4 Nf6 2.c4 c5 |
| 0x35 |  0x60002db, 0xd000fad, 0x600029a, 0xe000ca2, 0x60006e3, | 1.d4 Nf6 2.c4 c5 3.d5 |
| 0x36 |  0x60002db, 0xd000fad, 0x600029a, 0xe000ca2, 0x60006e3, 0xe000c61, | 1.d4 Nf6 2.c4 c5 3.d5 b5 |
| 0x37 |  0x60002db, 0xd000fad, 0x600029a, 0xe000ca2, 0x60006e3, 0xe000c61, 0x6c006a1, 0xe000c28, | 1.d4 Nf6 2.c4 c5 3.d5 b5 4.cxb5 a6 |
| 0x38 |  0x60002db, 0xd000fad, 0x600029a, 0xe000d2c, 0x6000396, | 1.d4 Nf6 2.c4 e6 3.g3 |
| 0x39 |  0x60002db, 0xd000fad, 0x600029a, 0xe000d2c, 0x6000396, 0xe000ce3, | 1.d4 Nf6 2.c4 e6 3.g3 d5 |
| 0x3A |  0x60002db, 0xd000fad, 0x600029a, 0xe000d2c, 0x5000052, | 1.d4 Nf6 2.c4 e6 3.Nc3 |
| 0x3B |  0x60002db, 0xd000fad, 0x600029a, 0xe000d2c, 0x5000052, 0xc000f59, | 1.d4 Nf6 2.c4 e6 3.Nc3 Bb4 |
| 0x3C |  0x60002db, 0xd000fad, 0x600029a, 0xe000d2c, 0x5000052, 0xc000f59, 0x6000314, | 1.d4 Nf6 2.c4 e6 3.Nc3 Bb4 4.e3 |
| 0x3D |  0x60002db, 0xd000fad, 0x600029a, 0xe000d2c, 0x5000052, 0xc000f59, 0x6000314, 0x900cf3f, | 1.d4 Nf6 2.c4 e6 3.Nc3 Bb4 4.e3 O-O |
| 0x3E |  0x60002db, 0xd000fad, 0x600029a, 0xe000d2c, 0x5000052, 0xc000f59, 0x20000ca, | 1.d4 Nf6 2.c4 e6 3.Nc3 Bb4 4.Qc2 |
| 0x3F |  0x60002db, 0xd000fad, 0x600029a, 0xe000d2c, 0x5000052, 0xc000f59, 0x20000ca, 0x900cf3f, | 1.d4 Nf6 2.c4 e6 3.Nc3 Bb4 4.Qc2 O-O |
| 0x40 |  0x60002db, 0xd000fad, 0x600029a, 0xe000d2c, 0x5000052, 0xc000f59, 0x20000ca, 0x900cf3f, 0x6000210, 0x4ca00652, 0x2800292, | 1.d4 Nf6 2.c4 e6 3.Nc3 Bb4 4.Qc2 O-O 5.a3 Bxc3+ 6.Qxc3 |
| 0x41 |  0x60002db, 0xd000fad, 0x600029a, 0xe000d2c, 0x5000052, 0xe000ce3, | 1.d4 Nf6 2.c4 e6 3.Nc3 d5 |
| 0x42 |  0x60002db, 0xd000fad, 0x600029a, 0xe000d2c, 0x5000195, | 1.d4 Nf6 2.c4 e6 3.Nf3 |
| 0x43 |  0x60002db, 0xd000fad, 0x600029a, 0xe000d2c, 0x5000195, 0xe000c69, | 1.d4 Nf6 2.c4 e6 3.Nf3 b6 |
| 0x44 |  0x60002db, 0xd000fad, 0x600029a, 0xe000d2c, 0x5000195, 0xe000c69, 0x6000210, | 1.d4 Nf6 2.c4 e6 3.Nf3 b6 4.a3 |
| 0x45 |  0x60002db, 0xd000fad, 0x600029a, 0xe000d2c, 0x5000195, 0xe000c69, 0x6000396, | 1.d4 Nf6 2.c4 e6 3.Nf3 b6 4.g3 |
| 0x46 |  0x60002db, 0xd000fad, 0x600029a, 0xe000d2c, 0x5000195, 0xe000c69, 0x6000396, 0xc000ea8, | 1.d4 Nf6 2.c4 e6 3.Nf3 b6 4.g3 Ba6 |
| 0x47 |  0x60002db, 0xd000fad, 0x600029a, 0xe000d2c, 0x5000195, 0x4c000f59, | 1.d4 Nf6 2.c4 e6 3.Nf3 Bb4+ |
| 0x48 |  0x60002db, 0xd000fad, 0x600029a, 0xe000d2c, 0x5000195, 0xe000ce3, | 1.d4 Nf6 2.c4 e6 3.Nf3 d5 |
| 0x49 |  0x60002db, 0xd000fad, 0x600029a, 0xe000d2c, 0x5000195, 0xe000ce3, 0x5000052, | 1.d4 Nf6 2.c4 e6 3.Nf3 d5 4.Nc3 |
| 0x4A |  0x60002db, 0xd000fad, 0x600029a, 0xe000dae, | 1.d4 Nf6 2.c4 g6 |
| 0x4B |  0x60002db, 0xd000fad, 0x600029a, 0xe000dae, 0x5000052, 0xc000f76, | 1.d4 Nf6 2.c4 g6 3.Nc3 Bg7 |
| 0x4C |  0x60002db, 0xd000fad, 0x600029a, 0xe000dae, 0x5000052, 0xc000f76, 0x600031c, | 1.d4 Nf6 2.c4 g6 3.Nc3 Bg7 4.e4 |
| 0x4D |  0x60002db, 0xd000fad, 0x600029a, 0xe000dae, 0x5000052, 0xc000f76, 0x600031c, 0xe000ceb, | 1.d4 Nf6 2.c4 g6 3.Nc3 Bg7 4.e4 d6 |
| 0x4E |  0x60002db, 0xd000fad, 0x600029a, 0xe000dae, 0x5000052, 0xc000f76, 0x600031c, 0xe000ceb, 0x400014c, 0x900cf3f, | 1.d4 Nf6 2.c4 g6 3.Nc3 Bg7 4.e4 d6 5.Be2 O-O |
| 0x4F |  0x60002db, 0xd000fad, 0x600029a, 0xe000dae, 0x5000052, 0xc000f76, 0x600031c, 0xe000ceb, 0x400014c, 0x900cf3f, 0x5000195, | 1.d4 Nf6 2.c4 g6 3.Nc3 Bg7 4.e4 d6 5.Be2 O-O 6.Nf3 |
| 0x50 |  0x60002db, 0xd000fad, 0x600029a, 0xe000dae, 0x5000052, 0xc000f76, 0x600031c, 0xe000ceb, 0x6000355, | 1.d4 Nf6 2.c4 g6 3.Nc3 Bg7 4.e4 d6 5.f3 |
| 0x51 |  0x60002db, 0xd000fad, 0x600029a, 0xe000dae, 0x5000052, 0xc000f76, 0x600031c, 0xe000ceb, 0x6000355, 0x900cf3f, | 1.d4 Nf6 2.c4 g6 3.Nc3 Bg7 4.e4 d6 5.f3 O-O |
| 0x52 |  0x60002db, 0xd000fad, 0x600029a, 0xe000dae, 0x5000052, 0xc000f76, 0x600031c, 0xe000ceb, 0x6000355, 0x900cf3f, 0x4000094, | 1.d4 Nf6 2.c4 g6 3.Nc3 Bg7 4.e4 d6 5.f3 O-O 6.Be3 |
| 0x53 |  0x60002db, 0xd000fad, 0x600029a, 0xe000dae, 0x5000052, 0xc000f76, 0x600031c, 0xe000ceb, 0x5000195, 0x900cf3f, | 1.d4 Nf6 2.c4 g6 3.Nc3 Bg7 4.e4 d6 5.Nf3 O-O |
| 0x54 |  0x60002db, 0xd000fad, 0x600029a, 0xe000dae, 0x5000052, 0xc000f76, 0x600031c, 0xe000ceb, 0x5000195, 0x900cf3f, 0x400014c, | 1.d4 Nf6 2.c4 g6 3.Nc3 Bg7 4.e4 d6 5.Nf3 O-O 6.Be2 |
| 0x55 |  0x60002db, 0xd000fad, 0x600029a, 0xe000dae, 0x5000052, 0xc000f76, 0x600031c, 0xe000ceb, 0x5000195, 0x900cf3f, 0x400014c, 0xe000d24, | 1.d4 Nf6 2.c4 g6 3.Nc3 Bg7 4.e4 d6 5.Nf3 O-O 6.Be2 e5 |
| 0x56 |  0x60002db, 0xd000fad, 0x600029a, 0xe000dae, 0x5000052, 0xc000f76, 0x600031c, 0xe000ceb, 0x5000195, 0x900cf3f, 0x400014c, 0xe000d24, 0x100c107, | 1.d4 Nf6 2.c4 g6 3.Nc3 Bg7 4.e4 d6 5.Nf3 O-O 6.Be2 e5 7.O-O |
| 0x57 |  0x60002db, 0xd000fad, 0x600029a, 0xe000dae, 0x5000052, 0xc000f76, 0x600031c, 0xe000ceb, 0x5000195, 0x900cf3f, 0x400014c, 0xe000d24, 0x100c107, 0xd000e6a, 0x60006e3, 0xd000ab4, | 1.d4 Nf6 2.c4 g6 3.Nc3 Bg7 4.e4 d6 5.Nf3 O-O 6.Be2 e5 7.O-O Nc6 8.d5 Ne7 |
| 0x58 |  0x60002db, 0xd000fad, 0x600029a, 0xe000dae, 0x5000052, 0xe000ce3, | 1.d4 Nf6 2.c4 g6 3.Nc3 d5 |
| 0x59 |  0x60002db, 0xd000fad, 0x600029a, 0xe000dae, 0x5000052, 0xe000ce3, 0x5000195, | 1.d4 Nf6 2.c4 g6 3.Nc3 d5 4.Nf3 |
| 0x5A |  0x60002db, 0xd000fad, 0x600029a, 0xe000dae, 0x5000052, 0xe000ce3, 0x6c006a3, 0xdc00b63, | 1.d4 Nf6 2.c4 g6 3.Nc3 d5 4.cxd5 Nxd5 |
| 0x5B |  0x60002db, 0xd000fad, 0x600029a, 0xe000dae, 0x5000052, 0xe000ce3, 0x6c006a3, 0xdc00b63, 0x600031c, 0xda008d2, 0x6a00252, 0xc000f76, | 1.d4 Nf6 2.c4 g6 3.Nc3 d5 4.cxd5 Nxd5 5.e4 Nxc3 6.bxc3 Bg7 |
| 0x5C |  0x60002db, 0xd000fad, 0x5000195, | 1.d4 Nf6 2.Nf3 |
| 0x5D |  0x60002db, 0xd000fad, 0x5000195, 0xe000ca2, | 1.d4 Nf6 2.Nf3 c5 |
| 0x5E |  0x60002db, 0xd000fad, 0x5000195, 0xe000ce3, | 1.d4 Nf6 2.Nf3 d5 |
| 0x5F |  0x60002db, 0xd000fad, 0x5000195, 0xe000d2c, | 1.d4 Nf6 2.Nf3 e6 |
| 0x60 |  0x60002db, 0xd000fad, 0x5000195, 0xe000d2c, 0x40000a6, | 1.d4 Nf6 2.Nf3 e6 3.Bg5 |
| 0x61 |  0x60002db, 0xd000fad, 0x5000195, 0xe000d2c, 0x600029a, | 1.d4 Nf6 2.Nf3 e6 3.c4 |
| 0x62 |  0x60002db, 0xd000fad, 0x5000195, 0xe000dae, | 1.d4 Nf6 2.Nf3 g6 |
| 0x63 |  0x60002db, 0xd000fad, 0x5000195, 0xe000dae, 0x40000a6, | 1.d4 Nf6 2.Nf3 g6 3.Bg5 |
| 0x64 |  0x60002db, 0xd000fad, 0x5000195, 0xe000dae, 0x600029a, | 1.d4 Nf6 2.Nf3 g6 3.c4 |
| 0x65 |  0x60002db, 0xd000fad, 0x5000195, 0xe000dae, 0x600029a, 0xc000f76, | 1.d4 Nf6 2.Nf3 g6 3.c4 Bg7 |
| 0x66 |  0x60002db, 0xd000fad, 0x5000195, 0xe000dae, 0x600029a, 0xc000f76, 0x5000052, | 1.d4 Nf6 2.Nf3 g6 3.c4 Bg7 4.Nc3 |
| 0x67 |  0x60002db, 0xd000fad, 0x5000195, 0xe000dae, 0x600029a, 0xc000f76, 0x5000052, 0x900cf3f, | 1.d4 Nf6 2.Nf3 g6 3.c4 Bg7 4.Nc3 O-O |
| 0x68 |  0x60002db, 0xd000fad, 0x5000195, 0xe000dae, 0x6000396, | 1.d4 Nf6 2.Nf3 g6 3.g3 |
| 0x69 |  0x60002db, 0xd000fad, 0x5000195, 0xe000dae, 0x6000396, 0xc000f76, 0x400014e, | 1.d4 Nf6 2.Nf3 g6 3.g3 Bg7 4.Bg2 |
| 0x6A |  0x600031c, | 1.e4 |
| 0x6B |  0x600031c, 0xe000ca2, | 1.e4 c5 |
| 0x6C |  0x600031c, 0xe000ca2, 0x6000292, | 1.e4 c5 2.c3 |
| 0x6D |  0x600031c, 0xe000ca2, 0x6000292, 0xe000ce3, 0x6c00723, 0xac00ee3, 0x60002db, | 1.e4 c5 2.c3 d5 3.exd5 Qxd5 4.d4 |
| 0x6E |  0x600031c, 0xe000ca2, 0x6000292, 0xe000ce3, 0x6c00723, 0xac00ee3, 0x60002db, 0xd000fad, | 1.e4 c5 2.c3 d5 3.exd5 Qxd5 4.d4 Nf6 |
| 0x6F |  0x600031c, 0xe000ca2, 0x6000292, 0xd000fad, 0x6000724, 0xd000b63, | 1.e4 c5 2.c3 Nf6 3.e5 Nd5 |
| 0x70 |  0x600031c, 0xe000ca2, 0x6000292, 0xd000fad, 0x6000724, 0xd000b63, 0x60002db, 0xec0089b, | 1.e4 c5 2.c3 Nf6 3.e5 Nd5 4.d4 cxd4 |
| 0x71 |  0x600031c, 0xe000ca2, 0x60002db, 0xec0089b, | 1.e4 c5 2.d4 cxd4 |
| 0x72 |  0x600031c, 0xe000ca2, 0x5000052, | 1.e4 c5 2.Nc3 |
| 0x73 |  0x600031c, 0xe000ca2, 0x5000052, 0xd000e6a, | 1.e4 c5 2.Nc3 Nc6 |
| 0x74 |  0x600031c, 0xe000ca2, 0x5000052, 0xd000e6a, 0x6000396, | 1.e4 c5 2.Nc3 Nc6 3.g3 |
| 0x75 |  0x600031c, 0xe000ca2, 0x5000052, 0xd000e6a, 0x6000396, 0xe000dae, | 1.e4 c5 2.Nc3 Nc6 3.g3 g6 |
| 0x76 |  0x600031c, 0xe000ca2, 0x5000052, 0xd000e6a, 0x6000396, 0xe000dae, 0x400014e, 0xc000f76, | 1.e4 c5 2.Nc3 Nc6 3.g3 g6 4.Bg2 Bg7 |
| 0x77 |  0x600031c, 0xe000ca2, 0x5000052, 0xd000e6a, 0x6000396, 0xe000dae, 0x400014e, 0xc000f76, 0x60002d3, | 1.e4 c5 2.Nc3 Nc6 3.g3 g6 4.Bg2 Bg7 5.d3 |
| 0x78 |  0x600031c, 0xe000ca2, 0x5000195, | 1.e4 c5 2.Nf3 |
| 0x79 |  0x600031c, 0xe000ca2, 0x5000195, 0xe000ceb, | 1.e4 c5 2.Nf3 d6 |
| 0x7A |  0x600031c, 0xe000ca2, 0x5000195, 0xe000ceb, 0x44000161, | 1.e4 c5 2.Nf3 d6 3.Bb5+ |
| 0x7B |  0x600031c, 0xe000ca2, 0x5000195, 0xe000ceb, 0x60002db, | 1.e4 c5 2.Nf3 d6 3.d4 |
| 0x7C |  0x600031c, 0xe000ca2, 0x5000195, 0xe000ceb, 0x60002db, 0xec0089b, | 1.e4 c5 2.Nf3 d6 3.d4 cxd4 |
| 0x7D |  0x600031c, 0xe000ca2, 0x5000195, 0xe000ceb, 0x60002db, 0xec0089b, 0x5c0055b, | 1.e4 c5 2.Nf3 d6 3.d4 cxd4 4.Nxd4 |
| 0x7E |  0x600031c, 0xe000ca2, 0x5000195, 0xe000ceb, 0x60002db, 0xec0089b, 0x5c0055b, 0xd000fad, | 1.e4 c5 2.Nf3 d6 3.d4 cxd4 4.Nxd4 Nf6 |
| 0x7F |  0x600031c, 0xe000ca2, 0x5000195, 0xe000ceb, 0x60002db, 0xec0089b, 0x5c0055b, 0xd000fad, 0x5000052, | 1.e4 c5 2.Nf3 d6 3.d4 cxd4 4.Nxd4 Nf6 5.Nc3 |
| 0x80 |  0x600031c, 0xe000ca2, 0x5000195, 0xe000ceb, 0x60002db, 0xec0089b, 0x5c0055b, 0xd000fad, 0x5000052, 0xe000c28, | 1.e4 c5 2.Nf3 d6 3.d4 cxd4 4.Nxd4 Nf6 5.Nc3 a6 |
| 0x81 |  0x600031c, 0xe000ca2, 0x5000195, 0xe000ceb, 0x60002db, 0xec0089b, 0x5c0055b, 0xd000fad, 0x5000052, 0xe000c28, 0x400015a, | 1.e4 c5 2.Nf3 d6 3.d4 cxd4 4.Nxd4 Nf6 5.Nc3 a6 6.Bc4 |
| 0x82 |  0x600031c, 0xe000ca2, 0x5000195, 0xe000ceb, 0x60002db, 0xec0089b, 0x5c0055b, 0xd000fad, 0x5000052, 0xe000c28, 0x400014c, | 1.e4 c5 2.Nf3 d6 3.d4 cxd4 4.Nxd4 Nf6 5.Nc3 a6 6.Be2 |
| 0x83 |  0x600031c, 0xe000ca2, 0x5000195, 0xe000ceb, 0x60002db, 0xec0089b, 0x5c0055b, 0xd000fad, 0x5000052, 0xe000c28, 0x4000094, | 1.e4 c5 2.Nf3 d6 3.d4 cxd4 4.Nxd4 Nf6 5.Nc3 a6 6.Be3 |
| 0x84 |  0x600031c, 0xe000ca2, 0x5000195, 0xe000ceb, 0x60002db, 0xec0089b, 0x5c0055b, 0xd000fad, 0x5000052, 0xe000c28, 0x40000a6, | 1.e4 c5 2.Nf3 d6 3.d4 cxd4 4.Nxd4 Nf6 5.Nc3 a6 6.Bg5 |
| 0x85 |  0x600031c, 0xe000ca2, 0x5000195, 0xe000ceb, 0x60002db, 0xec0089b, 0x5c0055b, 0xd000fad, 0x5000052, 0xe000c28, 0x40000a6, 0xe000d2c, | 1.e4 c5 2.Nf3 d6 3.d4 cxd4 4.Nxd4 Nf6 5.Nc3 a6 6.Bg5 e6 |
| 0x86 |  0x600031c, 0xe000ca2, 0x5000195, 0xe000ceb, 0x60002db, 0xec0089b, 0x5c0055b, 0xd000fad, 0x5000052, 0xe000dae, | 1.e4 c5 2.Nf3 d6 3.d4 cxd4 4.Nxd4 Nf6 5.Nc3 g6 |
| 0x87 |  0x600031c, 0xe000ca2, 0x5000195, 0xe000ceb, 0x60002db, 0xec0089b, 0x5c0055b, 0xd000fad, 0x5000052, 0xe000dae, 0x4000094, 0xc000f76, 0x6000355, | 1.e4 c5 2.Nf3 d6 3.d4 cxd4 4.Nxd4 Nf6 5.Nc3 g6 6.Be3 Bg7 7.f3 |
| 0x88 |  0x600031c, 0xe000ca2, 0x5000195, 0xe000ceb, 0x60002db, 0xec0089b, 0x5c0055b, 0xd000fad, 0x5000052, 0xe000dae, 0x4000094, 0xc000f76, 0x6000355, 0x900cf3f, | 1.e4 c5 2.Nf3 d6 3.d4 cxd4 4.Nxd4 Nf6 5.Nc3 g6 6.Be3 Bg7 7.f3 O-O |
| 0x89 |  0x600031c, 0xe000ca2, 0x5000195, 0xe000ceb, 0x60002db, 0xec0089b, 0x5c0055b, 0xd000fad, 0x5000052, 0xd000e6a, | 1.e4 c5 2.Nf3 d6 3.d4 cxd4 4.Nxd4 Nf6 5.Nc3 Nc6 |
| 0x8A |  0x600031c, 0xe000ca2, 0x5000195, 0xe000ceb, 0x60002db, 0xec0089b, 0x5c0055b, 0xd000fad, 0x5000052, 0xd000e6a, 0x40000a6, | 1.e4 c5 2.Nf3 d6 3.d4 cxd4 4.Nxd4 Nf6 5.Nc3 Nc6 6.Bg5 |
| 0x8B |  0x600031c, 0xe000ca2, 0x5000195, 0xe000ceb, 0x60002db, 0xec0089b, 0x5c0055b, 0xd000fad, 0x5000052, 0xd000e6a, 0x40000a6, 0xe000d2c, 0x20000cb, | 1.e4 c5 2.Nf3 d6 3.d4 cxd4 4.Nxd4 Nf6 5.Nc3 Nc6 6.Bg5 e6 7.Qd2 |
| 0x8C |  0x600031c, 0xe000ca2, 0x5000195, 0xe000d2c, | 1.e4 c5 2.Nf3 e6 |
| 0x8D |  0x600031c, 0xe000ca2, 0x5000195, 0xe000d2c, 0x60002d3, | 1.e4 c5 2.Nf3 e6 3.d3 |
| 0x8E |  0x600031c, 0xe000ca2, 0x5000195, 0xe000d2c, 0x60002db, 0xec0089b, 0x5c0055b, | 1.e4 c5 2.Nf3 e6 3.d4 cxd4 4.Nxd4 |
| 0x8F |  0x600031c, 0xe000ca2, 0x5000195, 0xe000d2c, 0x60002db, 0xec0089b, 0x5c0055b, 0xe000c28, | 1.e4 c5 2.Nf3 e6 3.d4 cxd4 4.Nxd4 a6 |
| 0x90 |  0x600031c, 0xe000ca2, 0x5000195, 0xe000d2c, 0x60002db, 0xec0089b, 0x5c0055b, 0xe000c28, 0x4000153, | 1.e4 c5 2.Nf3 e6 3.d4 cxd4 4.Nxd4 a6 5.Bd3 |
| 0x91 |  0x600031c, 0xe000ca2, 0x5000195, 0xe000d2c, 0x60002db, 0xec0089b, 0x5c0055b, 0xd000e6a, | 1.e4 c5 2.Nf3 e6 3.d4 cxd4 4.Nxd4 Nc6 |
| 0x92 |  0x600031c, 0xe000ca2, 0x5000195, 0xe000d2c, 0x60002db, 0xec0089b, 0x5c0055b, 0xd000e6a, 0x5000052, | 1.e4 c5 2.Nf3 e6 3.d4 cxd4 4.Nxd4 Nc6 5.Nc3 |
| 0x93 |  0x600031c, 0xe000ca2, 0x5000195, 0xe000d2c, 0x60002db, 0xec0089b, 0x5c0055b, 0xd000e6a, 0x5000052, 0xa000ef2, | 1.e4 c5 2.Nf3 e6 3.d4 cxd4 4.Nxd4 Nc6 5.Nc3 Qc7 |
| 0x94 |  0x600031c, 0xe000ca2, 0x5000195, 0xe000d2c, 0x60002db, 0xec0089b, 0x5c0055b, 0xd000fad, | 1.e4 c5 2.Nf3 e6 3.d4 cxd4 4.Nxd4 Nf6 |
| 0x95 |  0x600031c, 0xe000ca2, 0x5000195, 0xe000d2c, 0x60002db, 0xec0089b, 0x5c0055b, 0xd000fad, 0x5000052, | 1.e4 c5 2.Nf3 e6 3.d4 cxd4 4.Nxd4 Nf6 5.Nc3 |
| 0x96 |  0x600031c, 0xe000ca2, 0x5000195, 0xe000d2c, 0x60002db, 0xec0089b, 0x5c0055b, 0xd000fad, 0x5000052, 0xe000ceb, | 1.e4 c5 2.Nf3 e6 3.d4 cxd4 4.Nxd4 Nf6 5.Nc3 d6 |
| 0x97 |  0x600031c, 0xe000ca2, 0x5000195, 0xd000e6a, | 1.e4 c5 2.Nf3 Nc6 |
| 0x98 |  0x600031c, 0xe000ca2, 0x5000195, 0xd000e6a, 0x4000161, | 1.e4 c5 2.Nf3 Nc6 3.Bb5 |
| 0x99 |  0x600031c, 0xe000ca2, 0x5000195, 0xd000e6a, 0x4000161, 0xe000dae, | 1.e4 c5 2.Nf3 Nc6 3.Bb5 g6 |
| 0x9A |  0x600031c, 0xe000ca2, 0x5000195, 0xd000e6a, 0x60002db, 0xec0089b, 0x5c0055b, | 1.e4 c5 2.Nf3 Nc6 3.d4 cxd4 4.Nxd4 |
| 0x9B |  0x600031c, 0xe000ca2, 0x5000195, 0xd000e6a, 0x60002db, 0xec0089b, 0x5c0055b, 0xe000d24, | 1.e4 c5 2.Nf3 Nc6 3.d4 cxd4 4.Nxd4 e5 |
| 0x9C |  0x600031c, 0xe000ca2, 0x5000195, 0xd000e6a, 0x60002db, 0xec0089b, 0x5c0055b, 0xe000dae, | 1.e4 c5 2.Nf3 Nc6 3.d4 cxd4 4.Nxd4 g6 |
| 0x9D |  0x600031c, 0xe000ca2, 0x5000195, 0xd000e6a, 0x60002db, 0xec0089b, 0x5c0055b, 0xd000fad, 0x5000052, | 1.e4 c5 2.Nf3 Nc6 3.d4 cxd4 4.Nxd4 Nf6 5.Nc3 |
| 0x9E |  0x600031c, 0xe000ca2, 0x5000195, 0xd000e6a, 0x60002db, 0xec0089b, 0x5c0055b, 0xd000fad, 0x5000052, 0xe000ceb, | 1.e4 c5 2.Nf3 Nc6 3.d4 cxd4 4.Nxd4 Nf6 5.Nc3 d6 |
| 0x9F |  0x600031c, 0xe000ca2, 0x5000195, 0xd000e6a, 0x60002db, 0xec0089b, 0x5c0055b, 0xd000fad, 0x5000052, 0xe000ceb, 0x40000a6, | 1.e4 c5 2.Nf3 Nc6 3.d4 cxd4 4.Nxd4 Nf6 5.Nc3 d6 6.Bg5 |
| 0xA0 |  0x600031c, 0xe000ca2, 0x5000195, 0xd000e6a, 0x60002db, 0xec0089b, 0x5c0055b, 0xd000fad, 0x5000052, 0xe000d24, | 1.e4 c5 2.Nf3 Nc6 3.d4 cxd4 4.Nxd4 Nf6 5.Nc3 e5 |
| 0xA1 |  0x600031c, 0xe000ca2, 0x5000195, 0xd000e6a, 0x60002db, 0xec0089b, 0x5c0055b, 0xd000fad, 0x5000052, 0xe000d24, 0x150006e1, 0xe000ceb, | 1.e4 c5 2.Nf3 Nc6 3.d4 cxd4 4.Nxd4 Nf6 5.Nc3 e5 6.Ndb5 d6 |
| 0xA2 |  0x600031c, 0xe000ca2, 0x5000195, 0xd000e6a, 0x60002db, 0xec0089b, 0x5c0055b, 0xd000fad, 0x5000052, 0xe000d24, 0x150006e1, 0xe000ceb, 0x40000a6, 0xe000c28, | 1.e4 c5 2.Nf3 Nc6 3.d4 cxd4 4.Nxd4 Nf6 5.Nc3 e5 6.Ndb5 d6 7.Bg5 a6 |
| 0xA3 |  0x600031c, 0xe000ca2, 0x5000195, 0xd000e6a, 0x60002db, 0xec0089b, 0x5c0055b, 0xd000fad, 0x5000052, 0xe000d24, 0x150006e1, 0xe000ceb, 0x40000a6, 0xe000c28, 0x5000850, 0xe000c61, | 1.e4 c5 2.Nf3 Nc6 3.d4 cxd4 4.Nxd4 Nf6 5.Nc3 e5 6.Ndb5 d6 7.Bg5 a6 8.Na3 b5 |
| 0xA4 |  0x600031c, 0xe000caa, | 1.e4 c6 |
| 0xA5 |  0x600031c, 0xe000caa, 0x60002db, 0xe000ce3, | 1.e4 c6 2.d4 d5 |
| 0xA6 |  0x600031c, 0xe000caa, 0x60002db, 0xe000ce3, 0x6000724, | 1.e4 c6 2.d4 d5 3.e5 |
| 0xA7 |  0x600031c, 0xe000caa, 0x60002db, 0xe000ce3, 0x6000724, 0xc000ea5, | 1.e4 c6 2.d4 d5 3.e5 Bf5 |
| 0xA8 |  0x600031c, 0xe000caa, 0x60002db, 0xe000ce3, 0x6c00723, 0xec00aa3, | 1.e4 c6 2.d4 d5 3.exd5 cxd5 |
| 0xA9 |  0x600031c, 0xe000caa, 0x60002db, 0xe000ce3, 0x6c00723, 0xec00aa3, 0x600029a, 0xd000fad, 0x5000052, | 1.e4 c6 2.d4 d5 3.exd5 cxd5 4.c4 Nf6 5.Nc3 |
| 0xAA |  0x600031c, 0xe000caa, 0x60002db, 0xe000ce3, 0x5000052, | 1.e4 c6 2.d4 d5 3.Nc3 |
| 0xAB |  0x600031c, 0xe000caa, 0x60002db, 0xe000ce3, 0x5000052, 0xec008dc, 0x5c0049c, | 1.e4 c6 2.d4 d5 3.Nc3 dxe4 4.Nxe4 |
| 0xAC |  0x600031c, 0xe000caa, 0x60002db, 0xe000ce3, 0x500004b, 0xec008dc, 0x5c002dc, | 1.e4 c6 2.d4 d5 3.Nd2 dxe4 4.Nxe4 |
| 0xAD |  0x600031c, 0xe000ce3, 0x6c00723, 0xd000fad, | 1.e4 d5 2.exd5 Nf6 |
| 0xAE |  0x600031c, 0xe000ce3, 0x6c00723, 0xac00ee3, | 1.e4 d5 2.exd5 Qxd5 |
| 0xAF |  0x600031c, 0xe000ce3, 0x6c00723, 0xac00ee3, 0x5000052, | 1.e4 d5 2.exd5 Qxd5 3.Nc3 |
| 0xB0 |  0x600031c, 0xe000ce3, 0x6c00723, 0xac00ee3, 0x5000052, 0xa0008e0, | 1.e4 d5 2.exd5 Qxd5 3.Nc3 Qa5 |
| 0xB1 |  0x600031c, 0xe000ceb, | 1.e4 d6 |
| 0xB2 |  0x600031c, 0xe000ceb, 0x60002db, | 1.e4 d6 2.d4 |
| 0xB3 |  0x600031c, 0xe000ceb, 0x60002db, 0xd000fad, | 1.e4 d6 2.d4 Nf6 |
| 0xB4 |  0x600031c, 0xe000ceb, 0x60002db, 0xd000fad, 0x5000052, | 1.e4 d6 2.d4 Nf6 3.Nc3 |
| 0xB5 |  0x600031c, 0xe000ceb, 0x60002db, 0xd000fad, 0x5000052, 0xe000dae, | 1.e4 d6 2.d4 Nf6 3.Nc3 g6 |
| 0xB6 |  0x600031c, 0xe000ceb, 0x60002db, 0xd000fad, 0x5000052, 0xe000dae, 0x600035d, 0xc000f76, 0x5000195, | 1.e4 d6 2.d4 Nf6 3.Nc3 g6 4.f4 Bg7 5.Nf3 |
| 0xB7 |  0x600031c, 0xe000ceb, 0x60002db, 0xd000fad, 0x5000052, 0xe000dae, 0x5000195, 0xc000f76, | 1.e4 d6 2.d4 Nf6 3.Nc3 g6 4.Nf3 Bg7 |
| 0xB8 |  0x600031c, 0xe000d24, | 1.e4 e5 |
| 0xB9 |  0x600031c, 0xe000d24, 0x600035d, | 1.e4 e5 2.f4 |
| 0xBA |  0x600031c, 0xe000d24, 0x5000052, | 1.e4 e5 2.Nc3 |
| 0xBB |  0x600031c, 0xe000d24, 0x5000195, | 1.e4 e5 2.Nf3 |
| 0xBC |  0x600031c, 0xe000d24, 0x5000195, 0xd000e6a, | 1.e4 e5 2.Nf3 Nc6 |
| 0xBD |  0x600031c, 0xe000d24, 0x5000195, 0xd000e6a, 0x4000161, | 1.e4 e5 2.Nf3 Nc6 3.Bb5 |
| 0xBE |  0x600031c, 0xe000d24, 0x5000195, 0xd000e6a, 0x4000161, 0xe000c28, | 1.e4 e5 2.Nf3 Nc6 3.Bb5 a6 |
| 0xBF |  0x600031c, 0xe000d24, 0x5000195, 0xd000e6a, 0x4000161, 0xe000c28, 0x4000858, | 1.e4 e5 2.Nf3 Nc6 3.Bb5 a6 4.Ba4 |
| 0xC0 |  0x600031c, 0xe000d24, 0x5000195, 0xd000e6a, 0x4000161, 0xe000c28, 0x4000858, 0xd000fad, | 1.e4 e5 2.Nf3 Nc6 3.Bb5 a6 4.Ba4 Nf6 |
| 0xC1 |  0x600031c, 0xe000d24, 0x5000195, 0xd000e6a, 0x4000161, 0xe000c28, 0x4000858, 0xd000fad, 0x100c107, | 1.e4 e5 2.Nf3 Nc6 3.Bb5 a6 4.Ba4 Nf6 5.O-O |
| 0xC2 |  0x600031c, 0xe000d24, 0x5000195, 0xd000e6a, 0x4000161, 0xe000c28, 0x4000858, 0xd000fad, 0x100c107, 0xe000c61, 0x4000611, | 1.e4 e5 2.Nf3 Nc6 3.Bb5 a6 4.Ba4 Nf6 5.O-O b5 6.Bb3 |
| 0xC3 |  0x600031c, 0xe000d24, 0x5000195, 0xd000e6a, 0x4000161, 0xe000c28, 0x4000858, 0xd000fad, 0x100c107, 0xc000f74, | 1.e4 e5 2.Nf3 Nc6 3.Bb5 a6 4.Ba4 Nf6 5.O-O Be7 |
| 0xC4 |  0x600031c, 0xe000d24, 0x5000195, 0xd000e6a, 0x4000161, 0xe000c28, 0x4000858, 0xd000fad, 0x100c107, 0xc000f74, 0x3000144, 0xe000c61, 0x4000611, 0xe000ceb, | 1.e4 e5 2.Nf3 Nc6 3.Bb5 a6 4.Ba4 Nf6 5.O-O Be7 6.Re1 b5 7.Bb3 d6 |
| 0xC5 |  0x600031c, 0xe000d24, 0x5000195, 0xd000e6a, 0x4000161, 0xe000c28, 0x4000858, 0xd000fad, 0x100c107, 0xc000f74, 0x3000144, 0xe000c61, 0x4000611, 0xe000ceb, 0x6000292, 0x900cf3f, | 1.e4 e5 2.Nf3 Nc6 3.Bb5 a6 4.Ba4 Nf6 5.O-O Be7 6.Re1 b5 7.Bb3 d6 8.c3 O-O |
| 0xC6 |  0x600031c, 0xe000d24, 0x5000195, 0xd000e6a, 0x4000161, 0xe000c28, 0x4000858, 0xd000fad, 0x100c107, 0xc000f74, 0x3000144, 0xe000c61, 0x4000611, 0xe000ceb, 0x6000292, 0x900cf3f, 0x60003d7, | 1.e4 e5 2.Nf3 Nc6 3.Bb5 a6 4.Ba4 Nf6 5.O-O Be7 6.Re1 b5 7.Bb3 d6 8.c3 O-O 9.h3 |
| 0xC7 |  0x600031c, 0xe000d24, 0x5000195, 0xd000e6a, 0x4000161, 0xe000c28, 0x4000858, 0xd000fad, 0x100c107, 0xc000f74, 0x3000144, 0xe000c61, 0x4000611, 0xe000ceb, 0x6000292, 0x900cf3f, 0x60003d7, 0xd000aa0, 0x400044a, 0xe000ca2, 0x60002db, | 1.e4 e5 2.Nf3 Nc6 3.Bb5 a6 4.Ba4 Nf6 5.O-O Be7 6.Re1 b5 7.Bb3 d6 8.c3 O-O 9.h3 Na5 10.Bc2 c5 11.d4 |
| 0xC8 |  0x600031c, 0xe000d24, 0x5000195, 0xd000e6a, 0x4000161, 0xe000c28, 0x4000858, 0xd000fad, 0x100c107, 0xc000f74, 0x3000144, 0xe000c61, 0x4000611, 0x900cf3f, | 1.e4 e5 2.Nf3 Nc6 3.Bb5 a6 4.Ba4 Nf6 5.O-O Be7 6.Re1 b5 7.Bb3 O-O |
| 0xC9 |  0x600031c, 0xe000d24, 0x5000195, 0xd000e6a, 0x4000161, 0xd000fad, | 1.e4 e5 2.Nf3 Nc6 3.Bb5 Nf6 |
| 0xCA |  0x600031c, 0xe000d24, 0x5000195, 0xd000e6a, 0x400015a, | 1.e4 e5 2.Nf3 Nc6 3.Bc4 |
| 0xCB |  0x600031c, 0xe000d24, 0x5000195, 0xd000e6a, 0x400015a, 0xd000fad, | 1.e4 e5 2.Nf3 Nc6 3.Bc4 Nf6 |
| 0xCC |  0x600031c, 0xe000d24, 0x5000195, 0xd000e6a, 0x60002db, 0xec0091b, | 1.e4 e5 2.Nf3 Nc6 3.d4 exd4 |
| 0xCD |  0x600031c, 0xe000d24, 0x5000195, 0xd000e6a, 0x60002db, 0xec0091b, 0x5c0055b, | 1.e4 e5 2.Nf3 Nc6 3.d4 exd4 4.Nxd4 |
| 0xCE |  0x600031c, 0xe000d24, 0x5000195, 0xd000e6a, 0x5000052, | 1.e4 e5 2.Nf3 Nc6 3.Nc3 |
| 0xCF |  0x600031c, 0xe000d24, 0x5000195, 0xd000e6a, 0x5000052, 0xd000fad, | 1.e4 e5 2.Nf3 Nc6 3.Nc3 Nf6 |
| 0xD0 |  0x600031c, 0xe000d24, 0x5000195, 0xd000fad, | 1.e4 e5 2.Nf3 Nf6 |
| 0xD1 |  0x600031c, 0xe000d24, 0x5000195, 0xd000fad, 0x5c00564, 0xe000ceb, | 1.e4 e5 2.Nf3 Nf6 3.Nxe5 d6 |
| 0xD2 |  0x600031c, 0xe000d2c, | 1.e4 e6 |
| 0xD3 |  0x600031c, 0xe000d2c, 0x60002d3, | 1.e4 e6 2.d3 |
| 0xD4 |  0x600031c, 0xe000d2c, 0x60002db, | 1.e4 e6 2.d4 |
| 0xD5 |  0x600031c, 0xe000d2c, 0x60002db, 0xe000ce3, | 1.e4 e6 2.d4 d5 |
| 0xD6 |  0x600031c, 0xe000d2c, 0x60002db, 0xe000ce3, 0x6000724, 0xe000ca2, | 1.e4 e6 2.d4 d5 3.e5 c5 |
| 0xD7 |  0x600031c, 0xe000d2c, 0x60002db, 0xe000ce3, 0x6000724, 0xe000ca2, 0x6000292, | 1.e4 e6 2.d4 d5 3.e5 c5 4.c3 |
| 0xD8 |  0x600031c, 0xe000d2c, 0x60002db, 0xe000ce3, 0x5000052, | 1.e4 e6 2.d4 d5 3.Nc3 |
| 0xD9 |  0x600031c, 0xe000d2c, 0x60002db, 0xe000ce3, 0x5000052, 0xc000f59, | 1.e4 e6 2.d4 d5 3.Nc3 Bb4 |
| 0xDA |  0x600031c, 0xe000d2c, 0x60002db, 0xe000ce3, 0x5000052, 0xc000f59, 0x6000724, | 1.e4 e6 2.d4 d5 3.Nc3 Bb4 4.e5 |
| 0xDB |  0x600031c, 0xe000d2c, 0x60002db, 0xe000ce3, 0x5000052, 0xc000f59, 0x6000724, 0xe000ca2, | 1.e4 e6 2.d4 d5 3.Nc3 Bb4 4.e5 c5 |
| 0xDC |  0x600031c, 0xe000d2c, 0x60002db, 0xe000ce3, 0x5000052, 0xc000f59, 0x6000724, 0xe000ca2, 0x6000210, 0x4ca00652, 0x6800252, | 1.e4 e6 2.d4 d5 3.Nc3 Bb4 4.e5 c5 5.a3 Bxc3+ 6.bxc3 |
| 0xDD |  0x600031c, 0xe000d2c, 0x60002db, 0xe000ce3, 0x5000052, 0xd000fad, | 1.e4 e6 2.d4 d5 3.Nc3 Nf6 |
| 0xDE |  0x600031c, 0xe000d2c, 0x60002db, 0xe000ce3, 0x5000052, 0xd000fad, 0x40000a6, | 1.e4 e6 2.d4 d5 3.Nc3 Nf6 4.Bg5 |
| 0xDF |  0x600031c, 0xe000d2c, 0x60002db, 0xe000ce3, 0x500004b, | 1.e4 e6 2.d4 d5 3.Nd2 |
| 0xE0 |  0x600031c, 0xe000d2c, 0x60002db, 0xe000ce3, 0x500004b, 0xe000ca2, | 1.e4 e6 2.d4 d5 3.Nd2 c5 |
| 0xE1 |  0x600031c, 0xe000d2c, 0x60002db, 0xe000ce3, 0x500004b, 0xd000fad, | 1.e4 e6 2.d4 d5 3.Nd2 Nf6 |
| 0xE2 |  0x600031c, 0xe000d2c, 0x60002db, 0xe000ce3, 0x500004b, 0xd000fad, 0x6000724, 0x1d000b73, | 1.e4 e6 2.d4 d5 3.Nd2 Nf6 4.e5 Nfd7 |
| 0xE3 |  0x600031c, 0xe000d2c, 0x60002db, 0xe000ce3, 0x500004b, 0xd000fad, 0x6000724, 0x1d000b73, 0x4000153, 0xe000ca2, 0x6000292, 0xd000e6a, 0x500018c, | 1.e4 e6 2.d4 d5 3.Nd2 Nf6 4.e5 Nfd7 5.Bd3 c5 6.c3 Nc6 7.Ne2 |
| 0xE4 |  0x600031c, 0xe000dae, | 1.e4 g6 |
| 0xE5 |  0x600031c, 0xe000dae, 0x60002db, | 1.e4 g6 2.d4 |
| 0xE6 |  0x600031c, 0xe000dae, 0x60002db, 0xc000f76, | 1.e4 g6 2.d4 Bg7 |
| 0xE7 |  0x600031c, 0xe000dae, 0x60002db, 0xc000f76, 0x5000052, | 1.e4 g6 2.d4 Bg7 3.Nc3 |
| 0xE8 |  0x600031c, 0xe000dae, 0x60002db, 0xc000f76, 0x5000052, 0xe000ceb, | 1.e4 g6 2.d4 Bg7 3.Nc3 d6 |
| 0xE9 |  0x600031c, 0xd000fad, | 1.e4 Nf6 |
| 0xEA |  0x600031c, 0xd000fad, 0x6000724, 0xd000b63, | 1.e4 Nf6 2.e5 Nd5 |
| 0xEB |  0x600031c, 0xd000fad, 0x6000724, 0xd000b63, 0x60002db, 0xe000ceb, | 1.e4 Nf6 2.e5 Nd5 3.d4 d6 |
| 0xEC |  0x600031c, 0xd000fad, 0x6000724, 0xd000b63, 0x60002db, 0xe000ceb, 0x5000195, | 1.e4 Nf6 2.e5 Nd5 3.d4 d6 4.Nf3 |
| 0xED |  0x600035d, | 1.f4 |
| 0xEE |  0x6000396, | 1.g3 |
| 0xEF |  0x5000195, | 1.Nf3 |
| 0xF0 |  0x5000195, 0xe000ca2, | 1.Nf3 c5 |
| 0xF1 |  0x5000195, 0xe000ca2, 0x600029a, | 1.Nf3 c5 2.c4 |
| 0xF2 |  0x5000195, 0xe000ce3, | 1.Nf3 d5 |
| 0xF3 |  0x5000195, 0xe000ce3, 0x600029a, | 1.Nf3 d5 2.c4 |
| 0xF4 |  0x5000195, 0xe000ce3, 0x60002db, | 1.Nf3 d5 2.d4 |
| 0xF5 |  0x5000195, 0xe000ce3, 0x6000396, | 1.Nf3 d5 2.g3 |
| 0xF6 |  0x5000195, 0xe000dae, | 1.Nf3 g6 |
| 0xF7 |  0x5000195, 0xd000fad, | 1.Nf3 Nf6 |
| 0xF8 |  0x5000195, 0xd000fad, 0x600029a, | 1.Nf3 Nf6 2.c4 |
| 0xF9 |  0x5000195, 0xd000fad, 0x600029a, 0xe000ca2, | 1.Nf3 Nf6 2.c4 c5 |
| 0xFA |  0x5000195, 0xd000fad, 0x600029a, 0xe000d2c, | 1.Nf3 Nf6 2.c4 e6 |
| 0xFB |  0x5000195, 0xd000fad, 0x600029a, 0xe000dae, | 1.Nf3 Nf6 2.c4 g6 |
| 0xFC |  0x5000195, 0xd000fad, 0x600029a, 0xe000dae, 0x5000052, | 1.Nf3 Nf6 2.c4 g6 3.Nc3 |
| 0xFD |  0x5000195, 0xd000fad, 0x6000396, | 1.Nf3 Nf6 2.g3 |
| 0xFE |  0x5000195, 0xd000fad, 0x6000396, 0xe000dae | 1.Nf3 Nf6 2.g3 g6 |

*Note: 0x00 is unused, and 0xFF indicates the end of the stored line lookup table*

##### Moves
###### Move Structure
| **Bit**   | 31-24 | 23-16 | 15-0 |
| --- | --- | --- | --- |
| **Name**  | SAN Conversion | Undo | Stockfish-compatible Move |

###### Move Specification
| **Bit**   | 31 | 30 | 29 |  28 | 27 | 26-24 | 23-21 | 20-18 | 17-16 | 15-14 | 13-12 | 11-6 | 5-0 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| **Name**  | Special Flag (unused) | Check | Ambiguous Move: Include Rank in SAN | Ambiguous Move: Include File in SAN | Black to Move | Moving Piece | Captured Piece | En Passant File (unused) | Castling Flags (unused) | Special Move Flag | Promotion Piece Type | Origin Square | Destination Square |

###### Moving/Captured Piece
| Value | Piece |
| --- | --- |
| 1 | King |
| 2 | Queen |
| 3 | Rook |
| 4 | Bishop |
| 5 | Knight |
| 6 | Pawn |

###### Special Move Flags
| Value | Special Move |
| --- | --- |
| 0 | None |
| 1 | Promotion |
| 2 | En Passant |
| 3 | Castling |

###### Promotion Piece Types
| Value | Piece Type |
| --- | --- |
| 0 | KNIGHT |
| 1 | BISHOP |
| 2 | ROOK |
| 3 | QUEEN |

Note: This is equal to the Stockfish [PieceType](https://github.com/official-stockfish/Stockfish/blob/7c30091a92abddb8265e53768b32751c49642040/src/types.h#L197) - 2

###### Squares

[Little-endian Rank/File Mapping](https://www.chessprogramming.org/Square_Mapping_Considerations#LittleEndianRankFileMapping)

|       | A  | B  | C  | D  | E  | F  | G  | H  |
|  ---  |--- |--- |--- |--- |--- |--- |--- |--- |
| **8** | 56 | 57 | 58 | 59 | 60 | 61 | 62 | 63 | 
| **7** | 48 | 49 | 50 | 51 | 52 | 53 | 54 | 55 | 
| **6** | 40 | 41 | 42 | 43 | 44 | 45 | 46 | 47 | 
| **5** | 32 | 33 | 34 | 35 | 36 | 37 | 38 | 39 | 
| **4** | 24 | 25 | 26 | 27 | 28 | 29 | 30 | 31 | 
| **3** | 16 | 17 | 18 | 19 | 20 | 21 | 22 | 23 | 
| **2** | 8  | 9  | 10 | 11 | 12 | 13 | 14 | 15 | 
| **1** | 0  | 1  | 2  | 3  | 4  | 5  | 6  | 7  | 

#### Final Material Signature
| **Bit**   | 23-22 | 21-20 | 19-18 | 17-16 | 15-12 | 11-10 | 9-8 | 7-6 | 5-4 | 3-0 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| **Name**  | Number of White Queens | Number of White Rooks | Number of White Bishops | Number of White Knights | Number of White Pawns | Number of Black Queens | Number of Black Rooks | Number of Black Bishops | Number of Black Knights | Number of Black Pawns |

*Note: Only counts from 0 to 7 are possible for pawns, and only counts from 0 to 3 are possible for pieces.*

#### Number of Plies / Home Pawn Data
| **Bit**   | 71-70 | 69-0 |
| --- | --- | --- |
| **Name**  | Number of Plies High | Home Pawn Data |

##### Number of Plies
| **Bit**   | 9-8 | 7-0 |
| --- | --- | --- |
| **Name**  | Number of Plies High | Number of Plies Low |

##### Home Pawn Data
| **Bit**   |  69-65 | 63-0 |
| --- | --- | --- |
| **Name**  | Data Length | Data |

Data Length is a uint6 containing the number of valid entries in Data.
Data is an array of uint4 index values indicating that the pawn at that index moved from its home square in the order the array is populated. The first index is in the most significant nibble of Data.

# The NameBase File (.sn4)

> There are four NameBases, one each for PLAYER, EVENT , SITE and ROUND tags
> Contains all Player, Event, Site and Round names used in the database. Each name is stored only once even if it occurs in many games, and there is a database restriction on the number of unique names. The limits are -
>
> Player names: 2^20 - 1
> Event names: 2^19 - 1
> Site names: 2^19 - 1
> Round names: 2^18 - 1
> 
> and are defined in namebase.h The name file is usually the smallest of the three database files.

# TODO

# The Game File (.sg4)
> This file contains the actual moves, variations and comments of each game.
>
> The move encoding format is very compact: most moves take only a single byte (8 bits)! This is done by storing the piece to move in 4 bits (2^4 = 16 pieces) and the move direction in another 4 bits. Only Queen diagonal moves cannot be stored in this small space. This compactness is the reason Scid does not support chess variants.

# TODO
