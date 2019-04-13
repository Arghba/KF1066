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


---

---

---

# General Information


1. Pipe bombs can be detonated with high rate of fire weapons to get much more damage than it should in usual circumstances.
2. PipeBombs can be detonated if you throw grenades, do the shoot-go to spectator trick. Even on other person's pipes.
3. PipeBombs can be detonated if NPC walks, you suicide near it.
4. PipeBombs can not be detonated if placed on some spots - some meshes on the floor, some stairs, etc.
5. PipeBombs spam in logs during detonation.

# Exploits reasons
1. 2.`KFMod/PipeBombProjecile.uc#485`
```unrealscript
function TakeDamage( int Damage, Pawn InstigatedBy, Vector Hitlocation, Vector Momentum, class<DamageType> damageType, optional int HitIndex)
{
	...
	// no check if we are triggered or no
	// no checks if damage is dealth by players or zeds
	// ^ and what damage to deflect
	...
}
```

3. `KFMod/PipeBombProjecile.uc#111`
```unrealscript
function Timer()
{
    ...
    foreach VisibleCollidingActors( class 'Pawn', CheckPawn, DetectionRadius, Location )
    {
        ...
        // no checks on NPC and pawn health
        ...
    }
    ...
}
```

5. You've set default settings in a weird way.

`KFMod/PipeBombProjecile.uc#46`
```unrealscript
{
static function PreloadAssets()
{
	default.ExplodeSounds[0] = sound(DynamicLoadObject(default.ExplodeSoundRefs[0], class'Sound', true));
	...
}
```
`KFMod/PipeBombProjecile.uc#46`
```unrealscript
static function bool UnloadAssets()
{
	default.ExplodeSounds[0] = none;
	...
}
```

## #1: Nuka Pipes
Due to `PipeBombProjecile`'s `TakeDamage(...)` you can detonate them multiple times with high firerate weapons (like medguns, fal, etc and flamethrower, shotguns), and it will deal tremendous amount of damage - from 8k to 17k, depends how succesfull was your aim.
Aaaand besides that when multiple pipe bombs explode or you detonate one with methods described above - you will get huge log spam about out of bound sound array.

[Video demonstration](https://youtu.be/agHeuTY3Afg)

## #2: Pipe Bomb Detonation by Teammates
Almost similar to teamkilling projectile (shoot - spectate), but happens due to `PipeBombProjecile`'s `TakeDamage(...)`. If do the same trick to your teammates pipes instead of his pawn, it will detonate and make the fly awayyy.

## #3: Pipes react to corpses and NPC's
If you suicide on any pipe or lure KFO NPC's - it will detonate...

## #4: Pipes refuse to detonate
If placed on certain surfaces with complex blocking volumes, inside meshes on the floor, etc. Actors nearby just can't be traced and pipebomb stays as is without detonation.

## #5: Pipes Spam to Log
You will get loots of this lines while playing with pipes.

`Error: PipeBombProjectile KF-Westlondon.PipeBombProjectile (Function KFMod.PipeBombProjectile.Explode:005D) Accessed array 'ExplodeSounds' out of bounds (0/0)`

# Proposed solutions
1. 2.Additional checks for better damage detection and to allow it trigger only once.
```unrealscript
function TakeDamage( int Damage, Pawn InstigatedBy, Vector Hitlocation, Vector Momentum, class<DamageType> damageType, optional int HitIndex)
{
    if ( bTriggered || Damage < 5 )
        return;

    if ( Monster(InstigatedBy) == none && class<KFWeaponDamageType>(damageType) != none && class<KFWeaponDamageType>(damageType).default.bDealBurningDamage )
        return; // make pipebombs immune to fire, unless instigated by monsters

    // original pipebomb TakeDamage code
    ...
}
```

3. 4.`KFMod/PipeBombProjecile.uc#111`
```unrealscript
function Timer()
{
    ...
    local vector DetectLocation;
    local bool bSameTeam; //pawn is from the same team as instigator

    DetectLocation = Location;
    DetectLocation.Z += 25; // raise a detection poin half a meter up to prevent small objects on the ground bloking the trace

    ...
    foreach VisibleCollidingActors( class 'Pawn', CheckPawn, DetectionRadius, DetectLocation )
    {
        // don't trigger pipes on NPC  -- PooSH
        bSameTeam = KF_StoryNPC(CheckPawn) != none || (CheckPawn.PlayerReplicationInfo != none && CheckPawn.PlayerReplicationInfo.Team.TeamIndex == PlacedTeam);
        if( CheckPawn == Instigator || (bSameTeam && KFGameType(Level.Game).FriendlyFireScale > 0) )
        {
            // Make the thing beep if someone on our team is within the detection radius
            // This gives them a chance to get out of the way
            ThreatLevel += 0.001;
        }
        else
        {
            if( CheckPawn.Health > 0 //don't trigger pipes by dead bodies  -- PooSH
                            && CheckPawn != Instigator && CheckPawn.Role == ROLE_Authority
                            && !bSameTeam )
            {
                if( KFMonster(CheckPawn) != none )
                {
                    ThreatLevel += KFMonster(CheckPawn).MotionDetectorThreat;
                    if( ThreatLevel >= ThreatThreshhold )
                    {
                        bEnemyDetected=true;
                        SetTimer(0.15,True);
                    }
                }
                else
                {
                    bEnemyDetected=true;
                    SetTimer(0.15,True);
                }
            }
        }
    }
    ....
    else
    {
        bAlwaysRelevant=true;
        Countdown--;

        if( CountDown > 0 )
        {
            PlaySound(BeepSound,SLOT_Misc,2.0,,150.0);
        }
        else
        {
            Explode(DetectLocation, vector(Rotation)); // we use DetectLocation insdeat of original Location, to apply the fix
        }
    }
    ...
}
```

5. Set defaults inside `defaultproperties`.
```unrealscript
defaultproperties
{
	ExplodeSounds(0)=SoundGroup'Inf_Weapons.antitankmine.antitankmine_explode01'
}
```
