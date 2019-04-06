# Genaral Information
1. Almost all non-hitscan KF weapons can be used to teamkill players.
2. PipeBombs can be detonated from other players.
3. PipeBombs can be detonated with high rate of fire weapons to get much more damage than it should in usual circumstaces.
4. PipeBombs spam in logs during detonation.

# Exploits reasons
1. `KFMod/KFPawn.uc#2250`
```unrealscript
function TakeDamage(int Damage, Pawn instigatedBy, Vector hitlocation, Vector momentum, class<DamageType> damageType, optional int HitIdx )
{
	...
	// no checks if InstigatedBy is none
	...
}
```
2. 3.`KFMod/PipeBombProjecile.uc#485`
```unrealscript
function TakeDamage( int Damage, Pawn InstigatedBy, Vector Hitlocation, Vector Momentum, class<DamageType> damageType, optional int HitIndex)
{
	...
	// no checks if damage is dealth by players or zeds
	// no check if we are triggered or no
	// ^ and what damage to deflect
	...
}
```
4. You've set default settings in a weird way.

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

## #1: Teamkills
Because we don't have some important checks inside `TakeDamage(...)` when some one shoots harpoon, orca granade, m99, xbow, flamethrower, etc and go to spectators quickly, projectile hits `Pawns` and *DEALS* damage like it was made by an enemy. It's being used to troll players a lot. Harpoon kills are easies to pull due to it's slow projectiles and delayed detonation.

[Video demonstration #1](STUB!)

## #2: Pipe Bomb Detonation
Almost similar to the last case, but happens due to `PipeBombProjecile`'s `TakeDamage(...)`. If do the same trick to your teammates pipes instead of his pawn, it will detonate and make the fly awayyy.

[Video demonstration #1](STUB!)

## #3: Nuka Pipes
Due to `PipeBombProjecile`'s `TakeDamage(...)` you can detonate them multiple times with high firerate weapons (like medguns, fal, etc and flamethrower, shotguns), and it will deal tremendous amount of damage - from 8k to 17k, depends how succesfull was your aim.
Aaaand besides that when multiple pipe bombs explode or you detonate one with methods described above - you will get huge log spam about out of bound sound array.

[Video demonstration](https://youtu.be/agHeuTY3Afg)


# Proposed solution
1. Since you are going to edit `KFPawn` to fix dosh exploits, you can simply:
```unrealscript
function TakeDamage(int Damage, Pawn instigatedBy, Vector hitlocation, Vector momentum, class<DamageType> damageType, optional int HitIdx )
{
	// local vars
	// before any calculation and even before super.TakeDamage()

	// just in case
   	if ( Damage <= 0 )
	   return; 

	// copy-pasted from KFHumanPawn to check for nones
	if( Controller!=None && Controller.bGodMode )
		return;

	KFDamType = class<KFWeaponDamageType>(damageType);
	if ( InstigatedBy == none )
	{
		// Player received non-zombie KF damage from unknown source.
		// Let's assume that it is friendly damage, e.g. from just
		disconnected/crashed/cheating teammate
		// and ignore it.
		if ( KFDamType != none && class<DamTypeZombieAttack (KFDamType) == none )
			return;
    }
	else
	{
		if ( KFMonster(InstigatedBy) != none )
		{
			KFMonster(InstigatedBy).bDamagedAPlayer = true;
		}
		else if( KFHumanPawn(InstigatedBy) != none )
		{
			InstigatorPRI = KFPlayerReplicationInfo(InstigatedBy.PlayerReplicationInfo);
			// Don't allow momentum from a player shooting a player
			Momentum = vect(0,0,0);
			// no damage from spectators (i.e. fire missile -> spectate -> missile hits player
			if ( InstigatorPRI != none && InstigatorPRI.bOnlySpectator )
				return;
		}
	
	// here starts your original kfpawn code
	...
}
```

2. 3.Additional checks for better damage detection and to allow it trigger only once.
```unrealscript
function TakeDamage( int Damage, Pawn InstigatedBy, Vector Hitlocation, Vector Momentum, class<DamageType> damageType, optional int HitIndex)
{
    if ( bTriggered || Damage < 5 )
        return;

    if ( Monster(InstigatedBy) == none && class<KFWeaponDamageType>(damageType) != none && class<KFWeaponDamageType>(damageType).default.bDealBurningDamage )
        return; // make pipebombs immune to fire, unless instigated by monsters

    super.TakeDamage(Damage, InstigatedBy, Hitlocation, Momentum, damageType, HitIndex);
}
```
4. Set defaults inside `defaultproperties`.
```unrealscript
defaultproperties
{
	ExplodeSounds(0)=SoundGroup'Inf_Weapons.antitankmine.antitankmine_explode01'
}
```
