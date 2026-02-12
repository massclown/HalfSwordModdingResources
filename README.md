# HalfSwordModdingResources
A collection of modding resources for Half Sword.

## Engine
Half Sword Early Access (as of v0.6 released 31 Jan 2026) is built on Unreal Engine 5.4.4. 

Half Sword Demo v0.4 or Playtest are built on Unreal Engine 5.4.4. 

(The older versions like Demo v0.3 used to be on UE 5.1)

## Save file editing
The save files are in `%LOCALAPPDATA%\HalfSwordUE5\Saved\SaveGames` (usually meaning `C:\Users\YOUR_USERNAME_HERE\AppData\Local\HalfSwordUE5\Saved\SaveGames`) and the save file is `GameProgress.sav`. Make a copy of that.

The easiest to use is the uesave webpage https://trumank.github.io/uesave/ made by Truman Kilen. Load your `GameProgress.sav` file there, make changes (most of the variables are self-explanatory), download the edited save file, replace the local one with the new one. Again, make sure you have a good backup copy before all that.

## Modding approaches
In general, at run time you can inject your code or modify/objects variables in memory, or modify game assets on disk.

For Half Sword, as it is based on Unreal Engine, all of that can be done with ready-made tools (e.g. hooking engines to place hooks over the game's functions, or entire modding frameworks like UE4SS). 

Most code injection tools place a specially named DLL file (e.g. `dwmapi.dll`) with the tool's functionality in the game's directory right next to `HalfSwordUE5-Win64-Shipping.exe` and that library will be loaded by the operating system into the game process beacuse of that special name. Modifying assets on disk requires unpacking and re-packing them.

For an introduction to UE modding, take a look at this guide by Dmgvol: https://github.com/Dmgvol/UE_Modding

The rest of this document assumes that you are familiar with that, and can already do something to the game and are looking for exact hints for things you want to find/change.

## Finding interesting objects in UE
Note that UE has its own object system that builds on top of C++ object system. Objects contain other objects as members. Some objects are arrays or other collections of objects that you have to access accordingly. 

### Maps (Early Access)
* Abyss
* Alley
* Cellar
* Forest ("Ambush")
* Hall ("EastTower)
* Slums
* Yard (same as in Playtest)
* "LordsHall" (final boss map)

### Map (Demo v0.4)
* In the Gauntlet mode, the map class name is `Arena_Cutting_Map_C`
* In the Abyss mode, the map class name is `Abyss_Map_Open_Intermediate_C`
Find the first instance of this object in memory and you get the current map.

### GI_Settings
TBD

### WorldSettings
This is a standard UE object that holds interesting settings like the game speed. Find the first instance of the `WorldSettings` class and you have it.
* The game speed is `TimeDilation` under `WorldSettings` (this is standard for most UE-based games)
  * If `TimeDilation` is 1.0, then it is normal speed
  * `TimeDilation` < 1.0, means speed is slower, 0.1 = 10x slower
  * `TimeDilation` > 1.0, means speed is faster, 10.0 = 10x faster
  * A good value is usually 0.1 ~ 5.0 or as you wish, as long as it does not crash.
* Various interesting physics engine properties are also under `WorldSettings`
  * `bWorldGravitySet` is the boolean flag to toggle gravity override if true
  * `WorldGravityZ` is the gravity force, standard is 980.0

### Player character (Early Access)
TBD 

### Player character (Demo / Playtest)
* In the Gauntlet mode, the player character is a member of the map object named `Player (Temp)`
* In the Abyss mode, the player character is a member of the map object named `Player Willie`

Alternatively, you can find all instances of the `Willie_BP_C` class and then check which of those are controlled by the player (must be only one).

#### Character attributes
All these are members of the player object, **which also apply to other NPCs** as they are inherited from the same base class.

Only the game devs have a good idea what all these mean, but we can guess of course, as the names are pretty self-explanatory, but some variables/functions appear to do nothing, so try youself.

| Attribute/stat | Where to find it | Comments |
| ------------------ | ------------------ | ------------------ |
| Health | `Health` | 0.0-100.0, float |
| Consciousness | `Consciousness` | 0.0-100.0, float |
| Stamina | `Stamina` | 0.0-100.0, float |
| Tonus | `All Body Tonus` | 0.0-100.0, float |
| Invulnerability status | `Invulnerable` | boolean, true if invulnerable, false if vulnerable |
| Team | `Team Int` | 1-4, can be any integer number |
| Score | `Score` | Only in Abyss mode |
| Fallen? | `Fallen` | boolean |
| Character dead? | `DED` | boolean |
| Get Up Rate | `Get Up Rate` | float, usually 0.0-1.0 but can be more, how fast does the character get up from the ground |
| Regen Rate | `Regen Rate` | float, main health regeneration rate, above zero means regeneration is working |
| Running Speed Rate | `Running Speed Rate` | float, makes character run faster or slower |
| Walk Speed Rate | `Walk Speed Rate Run` | float, makes character walk faster or slower |
| Aiming speed | `Walk Speed Rate Aim` | float, makes character aim faster or slower |
| Muscle Power | `Muscle Power` | float, this seems to be the base modifier for character strength |
| Hand grip strength | `R Grab Force Limit`, `L Grab Force Limit`| float, maximum force when the hand grip breaks and releases |
| Hand rigidity | `Hands Rigidity (Gauntlets)` | float, makes the hands more or less rigid |

On top of that, the character object has the following objects as members:
| Attribute/stat | Where to find it | Comments |
| ------------------ | ------------------ | ------------------ |
| Controller object | `Controller` | UE `Controller` class, which is actually an instance of the UE `PlayerController` class for the player character, or `AIController`(`AI_BP_C`) class for NPCs |
| Mesh object | `Mesh` | UE `Mesh` class |

The player object has the following functions that can be called on it:
| Function name | Where to find it | Comments |
| ------------------ | ------------------ | ------------------ |
| Save loadout | `Save Loadout` | Used to work in Playtest only, before Demo v0.4, no idea what is the current state |
| Kill the character | `Death` | Triggers a simple death animation and kills the character |
| Explode Head | `Explode Head` | No longer works. Before Playtest, in Demo v0.3, it exploded the head and killed the character |
| Spill Guts | `Spill Guts` | No longer works. Before Playtest, in Demo v0.3, it exploded the stomach, spilling guts and killed the character |
| Snap Neck | `Snap Neck` | Since Demo v0.4, appears to break the neck of the character |

### NPCs spawned by the game
* In the Gauntlet mode:
   * the enemy NPCs are in an array which is member of the map object named `Enemies`
* In the Abyss mode:
   * the enemy NPCs are in an array which is member of the map object named `Enemy Array`
   * the boss is the `Boss` object under 'Current Boss Arena' under the map object.
     * the status of the boss is the `Boss Alive` variable under the map object.
   * any other NPCs spawned in a boss arena are inside `Spawned Enemies` array under 'Current Boss Arena' under the map object.

### Assets
Assets can be explored with any asset viewing tool for UE 5, e.g. the latest version of FModel.

The assets have obvious paths like `/Game/Assets/Weapons/`, `/Game/Assets/Armor/` and so on.

### Spawning anything
Spawning the assets or NPCs in the mod can be done by calling UE's `UWorld::SpawnActor()` function and supplying the class and other parameters, e.g. location or `FTransform` (location+rotation+scale).

The assets must be loaded before they can be spawned.

#### Despawning 
To despawn an object, call its `K2_DestroyActor()` method.
