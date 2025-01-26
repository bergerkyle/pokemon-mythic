# Ikigai OW Encounter Tools
```
Based off RHH's pokeemerald-expansion 1.9.2 https://github.com/rh-hideout/pokeemerald-expansion/
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

PetalburgCity_OnTransition:
    setobjectaswildencounter LOCALID_OW_MON, ENCOUNTER_GOOD_ROD
    end

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

## Commits
### Start Overworld Encounters & Get Overworld Species
Start a wild battle against an overworld mon object event with a defined level.
```
.macro startoverworldencounter level=req
callnative GetOverworldMonSpecies
playmoncry VAR_0x8004, CRY_MODE_ENCOUNTER
createmon B_SIDE_OPPONENT, B_FLANK_LEFT, VAR_0x8004, \level, isShiny=VAR_0x8005
waitmoncry
dowildbattle
.endm
```

This commit adds a function, `GetOverworldMonSpecies`, that sets `VAR_0x8004` to a species based on the last selected object event and `VAR_8005` to it's shinyness. If the object event isn't tied to a species, it returns `SPECIES_NONE`, however this can be changed very easily. `GetOverworldMonSpecies` can be called at any point in a `.inc`/`.pory` scripts though `callnative GetOverworldMonSpecies`/`callnative(GetOverworldMonSpecies)` or normally in C. It is used in the `startoverworldencounter` macro which plays the relevant species cry before starting a wild encounter.

> **Note**: ~~There is a known bug that I haven't yet squashed where adding a value an objects Sight Radius / Berry Tree ID may cause a species value to not be obtained correctly.~~ The function that caused this bug was removed in `pokeemerald-expansion 1.10.0`. [This was pull request that removed it](https://github.com/rh-hideout/pokeemerald-expansion/pull/5475).

#### October 3rd 2024
[Commit: de53fa5e2d](https://github.com/HashtagMarky/pokeemerald/commit/de53fa5e2d1e46efe09e1d829137d030be81d402)

---
### Set Object as Wild Encounter
Sets a variable object event to an overworld mon for a wild encounter. It should be noted, even though `.byte 0xe5` was used, you should change it to the next free value in `gScriptCmdTable` found in `scr_cmd_table.inc`.
```
#define ENCOUNTER_FIXED         0
#define ENCOUNTER_LAND          1
#define ENCOUNTER_SURF          2
#define ENCOUNTER_ROCK_SMASH    3
#define ENCOUNTER_OLD_ROD       4
#define ENCOUNTER_GOOD_ROD      5
#define ENCOUNTER_SUPER_ROD     6
#define ENCOUNTER_TYPES         7

.macro setobjectaswildencounter localId:req, encounterType=ENCOUNTER_FIXED
.byte 0xe5
.2byte \localId
.byte \encounterType
release
.endm
```
![Variable Object Event Usage Example](https://github.com/user-attachments/assets/6fa37858-f964-4916-8d35-c973f570b269)

This commit adds a few functions in order to change a variable object event into a species ready for an overworld encounter, by using the `setobjectaswildencounter` macro. The called function will read the object associated with the `localId` that is given, checking if it is a variable object event. If this check is passed, the variable associated with it is automatically set to allow for an overworld species to spawn.

By setting the encounter type to `ENCOUNTER_FIXED`, the species will be set using the function `ReturnFixedSpeciesEncounter`. This can be supplemented by adding to the switch statement, potentially with random chances or even by using map headers to determine encounters by map section or type. Using any other type will return a random species from the relevant encounter tables used.

This also uses `SPAWN_ODDS` to define whether or not the object should be shown by setting it's flag. Objects are set to always spawn by default, but if you would like to decrease these odds, I would recommend giving the object temporary flags in order to allow them a new chance to spawn when transitioning to the map again. This also allows encounters to be removed after battle, and have a chance to respawn after leaving the area.

> **Note**: I haven't played around with variable object events too much, and these may not play too nicely everywhere in vanilla as they are set at various points. The script `Common_EventScript_SetupRivalGfxId` is one way this is done, setting `VAR_OBJ_GFX_ID_0` and `VAR_OBJ_GFX_ID_3` when transitioning to many maps.

#### October 4th 2024
[Commit: 28156b0b5f](https://github.com/HashtagMarky/pokeemerald/commit/28156b0b5fa6933f42c5673026d046d5e6c2e566)

---
### Automatically Give Wild Encounters the Same Level as Normal Wild Encounters

> **Note**: The changes also refactor some of the functions in `wild_encounter.c` to return `struct WildMon` instead of just a species constant.

Thanks to the changes in this [Pull Request](https://github.com/HashtagMarky/pokeemerald/pull/1) by [LordRainDance2](https://github.com/lordraindance2), overworld encounters can now have their levels dynamically set the same way normal wild encounters do. This also allows for a common event script to be defined using the new macro `startwildoverworldencounter` which remove the need to make a script for every single overworld encounter, working as an useall solution. The individual macros can still be used in individual scripts, however.

```
Common_EventScript_OWWildEncounter::
	lock
	faceplayer
	startwildoverworldencounter
	release
	end
```

This PR make it easier to focus on implementing the encounters with minimal setup or making more scripts, though the objects still have to be defined as wild encounters though in poryscript.

## Further Work
My own implementation of dedicated overworld encounters feature is still a giant WIP, but if I make any updates on these tools or develop any others I will update this page. If you find any bugs, or even have any fixes, please feel free to submit a PR or ping me (HashtagMarky) a message on the TAH Discord.  
