# Genaral Information
1. Buzzsaw Projectiles that are sticked to objects are not replicated properly.
2. Multiple Buzzsaw Projectiles generate enormous ammount of sound spam.
3. Buzzsaw Projectiles tend to fly to areas where players can't reach them, and they stay there forever.

### Exploits reasons

1. `KFMod/CrossBuzzsawBlade.uc#357`
```unrealscript
simulated function Stick(actor HitActor, vector HitLocation)
{
  // doesn't forcet netupdate on sticking
  // in result blades are not replicated
}
```
2. No time limit for shutting up `AmbientSound`.
3. There is no `LifeSpan` for projectiles or level cleanup code for `KFMod/KFGameType.uc#2250 CloseShops()`.

### Proposed Solution
1. Crossbow arrows despawn and no one dies because of that fact. You can just add destroy timer for `CrossbuzzsawBlade`'s - `LifeSpan=80` or anything you find suitable.
2. If you don't want them to despawn (concerns about low ammo pool, etc) then simply add this code for level cleanup:

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
