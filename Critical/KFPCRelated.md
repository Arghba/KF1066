# General information
1. You can buy any variant of every weapon (camo, gold, neon, etc) at the same time with vanilla variant - you can have Neon, Gold, usual AK's, or golden AA12 + usual AA12, or Camo m32 + usual m32, etc. Many of this 'loadouts' have separate ammo pools, meaning you can double your firepower and achieve monstreous DPS / ammo count.

# Detailed exploits description

## Exploits reasons

`KFMod/FragFire.uc#123`

```unrealscript
function DoFireEffect()
{
    // no current ammo checks
}
```

## #1: Infinite Grenades
Easily can be done in every game. You can see how it's done in demonstration.

[Video demonstration](https://www.youtube.com/watch?v=pIejlEql4Tc&t=90s)

## #2: Throwing more nades than you carry
Go sharpshooter, equip LAR, spam grenade bind with some easy to find timing -> you can throw 6-10 nades depending how good you kept the timing.

[Video demonstration](https://youtu.be/7Un8IUtV8mU)

## #3: Crashing Servers..Again
Yup, some magic apps let you do this in every game.

[Video demonstration](https://youtu.be/icWPlrSpDKQ)

# Proposed solution

All three cases can be easily fixed. And we can just call fast grenade tossing with LAR a feature, since with this fix you won't be able to toss more than 5 (your max ammount). 

`KFMod/FragFire.uc#123`

```unrealscript
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

