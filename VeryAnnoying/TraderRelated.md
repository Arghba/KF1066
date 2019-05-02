# Genaral Information
On most custom maps and even on some official ones you get huge amount of log spam.

`Warning: ShopVolume KF-Manor.ShopVolume3 (Function KFMod.ShopVolume.Touch:0088) Accessed None 'MyTrader'`

`Warning: ShopVolume KF-Manor.ShopVolume3 (Function KFMod.ShopVolume.UnTouch:0051) Accessed None 'MyTrader'`

`Warning: ShopVolume KF-Manor.ShopVolume3 (Function KFMod.ShopVolume.Touch:0088) Accessed None 'MyTrader'`

`Warning: ShopVolume KF-Manor.ShopVolume3 (Function KFMod.ShopVolume.UsedBy:0099) Accessed None 'MyTrader'`

And sometimes you will get `Accessed None 'Other'` if someone leaves game / suicides during trading.

### Bug reasons
Many authors don't use `WeaponLocker` aka Trader models (nigga or 3d printer) for their traders. And `ShopVolume`s keep calling for non existing `WeaponLocker` and spam in chat.

### Proposed Solution
We need to add checks if we even have `WeaponLocker`s nearby and if there are none just skip their calls - trader sounds, reactions etc.

`KFMod/ShopVolume.uc`
```unralscript
#18

```
#
