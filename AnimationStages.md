Made as of Game Version `0.2.5`

There are 2 sections to this tutorial, **Theory** and **Practice**. If you wish to go straight to making animations, skip to **Practice**.

It's recommend to gain a bit of knowledge on the topic first so you can grasp how the game functions. Without doing so it may be difficult to expand or incorporate more advanced methods not specifically covered in the steps. A test mod is provided at the bottom of the page.

Be aware, this tutorial assumes that you have basic knowledge of Godot, Blender, and their interfaces. Novice developers, if you have trouble following along, you may want to brush up on the fundamentals first.

# Theory

**File Names/Usage**
- Dolls and Animations are saved as .glb files. -3D Model File
- Animation resources are referenced to Godot as .tres files. -Text File
- Stage scripts are read as .gd files. -Text File
- Godot scenes/nodes are read as .tscn files. -Text File

## **Understanding Dolls & Stages**
**Doll** is just the name Rahi gave to the actual armature file and what is used to reference it in files. Dolls are created in Blender, as `DollSkeleton.blend` files: this is where the armature and animations are made. If you have experience with 3D software you'll easily understand armature; if not, this is essentially the doll's "skeleton", which is animated by manipulating its bones. Animations, or actions, are stored, or "stashed", as Non-Linear Animation (NLA) tracks. Little to no animation knowledge is required to use keyframes in the action editor.

To use Dolls in Godot, we export them to binary files (`DollSkeleton.glb`) and import them into the project. In Godot, the resource will be treated as a scene file with two major nodes: Skeleton for the armature, and AnimationPlayer for the animations: to edit these nodes, we can create an inherited scene called `DollSkeleton.tscn` and work from there. The DollSkeleton is then instanced in another scene, `Doll3D.tscn`, where we also set up the scripting for the jiggle and deform bones, the bone attachments, the camera, et cetera.

When making new animations for the pre-existing skeleton, we can choose to import only the AnimationPlayer from our .glb files with a custom script, "CustomImportScriptOnlyLeaveAnimPlayer". This can be added on to Doll3D.tscn to easily extend the animations without touching anything else. Officially, new animation skeletons can be added easily, such as DollSkeleton2 and DollSkeleton3. For mods, it's best to make our own custom Doll3D.tscn, and then overwrite one of the extension DollSkeletons: this tutorial will overwrite DollSkeleton2.

**Stages**, or **StageScenes**, are the 3D render of your doll/character while ingame. Stages define the properties of the 3D scene in the render and are typically named after the subject of the animations (i.e. `StageScene.Grope`, `StageScene.PuppySexStart`, `StageScene.Birth`). Stages are stored in `res://Player/StageScene3D/Scenes(1-3)`

> [!NOTE]
> If you haven't already figured it out, even though the character dolls are built with two-dimensional body parts, they're rigged to a three-dimensional armature. So while the animations in the player panel are all being rendered in 3D, the artstyle creates the illusion of 2D animation. This style of cutout animation makes it easy for modders to make new body parts with basic painting tools, while allowing us to use a 3D workflow for animations.

Most stage names are self explanatory. Here are a few examples:

**StageScene.Solo**
- Used for when the game renders only 1 character (Hinted by the name Solo) for general animations. Your character playing the walking animation when you roam around the map is `StageScene.Solo`
```gdscript
playAnimation(StageScene.Solo, "walk")
```

**StageScene.Duo**
- Used for when the game renders 2 characters for non-sex-animations (Also hinted by the name). Your character talking to another in a cutscene is `StageScene.Duo`
```gdscript
playAnimation(StageScene.Duo, "stand", {npc="eliza"})
```

**StageScene.SexCowgirlAlt**
- Used for when the game renders 2 characters for animations in the "Sex Cowgirl Alt" position during sex.

```gdscript
playAnimation(StageScene.SexCowgirlAlt, "fast", {npc="pc", pc="nova", bodyState={naked=true, hard=true}, npcBodyState={naked=true, hard=true}})
```
This will have your player character riding Nova while both of you are naked and erect (if your player character has a penis). Sounds like fun, but let's not get distracted. See the next section for more information on using arguments in animations.

