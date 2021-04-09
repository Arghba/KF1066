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
#

# Medic Grenade Relevance issue
### Genaral Information
If player **A** throws medic grenade behind player **B** -> later won't see green smoke (emitter) if he turns around. This is related to all grenade types, it's just most visible on healing ones, since players "ignore" your medic grenades that you threw behind them and that causes many gameplay issues.

### Proposed Solution
`KFMod/Nade.uc#161`
```unrealscript
defaultproperties
{
  // we need to add this
  bAlwaysRelevant=True
  ...
}
```
#
