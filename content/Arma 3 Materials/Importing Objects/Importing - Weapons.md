---
title: How to set up a weapon for Arma 3
draft: false
tags:
  - import
  - weapons
  - object
  - configcpp
---
## Additional functionality compared to A2/OA
---
- Slotable weapon accessories
- Custom reload animations
- Adjustable sights
- Underwater weapons
- Ammo changes on fly and on hit
- Rotating muzzle-flash
- Explosion shielding

---
## Model requirements {p3d}

- proxies for slotable accessories
    - muzzle accessory should be on proxy **\A3\data_f\proxies\weapon_slots\MUZZLE**
    - optics should be on proxy **\A3\data_f\proxies\weapon_slots\TOP**
    - side accessory should be on proxy **\A3\data_f\proxies\weapon_slots\SIDE**
    - bipod accessory should be on proxy **\A3\data_f_mark\proxies\weapon_slots\UNDERBARREL**
    - all these proxies could be redefined in _cfgWeapons >> Weapon >> WeaponSlotsInfo >> XXX >> linkProxy_ parameter where XXX is the slot name

- selections for folding iron sights
    - You need to create selections and axes for iron sights if You want them folded once the optics is put on the weapon
    - Front part should be named **ForeSight** with **ForeSight_axis** in memory lod
    - Rear part should be named **BackSight** with **BackSight_axis** in memory lod

- adjustable sights for grenade launchers
    - there needs to be a selection that is going to rotate (in case of collimator sights), default naming is **OP**
    - this selection needs to have an axis in memory lod, default naming is **OP_axis**
    - there needs to be a focus point, the best place is the red dot of collimator, with memory point **OP_look** by default and several points for eye, usually **OP_eyeX** where X is the number of the point. Good practice is to place them in same distance form focus point

---
## Model config changes {model.cfg}

- custom reload animation have a good use of newly added parameter **unHideValue** for hide type of animations - you are now able to make asymmetrical animations eg. for hiding of magazine:

**{model.cfg}**
```cpp
class magazine_hide
{
	type = "hide";
	source = "reloadMagazine";
	selection = "magazine";
	minValue = 0.000000;
	maxValue = 1.00000;
	hideValue = 0.220;
	unhideValue = 0.550;
};
```

- the animations could look a bit better by simply adding a translation for the magazine and adding an axis for that in model - magazine should translate at first, then disappear, appear and translate back

**{model.cfg}**
```cpp
class magazine_reload_move_1
{
	type = "translation";
	source = "reloadMagazine";
	selection = "magazine";
	axis = "magazine_axis";
	minValue = 0.145;
	maxValue = 0.170;
	offset0 = 0.0;
	offset1 = 0.5;
};
```

- foldable iron sights use **hasOptics** controller

**{model.cfg}**
```cpp
class BackSight_optic
{
	type = "rotation";
	source = "hasOptics";
	selection = "BackSight";
	axis = "BackSight_axis";
	memory = 1;
	minValue = 0.0000000;
	maxValue = 1.0000000;
	angle0 = 0.000000;
	angle1 = (rad 90);
};
```

- new animation controllers **zeroing1** and **zeroing2** take values from _discreteDistance[]_ of first and second muzzle of the weapon. The value is index number of current zeroing in the array starting with zero (that means the first value is 0, second is 1, the last is number of discrete distances plus one). It might be used for iron sights of the weapon if desired but better use is for UGL collimator sights rotation:

**{model.cfg}**
```cpp
class OP_ROT
{
	type="rotation";
	source = "zeroing2";		// use second muzzle zeroing for rotation
	sourceAddress = "loop";		// loop when phase out of bounds
	selection = "OP";			// selection we want to rotate
	axis = "OP_axis";			// has its own axis
	minValue = 0;
	maxValue = 3;				// this weapon has array with 4 distances
	angle0 = "rad 0";
	angle1 = "rad 65";
};
```

- rotating muzzle flashes are done using a new animation source **ammoRandom** which changes it is value every time weapon is fired. Various degrees of rotation may be set up by using correct muzzle flash shape and minValue maxValue combination.

