# Genaral Information
1. Buzzsaw Projectiles that are sticked to objects are not replicated properly.
2. Multiple Buzzsaw Projectiles generate enormous ammount of sound spam.
3. Buzzsaw Projectiles tend to fly to areas where players can't reach them, and they stay there forever.

### Exploits reasons

1. In general - saws are not being destroyed properly for clients +

`KFMod/CrossBuzzsawBlade.uc#357`
```unrealscript
simulated function Stick(actor HitActor, vector HitLocation)
{
  // doesn't forcet netupdate on sticking
  // in result blades are not replicated
}
```
2. No time limit for shutting up `AmbientSound`.
3. There is no `LifeSpan` for projectiles or level .
4. There is no level cleanup for Buzzsaw Projectiles inside `KFMod/KFGameType.uc#2250 CloseShops()`.

### Proposed Solution
2. Add a timer to shut up AmbientSound.

`KFMod/CrossBuzzsawBlade.uc`
```unralscript
// add our shutup float
var float ShutMeUpTime;

#70
simulated function PostBeginPlay()
{
  // add our float and start to count it
  ShutMeUpTime += Level.TimeSeconds;
  ...
}

#103
simulated function Tick( float DeltaTime )
{
  ...
  // shut me up when i reach the time limit!
  if(AmbientSound != None && ShutMeUpTime > Level.TimeSeconds)
    AmbientSound = None; // make sure I'll shutup
}
...

```
#

3. Crossbow arrows despawn and no one dies because of that fact. You can just add destroy timer inside `CrossbuzzsawBlade.uc#408 defaultproperties{}` - `LifeSpan=80`. Or 60, 70, whatever you find more suitable.
#

4. If you don't want them to despawn (concerns about low ammo pool, etc) then simply add this code for level cleanup:

`KFMod/KFGameType.uc` -> `State MatchInProgress` -> `#2250: CloseShops()`:
```unrealscript
local CrossbuzzsawBlade CrossbuzzsawBlade;

foreach DynamicActors(class'CrossbuzzsawBlade', CrossbuzzsawBlade)
{
  if(CrossbuzzsawBlade == none)
    continue;
  if(CrossbuzzsawBlade.ImpactActor != none)
    CrossbuzzsawBlade.Destroy();
}
```
#