**StageScene.TFLook**
- Used for when the game renders your character for animations to look at their own body during transformation pill cutscenes.
```gdscript
playAnimation(StageScene.TFLook, "crotch", {pc="eliza", bodyState={exposedCrotch=true, hard=true}})
```
This will tell the game to set Eliza as the main focus character, to look at her crotch, expose it, and make her penis erect. (She presumably grew one in this scene.)

## **Using Animations in Scenes**

Calling to play an animation in a scene is playAnimation() and uses the following parameters:
```gdscript
playAnimation(The StageID[1], The AnimationID[2], Any Arguments[3])
```

[1] The stage will be set by the game with this parameter. It will look for the StageScene with that ID.
- Example: `StageScene.Solo`, `StageScene.BDSMMachineAltFuck`, `"GivingBirth"`, `"YourStage"`

[2] The Animation ID tells the dolls what animation to play.
- Example: `"jog"`, `"grope"`, `"firepistol"`

[3] The arguments to support the Animation ID.
- Example: `{pc="artica"}`, `{pc="jacki", bodyState={underwear=hasUnderwear}}`, `{npc="socket", further=true, npcAction=["stand", "res://Inventory/UnriggedModels/BigWrench/BigWrench.tscn"]}`

In Stages, `playAnimation()` will be received as:
```gdscript
func playAnimation(animID, _args = {}):
```

