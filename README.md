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
| Stored Line Code | 1 | UNKNOWN | UNKNOWN |
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
# TODO

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
# TODO

#### Final Material
# TODO

#### Number of Plies / Home Pawn Data
| **Bit**   | 71-70 | 69-0 |
| --- | --- | --- |
| **Name**  | Number of Plies High | Home Pawn Data |

##### Number of Plies
| **Bit**   | 9-8 | 7-0 |
| --- | --- | --- |
| **Name**  | Number of Plies High | Number of Plies Low |

##### Home Pawn Data
| **Bit**   |  69-64 | 63-0 |
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
