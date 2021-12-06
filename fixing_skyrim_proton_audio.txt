# Intro

Instructions adapted from: https://old.reddit.com/r/linux_gaming/comments/99i4se/skyrim_on_linux_steam_play_no_voiceno_music_audio/

Skyrim has audio issues with music and voice in Wine for me.  I installed Skyrim
Special Edition using Steam's compatibility (named Proton).  So, we can't use
the system-wide Wine for this, but Proton doesn't provide all the right scripts.

(note, in the above thread it reports that Proton 3.16 fixes this, so consider
trying that first; I totally didn't)


# Fix

Here's how I fixed it.

First, we need to use Proton's Wine install:

    export PATH="$HOME/.steam/debian-installation/steamapps/common/Proton - Experimental/files/bin:$PATH"

Let's also change directory to the right game.  This is Skyrim SE's directory:

    cd "~/.steam/steam/steamapps/compatdata/489830"

Next, we don't have a winecfg for Proton's Wine, but winecfg is normally a
script that calls another script, <prefix>/lib/wine/wineapploader, and executes
`<name>.exe` (minus anything after and including a hyphen).  You can read the
source to verify this by opening the system /usr/lib/wine/wineapploader if you
have Wine installed via the package manager.  Here is the re-created command to
do so (remember to run in the above directory; you might be able to get away
with just setting your prefix to the above cd'd path plus `/pfx`, but I did not
bother testing this):

    WINEPREFIX="$PWD/pfx" wine "winecfg.exe" ""

Go to the Libraries tab, then add `xaudio2_6` and `xaudio2_7` as new overrides,
setting them to `native` or `native,builtin`.

If all is fixed, you should hear title screen music.

Based on how Proton seems to have multiple Wine prefixes depending on the game,
this *shouldn't* affect any other Wine install.  On the downside, you'll have to
edit this for other games manually, but on the upside, you hopefully won't
affect everything if it needs different fixes.


# Potentially automated fix

(this might be useful if it's a hassle to get winecfg working, or you want to
save/restore profiles quickly via scripts)

The settings you change just change the user.reg file.  Here is the diff of the
relevant section I generated:

    -[Software\\Wine\\DllOverrides] 1638769670
    -#time=1d7ea64cde320f6
    +[Software\\Wine\\DllOverrides] 1638769853
    +#time=1d7ea653ae5c8c0
     "api-ms-win-crt-conio-l1-1-0"="native,builtin"
     "api-ms-win-crt-heap-l1-1-0"="native,builtin"
     "api-ms-win-crt-locale-l1-1-0"="native,builtin"
    @@ -750,674 +750,678 @@
     "vcomp120"="native,builtin"
     "vcomp140"="native,builtin"
     "vcruntime140"="native,builtin"
    +"xaudio2_6"="native,builtin"
    +"xaudio2_7"="native,builtin"

If I'm right, you just need to add the last two lines in the right spot, namely
at the end of the [Software\\Wine\\DllOverrides] section.  However, it seems
like it also changes where the font paths are pointed, which doesn't quite make
sense for an xaudio DLL override, so your mileage may vary.