_args{} is a [dictionary](https://docs.godotengine.org/en/3.6/classes/class_dictionary.html), which tells the game:
- What character to make the focus
	- `pc="artica"` will make Artica the main focus/dominant character
- What to show on the character/scene
	- `bodyState={naked=true}, npcBodyState={exposedCrotch=true,hard=true}` will make the PC doll naked and the NPC doll erect with their penis out
	- `npcAction=["stand", "res://Inventory/UnriggedModels/BigWrench/BigWrench.tscn"]` will have the NPC standing while holding a wrench
	- See Doll3D.gd's [`applyBodyState` method](https://github.com/Alexofp/BDCC/blob/main/Player/Player3D/Doll3D.gd#L748) for a comprehensive list of bodyState arguments.
- What specific animation to play
	- `npcAction="sit"` will make the NPC character do a sitting animation

## **Inside of a Stage.gd File**
If we want to use our animations in-game, they need to be scripted. StageScene scripts are inherited from BaseStageScene3D, but we still have to do some setup for each one. 
```gdscript
extends BaseStageScene3D

onready var animationTree = $AnimationTree
onready var doll = $Doll3D

func _init():
	id = "StageID"

func _ready():
	animationTree.anim_player = animationTree.get_path_to(doll.getAnimPlayer2())
	animationTree.active = true
```

When readying the animationTree.anim_player property, you'll have to associate it with your custom AnimationPlayer. There are three possible paths:
- doll.getAnimPlayer(): DollSkeleton's AnimationPlayer.
- doll.getAnimPlayer2(): DollSkeleton2's AnimationPlayer.
- doll.getAnimPlayer3(): DollSkeleton3's AnimationPlayer.

We're going to overwrite DollSkeleton2 for this tutorial, so we're setting the path to doll.getAnimPlayer2(). But if you overwrite DollSkeleton3 instead, you'd set it to doll.getAnimPlayer3().

### Using Dolls

All of Rahi's non-solo Stages have a "pc" and "npc". 
PC will be the dominant in a sex scene or the main focus in a non sex scene. That makes NPC the submissive in a sex scene or the secondary focus. PC is often the player's character but can also be used to make a different character the main one, depending on the scene or even the animation.

Most of Rahi's StageScene have 2-3 dolls: 

```gdscript
onready var animationTree = $AnimationTree
onready var animationTree2 = $AnimationTree2
onready var animationTree3 = $AnimationTree3
onready var doll = $Doll3D
onready var doll2 = $Doll3D2
onready var doll3 = $Doll3D3

func playAnimation(animID, _args = {}):
	var fullAnimID = animID
	if(animID is Array):
		animID = animID[0]
	
	print("Playing duo: "+str(animID))
	var firstDoll = "pc"
	if(_args.has("pc")):
		firstDoll = _args["pc"]
	doll.prepareCharacter(firstDoll)
	var secondDoll = "pc"
	if(_args.has("npc")):
		secondDoll = _args["npc"]
	doll2.prepareCharacter(secondDoll)
( . . . )
```

If you wish to add more dolls, you'll have to add more Doll3D and AnimationTree nodes in your stage.tscn file.

### Methods

There are a few important functions for Rahi's stages:

#### `state_machine.travel()`
- The method that calls the animation to the doll. Takes the actual name of an animation stored in a Doll.glb file.
- Corresponds to the animation tree for each doll you have in the stage. If you have 3 dolls in a stage ([ButtStackSex.gd](https://github.com/Alexofp/BDCC/blob/main/Player/StageScene3D/Scenes/ButtStackSex.gd#L80-L82)), you'd have:
```gdscript
var state_machine = animationTree["parameters/StateMachine/playback"]
var state_machine2 = animationTree2["parameters/StateMachine/playback"]
var state_machine3 = animationTree3["parameters/StateMachine/playback"]

if(animID == "tease"):
		state_machine.travel("ButtStackSexTease_1-loop")
		state_machine2.travel("ButtStackSexTease_2-loop")
		state_machine3.travel("ButtStackSexTease_3-loop")
```
<div align="center">
   <sup>Each doll has their own animation to do in a stage</sup>
</div>

The available arguments in an existing stage are obviously limited to what the .gd file calls for. You can quite obviously add your own arguments and functions. Self problem-solving and improvision is key, this is just the run-down on Rahi's stages.

#### `func playAnimation(animID, _args = {})`
- Sets what is shown in the stage and passes the AnimID to either `state_machine.travel()` or `stateMachineTravel()`. Fairly self explanatory when looking at the function yourself. 
- Review [Using Animations in Scenes](#using-animations-in-scenes).

#### `func getSupportedStates()`
- An array that dictates what animationIDs the stage can receive. 
- The [CheckPaw](https://github.com/Alexofp/BDCC/blob/main/Player/StageScene3D/Scenes2/CheckPaw.gd#L95) stage has five states: `"tease"`, `"check"`, `"lick"`, `"beans"`, and `"crotch"`. If you call `playAnimation(StageScene.CheckPaw, "sit")`, the stage will error as `"sit"` is not a valid animationID.
- Some stages, such as `StageScene.Solo` and `StageScene.Duo`, are exceptions: these return `getSupportedStatesSolo()` inherited from BaseStageScene3D and containing standard base animationIDs like `"stand"`, `"walk"`, `"jog"`, etc.* These are then directly passed to BaseStageScene3D's `stateMachineTravel` method, instead of calling `state_machine.travel()` in `playAnimation` like most other stages do. For this reason, you'll probably want to reference other stages instead when making custom animations, at least when you're first starting out.


## **Inside of a Stage.tscn File**

Stage.tscn files hold the node structure for a scene, however there are only 3 node types you need to worry about:

#### [AnimationPlayer](https://docs.godotengine.org/en/3.6/classes/class_animationplayer.html)
- Child to DollSkeleton Spatial nodes
- Holds the references to .tres files for animations
- Useful Properties:
	- `speed`/`playback_speed` (float)

#### [AnimationTree](https://docs.godotengine.org/en/3.6/classes/class_animationtree.html#animationtree)
- Child to Base Scene node
- Creates transitions from animation to animation
- Blends animations together
- Outputs into a single animation for the armature
- Stored as a .tres file ([DollSoloAnimationTree.tres](https://github.com/Alexofp/BDCC/blob/main/Player/StageScene3D/Scenes/DollSoloAnimationTree.tres))

#### [Spatial](https://docs.godotengine.org/en/3.6/classes/class_spatial.html)
- Child to Base Scene node
- The actual 3D instance that renders the doll/character
- Useful Properties:
	- Transform properties `translation`, `rotation_degrees`, and `scale` (X, Y, Z)
	- `visible` (boolean)

.tscn files can be edited in text editors manually, but it's recommended to use the Godot engine, ***especially*** if you don't know what you're doing.

## **DollSoloAndCuffsTree.tres and DollSoloAnimationTree.tres**

These 2 files are essential to animations:

**DollSoloAndCuffsTree.tres**
- Tree Root for Animation Tree node
- Blends 2 animations together. This specifically is used to overwrite arm/leg animations when a character is cuffed, putting their hands behind their back or feet together.

**DollSoloAnimationTree.tres**
- StateMachine node for Tree Root
- Essentially a flow chart for animations. Used to advance from 1 animation to another in succession.
- Example: You call `state_machine.travel("Walk-loop")`, the Animation Tree will run `Standing-loop` -> `StandingToWalking` -> `Walk-loop`

In this tutorial, these 2 files will be combined into a single .tres file.

![GodotTree](https://github.com/iiHartMemphisii/BDCC-Wiki/blob/RealTrapping/images/AnimationStages/godottree.png)
<div align="center">
   <sup>Animation Tree editor in Godot. All animations start with and are connected to Standing-loop. The animations in your stage can be centered around a different animation if you wish.</sup>
</div>

# Practice

**What tools you'll need**
- Godot Engine 3.6.2 and BDCC project repository (see [How to setup development environment](https://github.com/Alexofp/BDCC/wiki/How-to-setup-development-environment))
- Blender (v3.6 recommended)
- Text File Editor

## **Setup**

**File Structure**

Set up the following file structure in your module. All the files you create for your custom animation will reside here:
- `res://Modules/MyModule/Player`
- `res://Modules/MyModule/Player/Player3D` (Stores the Doll.glb related files)
- `res://Modules/MyModule/Player/StageScene3D` (Stores the Stage related files)

It's recommended to keep folder structure similar to how the base game does. Only rename/reorder if you know what you're doing.

## **Blender**

*Keep in mind, this is NOT a Blender tutorial. If you need general assistance with using Blender, [consult the Blender documentation](https://docs.blender.org/manual/en/3.6/index.html).*

- Launch Blender and open `DollSkeleton.blend` in your BDCC project files. (`res://AssetsSource/character`)
- **(Heavily recommended but not required)** Change the Display Mode of the Outliner to Blender File and delete NLA actions (`Current File` -> `Actions`) you won't use in-game or as references (This drastically cuts down the file size and much easier to export), followed by deleting unused NLA tracks. Change the Display Mode of the Outliner back to View Layer.
- **(Recommended)** When looking through the body part meshes, you may notice that most of them are missing textures. To fix this, go to File > External Data > Find Missing Files, navigate to `res://AssetsSource/character/img`, and then select "Find Missing Files". Some textures may still be missing, but this should at least restore the major body parts.
- Unlink the current action and create a new Action OR make a Single User Copy of an existing track to use as a reference.
- Select the Doll3D armature in the outliner and change Object Interaction mode to "Pose"
- Enable Auto-Keying
- Animate!

![BlenderShowcase](https://github.com/iiHartMemphisii/BDCC-Wiki/blob/RealTrapping/images/AnimationStages/blendshow.png)

### Animation Tips

The Doll armature uses [Inverse Kinematics](https://docs.blender.org/manual/en/latest/animation/armatures/posing/bone_constraints/inverse_kinematics/introduction.html), meaning not all bones need to be individually animated. Moving the armature's feet will have the upper and lower leg follow accordingly, same with hands and arms.

There are only 9 main bones you need to keyframe:
- Hands Left and Right
- Feet Left and Right
- Hips
- Chest
- Neck
- Head
- Tail (1-5)
- Penis (1-3)
- Balls

Try moving bones around and see how the joints follow for yourself!

> [!NOTE]
> If you want your action/animation to be a loop, be sure to add "-loop" to the end of the name.
> 
> Example: `DoggyStyle3SexFast` -> `DoggyStyle3SexFast-loop`

Once you've finished animating your action, stash the action to the NLA stack. Export the Doll as a .glb file and you're ready for Godot. *Be sure to select all checkboxes under the include/limit to export setting.* You can name the file anything for ease of memory; for the rest of this tutorial, we'll refer to it as CustomDoll.glb. Place your exported file in `res://Modules/MyModule/Player/Player3D`.

![BlenderExport](https://github.com/iiHartMemphisii/BDCC-Wiki/blob/RealTrapping/images/AnimationStages/blenderexport.png)

## **Godot**

*Keep in mind, this is NOT a Godot tutorial. If you need assistance on Godot's complete interface, [consult the engine documentation](https://docs.godotengine.org/en/3.6/index.html).*

Open the BDCC project in the Godot Editor.

### Re-Importing CustomDoll.glb
When you add your CustomDoll.glb to your project, Godot will import it automatically, but some of the properties need to be adjusted. To do this, you have to [re-import the asset](https://docs.godotengine.org/en/3.6/tutorials/assets_pipeline/import_process.html):
- Set the Animation FPS property to 24 (The default is 15), and Max Linear Error to 0.02 (The default is 0.05). Leave all other settings as default (unless you know what you're doing). Hit Preset > Set as Default for 'Scene' so this is done automatically in the future.
- Attach the following custom script: `res://Player/Player3D/CustomImportScriptOnlyLeaveAnimPlayer.gd`. Since we'll be using the armature from the basegame DollSkeleton.gd, we only need the AnimationPlayer node. Re-import the resource to apply the script and the other properties you changed.
- Don't forget to add the .import files to your mod .import folder, either manually or with the in-game ModMaker tool.

### Setting up Doll3D.tscn (Standard Method)
- Duplicate `Player/Player3D/Doll3D.tscn` and move it to your mod's Player3D folder (`res://Modules/MyModule/Player/Player3D`). Rename it CustomDoll3D.tscn.
- Open your new doll scene in the editor. Under the main Doll3D node, delete the DollSkeleton2 node. Instance a new child scene from your CustomDoll.glb, and rename it DollSkeleton2.
- Set the new DollSkeleton2 node to have editable children, and reassign its AnimationPlayer's root node to the DollSkeleton node.

To test if everything is set up properly so far, go to the 3D screen, select DollSkeleton2's AnimationPlayer, and try previewing your animations with the animation editor. If the skeleton moves with your animations, you're in good shape!

#### Text Editor Method
*This isn't the usual way to make .tcsn files, but you might find it easier and faster than recreating them in the Godot engine for the same effect.*

- Duplicate Doll3D.tcsn `res://Player/Player3D` and move it to your own Player3D folder (`res://Modules/MyModule/Player/Player3D`). Rename it CustomDoll3D.tscn
- Open CustomDoll3D.tcsn in a text file editor and edit the 7th resource line, replacing Rahi's DollSkeleton2.glb with your own.
- Leave all other resources alone unless you know what you're doing.

`[ext_resource path="res://Player/Player3D/DollSkeleton2.glb" type="PackedScene" id=7]` -> `[ext_resource path="res://Modules/MyModuleTwerk/Player/Player3D/CustomDoll.glb" type="PackedScene" id=7]`

### CustomStage.gd creation

It's recommended to clone an existing Stage.gd file base as to not have to completely create a script file from scratch. You can rewrite your Stage.gd file from there. Just keep in mind that some scenes, such as Duo.gd, will point to nodes that your stage may not use (Like $Chair or $KidlatBox). It's up to you to clean this up and make it work. Alternatively, [see the example mod](#lotterustestzip).

The important things to check:
- Make sure the script is instanced from BaseStageScene3D, and ready however many animationTrees and dolls are going to be in your scene. Under `_init()`, assign a unique ID, like you would with most other game assets.
```gdscript
extends BaseStageScene3D

onready var animationTree = $AnimationTree
onready var doll = $Doll3D

func _init():
	id = "CustomStage"
```

> [!WARNING]
> When assigning the stage an ID, write it out as a string, like "CustomStage". Do **NOT** write it as StageScene.CustomStage, like most IDs you'll see in the game files; this will return an error. Names like `StageScene.Solo` and `StageScene.Yoga` are references to constants held in [StageScene.gd](https://github.com/Alexofp/BDCC/blob/main/Player/StageScene3D/StageScene.gd), which you are not going to overwrite. Simply assign it as a string directly. If you really want to, you can set up your own constants in your module.

- Since we're overwriting DollSkeleton2, make sure your `_ready()` method looks like this:
```gdscript
func _ready():
	animationTree.anim_player = animationTree.get_path_to(doll.getAnimPlayer2())
	animationTree.active = true
```
- Have `getSupportedStates()` return the array of animationIDs you want to be able to pass to the stage.
```gdscript
func getSupportedStates():
	return ["MyAction1", "MyAction2", "MyAction3"]
```
- Make sure `getVarNpcs()` corresponds with the number of dolls in the scene. If it's a solo scene, it should return `["pc"]`; if it's a duo scene, it should return `["pc", "npc"]`; threesome scenes return `["pc", "npc", "npc2"]`; and so on. A solo scene's method should simply look like this:
```gdscript
func getVarNpcs():
	return ["pc"]
```
- Generally speaking, make sure `canTransitionTo()` also corresponds with the number of dolls in the scene. If it's a solo scene, it should probably look like this:
```gdscript
func canTransitionTo(_actionID, _args = []):
	var firstDoll = "pc"
	if(_args.has("pc")):
		firstDoll = _args["pc"]
		
	if(doll.getCharacterID() != firstDoll):
		return false
	return true
```
- If you anticipate that any of the characters could be wearing cuff restraints during your animation, make sure that this is reflected by `updateSubAnims()`. Conversely, if they *shouldn't* be wearing cuff restraints because the animation doesn't support it, then make sure this method *isn't* in your script:
```gdscript
func updateSubAnims():
	if(doll.getArmsCuffed()):
		animationTree["parameters/CuffsBlend/blend_amount"] = 1.0
	else:
		animationTree["parameters/CuffsBlend/blend_amount"] = 0.0
``` 
- Under `playAnimation()`, first prepare your characters:
```gdscript
func playAnimation(animID, _args = {}):
	print("Playing: "+str(animID))
	if(_args.has("pc")):
		doll.prepareCharacter(_args["pc"])
	else:
		doll.prepareCharacter("pc")
	
	if(_args.has("bodyState")):
		doll.applyBodyState(_args["bodyState"])
	else:
		doll.applyBodyState({})
	
	updateSubAnims() #if your script has it

	if(_args.has("pcCum") && _args["pcCum"]):
		startCumPenis(doll)
	
	( . . . )
```
- Next, set up a link to state_machine for each doll in the scene, which we'll explore further in the next section.
```gdscript
func playAnimation(animID, _args = {}):
	( . . . )

	var state_machine = animationTree["parameters/StateMachine/playback"]

	( . . . )
```
- Finally, set up the logic for your animations, telling state_machine what animations to play based on the animationID passed. This can be a very basic series of if statements (or a match statement if you prefer), like this:
```gdscript
func playAnimation(animID, _args = {}):
	( . . . )
	if(animID == "MyAction1"):
		state_machine.travel("animation-1")
	elif(animID == "MyAction2"):
		state_machine.travel("animation-2")
	elif(animID == "MyAction3"):
		state_machine.travel("animation-3")
	else:
		Log.printerr("Action "+str(animID)+" is not found for stage "+str(id))
```

### CustomStage.tcsn creation

Now that the script is set up, it's time to set up the scene file.

- Create a new Inherited Scene of BaseStageScene3D.tscn (`res://Player/StageScene3D`) OR your own modified BaseStageScene3D.tscn if you have one.
- (Optional) Rename the BaseStageScene3D node to the name of your scene.
- Load CustomStage.gd as the script property for the base node

![BlenderShowcase](https://github.com/iiHartMemphisii/BDCC-Wiki/blob/RealTrapping/images/AnimationStages/loadscript.png)

#### Creating the platform

- Create a Sprite3D child node
- Load platform.png from `res://Player/Props`
- Enable Region
- Set property Region Rect (X, Y, W, H) -> `X = 1`, `Y = 1`, `W = 1022`, `H = 62`
- Set property Transform Translation (X, Y, Z) -> `X = 0`, `Y = -0.197`, `Z = -1.775`

You could also copy the platform node from another stage.

#### Adding Character Dolls

- Create an Instance Child Scene node using CustomDoll3D.tscn (`res://Modules/MyModule/Player/Player3D`)
- Create an AnimationTree node under the base node
- Transform/Move the 3D model to where you want it

![GodotNodes](https://github.com/iiHartMemphisii/BDCC-Wiki/blob/RealTrapping/images/AnimationStages/godot2.png)

#### Creating an [AnimationTree](https://docs.godotengine.org/en/stable/classes/class_animationtree.html#class-animationtree)

- Create an AnimationTree child node
- Load DollSoloAndCuffsTree.tres (`res://Player/StageScene3D/Scenes`) as the Tree Root property in the Inspector
- Make the Tree Root unique
- (Only if you wish to completely make your own node and know how) Delete the StateMachine node
- (Optional) Save it as CustomAnimationTree.tres with your stage files (`res://Modules/MyModule/Player/Player3D`)

<div align="center">
   <sup>Animation Nodes</sup>
</div>

![TreeNodes](https://github.com/iiHartMemphisii/BDCC-Wiki/blob/RealTrapping/images/AnimationStages/treenodes.png)

**If you deleted the StateMachine node:**
- Create a new StateMachine node
- Make sure the name of the node matches what you used for the path to state_machine in the gdscript. So if the node is named StateMachine, make sure the state_machine path is `"parameters/StateMachine/playback"`. This should be what the game does automatically, but it doesn't hurt to check.
	- Some animations, like StageScene.Solo and StageScene.Duo, have their StateMachine node called "AnimationNodeStateMachine". In these cases, the state_machine path would be `"parameters/AnimationNodeStateMachine/playback"`

**For all:**
- Open the StateMachine (AnimationNodeStateMachine) editor
- Create your [StateMachine Animation Tree](https://docs.godotengine.org/en/3.6/tutorials/animation/animation_tree.html#statemachine)

#### StateMachine Animation Tree Tips

To add your animations to the tree:
- Set the Doll3D node to have editable children.
- Assign the AnimationTree node's Anim Player property to DollSkeleton2's AnimationPlayer.

![AniLoad](https://github.com/iiHartMemphisii/BDCC-Wiki/blob/RealTrapping/images/AnimationStages/aniload.png)

If starting from scratch:
- Create a start animation
- Branch off/continue from there

If adding onto the existing tree:
- Add your animation node connections starting from Standing-loop
- Use existing nodes as examples for properties for your new animation nodes (`Switch Mode`, `Auto Advance`, `Xfade Time`, `Priority`)

![AniLoad](https://github.com/iiHartMemphisii/BDCC-Wiki/blob/RealTrapping/images/AnimationStages/xfade.png)

Once you have created your StateMachine, save your Tree Root. You can now load it if you wish to add more animations, nodes, properties, etc.

**If you wish to add more than 1 character to your stage, duplicate your Doll3D and AnimationTree nodes for however many characters you wish. Keep in mind to edit your CustomStage.gd accordingly.**

![Triples](https://github.com/iiHartMemphisii/BDCC-Wiki/blob/RealTrapping/images/AnimationStages/triple.png)

## **Adding your stage to the game**

Now that you've created the stage files:
- Open your Module.gd (`res://Modules/MyModule`)
- Add the `stageScenes = []` array along with your `CustomStage.tcsn` to `_init()`
```gdscript
extends Module

func _init():
	id = "MyModule"
	author = "You"

	scenes = [ . . . ]
	characters = [ . . . ]
(. . .)
	stageScenes = [
		"res://Modules/MyModule/Player/StageScene3D/CustomStage.tscn",
	]
```

## **Call your stage while in a scene**

```gdscript
extends SceneBase

func _init():
	sceneID = "YourScene"

func _run():
	if(state == ""):
		addCharacter("tavi")
		playAnimation("CustomStage", "MyAction1", {npc="tavi", npcAction="MyAction4"})

		saynn("You scream out in joy as you watch Tavi and your character do an odd dance together.")

		saynn("[say=pc]Victory at last![/say]")
```

Test File:
# [LotterusTest.zip](https://github.com/iiHartMemphisii/BDCC-Wiki/blob/RealTrapping/examples/LotterusTest.zip)	
