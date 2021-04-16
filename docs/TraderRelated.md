# Genaral Information

On most custom maps and even on some official ones you get huge amount of log spam.

```clike
Warning: ShopVolume KF-Manor.ShopVolume3 (Function KFMod.ShopVolume.Touch:0088) Accessed None 'MyTrader'
Warning: ShopVolume KF-Manor.ShopVolume3 (Function KFMod.ShopVolume.UnTouch:0051) Accessed None 'MyTrader'
Warning: ShopVolume KF-Manor.ShopVolume3 (Function KFMod.ShopVolume.Touch:0088) Accessed None 'MyTrader'
Warning: ShopVolume KF-Manor.ShopVolume3 (Function KFMod.ShopVolume.UsedBy:0099) Accessed None 'MyTrader'
```

And sometimes you will get `Accessed None 'Other'` if someone leaves game / suicides during trading.

## Bug reasons

Many authors don't use `WeaponLocker` aka Trader models (nigga or 3d printer) for their traders. And `ShopVolume`s keep calling for non existing `WeaponLocker` and spam in chat.

### Proposed Solution

We need to add checks if we even have `WeaponLocker`s nearby and if there are none just skip their calls - trader sounds, reactions etc.

`KFMod/ShopVolume.uc`

```clike
#18
function Touch( Actor Other )
{
  // to prevent accessed none warnings
  if (Other == none)
    return;
  ....
  // add a check if we even have a weaponlocker or no
  if (MyTrader != none)
    MyTrader.SetOpen(true);
  ...
}

#56
function UnTouch( Actor Other )
{
  // to prevent accessed none warnings
  if (Other == none)
    return;
  // add a check if we even have a weaponlocker or no
  if (MyTrader != none && Pawn(Other)!=None && PlayerController(Pawn(Other).Controller)!=None && KFGameType(Level.Game)!=None)
    MyTrader.SetOpen(false);
}

#61
function UsedBy(Pawn user)
{
  local string traderTag;

  // to prevent accessed none warnings
  if (user == none || KFHumanPawn(user) == none)
    return;
  // Set the pawn to an idle anim so he wont keep making footsteps
  User.SetAnimAction(User.IdleWeaponAnim);

  if (KFPlayerController(user.Controller)!=None && KFGameType(Level.Game)!=None && !KFGameType(Level.Game).bWaveInProgress)
  {
    // add a check if we even have a weaponlocker or no
    if (MyTrader != none)
      traderTag = string(MyTrader.Tag);
    else
      traderTag = "WeaponLocker"; // since we have none set it to something

    KFPlayerController(user.Controller).ShowBuyMenu(traderTag,KFHumanPawn(user).MaxCarryWeight);
  }
}
```
