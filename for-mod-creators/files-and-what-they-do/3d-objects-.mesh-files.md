---
description: Documentation on .mesh files and their properties.
---

# 3d objects: .mesh files

This page contains information on .mesh files and their properties.

* If you want to export or import mesh files, see [exporting-and-importing-meshes](../3d-modelling/exporting-and-importing-meshes/ "mention")
* If you want to learn how to edit a mesh's appearance, check [changing-materials-colors-and-textures.md](../modding-guides/items-equipment/editing-existing-items/changing-materials-colors-and-textures.md "mention")
* If you want to play around with mesh materials, see [textured-items-and-cyberpunk-materials.md](../modding-guides/everything-else/textured-items-and-cyberpunk-materials.md "mention")

{% hint style="info" %}
In the context of Cyberpunk, a mesh is the file that defines the topology[^1] and the materials[^2] of an object in the game world.

A mesh can have several **submeshes**, each of which has own material assignments.
{% endhint %}

## How the mesh is loaded

Meshes are loaded in .app files ( [#components](appearance-.app-files.md#components "mention") ) or .ent files ( [#mesh-component-entity-simple-entity](entity-.ent-files/#mesh-component-entity-simple-entity "mention")) via [components](components/ "mention"), e.g.[#entgarmentskinnedmeshcomponent](components/documented-components.md#entgarmentskinnedmeshcomponent "mention").&#x20;

### Shadows

There are two ways of adding shadows to  meshes:&#x20;

#### Component property

To make a mesh cast a real-time shadow, set the component's property `castShadows` to `Always`.

{% hint style="warning" %}
Depending on your geometry, this can impact performance.
{% endhint %}

#### Shadow mesh

Many meshes have dedicated shadow meshes, which have a much lower level of detail and will be hidden by default. You can open any clothing item's mesh entity to see this in action.

## Material assignment

### Step 1: Appearances

{% hint style="info" %}
This page only contains mesh-specific information. Find more details on materials, the [corresponding section](../materials/).&#x20;
{% endhint %}

This is how to determine which parts of the mesh have which material:

<figure><img src="../../.gitbook/assets/materials_explanation.png" alt=""><figcaption><p>Example: A mesh with two materials, one of them a local instance, one of them an external .mi file</p></figcaption></figure>

### ChunkMaterials

You assign materials based on the "chunks" (the individual submeshes) inside a mesh. Open the mesh file in Wolvenkit and open the "appearances" array, then make sure that each of your submeshes has an entry inside the array.

<figure><img src="../../.gitbook/assets/mesh_material_appearance.png" alt=""><figcaption><p>You may have to create additional entries in "chunkMaterials": Either duplicate an existing entry from the right-click menu, or select the array and use the yellow (+) in the side panel.</p></figcaption></figure>

### Material definition

Materials are defined in the array **`materialEntries`** inside your mesh:

<figure><img src="../../.gitbook/assets/materials_materialentries_overview.png" alt=""><figcaption><p>For a detailed example, see <a href="../materials/re-using-materials-.mi.md#maximally-lazy-external-materials">re-using materials</a></p></figcaption></figure>

{% hint style="warning" %}
While you can mix external and local materials, you can not mix preloaded and non-preloaded ones. For details, see [below](3d-objects-.mesh-files.md#preload...-what).
{% endhint %}

<table><thead><tr><th width="202">Property</th><th>Description</th></tr></thead><tbody><tr><td>index</td><td><strong>numerical index</strong> of corresponding material in target list (as defined by <code>isLocalInstance</code>)</td></tr><tr><td>isLocalInstance</td><td>Selects the material target list.<br><strong>True:</strong> <a href="3d-objects-.mesh-files.md#materialinstance-the-local-material">local material</a> in <code>localMaterialBuffer.materials</code> or <code>preloadLocalMaterialInstances</code><br><strong>False:</strong> <a href="3d-objects-.mesh-files.md#material-reference-a-material-somewhere-else">material reference</a> in<code>externalMaterials</code> or <code>preloadExternalMaterials</code><br><br>For more information on this, see the page for <a href="../materials/re-using-materials-.mi.md#maximally-lazy-external-materials">local/external materials</a>.</td></tr><tr><td>name</td><td><strong>unique</strong> name of material, used to select the material via <code>chunkMaterial</code></td></tr></tbody></table>

#### Preload… what?

Many of CDPR's early meshes use `preloadLocalMaterialInstances` instead of `localMaterialBuffer.materials`. As far as we are concerned, you can use the two interchangeably, **but**:&#x20;

If you are using **a mix of local and external materials**, you **must** use the corresponding lists:

| local                           | external                   |
| ------------------------------- | -------------------------- |
| `localMaterialBuffer.materials` | `externalMaterials`        |
| `preloadLocalMaterialInstances` | `preloadExternalMaterials` |

&#x20;If you mix the two, the materials outside of `preload`… will appear as transparent the first 1-2 times you trigger your item's appearance.

### Step 3: Material definition

A material definition can be in a `CMaterialInstance` inside the mesh or in a [`.mi` file](../materials/re-using-materials-.mi.md) in the project. For local materials, Wolvenkit will display material names as defined in the `materialEntries`.

For more details on material definitions, [check here](../materials/).

### MaterialInstance: The local material

The materials themselves are inside the array `localMaterialBuffer.materials` (or `preloadLocalMaterials` in case of older meshes).&#x20;

{% hint style="success" %}
You can't go wrong by using those. However, if you don't have any properties that are unique to your mesh or appearance (for example a custom normal map), you might consider [creating and using an external material instead](../materials/re-using-materials-.mi.md).
{% endhint %}

A material instance looks like this:

<figure><img src="../../.gitbook/assets/material_docu_material_instance.png" alt=""><figcaption><p>baseMaterial picks the material (shader), while "values" contains <a href="3d-objects-.mesh-files.md#checking-material-properties">properties</a> to adjust it.</p></figcaption></figure>

{% hint style="info" %}
You can find a guide about [texture editing](../modding-guides/items-equipment/editing-existing-items/changing-materials-colors-and-textures.md) and [adding custom textures](../modding-guides/items-equipment/editing-existing-items/changing-materials-colors-and-textures.md#step-4-optional-custompathing) in the [modding-guides](../modding-guides/ "mention") section.

For an overview of materials that you might want to use for something, check [here](../../modding-know-how/references-lists-and-overviews/cheat-sheet-materials.md).&#x20;

For how to find out which properties a material has, check [here](../materials/#checking-material-properties).
{% endhint %}

### Material reference: [reusing materials](../materials/re-using-materials-.mi.md#maximally-lazy-external-materials)

A relative path to an external material, usually encapsulated in a [.mi file](../materials/re-using-materials-.mi.md#.mi-files-to-the-rescue). Use this if you don't need to add extra properties.

[^1]: vertices, edges – everything that makes up the 3d object's surface information

[^2]: the surface shader and its properties
