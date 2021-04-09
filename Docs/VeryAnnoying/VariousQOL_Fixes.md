#
Here are some little thing that can be easily fixed and the won't break any mod compability or game mechanics.
#

# GameLenght Controll From MapVote
### Genaral Information
Currently you can not change game lenght (Short, Mid, Long) from map vote, only from web admin / KillingFloor.ini. Because of that you can't have a server with both 7 and 10 wave votings. This complicates things too much and is very illogical.

### Proposed Solution
`KFMod/KFGameType.uc#658`
```unrealscript
event InitGame( string Options, out string Error )
{
  ...
  // try to get the value from commandline
  // if we fail just use the config value
  KFGameLength = GetIntOption(Options, "GameLength", KFGameLength);
  // fallback, if user defines wrong number
  if(KFGameLength < 0 || KFGameLength > 3)
  {
    log("GameLength must be in [0..3]: 0-short, 1-medium, 2-long, 3-custom", class.name);
    // force long game on worng number
    KFGameLength = GL_Long;
  }
  ...
}
```
#

# KFPawn `SoundGroup` related Log Spam
### Genaral Information
You will get lots of 

`Warning: Failed to load 'Class XGame.xJuggMaleSoundGroup': Failed to find object 'Class XGame.xJuggMaleSoundGroup'`

`Error: KFHumanPawn KF-Hospitalhorrors-LE.KFHumanPawn (Function XGame.xPawn.PlayTakeHit:00A5) Accessed null class context 'SoundGroupClass'`

lines in logs. Happens on every game.

### Proposed Solution
`KFMod/KFPawn.uc#190`
```unrealscript
simulated function PostBeginPlay()
{
  Super(UnrealPawn).PostBeginPlay();
  // double check if SoundGroupClass is not none
  if(SoundGroupClass == none)
    SoundGroupClass = Class'KFMod.KFMaleSoundGroup';
  // all other code
}
```
`KFMod/KFPawn.uc#3985`
```unrealscript
function Sound GetSound(xPawnSoundGroup.ESoundType soundType)
{
  ...
  // yet another check in a function that actually uses sound group
  if(SoundGroupClass == none)
    SoundGroupClass = Class'KFMod.KFMaleSoundGroup';
  ...
}
```
#

# Weapon Pickup Log Spam
### Genaral Information
**All** weapon pickups that are thrown by a player (or else has `bDropped=true`) spam to log while being destroyed, because `KFMod/KFWeaponPickup.uc#412: Destroyed()` lacks checks if we have inventory or no.

`Warning: MK23Pickup KF-WinLondon.MK23Pickup (Function KFMod.KFWeaponPickup.Destroyed:0019) Accessed None 'Inventory'`

### Proposed Solution
`KFMod/KFWeaponPickup.uc#412: Destroyed()` add a check for `Inventory != none`.
```unrealscript
function Destroyed()
{
  if ( bDropped && Inventory != none && KFGameType(Level.Game) != none )
    KFGameType(Level.Game).WeaponDestroyed(class<Weapon>(Inventory.Class));

  super.Destroyed();
}
```
#

# Weapon Pickups Weird Ammo Calculation
### Genaral Information
1. You drop a drop a gun (WeaponPickup) while not being dead - other players that will pick it up get exact the same ammo that you had.
2. You die and gun drops - it's ammo count will be reseted to default values D:
Its very frustrating when you pick up your own gun after recent wipe and find out that all you ammo is gone and dosh wasted. Or even worse when you pick a gun during kite to find out it has only 1-2 mags.

### Proposed Solution
We need to edit `GiveAmmo(int m, WeaponPickup WP, bool bJustSpawned)` function for `KFMod/KFWeapon.u#1557` and `KFMod/HuskGun.u#158`. It uses `bThrown` flag to check if it was thrown by player or not. But that flag is being set from `DropFrom(vector StartLocation)` and its `True` only if pawn has more than 0 health.. If we simply ignore that flag nothing will change (if we don't count this bugfix).

```unrealscript
function GiveAmmo(int m, WeaponPickup WP, bool bJustSpawned)
{
    ...
    InitialAmount = FireMode[m].AmmoClass.Default.InitialAmount;
    
    if(WP!=none) // remove WP.bThrown==true check
        InitialAmount = WP.AmmoAmount[m];
    ...
}
```
#

# Penetrating Pistols Log Spam
### Genaral Information
You will get huge log spam while playing with mk / deagle / magnum / their dual variants.

`Warning: GoldenDeagleFire KF-WinHospital.GoldenDeagle.GoldenDeagleFire0 (Function KFMod.DeagleFire.DoTrace:04BB) Accessed None 'IgnoreActors'`

Happens because some zeds (`IgnoreActors`) get killed right after they get traced, but before we set their collisions. In result we got `none` errors.

### Proposed Solution
You need to add `IgnoreActors != none` check for all these classes `KFMod/DeagleFire.uc`, `Dual44MagnumFire.uc`, `DualDeagleFire.uc`, `DualMK23Fire.uc`, `Magnum44Fire.uc`, `MK23Fire.uc` inside `function DoTrace(Vector Start, Rotator Dir)`.

```unrealscript
function DoTrace(Vector Start, Rotator Dir)
{
  ...
  // these lines are end the very end
  // Turn the collision back on for any actors we turned it off
  if ( IgnoreActors.Length > 0 )
  {
    for (i=0; i<IgnoreActors.Length; i++)
    {
      if(IgnoreActors[i] == none)
        continue;
      IgnoreActors[i].SetCollision(true);
    }
  }
}
```
#

# Inadequate Match End
### Genaral Information
If a lone player joins to empty server as a spectator / usual player and then leave, game thinks that team is wiped and triggers voting -> map switch.

### Proposed Solution
Add an additional check to your `KFMod/KFGameType` -> `#4736: CheckEndGame(...)` so lobby state will be excluded.
```unrealscript
function bool CheckEndGame(PlayerReplicationInfo Winner, string Reason)
{
  ...
  if(Level.Game.IsInState('PendingMatch'))
    return false;
  ...
}
```
#

# Missing 'NotifyGameEvent'
### Genaral Information
Some mods and old custom maps refer to this function inside `KFMod/KFGameType.uc`, but you removed it completely 2-3 patches ago so it can lead to crashes (if you are unlucky on those maps) or get a log spam.

`Warning: Missing Function Function KFMod.KFGameType.NotifyGameEvent`

### Proposed Solution
Just add a stub function, so at least you won't crash.
`KFMod/KFGameType.uc`
```unrealscript.
function NotifyGameEvent(int EventNumIn);
```
#
