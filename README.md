# Outfits System

![Gif that shows debugging functionality that is unique to pokeemerald-expansion such as rerolling Trainer ID, Cheat Start, PC from Debug Menu, Debug PC Fill, Pokémon Sprite Visualizer, Debug Warp to Map, and Battle Debug Menu](https://github.com/user-attachments/assets/cf9dfbee-4c6b-4bca-8e0a-07f116ef891c) ![Gif that shows overworld functionality that is unique to pokeemerald-expansion such as indoor running, BW2 style map popups, overworld followers, DNA Splicers, Gen 1 style fishing, OW Item descriptions, Quick Run from Battle, Use Last Ball, Wild Double Battles, and Catch from EXP](https://github.com/user-attachments/assets/383af243-0904-4d41-bced-721492fbc48e) ![Gif that shows off a number of modern Pokémon battle mechanics happening in the pokeemerald-expansion engine: 2 vs 1 battles, modern Pokémon, items, moves, abilities, fully customizable opponents and partners, Trainer Slides, and generational gimmicks](https://github.com/user-attachments/assets/50c576bc-415e-4d66-a38f-ad712f3316be)

This is a feature branch that implements _Outfits System_ into Pokémon Emerald, which also adds:
*  SDH's [commit](<https://github.com/ShinyDragonHunter/pokeemerald/commit/05f8f2688b22454e9d2400db1621375f1e4ccb3c>) for simplifying the player states system and thus making this system easier to implement. (Required)
* An item called _`OUTFIT BOX`_ for storing and changing the player's current outfit.
* A menu for changing outfits that is called in the OUTFIT BOX item. (Can also be configured to be called somewhere else if wanted)
* Several scripting macros for unlocking an outfit, checking the state of an outfit and buffers an outfit's name/description.

This is _not_ a faithful port of Gen 6's Outfits System. So, if you want it to behave similarly to Gen 6's system, you're likely gonna have to go on your own there. However, questions regarding the Outfits System is welcomed.

Note that this feature branch is still missing some of the features that's necessary, such as the ability to purchase outfits. So, stay tune for more until then!

**`pokeemerald-expansion`** offers hundreds of features from various [core series Pokémon games](https://bulbapedia.bulbagarden.net/wiki/Core_series), along with popular quality-of-life enhancements designed to streamline development and improve the player experience. A full list of those features can be found in [`FEATURES.md`](FEATURES.md).

<!-- TODO: Too busy to fix, so I'll put it here -->
### - Trainer Card shows Player's trainer pic for Link (Cable) Players

## Credits

If you use **`pokeemerald-expansion`**, please credit **RHH (Rom Hacking Hideout)**. Optionally, include the version number for clarity.

```
Based off RHH's pokeemerald-expansion 1.14.1 https://github.com/rh-hideout/pokeemerald-expansion/
```
These are some of the tools I've made for my hack, aiming to provide a framework to develop your own overworld encounter feature. These rely heavily on **merrp**'s followers branch, so credit goes to them as well as the maintainers who have been recently added it to **pokeemerald-expansion**.

> **Note**: This branch tracks **pret**'s **pokeemerald**. Please cherry-pick the linked commits or copy them directly.

## Features
- [x] Create a wander in grass movement type for overworld encounters. [Found here](https://github.com/Pawkkie/Team-Aquas-Asset-Repo/wiki/Create-Wander-in-Grass-Movement-Type).
- [x] Create a way to start a wild encounter based on an overworld mon object event, without having to specify the species in a script.
- [x] Create a way to specify the species of an overworld mon object event from a list of species.
- [x] Create a way to specify the species of an overworld mon object event from map wild encounter tables.

### Example - Petalburg Good Rod Encounters
![](https://github.com/HashtagMarky/pokeemerald/blob/ikigai/ow-encounters/ikiagi_ow_encounters.gif)

#### .inc format
```
.set LOCALID_OW_MON 1

PetalburgCity_MapScripts::
    map_script MAP_SCRIPT_ON_TRANSITION, PetalburgCity_OnTransition
    .byte 0

If you're new to git and GitHub, [Team Aqua's Asset Repo](https://github.com/Pawkkie/Team-Aquas-Asset-Repo/) has a [guide to forking and cloning the repository](https://github.com/Pawkkie/Team-Aquas-Asset-Repo/wiki/The-Basics-of-GitHub). Then you can follow one of the following guides:

PetalburgCity_EventScript_OverworldMon::
    lock
    callnative GetOverworldMonSpecies
    bufferspeciesname STR_VAR_1, VAR_0x8004
    msgbox PetalburgCity_Text_OverworldMon, MSGBOX_DEFAULT
    closemessage
    startoverworldencounter 5
    release
    end
```

#### .pory format
```
const LOCALID_OW_MON = 1

mapscript PetalburgCity_MapScripts
{
    MAP_SCRIPT_ON_TRANSITION: PetalburgCity_OnTransition
}

script PetalburgCity_OnTransition
{
    setobjectaswildencounter(LOCALID_OW_MON, ENCOUNTER_GOOD_ROD)
}

PetalburgCity_EventScript_OverworldMon::
    lock
    callnative(GetOverworldMonSpecies)
    bufferspeciesname(STR_VAR_1, VAR_0x8004)
    msgbox(PetalburgCity_Text_OverworldMon, MSGBOX_DEFAULT)
    closemessage
    startoverworldencounter(5)
    release
    end
```

For help setting up mapscripts, please see the [Team Aqua's Hideout Video Tutorial](https://youtu.be/b7AQP6WND-4?si=7q06ybC2k65GkQI6), and if you need it, the [poryscript guide](https://github.com/huderlem/poryscript#mapscripts-statement).

Our community uses the [ROM Hacking Hideout (RHH) Discord server](https://discord.gg/6CzjAG6GZk) to communicate and organize. Most of our discussions take place there, and we welcome anybody to join us!
