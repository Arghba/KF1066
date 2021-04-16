# Genaral Information

1. Buzzsaw Projectiles that are sticked to objects are not replicated properly.
2. Multiple Buzzsaw Projectiles generate enormous ammount of sound spam.
3. Buzzsaw Projectiles tend to fly to areas where players can't reach them, and they stay there forever.

## Exploits reasons

1. In general - saws are not being destroyed properly for clients +

`KFMod/CrossBuzzsawBlade.uc#357`

```clike
simulated function Stick(actor HitActor, vector HitLocation)
{
  // doesn't forcet netupdate on sticking
  // in result blades are not replicated
}
```

2. No time limit for shutting up `AmbientSound`.
3. There is no `LifeSpan` for projectiles.
4. There is no level cleanup for Buzzsaw Projectiles inside `KFMod/KFGameType.uc#2250 CloseShops()`.

### Proposed Solution

1. The most complicated part xD thanks to [Poosh's Scrn](https://github.com/poosh/KF-ScrnBalance/blob/master/Classes/ScrnCrossbuzzsawBlade.uc) tho, we have a working solution.

`KFMod/CrossBuzzsawBlade.uc`

```clike
#133
simulated function PostNetReceive()
{
  // add a role check
  if(Role < ROLE_Authority)
  {
    // destroy this nonsense imidiately
    if (bHidden)
      Destroy();
    else if (ImpactActor!=None && Base != ImpactActor)
      GoToState('OnWall');
  }
}

#357
simulated function Stick(actor HitActor, vector HitLocation)
{
  ...
  // force replication after we get sticked
  NetUpdateTime = Level.TimeSeconds - 1;
}

// new function!
// destroy an actor and make sure it will be destroyed on clients too
simulated function ReplicatedDestroy()
{
  if (Level.NetMode == NM_Client || Level.NetMode == NM_StandAlone)
  {
    Destroy();
  }
  else
  {
    // set everything to null
    bHidden = true;
    SetCollision(false, false);
    SetPhysics(PHYS_None);
    Velocity = vect(0,0,0);
    Speed = 0;
    // force it to replicate
    NetUpdateTime = Level.TimeSeconds - 1;
    // now destroy
    SetTimer(1.0, false);
   }
}

function Timer()
{
  Destroy();
}
```

Now start to add our new destroy function to `OnWall` state. And force replication there aswell.

```clike
simulated state OnWall
{
  ...
  function ProcessTouch (Actor Other, vector HitLocation)
  {
    // replace Destroy() with ReplicatedDestroy();
  }

  simulated function Tick( float Delta )
  {
    // new function!
    if (Base == None)
      ReplicatedDestroy();
  }

  simulated function BeginState()
  {
    ...
    // just add this
    NetUpdateFrequency = 2.0;
  }
}
```

2. Add a timer to shut up AmbientSound.

`KFMod/CrossBuzzsawBlade.uc`

```clike
// add our shutup float
var float ShutMeUpTime;

#70
simulated function PostBeginPlay()
{
  // add our float and give it a value
  ShutMeUpTime += Level.TimeSeconds;
  ...
}

#103
simulated function Tick( float DeltaTime )
{
  ...
  // shut me up when i reach the time limit!
  if (AmbientSound != None && ShutMeUpTime > Level.TimeSeconds)
    AmbientSound = None; // make sure I'll shutup
}

defaultproperties
{
  ...
  ShutMeUpTime=10.0 //seconds sounds reasonable
}
```

3. Crossbow arrows despawn and no one dies because of that fact. You can just add destroy timer inside `CrossbuzzsawBlade.uc#408 defaultproperties{}` - `LifeSpan=80`. Or 60, 70, whatever you find more suitable.

4. If you don't want them to despawn (concerns about low ammo pool, etc) then simply add this code for level cleanup:

`KFMod/KFGameType.uc` -> `State MatchInProgress` -> `#2250: CloseShops()`:

```clike
local CrossbuzzsawBlade CrossbuzzsawBlade;

foreach DynamicActors(class'CrossbuzzsawBlade', CrossbuzzsawBlade)
{
  if (CrossbuzzsawBlade == none)
    continue;
  if (CrossbuzzsawBlade.ImpactActor != none)
    CrossbuzzsawBlade.Destroy();
}
```
