---
title: Weapon Creation - Asset Preparation
draft: false
tags:
  - modeling
  - modding
  - armareforger
  - import
---
## Materials
[Sample Files](https://github.com/BohemiaInteractive/Arma-Reforger-Samples)

## Prepare the Mesh
---
This tutorial will cover the procedure for integrating a new weapon into Enfusion Workbench. While it may focus more on Blender users, other software users shouldn't worry, as most of the processes shown here can be similarly performed in other software.

### Object Orientation

One of the most important thing to begin with is making sure that your **model is properly orientated**. As per [Arma Reforger:FBX Import](https://community.bistudio.com/wiki/Arma_Reforger:FBX_Import#Alignment "Arma Reforger:FBX Import") page:

> [!info]
> Everything must be oriented as pointing along/towards the Y+ axis in Blender and 3dsMax and along the Z+ axis in Maya.

That means, when you are **importing an A3/external weapon you need to rotate it by 90 degrees to the left.**

### Object Cutting

You may already have some bits present in the mesh. Since Enfusion allows you to assemble a weapon from multiple parts, we will split our mesh into several separate objects to achieve much higher customization of the weapon.

![[600px-armareforger-new-weapon-cutting-parts.gif]]

In this case, the sample contains two parts that could potentially be moved to separate files, allowing you to have three variants in total quite easily. As shown in the video above, the grip and iron sights were moved to separate FBX files. The magazine was already separated for that particular mesh, so the only remaining task is to add slot points for those accessories.

### Object Naming

When it comes to naming, there are a few important rules to keep in mind:

1. **_LODx** suffix is used to indicate **Level of Details**.
2. **UBX_, UCX_, USP_, UCS_, UCL_, UTM_** prefixes are used to mark **Colliders**.
3. **OCC_** prefix is used for **Geometry Occluders**.

Besides that, there are also additional guidelines regarding the naming of objects—**[slot/snap points naming convention](https://community.bistudio.com/wiki/Arma_Reforger:Weapon_Slots_And_Bones "Arma Reforger:Weapon Slots And Bones")**—which do not affect how the mesh is processed by the engine (like those mentioned above) but are there to ensure **consistency**. It's also worth noting that some base weapon prefabs use some of these names **by default**. They can be changed, but it's strongly advised to follow these rules nevertheless.

### Add Slots/Snap Points

If you have experience with Arma 3 modding, the concept of having slots and snap points should be fairly familiar to you. These dummy objects serve as memory points. However, there is one major difference: you no longer need to place two points to create an axis. Instead, the rotation of the dummy object is used.

In Blender, you can use one of the empty objects, such as Plain Axis, to create these helper points.

![[armareforger-new-weapon-empty-create.png]]

Please notice the Plain Axis gizmo - this is the orientation of the model. 

Make sure that your empty object is **properly aligned**.
![[armareforger-new-weapon-empty-rotate.png]]

> [!info]
> One easy method to ensure slot and snap points are correctly aligned is to create the slot points while the mesh is still in one piece. Then, copy and paste the mesh and empty socket to a new empty scene. After that, the only thing left to do is rename **slot_XX** to **snap_XX**.

On **Sample Weapon**, the following slots were created:

- **slot_ironsight_front & slot_ironsight_rear** - slots for picatinny mounted ironsights
- **slot_magazine** - slot for magazine well
- **slot_optics** - slot for top-mounted optics
- **slot_underbarrel** - slot for bottom-mounted accessories like bipods

Additionally, the following empty objects were created for various components:

- **snap_hand_right** - **IK target** for the right hand when using the **weapon deployment** feature
- **snap_hand_left** - **IK target** for the left hand when using the **weapon deployment** feature
- **barrel_chamber & barrel_muzzle** - these points are used in **MuzzleComponent** to determine the location and direction where the bullet is spawned
- **eye** - point for aiming down sight view, used in **SightsComponent**
![[armareforger-new-weapon-slots-overview.png]]

### Colliders & Material Names

Colliders are a special type of object used to calculate various kinds of collisions, whether it be for physics simulation or tracing bullet penetration. There are a few rules regarding these colliders, most of which are listed on the [FBX Import - Colliders usage](https://community.bistudio.com/wiki/Arma_Reforger:FBX_Import#Collider_usage "Arma Reforger:FBX Import") page.

Collider with **Weapon** Layer Preset:
![[armareforger-new-weapon-colliders-geo.png]]
Collider with **FireGeo** Layer Preset:
![[armareforger-new-weapon-colliders-firegeo.png]]

A weapon requires at least one collider with the **Weapon [layer preset](https://community.bistudio.com/wiki/Arma_Reforger:Collision_Layer "Arma Reforger:Collision Layer")**. Without it, weapon actions like equip will be missing. If your geometry is simple enough, you can use one collider for both weapon and fire geometry collisions. Otherwise, you might need two colliders:

- One for **Weapon** collision - this should be a very simple collider (e.g., convex).
- Another for **FireGeo** collision - this can be more detailed; a trimesh can be used to provide the best experience.

When importing assets from previous Arma games, you are likely to already have convex components ready from **Geometry, Fire Geometry, View Geometry**, or **Geometry Physx LODs**. In this case, the **Fire Geometry LOD from Arma 3 was used as FireGeo**, and the **mesh from Geometry LOD** was used for the **Weapon** layer.

You might ask how to assign **Layer Presets**. If you are using Blender, there is a handy tool called [Objects Tool](https://community.bistudio.com/wiki/Arma_Reforger:Enfusion_Blender_Tools:_Objects_Tools "Arma Reforger:Enfusion Blender Tools: Objects Tools"), which is part of [Enfusion Blender Tools](https://community.bistudio.com/wiki/Arma_Reforger:Enfusion_Blender_Tools "Arma Reforger:Enfusion Blender Tools"), to assist you with assigning the correct game materials and layer presets on colliders. Otherwise, refer to the [FBX Import](https://community.bistudio.com/wiki/Arma_Reforger:FBX_Import#Setting_Layer_Preset_on_colliders_.28usage_parameter.29 "Arma Reforger:FBX Import") page for instructions on how to set that parameter in 3DS Max or Maya.

> [!info]
> You can use [Model Quality Assurance](https://community.bistudio.com/wiki/Arma_Reforger:Enfusion_Blender_Tools#Model_Quality_Assurance "Arma Reforger:Enfusion Blender Tools") to verify if your colliders are convex by **checking UCX Collider** option.

### Skeleton Setup & Mesh Rigging
#### Skeleton Creation

Whether it's a new model or a mesh imported from P3D, it will most likely be necessary to prepare a skeleton. In **Blender**, skeletons are called **[Armatures](https://docs.blender.org/manual/en/latest/animation/armatures/index.html)**, and their creation process is quite straightforward. Start with the [creation of the **Armature itself**](https://docs.blender.org/manual/en/latest/animation/armatures/introduction.html#your-first-armature). This can be done by selecting the **Armature** option from the **Add** menu in the top section of the viewport while in **Object** mode.
![[armareforger-new-weapon-adding-armature.gif]]

> [!warning]
>It is recommended to name your armature **Armature** to avoid an artificial bone in the skeleton hierarchy when the mesh is imported into Workbench. While this might not cause issues with rifles, it is essential when importing character-related gear.

Once it is created, you might notice that the root bone is quite large and may want to resize it. To do this, switch to **Edit Mode**, select the **Bone**, and then reduce its size to a reasonable level.

> [!warning]
> Armature should be located at **0,0,0 point and be using 1.0 scale**. Otherwise you might encounter some problems when trying to animate it in Blender at a later stage.

The next step is to rotate this bone towards the **front of the weapon** and then rename it to **w_root**. This bone will serve as the **root bone** of the mesh.
![[armareforger-new-weapon-rotating-naming-bone.gif]]

#### Bones Creation

After the basics of the armature are done, it is time to add bones for all animated parts of the weapon. While in **Edit Mode** and with **Armature** selected, you can add new bones through either of these methods:

- **Add → Single Bone** option in the top bar of the viewport
- By [duplicating a bone](https://docs.blender.org/manual/en/latest/animation/armatures/bones/editing/duplicate.html) already present in the armature, like **w_root**

If you choose the first option, you will need to resize and rotate the new bone to the desired size. Duplication doesn't have this issue, so it was chosen to create the fire mode switch bone - **w_fire_mode**. 

It is also important to maintain the hierarchy of bones. In this case, all bones responsible for moving parts of the weapon, like **w_fire_mode**, should be **parented to w_root**. This can be done by selecting the bone in **Edit Mode** and then changing the Parent property in the Relations section of the Bone properties tab.
![[armareforger-new-weapon-duplicating-bones.gif]]

> [!info]
> Consider setting the bone visibility to **In Front** - this will help with placement of the bones. ![[armareforger-new-weapon-set-bone-in-front.png]]

There are a few things to keep in mind when creating bones. Below is a list of those considerations:

- Bones will serve as axes for any further motion, so try to place them in **spots that would result in reasonable motion**, like the center of the **fire selector pivot point** or the **center of the bolt**.
- **Keep Y+** orientation of the bones. This is especially handy if an action uses the center of the bone for operations.
- While it's not necessary, it is recommended to use the vanilla **[naming convention for bones](https://community.bistudio.com/wiki/Arma_Reforger:Weapon_Slots_And_Bones#Bones "Arma Reforger:Weapon Slots And Bones")**. This is particularly useful when dealing with animation export, as vanilla animation export profiles expect such bone names.
    - _It is still possible to use custom names, but this might require custom export profiles. More about this will be mentioned in the chapter covering weapon animation._

With this knowledge in mind, it should be possible to create the rest of the bones like **w_trigger**, **w_charging_handle**, and **w_bolt**.
![[armareforger-new-weapon-bones-end-result.png]]
#### Mesh Skinning

Skinning of the mesh depends on the software that you are using and if you don't know how to do it in software of your choice, it is recommended **to search for some tutorials on the web what skinning of bones is**.

> [!info]
> If you've **imported your model from P3D** via Enfusion Blender Tools then you will have some vertex groups already - in this case you can try to **rename** those so they **match the skeleton bone names**.

If you are using **Blender**, here is a short instruction on how to quickly set up skinning on an object. In the example below, the **w_bolt** bone will be set:

1. Switch to **Object Mode**.
2. Select the object you want to skin - in this case, it is **body_02_LOD0**.
3. In the **[Modifiers Properties](https://docs.blender.org/manual/en/latest/modeling/modifiers/introduction.html#interface)** tab, add the **[Armature](https://docs.blender.org/manual/en/latest/modeling/modifiers/deform/armature.html)** modifier via the **Add Modifier** button.
4. In the **Object** property of the **Armature** modifier, select the **Armature** (skeleton) that should influence this object - in this case, it is **Armature**.
5. In **Object Data Properties**, expand the [**Vertex Group** panel](https://docs.blender.org/manual/en/latest/modeling/meshes/properties/vertex_groups/vertex_groups.html).
6. Switch to **[Edit Mode](https://docs.blender.org/manual/en/latest/editors/3dview/modes.html)** and then activate **face selection mode**.
7. Select the faces that should belong to the bolt.
8. In the **Vertex Group** section, click on the plus button to **[Add Vertex Group](https://docs.blender.org/manual/en/latest/modeling/meshes/properties/vertex_groups/assigning_vertex_group.html#creating-vertex-groups)**.
9. Double-click with the **Left Mouse Button** on it and change the name of that new vertex group from **Group** to **w_bolt**.
10. Click on the **Assign** button in the **Vertex Groups** section (assuming you still have the bolt faces selected in the viewport) - this will assign your current selection in the viewport to the **w_bolt** vertex group with full influence (influence is controlled by the **[Weight](https://docs.blender.org/manual/en/latest/modeling/meshes/properties/vertex_groups/vertex_weights.html)** property).
![[armareforger-new-weapon-set-skinning.gif]]
At this stage the **w_bolt** bone should be successfully rigged but that is not the end of the process!

> [!warning]
>   Below section applies to any 3D software!

When creating skinned objects, **Enfusion Workbench** expects that the whole object is skinned to a bone.

> [!warning]
>  Otherwise, the importer will try to "fix" it by skinning the remaining faces to some root bone, and you will see the following message in the console log:
>  ```RESOURCES (W): Missing some mesh skinning weights (Object_LOD0). Weighting them to root```

In Blender, this means that any object with an **Armature** modifier must be fully skinned to some existing bone through vertex groups. In this case, it means that all faces, **besides those belonging to** the **w_bolt** vertex group, on the **body_02_LOD0** object must be skinned to the **w_root** bone. This was done by selecting the faces belonging to **w_bolt** and then [inverting the selection using the Ctrl + I shortcut](https://docs.blender.org/manual/en/latest/interface/keymap/industry_compatible.html#selection). After that, a new vertex group called **w_root** was created, and the current selection was assigned to it via the **Assign** button.
![[armareforger-new-weapon-set-skinning-root.gif]]
#### Colliders Rigging

If you have an animated collider, please keep in mind that **only trimesh colliders can be skinned**. In all other cases, you have to use **100% weight**.

In Blender, it gets even more tricky. You need to use **Object Relations** (in the **Object tab**) if you want to connect a non-trimesh collider to a skeleton bone.
![[armareforger-new-weapon-relations.png]]

> [!info]
> It is also recommended to parent slots on the weapon to the **Armature** - it is not necessary to parent it to the w_root bone though. This should make it easier to snap magazine in the reload sequence for instance.

## FBX Export Settings

Most of the general rules can be found on the **[FBX Import page](https://community.bistudio.com/wiki/Arma_Reforger:FBX_Import#FBX_Export "Arma Reforger:FBX Import")**. In principle, when exporting from software like 3DS Max, you need to ensure that you are exporting in **binary format in version 2014/2015**. Furthermore, **Triangulation** and **Preserve Edge Orientation** should be turned off.

For **Blender**, there are three important things to keep in mind when exporting FBX:

![[armareforger-new-weapon-export-blender2.png]]

1. **Object Types**:

For an animated object like a weapon you need to have checked at least:

**Empty** - which handles all the snap points

**Armature** - exports the skeleton of your weapon

**Mesh** - self explanatory

2. **Custom Properties**:

Without this option all the custom properties like **LayerPresets** would be lost!

3. **Leaf bones**:

Leaf bones are completely unnecessary in Enfusion and it's better to have that option **turned off.**

## Textures Preparation

When importing weapon textures from previous Real Virtuality games like Arma 3, there is no straightforward or automated method to convert spec-gloss textures to PBR ([Physically Based Rendering](https://en.wikipedia.org/wiki/Physically_based_rendering)) metal-rough textures, which are the current industry standard. Therefore, in most cases, it's much easier to create textures from scratch using software like Substance Painter.

There are many resources available online on how to create proper PBR textures, and it's highly recommended to search for them using popular search engines. For Substance Painter, it's worth looking at the [Substance Painter PBR guide](https://substance3d.adobe.com/tutorials/courses/the-pbr-guide-part-1).

If you still want to try converting **RV spec-gloss textures to PBR**, you can follow this [Converting A2/A3/Dayz textures to PBR for Reforger](https://www.mod-fusion.com/post/converting-a2-a3-dayz-textures-to-pbr-for-reforger) tutorial.

## Model Import & Registration

Once the mesh is successfully prepared and all selections, sockets, and snap points are in place, it's time to try our asset in the game. The majority of the process is already well described on the [FBX Import - Import process in the Workbench](https://community.bistudio.com/wiki/Arma_Reforger:FBX_Import#Import_process_in_the_Workbench "Arma Reforger:FBX Import") page.

In principle, all you have to do is right-click on your FBX files and select the **Register and Import** option from the context menu.
![[armareforger-new-weapon-mesh-register.gif]]

### Colliders & Materials Check

After the initial import is done, it's time to ensure that materials and colliders are using the correct settings. By default, Enfusion will try to assign materials based on the name of the assigned texture in the mesh. If it fails to find the texture, a new dummy material (see the area marked in orange on the screen below) will be created next to the FBX model.

There are two typical errors when it comes to collider configuration:



![[1024px-armareforger-new-weapon-collider-errors.png]]

> [!warning]
>  - Make sure that you have "usage" property defined in the collider object properties: ![[armareforger-fbx-layers-blender-1.png]]
>  - Make sure that the correct material is assigned to all the colliders:![[armareforger-new-weapon-colliders-material.png]]
>  
>  More info can be found on [Arma Reforger:FBX Import](https://community.bistudio.com/wiki/Arma_Reforger:FBX_Import "Arma Reforger:FBX Import").


### Skeleton & Hierarchy

Next, we will take care of the bones. In this case, the magazine object has only empty objects (snap points), and to import them, checking the **Export Scene Hierarchy** in the Miscellaneous section of the Import Settings tab should be enough.

For **skinned assets like rifles**, the **Export Skinning** option should be used instead. It's also useful to know that **Export Scene Hierarchy** is not necessary when the **Export Skinning** option is selected.

The process of setting **Export Scene Hierarchy** on the magazine is showcased below, followed by the analogous process of using **Export Skinning** on **SampleWeapon_01.xob**.

If, for some reason, you don't see the bones icon on **SampleWeapon_01.xob** even after checking **Export Skinning** and reimporting the resource, make sure that you have properly skinned your model in the 3D software of your choice.

![[armareforger-new-weapon-scene-hierarchy.gif]]
^ Importing **hierarchy** on magazine

![[armareforger-new-weapon-importing-skinning.gif]]
^ Importing **skinning** on weapon

> [!info]
> Import Settings Tab: 
> ![[armareforger-new-weapon-import-settings.png]]

> [!info]
> Changes made in **Import Settings** tab are only applied to the model after **manually reimporting the model** via **Reimport resource (PC)** button.

## Texture Import

In principle, you can use the same procedures as those described on the [Weapon Modding](https://community.bistudio.com/wiki/Arma_Reforger:Weapon_Modding#From_Scratch "Arma Reforger:Weapon Modding") page.

By default, **Workbench** should already **create some of the materials based on their material name in the FBX**. So, in the case of [SampleWeapon_01.xob](enfusion://ResourceManager/~SampleMod_NewWeapon:Assets/Weapons/Rifles/SampleWeapon_01/SampleWeapon_01.xob), there should already be some **emats** that need to be properly configured.

- **SampleWeapon_01_Camo1.emat** material with two textures:
    - **SampleWeapon_01_Camo1_BCR** - Base color + Roughness
    - **SampleWeapon_01_Camo1_NMO** - Normal map
- **SampleWeapon_01_Camo2.emat** also with two textures:
    - **SampleWeapon_01_Camo2_BCR** - Base color + Roughness
    - **SampleWeapon_01_Camo2_NMO** - Normal map
![[armareforger-new-weapon-materials.png]]
Since in this example the magazine shares textures with the rifle itself, it is necessary to adjust the textures accordingly. To do this, double-click on the **[XOB file of the magazine](enfusion://ResourceManager/~SampleMod_NewWeapon:Assets/Weapons/Magazines/SampleWeapon_01/Magazine_65x39c_SampleWeapon_01_30rnd.xob)** to open it in a new **Resource Manager** tab. After that, you can set up the materials that were previously created for the weapon.

Materials can be assigned in two ways:

- Drag and drop the desired material onto the material icon in the **Materials** tab.
    - This action will automatically reimport the model with the selected material.
- Change the **Material Assign** in the **Visual section** of the **Import Settings**.
    - It will be necessary to click on the **Reimport resource (PC)** button after applying changes.
    
![[armareforger-new-weapon-adjusting-magazine-xob.gif]]

TODO: See the next step: **[Prefab Configuration](https://community.bistudio.com/wiki/Arma_Reforger:Weapon_Creation/Prefab_Configuration "Arma Reforger:Weapon Creation/Prefab Configuration")**.