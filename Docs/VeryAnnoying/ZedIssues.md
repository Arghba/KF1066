# Genaral Information
1. Siren damages players after she is decapped (has - health).
2. Husk shoots fireball after he is dead.
3. Crawler has collision issues.
4. Zed to zed damage spamms to log, a lot.
5. Fleshpound makes all zeds to spin like crazy.
6. Crawler spams in log when huks shoots his fireball near him / you get swarmed by their groups.
7. Husk spams in log while shooting and being killed imidiately after animation starts.

# Exploits reasons
1. `KFChar/ZombieSiren.uc#112 SpawnTwoShots()`
```unrealscript
simulated function SpawnTwoShots()
{
    // no checks if we are decapped or no
    if( bZapped )
    {
        return;
    }
    ...
}
```
2. If `Husk` started his fireball launching animation nothing can interupt him. Even if you wipe him to meat and blood, fireball will be spawned anyways and does his job.
3. Crawler jumps very fast and his pawn is not being destroyed that quick to make the transition smooth.
4. `KFMod/KFMonster.uc#2631 TakeDamage()`
```unrealscript
function TakeDamage(int Damage, Pawn instigatedBy, Vector hitlocation, Vector momentum, class<DamageType> damageType, optional int HitIndex )
{
    ...
    // 2nd check will spam log with none warnings if damage was dealth by a zed (no player pawn)
    if ( damageType != none && LastDamagedBy.IsPlayerPawn() && LastDamagedBy.Controller != none )
    {
        if ( KFMonsterController(Controller) != none )
        {
            KFMonsterController(Controller).AddKillAssistant(LastDamagedBy.Controller, FMin(Health, Damage));
        }
    }
    ...
}
```
5. `KFMod/FleshPoundAvoidArea.uc#41`
```unrealscript
function bool RelevantTo(Pawn P)
{
    // no zed health / specie check
    return ( KFMonst != none && VSizeSquared(KFMonst.Velocity) >= 75 && Super.RelevantTo(P)
    && KFMonst.Velocity dot (P.Location - KFMonst.Location) > 0  );
}
```
6. Animation package doesn't have Sequence for some animations.
7. During `SpawnTwoShots()` Husks doesnt check if `Controller` exists or no. For `ToggleAuxCollision()` inside `KFMonster` - no checks if we even have additional collision or no (destroyed, gone, etc).

## #1: Siren Aftershock
This happens like 100 times per game. Decapped / dead sirens still deal damage to you. Not the greatest issue especially when they are alone, but when when you get 2-3 sirens from a corner and you know that if you kill them fast right now you will be killed too from their after death screams. And you have to fall back to make distance AND THEN shoot. Or if a player is unaware of this 'feature' he will shoot and die / get wounded to 10-20hp at max.
[Video demonstration](STUB!)

## #2: Husk's Magic
Ok, this can be funny. But let's not forget that their fireballs move ALL zeds and make them jump if it explodes. A husk comes behind a far corner / building, starts to charge his weapon of doom, you kill it and.... fireball still flies to you, meanwhile moving all zeds and messing your aim.

[Video demonstration]()

## #3: Crawler bodyblocking
Crawler pawn take some time to get destroyed, and since you will hug him much more frequantly than other zeds you will notice this issue more than on other zeds (like charging gorefasts). While they die during jumps their corpse will block your movement like real, non dead zed does. It leads to situations when you need to rush through groups of crawlers, you move ahead while oneshoting them (lets say from musket, for example), and every jumping crawler will stop your movement. This is very annoying and happens on all games you join.

## #4: Zed2Zed Damage Log Spam
You open your server logs and find out that

`Warning: ZombieClot_STANDARD KF-Solo-Bioticslab.ZombieClot_STANDARD (Function KFMod.KFMonster.TakeDamage:02FC) Accessed None 'LastDamagedBy'`

Lines take the good half of the file. It makes navigating and administrating much harder in this mess and ofc this massive logging decreases preformance. I'm not providing any demonstration since you can see this in every game, server log.

## #5: Fleshpound Spinning Madness
When you rage a FP it during it's animation all zeds that touch him will be spinned several times. This makes headshots much much harder to achieve, and for dual, triple FP's almost suicidal mission.
And there are another serious issues with this.
1. If raged FP touches a zed and it doesnt instagib, it will become unable to hit the players (friendly).
2. If you kill a FP before it rage animation ends, it will launch all zeds that touch his `FleshPoundAvoidArea` to random directions, sometimes behind you. Very funny to get raged sc at your juicy ass.

