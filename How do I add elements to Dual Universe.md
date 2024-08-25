The source-of-truth for elements in Dual Univers are stored in the postgresql database. These can be modified, added, and deleted through Backoffice

Adding a brand new element to dual universe involves a few steps. In this example I'll add a new "Freight Wing XS" element. Slightly heavier, but slightly more lift.

1. Create the item in backoffice. The kind of element will determine where in the Item Heirarchy you will create the item definition. For example, a new type of Wing would go under GameplayObject/BaseItem/Consumable/Element/Construct/Element/Piloting/EngineUnit/Airfoil/Wing 2. Creating the new item there will inherit properties from all the catagories above. Clicking the "+" symbol next to "Wing 2" will present a pop up in BackOffice to name the new element.
2. Add properties to the new element. By default, the item YAML for our new element looks like this:
```
WingXtraSmallFreight:
parent: Wing2
```
But we want it to look more like the other Wing XS
```
WingXtraSmall2:
  parent: Wing2
  displayName: Wing
  maxPower: 62500
  maxLiftFactor: 12.5
  unitMass: 61.20
  unitVolume: 22.60
  level: 1
  scale: xs
  visibilityLOD: 2
  hitpoints: 50
  liftDragRatio: 7
```
So we need to copy and past all the properties from displayName to liftDragRatio and add them to our new element YAML. After adjusting some of the properties for out new wing, the resulting YAML would look like this. Note that we also changed the level to "2" signifying a tier 2 item. This will appear in the corner of the icon for the object in-game.
```
WingXtraSmallFreight:
  parent: Wing2
  displayName: Freight Wing
  maxPower: 100500
  maxLiftFactor: 15.5
  unitMass: 75.20
  unitVolume: 35.60
  level: 2
  scale: xs
  visibilityLOD: 2
  hitpoints: 55
  liftDragRatio: 7
```
Clicking "Save" will update the item definition in the database. The item can now be found in the market listings with zero orders, but it will have the "Error" icon, and also cannot be found in the crafting recipes.

##How do I make the new item craftable?

To make our new wing craftable we need to add a crafting recipe to the recipes table. We can do this through backoffice by going to the recipes tab. The only way to add recipes from backoffice is to upload them with the "Import from computer" function. We can download the "recipes. Yaml" by clicking "Download All" at the top of the page.

IMPORTANT: The behavior when uploading means that the recipes you upload are the ones that will be available in game. Meaning that you have to add the new recipe to the end of the existing recipes, and then re-upload the recipes. Yaml. If you remove anything from this file, it will be removed from the game.

To make our new recipe for our Freight Wing XS, we'll start with the copy of WingXtraSmall 2:
```
---
id: 422654402
time: 120
nanocraftable: true
in:
- screw_1: 1
- hydraulics_1: 1
- mobilepanel_1_xs: 1
- standardframe_1_xs: 1
out:
- WingXtraSmall2: 1
industries:
- IndustryAssemblyXS
- IndustryAssemblyXS2
- IndustryAssemblyXS3
- IndustryAssemblyXS4
```

Note the string of hyphens in the first line, we want to make sure those are preserved when we upload our new recipe. After making the changes to the new recipe to reflect the tier 2 components we need to make it, as well as changing the ID so that it can properly load into the game. I'm using an ID that is much higher than the very last item in the recipes. Yaml. In the default config, this is "2146605013". So i'll be starting with 5000000001.
```
---
id: 5000000001
time: 120
nanocraftable: true
in:
- screw_1: 1
- screw_2: 1
- hydraulics_2: 1
- mobilepanel_2_xs: 1
- standardframe_2_xs: 1
out:
- WingXtraSmallFreight: 1
industries:
- IndustryAssemblyXS
- IndustryAssemblyXS2
- IndustryAssemblyXS3
- IndustryAssemblyXS4

```
Past the new recipe into your recipes. Yaml. It is important to make sure that you have an empty line between the previous recipe, and the line of hyphens, as well as an extra empty line after the new recipe. Once added to recipes. Yaml, save the file and upload to the recipes page. The import process is fairly quick, taking less than a minute. When you search the recipes for the new element you should see the element be shown with the new ID. Clicking on the element name should link you to the item definition. On the item definition under "Recipes Producing" you should see the ID of the new recipe. Thats how we know the two are linked.

Great, now we have our new element and we can craft it. But now we have the next problem

##When I try to add the element to my construct I get a box saying "Mesh Missing"!

We need to link the new element to a mesh so that the game knows how to render our new element. To do this we need to make changes to the client.

In the client folder on the computer where you play myDU, all the element mesh definitions are stored under `Dual Universe\Game\data\resources_generated\elements`. To update one of these element definitions to link to our new element we need to find the relevant folder. So we navigate to `Dual Universe\Game\data\resources_generated\elements\wings\wing_004_s\`  (The names of each individual element are there, but not always one-to-one with the names of what you see in game). 

In each of these element folders you'll find a few more folders:
Defs: contains a sort of JSON-like .nqdef file that defines each element that should use the nodes and meshes in this folder.
Icons: the picture that gets used for the element in the game, shows up in inventory, crafting and markets.
Meshes: the actual mesh models to be used for the element in game.
Nodes: contains a .node file that links the element definition to the mesh, as well as any any animations, visual effects, as well as the orientation of the element in relation to the world in the game. 

To link our new element to the existing node, we just need to update the .nqdef in the defs folder. By default, the env_wing_004_s.nqdef file looks like this:

```
{
  "elements": {
    "WingXtraSmall 2": {
      "icon": "resources_generated/elements/wings/wing_004_s/icons/env_wing_004_s_icon. Png",
      "node": "resources_generated/elements/wings/wing_004_s/nodes/env_wing_004_s.node",
      "soundMaterial": "hard"
    }
  }
}
```
We just need to add a new entry to the "elements" record to ensure that the element renders properly. Its important to make sure that the new element record name matches the name we used for the item definition in Back office.

```
{
  "elements": {
    "WingXtraSmall 2": {
      "icon": "resources_generated/elements/wings/wing_004_s/icons/env_wing_004_s_icon. Png",
      "node": "resources_generated/elements/wings/wing_004_s/nodes/env_wing_004_s.node",
      "soundMaterial": "hard"
    },
    "WingXtraSmallFreight": {
      "icon": "resources_generated/elements/wings/wing_004_s/icons/env_wing_004_s_icon. Png",
      "node": "resources_generated/elements/wings/wing_004_s/nodes/env_wing_004_s.node",
      "soundMaterial": "hard"
    }
  }
}
```

And thats it, we added a new element, made it craftable, and then gave it a mesh for rendering. 

Final notes: There are many elements that are already in the game that have item definitions, and may also have entries in their respective .nqdef files. They're not intuitive so it takes some patience and effort to find them. Many such items appear in the game as "Error Items". I've found that some of these (like the larger cores), can be fixed by finding the item definition in BackOffice and removing the `hidden: true` property from their yaml. After that they show up in the game with no issues.
