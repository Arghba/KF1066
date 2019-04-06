Here are some little thing that can be easily fixed and the won't break any mod compability or game mechanics.

# Buzzsaw Projectiles
### Genaral Information
Very often saws go out of map / areas where players can reach it, or players itself doesn't pick up them and refill in trader. This leads to tremandous sound spam. You can't destroy them in any way, and have to deal with sound spam for whole game.

### Proposed Solution
1. You can just add despawn timer for `CrossbuzzsawBlade`'s. Crossbow arrows despawn and no one dies because of that fact..
2. If you don't want them to despawn (concerns about low ammo pool, etc) then simply add this code for level cleanup:

`KFMod/KFGameType.uc` -> `State MatchInProgress` -> `#2308: CloseShops()`:
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

# Grenades Log Spam
### Genaral Information
If you throw more than a single nade it will lead to log spam.

`Error: Nade KF-Corner3WaysFix.Nade (Function KFMod.Nade.Explode:0023) Accessed array 'ExplodeSounds' out of bounds (0/0)`

### Proposed Solution
1. `KFMod/Nade.uc#79` add a check if we have a corrupted array.
```unrealscript
simulated function Explode(vector HitLocation, vector HitNormal)
{
  ...
  // null reference fix
  if ( ExplodeSounds.length > 0 )
    PlaySound(ExplodeSounds[rand(ExplodeSounds.length)],,2.0);
  ...
 }
```
2. `KFMod/Nade.uc#380` convert `sound` to `SoundGroup`.
```unrealscript
defaultproperties
{
  ...
  ExplodeSounds(0)=SoundGroup'KF_GrenadeSnd.Nade_Explode_1'
  ExplodeSounds(1)=SoundGroup'KF_GrenadeSnd.Nade_Explode_2'
  ExplodeSounds(2)=SoundGroup'KF_GrenadeSnd.Nade_Explode_3'
}
```

# Pickups Log Spam
### Genaral Information
**All** pickups that are thrown by a player (or else has `bDropped=true`) spam to log while being destroyed, because `KFMod/KFWeaponPickup.uc#412: Destroyed()` lacks checks if we have inventory or no.

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

# Missing 'NotifyGameEvent'
### Genaral Information
Some mods and old custom maps refer to this function inside `KFMod/KFGameType.uc`, but removed it completely 2-3 patches ago so it can lead to crashes (if you are unlucky on those maps) or get a log spam.

`Warning: Missing Function Function KFMod.KFGameType.NotifyGameEvent`

### Proposed Solution
Just add a stub function, so at least you won't crash.
`KFMod/KFGameType.uc`
```unrealscript.
function NotifyGameEvent(int EventNumIn);
```
