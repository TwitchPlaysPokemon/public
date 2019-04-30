# Preface 

Most of this document is info-providing, but a few portions are more heavily opinion-based, and should just be considered as points for discussion.

# Features

## Deployed
1. Complete support for defiance double battles
1. Overlay support for 3v3 & 4v4
1. [Live data (HP, PP, moves, ability, etc) reading & writing for all Pokemon](#live-data-reading/writing)
1. Detection of all softlocks
1. Ability to retry broken matches, both automatically and manually
1. Faster PBR match setup (40s first start, 18s every match after) which also helps stability
1. Toggleable announcer during matches
1. Ability to avoid trying the same move / switch more than once (toggle on for defiance at least)
1. [Atomic input selections](##atomic-inputs)
1. Modifiable input selection timer (in seconds)
1. Modifiable match timer (a legitimate PBR feature, match ends after x minutes)
1. Per-mode token match cooldowns

## Pending
1. Support teams sizes other than 3v3 and 4v4
1. Global modifications to species, moves, and effectiveness
1. Battle royale matches (pbr double battles with chat split into four teams)
1. Mysterious surprise feature (won't affect gameplay)

# Live data reading/writing

## Caveats

Internally, PBR doesn't update internal data (hp, moves, etc) in a play-by-play fashion. Instead it usually* updates twice per turn, in this order:  
1: Update data to the result after all attacks are accounted for. Occurs immediately after all selections are made for the turn, but before the attack animations actually play.  
2: Update data to the result after residual damage like hail and poison. Occurs immediately after all move/attack animations finish playing, but before residual damage animations play.

As far as I can tell, PBR doesn't even store the intermediate HP values between multiple attacks to a single Pokemon per turn anywhere in memory.

Example 1: Pidgey gets damaged several times (attacked by 2 Pokemon + recoil + hail + poison). Internal HP is decreased exactly twice:    
1. decreased for both attacks + recoil before attack animations play  
2. decreased for hail + poison before the residual dmg animations play 

Example 2: Ditto transforms and then gets roared away in the same turn. The internal teams data never gets updated with Ditto's new moves, stats, etc, because Ditto doesn't have those at the end of the turn.

*Data may more than twice per turn after mid-turn user input, such as selecting which Pokemon to switch to after baton pass. 

## Modifying

Most live data is modifiable, but the caveats above apply and cause some limitations. These apply particularly to any volatile conditions.  Volatile conditions can only be written for active (currently sent out) Pokemon, because inactive Pokemon literally don't have bytes representing volatile conditions.   

### Nonvolatile conditions

- HP 
- status, including toxic countup and sleep countdown.  
  
These can all be set before the first Pokemon of the match get sent out.

Notes:

While the HP bar will report "FRZ", "SLP", etc, the first Pokemon sent out in the match will not appear to be frozen (not moving) or asleep (eyes closed). The animation does correct itself as soon as the Pokemon attempt to use their first move.

A Pokemon's ability or berry may trigger when being sent out (woken up by insomnia, using a sitrus berry on low HP)

### Volatile conditions

All Pokemon sent into battle can use a move on that turn. That move selection time is ideal for setting an active Pokemon's volatile condition. Setting such a condition in a gimmick might be described as:    
**Pokemon get \<volatile condition> immediately before using their first move.**

For example, if we wanted a mode where all Pokemon start at +1 defense, it is simply not possible to ensure Diglett gets +1 defense when it comes out mid-turn (think getting roared in, and then attacked) because the game will perform a single atomic step of sending in diglett and damaging it.  There is no way to stop the game after diglett is sent in, increase its defense, and allow it to proceed with taking damage based on it having +1.

Given these caveats, I think the "safe" conditions to set are:
- infatuation
- taunt
- torment
- focus (focus energy)
- curse
- confusion

Notes:

Curse would have to be described as mentioned previously: **Pokemon get cursed immediately before using their first move.**

Confusion is problematic because of how it can be cured.  A Pokemon confused and having either own tempo or a persim berry will be cured after any Pokemon- including itself- finishes using a move.  This basically means if it's the fasted Pokemon, it may self-hit before getting cured. We could just remove own tempo and persim berries if we want a confusion gimmick.

Pokemon can be infatuated with themselves. Remember that infatuation will end if the infatuatee leaves the field. 


# Species, moves, and effectiveness

We can also make global modifications to these entities.  These seem to work but aren't thoroughly tested yet.

Species
- base stats
- types
- probaby other things like weight

So if Ivysaur's data got changed to be a poison/ghost type, it would no longer be possible to have a regular Ivysaur in that match.

Moves
- base power
- accuracy
- type

Efectiveness
- change SE to NVE and back. Can be used to implement Inverse Battles.

# Overlay changes

See some mockups and past designs under [Resources](#resources)

According to the moveset team, doubles is generally faster than singles, and 4v4 doubles is roughly analogous to 3v3 singles.  Therefore we should aim for an overlay design that comfortably supports 4v4 doubles with targeting.

Matches greater than 4v4 generally take too long for TPP, especially in singles. That said, it'd be nice to support 6v6, as we might have some game formats (tournaments?) where a 6v6 as a finale could be appropriate.


## "New Overlay"

The "New Overlay" is being designed by Red, Lightning, and M4. This design will hopefully be released to the public sometime in 2019 Q2. After the overlay design is approved, it will need to be implemented in code, which will take several months at least.

## Temporary overlay redesign

While the new overlay is being worked on for a potentially long time, we have redesigned the current overlay so it can support 6v6 with doubles targeting.  The redesign's goal is to create an overlay with the best layout possible, but not with the impressive animations that the "New Overlay" is exepected to eventually have.

The redesign concepts were heavily influenced from multiple past overlays (see [Imgur Albums](###imgur-albums)) and also the "New Overlay", although the latter has not been released to the public yet.

Both the 2018 pbr overlay and 2019 temporary redesign are limited in their ability to support partial transparency. An overlay pixel is either solid a color or transparent- no slight darkening with opacity is possible.  This technical limitation is hoped to be overcome in the coding of the New Overlay.

Popularly requested improvements on the old (2018) overlay included:

- Show live pkmn data (HP, PP, etc)
- Remove stats or make them easier to read (could display on 2 rows, 3 stats per row, instead of the current 6 in a single row)
- Use a more legible font
- Easier to read pkmn names (could be white instead of blue / red)
- Easier to read ability / items (Could display on 2 rows instead of 1)
- Easier to read moves (could stop coloring moves according to the move percentage. Could also try all-caps names, which yields a bit more vertical space)
- Easier to read PP. Darken and cross out moves when they run out of PP.
- Wider sidebars, so there's more horizontal space for bet info. Darken PBR underneath the sidebars so that the sidebar info is more legible.

## Shrinking the PBR popup guis

The HP bars + move select popups as a whole can be shifted left/right/up/down or shrunk. 

Shrinking these popups a little lets us widen the sidebars.  This gives more space for vital info, but widen them too much and PBR might feel squished.

## Removing the PBR popup guis

One idea is to entirely remove the popups and replace them with our own custom ones. This is hopefully possible, but may require quite a lot of time to achieve on the PBR end of things.

We could then display moves, pp, etc. in big font for all the active Pokemon, and design better targeting / switch popups than what PBR currently has (they are really kind of awful imo).

## Separating active pkmn

 For each team, players need to easily tell:
- which pokemon are currently out
- which one is on the left side / top hp bar, and which is on the right side/ bottom hp bar

With the current static positioning (1-2-3-4, top to bottom) it's way too difficult to tell which pokemon are currently out. A little "button" indiciating which pokemon are out would be helpful, but for doubles with inputting, I really think it won't be enough.

Unfortunately, this also means that sidebar pokemon info will move around every time a switch is made.  It will be frustrating if a pokeset moves elsewhere while a player is reading it.   The active pokemon would also likely be outdated info for anyone with lag, or for everyone in speed mode when Pokemon are switched out very quickly.

#### Moving active pkmn out of the sidebars

We could add a custom GUI that doesn't popup but rather stays in place the whole match, and displays info of the active pkmn (2 in singles, up to 4 in doubles).  This frees up more space for player bets and othe sidebar info, especially in 4v4 - 6v6.

Moving the "Team Blue's x used y" text to the bottom of the screen would give more space for this concept, although our ability to do this is still very limited.  

In singles, the sidebar might be showing Pokemon 1 and 3, with 2 being in the active gui.  
In doubles, the sidebar might be showing Pokemon 2 and 4, with 3 and 1 being in the active gui.

# Match retrying & cancelling

As per mods' request, matches that break over 2 minutes after starting will be automatically cancelled, instead of retried via emulator restart.

1. Best way to deal with softlock from 0pp + more than one of the same move?
1. Should `!cancelmatch bluewins` be a thing?




# Targeting notation

### Background

[See doubles screens here](https://imgur.com/a/MkYUCXV)

Move selection goes: 
banette -> caterpie -> persian -> entei  
aka  
upper blue -> lower blue -> upper red -> lower red (going by the HP bar positions)  
aka  
blue's left mon -> blue's right mon -> red's left mon -> red's right mon (going by trainer POV)

You first select your move, and then the target for that move.

[See here for special targeting cases](https://bulbapedia.bulbagarden.net/wiki/Double_Battle#Effects_on_moves)

### General goals

- easy to explain and learn
- easy to make inputs (including on mobile)
- easy to understand inputs made by others
- both newbie and expert friendly

### Proposals

[Ax6's targeting proposal](https://gist.github.com/aaaaaa123456789/d9e3a0363b4234e5d76e63da5216e640)

### Active Pokemon positioning

If the active pokemon on a side are positioned left to right, the targeting notation of `L` and `R` makes the most sense. But if the active pokemon are positioned top and bottom, we may want to change that notation to `U` (upper) and `L` (lower) or similar. 

Note that the PBR HUD already positions the active pokemon health bars in a top-bottom fashion. 

While the actual Pokemon on the field are positioned in a left-right fashion from the trainer POV, it's far more difficult to see this than it is in a gen 6 double battle.  The camera rarely shows all four Pokemon at the same time- usually it just focuses on each of them in turn, or on both pokemon on a single team. Even when Pokemon attack, it's often difficult to tell the positions of either the attacker or defender.

There's also the fact that the red trainer's left-side pokemon is always to the right of their right-side pokemon, due to the camera angle.  


# Input selection process (moves, switch, targeting)

## Atomic inputs

As mentioned in [Features](#features), we can make input selections atomic with respect to player inputs.   
This prevents the red team from sitting on a 0pp move to react to blue's input. It also prevents blue team from sitting on a 0pp move, and watch red's input before making their selection.

If we make selections atomic, we would also prevent selection of the same move twice (as it'd be pointless to do so).

Another benefit is less delay when someone forgets to change move / tries to stall / leaves the stream / etc, particularly in Fragile mode.

This is more punishing for players who:
 - react too slowly
 - have difficulty reading PP (even though it's much easier to readnow)
 - miss the effects of Taunt, Torment, etc. that make their input invalid, as it will not be possible for the player to select an alternate input.

## Ally targeting

Some have suggested disallowing ally targeting some or all of the time, but ally targeting does have many strategic uses, such as:

**Moves**: sketch, transform, pain split, swagger, flatter, psych up, role play, power/guard/heart swap, acupressure, trick, switcheroo, embargo, mimic, sketch, worry seed, skill swap  
**Abilities**: water/volt absorb, motor drive, steadfast, anger point, flash fire, marvel scale, guts, quick/tangled feet, dry skin, poison heal, blaze/torrent/overgrow/swarm, own tempo

Ally targeting for doubles defiance in currently set to:

Key: `60%: 2%` means, `60%` of matches have a `2%` ally-target chance.

Note: a 25% ally-target chance currently shows up as `20% 20% 30% 30%` (self, ally, left foe, and right foe respectively).  This is because self-targeting is impossible for all ally-hit moves, with the exception of Acupressure, so the 25% comes from 20% / (20% + 30% + 30%)  This may be simplified to "Ally-hit chance: 25%" in the future.

```
35%: 0%
25%: 2%
20%: 5%
10%: 10%
 7%: 25%
 3%: 33%
```
as per a public-dev poll.
 
We might also move ally-target chance into a mode, for aesthetic purposes.  We'd need an icon though.

# Battle Royale

An imitation "Battle Royale" mode, which would just be a pbr doubles battle, with chat split into four teams. 

Example:

Chat is split into teams 1, 2, 3, and 4 (can be colors or whatever)

The 4v4 matchup might be:  
PBR's "blue corner": vaporeon, jolteon, flareon, umbreon  
vs  
PBR's "red corner": moltres, zapdos, articuno, and entei.  

It's doubles, so the initial matchup is umbreon and flareon vs moltres and zapdos.

Team 1 controls umbreon and jolteon.  
Team 2 controls flareon and vaporeon.  
Team 3 controls moltres and articuno.  
Team 4 controlls zapdos and entei.  

Each team can switch between their two Pokemon, if switching is on.  When any Pokemon faints, the other Pokemon on that team gets sent out, unless fainted.

If team 2 ends up playing both their Pokemon at once (usually because all team 1's Pokemon are fainted), then their second Pokemon is controlled by defiance, rather than by team 1. 

If all Pokemon on the red corner (team 3 + team 4) are fainted, and both teams on the blue corner (team 1, team 2) still have a Pokemon remaining, we can either:
- split winnings between teams 1 and 2
- declare either team 1 or 2 as the winner, based on tiebreak rules

## Issues
Moves like baton pass, u-turn, roar, metronome, etc. could cause team 1 to end up fighting with both their Pokemon, while team 2's Pokemon aren't fighting at all.  Should these moves be banned? Note we can still ensure that team 1's second Pokemon is controlled by defiance, and not by team 2. 

Do we need to remove/replace moves like surf for being OP? 

# Resources

### General

[Permitted doubles targets by move](https://bulbapedia.bulbagarden.net/wiki/Double_Battle#Effects_on_moves)  
[Ax6's targeting proposal](https://gist.github.com/aaaaaa123456789/d9e3a0363b4234e5d76e63da5216e640)

### Imgur albums
[Temporary overlay redesign concepts 2019-03](https://imgur.com/a/d0neSaT)  
[Temporary overlay redesign concepts 2019-01](https://imgur.com/a/MkYUCXV)  
[2018 overlay](https://imgur.com/a/rtzhZWZ)  
[Streamer's & Stadium 2 overlays](https://imgur.com/a/83rNOMn)

### Other links

[Official streamer's overlay reddit thread](http://redd.it/7cmbxw)  
[Overlay leak reddit thread](http://redd.it/7cjmxz)   
[Overlay reddit comments](https://www.reddit.com/r/twitchplaysPokemon/comments/a6tuwu/the_cutest_fusion_ever/ebydsau)  
[Temp overlay feedback from Zadck](https://docs.google.com/document/d/1UO7CrJKuk4KuynJPDRE5_J_iAOnsWA8UEE6BicyVGmE/edit)

### Videos
[Doubles Demo](https://www.youtube.com/watch?v=HJI9-ADqrUA)  
[A TPP Stadium 2 match](https://www.youtube.com/watch?v=-cYMaQbRc0g)
