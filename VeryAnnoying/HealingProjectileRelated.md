## Genaral Information
1. Healing moving players with med guns is pure HELL ON EARTH.
2. And even if you hit moving player most likely `ROBulletWhipAttachment` (the weapon) will block healing.
3. Med gun darts shake nearby players.

### Bugs Reasons
1. This is not that visible on low pings, but after 50-60 (not talking about >120-160) it will become ultra hard to hit moving players. And client will often see how their darts connect to target players (hit effects), but in reality server will decide to use `HealingProjectile`'s Location, will check Target's location, see that they differ (ofc coz of the ping), and say NOPE to your heals. Repeat 5-6-10-15 times till server allows your darts to do their job..
2. Ok, so you were be able to connect your darts. Aaaaand it hits to `ROBulletWhipAttachment`, NOPE please retry.
3. This is almost as annoying as the first case. You just stand near another player, medic decides to heal him -> your screen, aim starts to shake, nice! Target players will play teleport effect (i.e. the shake) anyways, so why you force nearby players to shake to D:

### Proposed Solution
`KFMod/HealingProjectile.uc`

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
#
