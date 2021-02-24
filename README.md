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

All numbers are stored in Big Endian format unless otherwise noted

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
| Length Low | 2 | uint | Two least significant bytes (Big Endian) of the entry length in the GameBase |
| Length High | 1 | uint | Most significant byte of the entry length in the GameBase. On Scid versions < 4 this value did not exist, so should be skipped if trying to read an older version file. |
| Flags | 2 | custom | Custom Index Flags (see below). |
| White/Black High | 1 | custom | Most significant bits of the White and Black Player IDs in the NameBase. |
| White ID Low | 2 | custom | Two least significant bytes of the White Player ID in the NameBase. |
| Black ID Low | 2 | custom | Two least significant bytes of the Black Player ID in the NameBase. |
| Event/Site/Round High | 1 | custom | Most significant bits of Event, Site, and Round IDs in the NameBase. |
| Event ID Low | 2 | custom | Least significant bytes of the Event ID in the NameBase. |
| Site ID Low | 2 | custom | Least significant bytes of the Site ID in the NameBase. |
| Round ID Low | 2 | custom | Least significant bytes of the Round ID in the NameBase. |
| Var Counts | 2 | UNKNOWN | UNKNOWN |
| ECO Code | 2 | uint | Encyclopedia of Chess Openings Code for the game. |
| Dates | 4 | custom | The date of the Game and date of the Event. |
| White Elo | 2 | uint | Elo rating of the White Player, limited to 4000 since Elo ratings are stored interally in 12 bits. |
| Black Elo | 2 | uint | Elo rating of the Black Player, limited to 4000 since Elo ratings are stored interally in 12 bits. |
| Final Mat Sig | 4 | UNKNOWN | UNKOWN |
| Number of Plies Low | 1 | uint | Least signficant byte of the number of plies (half moves) played in the Game. |
| Number of Plies High / Home Pawn Data | 9 | custom | Most significant bits of number of plies and Home Pawn Data. |

**Total Length of Index Entry: 47 bytes**

### Description of Custom Data Types

#### Flags

#### White/Black ID

#### Event/Site/Round ID

#### ECO Code

#### Final Mat Sig

#### Number of Plies

#### Home Pawn Data


# The NameBase File (.sn4)

There are four NameBases, one each for PLAYER, EVENT , SITE and ROUND tags

# The GameBase File (.sg4)
