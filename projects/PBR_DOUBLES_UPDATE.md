# Preface 

We plan to set up a demo for players before deciding various things, but to do so we'll first need to add some degree of overlay support for doubles targeting.

Most of this document is info-providing, but a few portions are more heavily opinion-based, and should just be considered as points for discussion.




# Features

*Might be deployed in phases, e.g. defiance doubles after anniversary run, then doubles with targeting some time later, and battle royale some time after that.*

## Complete but needs lots of testing

1. Support (excluding overlay changes) for 6v6 double battles with targeting
1. Detection of HP, PP, moves, ability, etc. for all Pokemon*
1. Detection of all softlocks
1. Ability to retry broken matches, both automatically and manually
1. Faster PBR match setup (45s first start, 30s every match after) which also helps stability
1. Toggleable announcer during matches
1. Toggleable ability to avoid trying the same move / switch more than once (toggle on for defiance at least)
1. Ability to make input selections atomic with respect to player inputs
1. Modifiable input selection timer (in seconds)
1. Modifiable match timer (a legitimate PBR feature, match ends after x minutes)
1. Per-mode token match cooldowns

## Pending

1. Overlay support for 4v4+
1. Overlay support for doubles with targeting
1. Overlay improvements for readability, particularly on mobile
1. Modifiable Pokemon types and base stats**
1. Battle royale matches (pbr double battles with chat split into four teams)
1. Mysterious surprise feature (won't affect gameplay)

*[caveat, see here](##live-data-caveats)

**accomplished by changing the type and base stats of an entire species for the match. So if Ivysaur's data got changed, it would no longer be possible to have a regular Ivysaur in that match.

# Overlay changes

See some mockups and past designs under [Resources](#resources)

According to the moveset team, doubles is generally faster than singles, and 4v4 doubles is roughly analogous to 3v3 singles.  Therefore we should aim for an overlay design that comfortably supports 4v4 doubles with targeting.

Matches greater than 4v4 generally take too long for TPP, especially in singles. That said, it'd be nice to support 6v6, as we might have some game formats (tournaments?) where a 6v6 as a finale could be appropriate.


## "New Overlay"

The "New Overlay" is being designed by Red, Lightning, and M4. This design will hopefully be released to the public in the next 0-3 weeks. After the overlay design is approved, it will need to be implemented in code, which will take some time- probably months.

## Current overlay improvements

While the new overlay is being worked on for a potentially long time, we plan to alter the current overlay to add support for 4v4 with doubles targeting.  The goal is to implement a decent looking targeting and 4v4 system within 4-6 weeks, possibly borrowing some inspiration from the new overlay design.

Some things that could be improved with the current overlay:

- Show live pkmn data (HP, PP, etc)
- Remove stats or make them easier to read (could display on 2 rows, 3 stats per row, instead of the current 6 in a single row)
- Use a more legible font
- Easier to read pkmn names (could be white instead of blue / red)
- Easier to read ability / items (Could display on 2 rows instead of 1)
- Easier to read moves (could stop coloring moves according to the move percentage. Could also try all-caps names, which yields a bit more vertical space)
- Easier to read PP (not sure how to achieve, maybe change to purple for <5 PP, or add more space between the PP bars). Darken moves when they run out of PP.
- Wider sidebars, so there's more horizontal space for bet info. Color background black, like in the stadium 2 days?

## Live data caveats

Internally, PBR updates team data all at once rather than in a play-by-play fashion. This update happens immediately after PBR finishes accepting move / switch selections.

Example 1: A Pokemon gets damaged 3 times in 1 turn (eg, attack + hail + psn). The internally stored HP doesn't experience 3 separate decreases. Instead, it decreases instantly (even before animations play) to the resulting HP after hail and psn. As far as I can tell, PBR doesn't even store the intermediate HP values anywhere in memory.

Example 2: Ditto transforms and then gets roared away in the same turn. The internal teams data never gets updated with Ditto's new moves, stats, etc, because Ditto doesn't have those at the end of the turn.

## Shrinking the PBR popup guis

The HP bars + move select popups as a whole can be shifted left/right/up/down or shrunk. 

Shrinking these popups a little lets us widen the sidebars.  This gives more space for vital info, but widen them too much and PBR might feel squished.

## Removing the PBR popup guis

One idea is to entirely remove the popups and replace them with our own custom ones. This is hopefully possible, but may require quite a lot of time to achieve on the PBR end of things.

We could then display moves, pp, etc. in big font for all the active Pokemon, and design better targeting / switch popups than what PBR currently has (they are really kind of awful imo).  I don't think this really has any downsides at all.

## Adding a permanent active pkmn gui

We could probably also shift the "Team Blue's x used y" text to the bottom of the screen, which then frees up the top third of the screen for a custom GUI that doesn't popup but rather stays in place the whole match, and displays info of the active pkmn (2 in singles, up to 4 in doubles).  With so much space available, we could make the active pkmn info much easier to read than if it was constrained to the sidebar space.

This applies to both singles and doubles.

### Omitting active pokemon in the sidebars

We could then omit the active pkmn displays in the sidebars, which frees up more space for player bets and othe sidebar info.

Unfortunately, this also means that the sidebar info would constantly be shifting every time a switch is made.  The sidebars would also likely show outdated info for anyone with lag, or for everyone in speed mode when pokemon are switched out very quickly.  
In singles, the sidebar might be showing pokemon 1 and 3, with 2 being in the active gui.  
In doubles, the sidebar might be showing pokemon 2 and 4, with 3 and 1 being in the active gui.




# Match retrying & cancelling

1. Best way to deal with softlock from 0pp + more than one of the same move?
1. Should matches that break after starting be automatically retried?
1. Should `!cancelmatch bluewins` be a thing?




# Targeting notation

## Background

[See doubles screens here](https://imgur.com/a/MkYUCXV)

Move selection goes: 
banette -> caterpie -> persian -> entei  
aka  
upper blue -> lower blue -> upper red -> lower red (going by the HP bar positions)  
aka  
blue's left mon -> blue's right mon -> red's left mon -> red's right mon (going by   trainer POV)

You first select your move, and then the target for that move.

[See here for special targeting cases](https://bulbapedia.bulbagarden.net/wiki/Double_Battle#Effects_on_moves)

## General goals

- easy to explain and learn
- easy to make inputs (including on mobile)
- easy to understand inputs made by others
- both newbie and expert friendly

## Proposals

[Ax6's targeting proposal](https://gist.github.com/aaaaaa123456789/d9e3a0363b4234e5d76e63da5216e640)




# Input selection process (moves, switch, targeting)

## Atomic inputs

As mentioned in [Features](#features), we can make input selections atomic with respect to player inputs.   
This prevents the red team from sitting on a 0pp move to react to blue's input. It also prevents blue team from sitting on a 0pp move, and watch red's input before making their selection.

If we make selections atomic, we would also prevent selection of the same move twice (as it'd be pointless to do so).

Another benefit is less delay when someone forgets to change move / tries to stall / leaves the stream / etc, particularly in Fragile mode.

This is more punishing for players, especially with pokesets with less than max PP, and since PP is very hard to read on the overlay.

## Ally targeting

Some have suggested disallowing ally targeting, but ally targeting does have many strategic uses, such as:

**Moves**: sketch, transform, pain split, swagger, flatter, psych up, role play, power/guard/heart swap, acupressure, trick, switcheroo, embargo, mimic, sketch, worry seed, skill swap  
**Abilities**: water/volt absorb, motor drive, steadfast, anger point, flash fire, marvel scale, guts, quick/tangled feet, dry skin, poison heal, blaze/torrent/overgrow/swarm, own tempo

Ally targeting in defiance is another question. Should it never happen? Happen 10-20% of the time? Should we have two doubles defiance modes, one with ally targeting and the other without?


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

Each team can switch between their two pokemon, if switching is on.  When any pokemon faints, the other pokemon on that team gets sent out, unless fainted.

If team 2 ends up playing both their pokemon at once (usually because all team 1's pokemon are fainted), then their second pokemon is controlled by defiance, rather than by team 1. 

If all pokemon on the red corner (team 3 + team 4) are fainted, and both teams on the blue corner (team 1, team 2) still have a Pokemon remaining, we can either:
- split winnings between teams 1 and 2
- declare either team 1 or 2 as the winner, based on tiebreak rules

## Issues
Moves like baton pass, u-turn, roar, metronome, etc. could cause team 1 to end up fighting with both their pokemon, while team 2's pokemon aren't fighting at all.  Should these moves be banned? Note we can still ensure that team 1's second Pokemon is controlled by defiance, and not by team 2. 

Do we need to remove/replace moves like surf for being OP? 

# Resources

### General

[Permitted doubles targets by move](https://bulbapedia.bulbagarden.net/wiki/Double_Battle#Effects_on_moves)  
[Ax6's targeting proposal](https://gist.github.com/aaaaaa123456789/d9e3a0363b4234e5d76e63da5216e640)

### Imgur albums

[Ugly doubles & 4v4 overlay examples](https://imgur.com/a/MkYUCXV)  
[Current overlay screenshots](https://imgur.com/a/rtzhZWZ)  
[Streamer's & Stadium 2 overlays](https://imgur.com/a/83rNOMn)

### Reddit links

[Official streamer's overlay thread](http://redd.it/7cmbxw)  
[Overlay leak thread](http://redd.it/7cjmxz)   
[Random overlay comments](https://www.reddit.com/r/twitchplayspokemon/comments/a6tuwu/the_cutest_fusion_ever/ebydsau)

### Videos
[A TPP Stadium 2 match](https://www.youtube.com/watch?v=-cYMaQbRc0g)
