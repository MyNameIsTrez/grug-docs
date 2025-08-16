---
weight: 200
---
# Writing Mods
Grug mods are located in the mods folder, the location of which is dependent on
the game (check your game's documentation for where exactly it is). The folder
contains any amount of subfolders. Each subfolder that contains an `about.json`
is a mod. `.grug` script files may be placed anywhere within the mod's
directory.
## about.json
The `about.json` is a [JSON](https://en.wikipedia.org/wiki/JSON) file in a mod
consists of an object with the following fields, in this order:

* `name` - Name of the mod.
* `version` - Version of the mod.
* `game_version` - Version of the game that this mod is made for.
* `author` - Author of the mod. (that's you!)
Each field is a string.

In addition, more fields may be available, depending on your game. See your
game's documentation for more information.
!!! note
    If you're unfamiliar with the JSON format, [this
    article](https://zipsegv.net/guides/json.html) may be helpful to you.

An example `about.json` is below:
```json
{
	"name": "magic",
	"version": "1.0.0",
	"game_version": "1.0.0",
	"author": "MyNameIsTrez"
}
```
## Script files
Script files may be placed anywhere in the mod, and may reside in subfolders.
An example directory structure of a mod is provided below:
```
magic
├── about.json
├── mage-human.grug
├── potions
│   └── health-tool.grug
└── weapons
    └── lightning-tool.grug
```
