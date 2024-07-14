---
title: Modeling - Importing a Weapon
draft: false
tags:
  - modeling
  - unreal
---
## Importing a Modded Weapon

Updated: 02/03/2024

**This guide outlines the process for importing a custom weapon into a game using the OHDCore Mod Kit.**

## Prerequisites
* Basic understanding of 3D modeling software (e.g., Blender). We're using Blender in this guide.
* Familiarity with Unreal Engine/OHDCore

**Steps:**

1. **Create Asset Folders:**

    * Within your mod project, create separate folders for "Faction", " Items" and "Kits."
    * Place the relevant blueprints and files within their respective folders. (copied from reference base files)

2. **Copy and Export Weapon Mesh:**

    * Locate the existing weapon mesh file (e.g., "`SK_XXXX`") within your game's assets.
    * Copy this file and place it inside a new folder within the "Items" directory.
    * Export the copied SK mesh for further editing.

3. **Import and Align Custom Model:**

    * Open the exported SK mesh and your custom weapon model in a 3D modeling software like Blender.
    * Import the custom model and meticulously align it with the reference SK mesh, ensuring proper scaling, positioning, and rotation. Ensure it matches the reference mesh as close as possible.

4. **Vertex Group Matching:**

    * Select individual components of the custom model and create vertex groups that closely resemble the corresponding vertex groups in the reference SK mesh.
    * This step ensures proper animation and deformation of the weapon during gameplay.

5. **Parenting and Rigging:**

    * Make your finalized custom mesh a child of the "`ROOTBONE`" of the reference SK mesh.
    * Delete the reference SK mesh once the hierarchy is established.
    * Select the weapon model first and then select the rig after and then use the "CTRL+P" function with "Empty Groups" to bind the custom mesh to the existing rig, ensuring proper animation functionality. (To make this easier, make the armature to be in front at all times.)

6. **Export and Import into Game Engine:**

    * Export the final mesh as an FBX file to your workspace folder.
    * Import the FBX file into your OHD mod project.

7. **Modify Kit and Faction Presets:**

    * Update the relevant kit and faction presets within your game to utilize the newly imported weapon.

8. **Blueprint Creation and Modification:**

    * Create a copy of the existing blueprint associated with the reference SK mesh (unless you intend to create custom animations).
    * Edit the copied blueprint and assign your newly imported model as the skeletal mesh for both the "world" and "1st person" hierarchy views.

9. **Save and Compile:**

    * Save all modifications made to the blueprint.
    * Compile the blueprint to ensure proper functionality within the game.

**Note:** This guide assumes a basic understanding of 3D modeling and game development concepts. 