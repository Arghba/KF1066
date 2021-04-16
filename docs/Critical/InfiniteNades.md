# General information

Hand Grenades doesn't check for ammo count before doing fire effect. Meaning:

1. You can bug yourself into "infinite grenade mode".
2. Spam grenades with several weapons and in result throw more grenades than you carry.
3. Crash servers with 3rd party cheaty apps.

## Detailed exploits description

## Exploits reasons

`KFMod/FragFire.uc#123`

```clike
function DoFireEffect()
{
  // no current ammo checks
}
```

## #1: Infinite Grenades
Easily can be done in every game. You can see how it's done in demonstration.

[Video demonstration](https://youtu.be/4-lobeyDn4g)

## #2: Throwing more nades than you carry
Go sharpshooter, equip LAR, spam grenade bind with some easy to find timing -> you can throw 6-10 nades depending how good you kept the timing.

[Video demonstration](https://youtu.be/7Un8IUtV8mU)

## #3: Crashing Servers..Again
Yup, some magic apps let you do this in every game.

[Video demonstration](https://youtu.be/icWPlrSpDKQ)

# Proposed solution

All three cases can be easily fixed. And we can just call fast grenade tossing with LAR a feature, since with this fix you won't be able to toss more than 5 (your max ammount). 

`KFMod/FragFire.uc#123`

```clike
var float PrevAmmo;  // new variable

function DoFireEffect()
{
  local float MaxAmmo,CurAmmo;
    
  Weapon.GetAmmoCount(MaxAmmo,CurAmmo);
  // do not let tossing if we run out of "ammo"
  if (CurAmmo==0 && PrevAmmo==0)
    return;
  PrevAmmo=CurAmmo;

  // original code starts from here
  ...
}
```