[Video demonstration #1](https://youtu.be/kLd-TJzyzBE)

[Video demonstration #2](https://youtu.be/DoqhCZAykvA)

## #6: Crawler Log Spam
Just tons of these lines in logs, during every game.

`Log: PlayAnim: Sequence 'Jump' not found for mesh 'Crawler_Freak'`

## #7: Husk Log Spam
Lots of lines while kiling husks during their fireball launch animation.

` Warning: ZombieHusk_STANDARD KF-Westlondon.ZombieHusk_STANDARD (Function KFChar.ZombieHusk.SpawnTwoShots:0135) Accessed None 'Controller'

Warning: ZombieHusk_STANDARD KF-Westlondon.ZombieHusk_STANDARD (Function KFMod.KFMonster.ToggleAuxCollision:0037) Accessed None 'MyExtCollision'
`

# Proposed Fixes
1. Add `bDecapitated` check to deflect damage. 

`KFChar/ZombieSiren.uc#112 SpawnTwoShots()`
```unrealscript
simulated function SpawnTwoShots()
{
    if( bZapped || bDecapitated )
    {
        return;
    }
    ...
}
```
And edit `ZombieDying` state so it won't let sirens to do any animation / damage, etc (well since she's dead...).
```unrealscript
State ZombieDying
{
ignores AnimEnd, Trigger, Bump, HitWall, HeadVolumeChange, PhysicsVolumeChange, Falling, BreathTimer, Died, RangedAttack, SpawnTwoShots;
}
```
#

2. Edit `ZombieDying` state so it won't let husks to do any animatin, damage, etc (well since she's dead...).
`KFChar/ZombieHusk`
```unrealscript
State ZombieDying
{
ignores AnimEnd, Trigger, Bump, HitWall, HeadVolumeChange, PhysicsVolumeChange, Falling, BreathTimer, Died, RangedAttack, SpawnTwoShots;
}
```
#

3. Edit crawler's  `PlayDying()` and `ZombieDying` state, so they will disable pawn collision on death.

`KFChar/ZombieCrawler`
```unrealscript
simulated function PlayDying(class<DamageType> DamageType, vector HitLoc)
{
    Super.PlayDying(DamageType, HitLoc);
    DisablePawnCollision();
}

State ZombieDying
{
ignores AnimEnd, Trigger, Bump, HitWall, HeadVolumeChange, PhysicsVolumeChange, Falling, BreathTimer, Died, RangedAttack;

    simulated function BeginState()
    {
        DisablePawnCollision();
        Super.BeginState();
    }
}

// we add this to disable our collision
// or we can add this to KFMonster and use this
// function for other problematic zeds like gorefasts, FP's
// laters can even launch zeds with their after-death collision
final function DisablePawnCollision()
{
	bBlockActors = false;
	bBlockPlayers = false;
	bBlockProjectiles = false;
	bProjTarget = false;
	bBlockZeroExtentTraces = false;
	bBlockNonZeroExtentTraces = false;
	bBlockHitPointTraces = false;
}
```
#

4. Add `LastDamagedBy != none` check.

`KFMod/KFMonster.uc#2631 TakeDamage()`
```unrealscript
function TakeDamage(int Damage, Pawn instigatedBy, Vector hitlocation, Vector momentum, class<DamageType> damageType, optional int HitIndex )
{
    ...
    if ( damageType != none && LastDamagedBy != none && LastDamagedBy.IsPlayerPawn() && LastDamagedBy.Controller != none )
    {
        if ( KFMonsterController(Controller) != none )
        {
            KFMonsterController(Controller).AddKillAssistant(LastDamagedBy.Controller, FMin(Health, Damage));
        }
    }
    ...
}
```
#

5. For less spinning madness let's add zed health check.

`KFMod/FleshPoundAvoidArea.uc#41`
```unrealscript
function bool RelevantTo(Pawn P)
{
    if(KFMonst != none && KFMonst.Health >= 1500) //or other suitable value to exclude buffy zeds
        return false;
    return ( KFMonst != none && VSizeSquared(KFMonst.Velocity) >= 75 && Super.RelevantTo(P)
    && KFMonst.Velocity dot (P.Location - KFMonst.Location) > 0  );
}
```
#

6. Fix the Sequence inside KF_Freaks_Trip.ukx / KF_Freaks2_Trip.ukx (don't remember which one xD) or just add a dud one, to prevent the spam.
#

7. Firstly for `KFMod/KFMonster.uc#776`
```unrealscript
// Setters for extra collision cylinders
simulated function ToggleAuxCollision(bool newbCollision)
{
    // lets check if we even have an additional collision
    // if no just skip this completely
    if(MyExtCollision == none)
        return;
    ...
}
```
And for `KFChar/ZombieHusk.uc#155` add `!= none` checks
```unrealscript
function SpawnTwoShots()
{
	...
	if(Controller != none)
		FireRotation = Controller.AdjustAim(SavedFireProperties,FireStart,600);

	foreach DynamicActors(class'KFMonsterController', KFMonstControl)
	{
        if(Controller == none || KFMonstControl != Controller)
        {
            if( PointDistToLine(KFMonstControl.Pawn.Location, vector(FireRotation), FireStart) < 75 )
            {
                KFMonstControl.GetOutOfTheWayOfShot(vector(FireRotation),FireStart);
            }
        }
	}
	....
}
```
#
