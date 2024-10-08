# Conversion from TinyMUX to RhostMUSH
City of Hope, a classic World of Darkness game, was released to the public some time ago. However, its database is circa 2013, and is based on TinyMUX 2.9+

This guide will walk you through getting it working on [RhostMUSH](https://github.com/RhostMUSH/trunk) with some modernizing and added qol/missing data in a jiffy.

## Step 1: Install RhostMUSH
This readme assumes you at least know the basics of what you're doing by installing softcode and running a game, have a host, know how to get the game running and back it up, edit its files, etc. If you don't, I'd suggest checking out these fine people over at the Rhost Dev server, `rhostdev.mushpark.com 4201`, or their [Discord](https://discord.gg/RxxtfVRYA4) - most coders there could easily get you going in a flash.

You should refer to the [official Rhost installation guide](https://github.com/RhostMUSH/trunk/wiki/Installation) while following these instructions. I can't write instructions that cover every key-press and command.

These instructions written on 8/1/2024 for Rhost 4.4.2. Rhost's default settings may change in the future, so these instructions and provided settings may not be accurate for future versions.

### Obtain RhostMUSH and merge in data
`git clone https://github.com/RhostMUSH/trunk rhost`

Copy the `Server` folder into the rhost directory and merge the contents. Overwrite whatever needs to be overwritten.

### Configure RhostMUSH
`cd rhost/Server`

`make confsource`

#### make confsource options:
* 7: Disable hardcoded +help
* 22: Read Mux Passwords
* B6: LBUF size. Option 4 (32K) is recommended for massively improved QoL.

#### Compile!
`[r]un` the compile and use `make links` when prompted.

#### Handle ulimit
_DO NOT SKIP THIS STEP!_

It's very important that RhostMUSH be given a sufficient stack size, or it will crash without warning under heavy load and likely corrupt your database. With 32kb LBUFs and more The RhostMUSH server requires an increased stack size over the defaults.

On Linux (I did my testing on Debian/Ubuntu) this is handled by the `ulimit` command.

The easiest way to ensure it always has sufficient stack size is to add a command to the `Startmush` file in `rhost/Server/game`.

For example, it could look like:

```
#!/bin/sh
#
#       Startmush - Kick off the netmush process.
#
ulimit -s 128000
lc_chk=`echo "$1"|tr '[:upper:]' '[:lower:]'`
```

## Step 2: Convert the Database
`cd ~/rhost/Server/convert`

`./doconvert.sh cityofhope.FLAT rhost.out`

(version TinyMUX 2.9+, defaults Y for everything)

Ignore the warning about 750 attributes. It's fine.

The converter will also create a file called `muxlocks.out`. Take note of this for later.

## Step 3: Load the Database

`cd ~/rhost/Server/game`

`./db_load data/netrhost.gdbm ~/rhost/Server/convert/rhost.out data/netrhost.db.new`

## Step 4: .conf files
Open up `rhost/Server/game/netrhost.conf` in your favorite editor and scroll through, making sure everything is set up correctly. Set up your port, mud name, etc.

At the bottom add this line:
```
include cityofhope.conf
```

`cityofhope.conf` also includes Rhost's `muxalias.conf` so no need to add that a second time.

It is, alternatively, highly recommended that you create something like a `mygame.conf` which will `include cityofhope.conf` and include your game-specific changes to rhost settings and etc.

## Step 5: Start up RhostMUSH
```
./Startmush -i

./Startmush
```
That will index helpfiles and then start the game.

## Step 6: Fix Sponge
```
@set *sponge=IMMORTAL !ROYALTY INHERIT
```

## Step 7: Softcode Function Wrappers
Rhost doesn't have some of the functions TinyMUX does and instead uses wrappers.

These are found in `rhost/Mushcode/softfunctions.minmax`

Paste the file's contents into your client to create the object then `@tel SoftFunctions=#32` to stuff it in the Data Closet. Finally you can `@reboot` to make sure it's all loaded.

## Step 8: Locks
First, enter this command:
```
@cmdquota me=99999
```

Then, paste in the contents of the `muxlocks.out` file generated by the converter.

## Step 9: Myrddin's MUSHCron
Get latest CRON from [here](https://bitbucket.org/myrddin0/myrddins-mush-code/src/master/)

Snip out the line that starts with `@create mushcron`

Go to the master room via `@tel me=#2`

Paste the remaining lines to patch it up.

Add the following to ensure the game dumps to flatfile periodically.

```
@tag/add cron=#756

&CRON_TIME_DUMP #cron=||||00 15 30 45|
&CRON_JOB_DUMP #cron=@dump/flat
```

## Step 10: Myrddin's BBS
Get latest BBS from [here](https://bitbucket.org/myrddin0/myrddins-mush-code/src/master/)

```
&dbref_bbs me=#52
&dbref_bbpocket me=#51
```
Run the install script after ensuring those attributes are set.

The 5.0 help has already been integrated into `plushelp.txt` but you may need to adjust it if there's a newer BBS out.

## Step 11: Install Help System.
Paste in `Help System.txt`'s contents to install support for the +help, +jhelp etc

## Step 12: Reindex help files.
In your `rhost/Server/game` directory, type:

`./Startmush -i`

This will reindex all the helpfiles.

## Step 13: Apply Patches
Paste in the contents of `Code Patches.txt` to the game to fix a bunch of bugs caused by the new code engine.

## Step 14: Apply WoD Data
Paste in the contents of `WoD Data.txt` to the game to add missing stats, merits, categories, etc.