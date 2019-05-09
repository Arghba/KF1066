It's a collection of pipe-related exploits and bugs. Some of them are critical, some are not. We decided to not split them up to keep everything pipe-related in one file.

# #1 Pipe bombs? More like nuke bombs!

## General Information

Shooting pipes with certain weapons causes them to explode several times potentially dealing up to 18k damage. This leads to a pipe abuse, where demos use Schneidzekk and a single pipe to obliterate everything. Or to a classic 2GUYS1PAT.MOV situation where 2 suicide bombers with a pipe and a hunting shotgun can destroy 6p hp Patriarch.

[Video demonstration](https://youtu.be/agHeuTY3Afg)

## Reasons

`KFMod/PipeBombProjecile.uc#485 (function TakeDamage)` does not have a check if the pipe was already triggered. So, when a player shoots the pipe with a weapon which has a very fast rate of fire, it triggers pipe's `TakeDamage` method several times causing pipe to explode on every call.

## Proposed solution

Put this at the very top of `TakeDamage` method:

```unrealscript
if ( bTriggered )
    return;
```

# #2 Detonating teammates' pipes with weapons

TODO: investigate

# #3 Detonating teammates' pipes with your own corpse

## General information

Going to spectator mode while standing near someone's pipes will trigger them.

## Reasons

`KFMod/PipeBombProjecile.uc#134 (function Timer)`. When scanning all nearby pawns with `foreach VisibleCollidingActors`, it sees dead player's pawn as an enemy, because this pawn does not have `PlayerReplicationInfo`.

## Proposed solution

Add an additional check to if-statement at `KFMod/PipeBombProjecile.uc#146`:

```unrealscript
CheckPawn.Health > 0
```

# #4 KFO NPCs always trigger planted pipes

## General information

[Ringmaster Lockheart](http://kf-wiki.com/wiki/Ringmaster_Lockheart) will destroy all the pipes he'll come across.

## Reasons

Yet again, no checks inside `KFMod/PipeBombProjecile.uc#134 (function Timer)` -> `foreach VisibleCollidingActors`.

## Proposed solution

```unrealscript
function Timer()
{
    local bool bSameTeam;

    ...

    foreach VisibleCollidingActors( ... )
    {
        bSameTeam = KF_StoryNPC(CheckPawn) != none || (CheckPawn.PlayerReplicationInfo != none && CheckPawn.PlayerReplicationInfo.Team.TeamIndex == PlacedTeam);

        ...
    }

    ...
}
```

And use this `bSameTeam` in `#138` and `#147` instead of just comparing `.TeamIndex` and `PlacedTeam`.

# #5 Not triggering pipe bombs

## General information

If the pipe is placed at certain spots (small meshes on the floor, some stairs), it will never be triggered by zeds.

## Reasons

The pipe's core is sunk inside some mesh and ray tracing done by `foreach VisibleCollidingActors` which is called from `KFMod/PipeBombProjecile.uc#134 (function Timer)` won't work.

## Proposed solution

Do ray tracing from half a meter higher than the pipe bomb's actual location.

```unrealscript
function Timer()
{
    local vector DetectLocation;

    DetectLocation = Location;
    DetectLocation.Z += 25;

    ...

    foreach VisibleCollidingActors( class 'Pawn', CheckPawn, DetectionRadius, DetectLocation )
    {
        ...
    }

    ...
}
```

# #6 Pipe log spam

## General information

Every pipe leaves the following log message on explode:

```
PipeBombProjectile KF-ThrillsChills.PipeBombProjectile (Function KFMod.PipeBombProjectile.Explode:005D) Accessed array 'ExplodeSounds' out of bounds (0/0)
```

## Reasons

`ExplodeSounds[0]` is not defined.

## Proposed solution

```unrealscript
defaultproperties
{
    ExplodeSounds(0)=SoundGroup'Inf_Weapons.antitankmine.antitankmine_explode01'
    ...
}
```
