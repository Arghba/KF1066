## Genaral Information
1. Healing moving players with med guns is pure HELL ON EARTH.
2. And even if you hit moving player most likely `ROBulletWhipAttachment` (the weapon) will block healing.
3. Med gun darts shake nearby players.
4. There is a redunant data in default values.

### Bugs Reasons
1. This is not that visible on low pings, but after 50-60 (not talking about >120-160) it will become ultra hard to hit moving players. And client will often see how their darts connect to target players (hit effects), but in reality server will decide to use `HealingProjectile`'s Location, will check Target's location, see that they differ (ofc coz of the ping), and say NOPE to your heals. Repeat 5-6-10-15 times till server allows your darts to do their job..
2. Ok, so you were be able to connect your darts. Aaaaand it hits to `ROBulletWhipAttachment`, NOPE please retry.
3. This is almost as annoying as the first case. You just stand near another player, medic decides to heal him -> your screen, aim starts to shake, nice! Target players will play teleport effect (i.e. the shake) anyways, so why you force nearby players to shake to D:
4. Why we even need `Damage` and `DamageRadius` for **Healing** darts.

### Proposed Solution
`KFMod/HealingProjectile.uc`
1. ==============
```unrealscript
#70
simulated function PostNetReceive()
{
    if( bHidden && !bHitHealTarget )
    {
        // use only HealLocation
        if( HealLocation != vect(0,0,0) )
        {
            // remove the log part too, why we need it?
            HitHealTarget(HealLocation,vector(HealRotation));
        }
        // HealLocation doesn't received from server, so don't call HitHealTarget
        // Actually PostNetReceive() shouldn't be called without HealLocation, but who knows
        // the devil that is living inside KF code? :)
        // (c) PooSH
    }
}

#135
simulated function Timer()
{
    if ( Role == ROLE_Authority)
        super.Timer();
    // client side stuff
    // Didn't receive HealingLocation from the Server - Destoy
    else if ( !bHitHealTarget )
        Destroy();
}
```
#

1.2. ==============
```unrealscript
#241
simulated function ProcessTouch(Actor Other, Vector HitLocation)
{
    ...
    //local vars
    ...
    // before any role check
    
    // KFBulletWhipAttachment is attached to KFPawns
    // so heal him!!
    if ( ROBulletWhipAttachment(Other) != none ) {
        Healed = KFHumanPawn(Other.Owner);
        if ( Healed == none || Healed.Health >= Healed.HealthMax )
            return;
    }
    else
        Healed = KFHumanPawn(Other);

    if( Healed != none )
    {
        if( Role == ROLE_Authority )
        {
            // original code
            ...
        }
        else
        {
            // client side
            bHidden = true;
            SetPhysics(PHYS_None);
            SetTimer(2.0, false); //give server some time to tell us we're healed somebody, or destroy
            return;
        }
    }
    Explode(HitLocation,-vector(Rotation));
}
```
#

3. ==============
```unrealscript
// why need to shake nearby players? make this a stub function
function ShakeView()
{
}
```
#

4. ==============
```unrealscript
defaultproperties
{
    // set to 0
    Damage=0
    DamageRadius=0
    ...
}
```
#