**{model.cfg}**
```cpp
class MuzzleFlashROT
{
	type = "rotationX";
	source = "ammoRandom";		// use ammo count as phase for animation
	sourceAddress = "loop";		// loop when phase out of bounds
	selection = "zasleh";		// selection we want to rotate
	axis = "";					// no own axis - center of rotation is computed from selection
	centerFirstVertex = true;	// use first vertex of selection as center of rotation
	minValue = 0;
	maxValue = 4;				// rotation angle will be 360/4 = 90 degrees
	angle0 = "rad 0";
	angle1 = "rad 360";
};
```

---
## New config parameters {config.cpp}
### Slotable weapons

- Available slots are defined in each weapon but are usually inherited from a parent weapon. They are stored as classes in **class WeaponSlotsInfo** which contains the slots and parameters for inventory
    - **Mass** is a new unit used to describe weight and volume of an object used. Each container has a set capacity in the same units.
    - allowedSlots[] is an array of slot numbers where you may put the weapon. 701 stands for vest, 801 stands for uniform, 901 stands for backpack
    - each weapon slot is a separate subclass in class WeaponSlotsInfo
        - parameter **linkProxy** defines a proxy in weapon model for said slot (see standard names on top)
        - parameter **displayName** describes a mouse-over name of slot in Inventory
        - array **compatibleItems[]** lists possible accessory placeable into that slot. Most weapons are able to have any RIS equipment, but eg. muzzle accessory differs according to caliber.
    - external classes _CowsSlot_ and _PointerSlot_ are used for standard optics and side accessory. That means these classes are outside cfgWeapons and changeable for all weapons at once.
```cpp
class SlotInfo;
class CowsSlot : SlotInfo
{
	// targetProxy
	linkProxy = "\A3\data_f\proxies\weapon_slots\TOP";

	// display name
	displayName = "$STR_A3_CowsSlot0";

	// class names with items supported by weapon 
	compatibleItems[] = { "optic_Arco", "optic_aco", "optic_ACO_grn", "optic_hamr", "optic_Holosight" };
};
class PointerSlot : SlotInfo
{
	// targetProxy
	linkProxy = "\A3\data_f\proxies\weapon_slots\SIDE";

	// display name
	displayName = "$STR_A3_PointerSlot0";

	// class names with items supported by weapon 
	compatibleItems[] = { "acc_flashlight", "acc_pointer_IR" };
};

class CfgWeapons
{
	class myWeapon
	{
		class WeaponSlotsInfo
		{
			mass = 4; // default mass of a weapon
			class MuzzleSlot : SlotInfo
			{
				// targetProxy
				linkProxy = "\A3\data_f\proxies\weapon_slots\MUZZLE";

				// display name
				displayName = "Muzzle Slot";

				// class names with items supported by weapon
				compatibleItems[] = {}; // moved to each weapon
			};
			class CowsSlot : CowsSlot {};
			class PointerSlot : PointerSlot {};
			allowedSlots[] = { 901 }; // you simply cannot put this into your pants
		};
	};
};
```

### Muzzle accessories

- suppressors are configured as a weapon inheriting some item abilities from class **ItemCore**
- the class itself consists only from scope, displayName, picture and model, there is a separate subclass **ItemInfo** with all the required parameters
    - there is subclass **MagazineCoef** inside class ItemInfo with parameter **initSpeed** - this is just a multiplier of initSpeed of weapon's magazine
    - subclass **AmmoCoef** of class ItemInfo has more parameters for the ammo shoot through the suppressor:
        - **hit** is the coefficient of hit of original ammo
        - **visibleFire**, **audibleFire**, **visibleFireTime** and **audibleFireTime** are coefficients for detection upon shooting the weapon
        - higher **cost** coefficient should make AI think more about shooting the suppressed weapon
        - **typicalSpeed** and **airFriction** coefficients change the ballistic characteristics of the ammo
    - there are alternate **muzzleEnd** and **alternativeFire** directly inside class ItemInfo to have different muzzle effects origin and muzzle flashes