# General information

There is 

1. Ability to bypass collision checks of almost all actors in the level, allowing players to reach the unreachable places.
2. Instantly kill *any* zeds.
3. Lag clients, crash servers.

# Detailed exploits description

## Exploits reasons

`KFMod/KFPawn.uc#2964`

```unrealscript
exec function TossCash( int Amount )
{

    ...

    if( Amount<=0 )
        Amount = 50;
    Controller.PlayerReplicationInfo.Score = int(Controller.PlayerReplicationInfo.Score); // To fix issue with throwing 0 pounds.
    if( Controller.PlayerReplicationInfo.Score<=0 || Amount<=0 )
        return;
    Amount = Min(Amount,int(Controller.PlayerReplicationInfo.Score));

    ...

}
```

1. Not capping the minimum amount of dosh per toss. Which results in the ability to spawn as many `CashPickup` actors as your total dosh amount (by setting pickups value to £1.)
2. Since there's no timeout for `TossCash` execution, all the money can be tossed out very fast.

## #1: Collision bypass

Having tons of `CashPickup` actors concentrated in one spot is very calculation-heavy for the engine. Once the certain amount of actors is reached, the engine starts skipping calculation of collision between some of them. (It's what we came up with.)

Players can use this to bypass intended map-designers' blocks and reach the unreachable spots by going straight through static meshes, various blocking volumes, etc.

However, this doesn't work with:
- BSP Geometry
- Meshes with built-in blocking collision

[Video demonstration #1](https://youtu.be/4-lobeyDn4g) ++

[Video demonstration #2](https://youtu.be/fbs7SBHWzlM)

# Proposed solution

Simply add a very small timeout after tossing.

After testing this with friends, we came up with `0.1f`. It prevents any dosh-related problems with a small exception (about this later) and it's not very restrictive so you still can please your inner Michelangelo:

![A truly majestic piece of art](https://i.imgur.com/ITaG6xL.jpg)

Proposed fix:

`KFMod/KFPawn.uc`

```unrealscript
var float AllowedTossCashTime;

...

exec function TossCash( int Amount )
{
  ...

  if( Level.TimeSeconds < AllowedTossCashTime )
    return;

  ...

  AllowedTossCashTime = Level.TimeSeconds + 0.1f;
}
```

