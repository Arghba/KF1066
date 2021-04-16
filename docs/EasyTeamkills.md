# General information

Shooting certain weapons and instantly going to spectator mode deals 100% damage to players. It is used a lot by griefers to kill others and ruin their games.

A lot of weapons can by used for that, but harpoon is the most viable option, since is has delayed detonation and teamkilling can be easily pulled off without using a macro. If you have a properly set up macro, HSG Shotgun is the best choice.

TODO: video demonstration

## Exploit reason

`KFMod/KFPawn.uc#2250`

```clike
function TakeDamage(int Damage, Pawn instigatedBy, Vector hitlocation, Vector momentum, class<DamageType> damageType, optional int HitIdx)
{
  ...
  // InstigatedBy == none if player became spectator, and there are no checks for that.
  // So the game thinks the damage is dealt by an enemy.
  ...
}
```

## Proposed solution

Since you are going to edit `KFPawn` to fix dosh exploits

```clike
function TakeDamage(int Damage, Pawn instigatedBy, Vector hitlocation, Vector momentum, class<DamageType> damageType, optional int HitIdx )
{
  // local vars
  // before any calculation and even before super.TakeDamage()

  // just in case
  if (Damage <= 0)
    return;

  // copy-pasted from KFHumanPawn to check for nones
  if (Controller!=None && Controller.bGodMode)
    return;

  KFDamType = class<KFWeaponDamageType>(damageType);
  if (InstigatedBy == none)
  {
    // Player received non-zombie KF damage from unknown source.
    // Let's assume that it is friendly damage, e.g. from just
    disconnected/crashed/cheating teammate
    // and ignore it.
    if (KFDamType != none && class<DamTypeZombieAttack (KFDamType) == none)
      return;
    }
  else
  {
    if (KFMonster(InstigatedBy) != none)
    {
      KFMonster(InstigatedBy).bDamagedAPlayer = true;
    }
    else if (KFHumanPawn(InstigatedBy) != none)
    {
      InstigatorPRI = KFPlayerReplicationInfo(InstigatedBy.PlayerReplicationInfo);
      // Don't allow momentum from a player shooting a player
      Momentum = vect(0,0,0);
      // no damage from spectators (i.e. fire missile -> spectate -> missile hits player
      if (InstigatorPRI != none && InstigatorPRI.bOnlySpectator)
        return;
    }

  // here starts your original kfpawn code
  ...
}
```
