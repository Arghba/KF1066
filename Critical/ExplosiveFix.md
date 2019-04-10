# Genaral Information
1. PipeBombs can be detonated with high rate of fire weapons to get much more damage than it should in usual circumstaces.
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
