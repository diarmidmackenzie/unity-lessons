# Lesson 1 - Adding a Zombie



In this first lesson we are going to take the basic Unity FPS sample game, and add a zombie.



### Choosing a 3D Model

We decided to use an existing animated 3D model.

We searched on sketchfab, and found this model, which seemed like a good choice:

[https://sketchfab.com/3d-models/zombie-walk-test-165fd9342c364216bfdd8c2f1102223c](https://sketchfab.com/3d-models/zombie-walk-test-165fd9342c364216bfdd8c2f1102223c)



What we liked about it:

- It's free to use under a Creative Commons Attribution license - we just need to attribute the creator like this:

  > "Zombie Walk Test" (https://skfb.ly/6RPU6) by OSCAR CREATIVO is licensed under Creative Commons Attribution (http://creativecommons.org/licenses/by/4.0/).

- It's detailed and realistic looking - and scary!

- It's already animated, so we don't need to worry about rigging and animating a model - we can add it straight into our game as it is.



### Getting the Model into Unity

First we download the 3D model using the link on the page (you'll need to create a free sketchfab account)

![image-20220214103651678](.\image-20220214103651678.png)

We choose to download in FBX format, which works well with Unity.

The download is a zip file, which we extract to a convenient location 

(I used [7zip](https://www.7-zip.org/download.html) to extract the zip file, and extracted into my Documents folder).



When you extract the zip file, you'll see two folders: source (containing an FBX model) and textures (containing various textures).

In the Unity Project panel, open up "Assets / FPS / Art / Models" and drag the FBX file into there.

Now drag the model into somewhere in the Game scene.  You should see the Zombie model in the scene (with no textures yet), and a new object called "Zombie Walk" in the Hierarchy on the left.

![image-20220214105111898](.\image-20220214105111898.png)



Select "Zombie Walk" in the Hierarchy, then in the "Inspector" panel on the right, click on Model / Select.

![image-20220214105238029](.\image-20220214105238029.png)



Click on the "Materials" tab, and click "Extract Materials..."

You'll be prompted to choose a folder for the materials.  Choose "Assets/FPS/Art/Materials/Enemies", and go ahead and extract the materials.

In the Project window, you'll now see you have a new material called "No name"



![image-20220214105542414](.\image-20220214105542414.png)



Right click, and rename this to "Zombie"


Now we want to set this material up to use the textures that came with the Zombie.

Select the material, and in the Inspector, click where it says Shader / Standard, and choose "Nature / Tree  Creator Bark"

![image-20220214105745982](.\image-20220214105745982.png)



What's going on here?  Our zombie isn't supposed made of tree bark...!

Well, unity has various different shaders, and each shader works with a different selection of textures.

If you look at the Textures folder in the zip file, you'll see there are several different texture maps in there, labelled, Diffuse, Gloss, Normal and Specular.

We want to use a shader that supports all these types of maps.

Actually, Nature / Tree Creator Bark only supports 3 out of 4 of these maps (it doesn't include Specular), but it's the closest we can get among the shaders available by default in Unity.



After choosing that shader, you should now see that the material has slots available for "Base", "Normal" and "Gloss" texture maps.

![image-20220214110320973](.\image-20220214110320973.png)



In the Project window, create a new folder FPS / Art / Textures / Zombie /

Now go back to the textures folder from the zip file, and drag the Diffuse, Gloss and Normal texture files into this folder...

![image-20220214110515421](.\image-20220214110515421.png)



Now, with the inspector open on the new "Zombie" Material, you should be able to click "select" on ech of the textures, and assign these textures to the material.

You can type "zom" in the search box to help find the textures you just uploaded.

![image-20220214110936064](.\image-20220214110936064.png)

Assign them as follows:

- Base -> Diffuse
- Normal -> Normal
- Gloss -> Gloss

Now look at your Zombie again in the game window.  You should see him looking distinctly more Zombie-like and terrifying:

![image-20220214111153548](.\image-20220214111153548.png)



### Adding Behaviour

So far the zombie will just stand still - like a waxwork at Madame Tussauds.

You can't interact with him - you can't even shoot at him.

So next, we'll incorporate the zombie into the game.

The simplest way to do this will be to build on one of the existing enemies, the Hover Bot.

Find the "Enemies" section of the Hierarchy.

Right click on the Enemy_HoverBot and copy, then right click & paste.  You'll now have a new entity called Enemy_HoverBot (1) at the bottom of the hierarchy.

![image-20220214111603539](.\image-20220214111603539.png)

Rename this new entity as "Zombie", and move him up to be with the other enemies.

![image-20220214111528696](.\image-20220214111528696.png)

Finally, so we can focus on the zombie, open the original Enemy_HoverBot in the inspector, and clear the checkbox to the left of the entity name.  This will remove him from the scene (temporarily - we can easily add him back later - or delete him if/when we decide we don't want him any more.)

![image-20220214111803305](.\image-20220214111803305.png)



We now have an Entity called "Zombie" - but he doesn't look like a Zombie yet.  Let's fix that.

If you expand "Zombie" in the hierarchy, you'll see a bunch of sub-entities.  BasicRobot is the one that represents the appearance, which we want to replace.

![image-20220214111931152](.\image-20220214111931152.png)

Right click on Zombie, and select "Unpack Prefab"

Now drag the "Zombie Walk" entity to become a child of "Zombie", alongside "Basic Robot", and disable the "BasicRobot" entity in the inspector panel by clearing the checkbox next to the entity name.

![image-20220214112108952](.\image-20220214112108952.png)

(we could have deleted BasicRobot, and will do so later, but disabling has the same effect for now, and means it's easy to go back, or compare against how things were set up before).

Finally, open the "Zombie_walk" object in the inspector and set Transform / Position to 0 , 0, 0.  This will ensure that the Zombie model is positioned in the correct place (as opposed to wherever you happened to drag him into the scene)

![image-20220214112855665](.\image-20220214112855665.png)



If you try playing the game now, you should see a Zombie that acts like a Hover Bot.

He'll move aggressively towards you and shoot you with lasers.



### Shut down the lasers!

There may be quite a few differences in the behaviour we want from the Zombie vs. the Hover Bot enemy, but for now we'll just fix the most egregious.

The easiest way to disable the lasers is to inpsect the Zombie / WeaponRoot / Weapon_EyeLazers entity, and in Weapon Controller (Script), set "Shoot Type" to "Manual"

![image-20220214113220416](.\image-20220214113220416.png)



Ideally we'd completely remove the entities and components associated with the lasers, as we don't need any of this for our zombie.

However, the way the code for the HoverBot is organized, it's not very easy to remove the lasers altogether without breaking some other things we do want for our Zombies (like following the player, having a health bar etc.)

In the future we'll come back and clean this all up, but for now, just switching off automatic lasers is good enough.



### Animation

We now have a Zombie that chases us around rather aggressively, but does so hovering just above the floor.

One of the motivations for choosing this particular zombie model was the fact that it came with a walking animation, so now let's try to get that working.



Open "Zombie Walk" in the inspector, click "Add Component", and select "Animator" (you can type the first few letters to filter on the various components available)

You  can see that the Animator requires a "Controller", and we don't have one of those yet - so let's create one...



In Project, under FPS / Animation / Controllers, right-click, and Create... / Animator Controller

Rename it to "Zombie Animator Controller"

![image-20220214114219879](.\image-20220214114219879.png)



Double click on this to open the Animator window. Right click in the middle of the window and Create State / Empty

![image-20220214114419795](.\image-20220214114419795.png)



Using the inspector panel on the right, rename this to "Walk", and set "Motion" to "mixamo.com" (which is the name that the walking animation was give in the FBX file that we imported)

![image-20220214114532850](.\image-20220214114532850.png)



Now, back in the animation window, right click on "Walk", select "Make Transition", and then drag the white arrow that appears back into the "Walk" box.

![image-20220214114740954](.\image-20220214114740954.png)



You should get a result that looks like this:



![image-20220214114825712](.\image-20220214114825712.png)



What this shows is that when the animator begins, we'll do the "Walk" animation, and then loop the same "walk" animation repeatedly.

Now, go back to "Zombie Walk" in the inspector, select the "Animator" component, and select the "Zombie Animator Controller" that we just created.

![image-20220214115143219](.\image-20220214115143219.png)



If you play the game now, you should see that the Zombie walk animation is in place.

But he's moving a bit too fast - so it looks like he's ice skating!

### Slowing Things Down

To slow down the movement speed, open the Zombie entity in the inspector, and find the "Nav Mesh Agent" component.

Under "steering" there is a setting "speed", set to 3.5.  Reduce this to 0.35, which is a good fit for the default speed of the zombie walking animation.

![image-20220214132504732](.\image-20220214132504732.png)

If you play the game now, you'll see the zombie is moving a lot more slowly, but there's still a problem with the animation.

The zombie's legs seem to be sliding forward through most of the animation, and then slide backwards again at the point that it wraps.

To see more clearly what's going on here. in the Project view, navigate to Assets / FPS / Art / Modelsm, and click on the "Zombie Walk.fbx" model.

In the bottom right corner of the screen you can see the animation, in which the zombie walks forwards, and then snaps back to his start position.

![image-20220214132952154](.\image-20220214132952154.png)

To get the Zombie moving properly, we need to eliminate this jump.

In the panel just above, check the boxes marked "Loop Time" and "Loop Pose".

![image-20220214133035575](.\image-20220214133035575.png)



Press "Apply" to commit these changes to the animation.

In the preview of the animation at the bottom of the panel, you'll see that he's now moving on the spot, rather than moving forwards.

Once we combine this with the in-game movement, we should end up with a realistic-looking walking movement.

Return to the game, and the Zombie's feet should have stopped sliding around.





### Turning the Zombie into a Prefab



As a final step, we'll make the Zombie into a Prefab.

Find the top-level Zombie entity in the Hierarchy, hold down the left mouse button, and drag it to Assets / FPS / Prefabs in the Project panel.

This creates a new "Zombie" prefab, and you'll see that the Zombie entity in the hierarchy has now turned to a solid blue icon.

![image-20220214133708512](.\image-20220214133708512.png)

![image-20220214133724107](.\image-20220214133724107.png)



The benefit of a prefab is that you can create multiple instances of the entity, and apply changes to all of them at once, just by modifying the prefab.

It's time to create some more zombies...



With the "scene" view open, drag the zombie prefab into the scene, and it will create a new zombie that will behave the same as the first one. 

You can add these wherever you like...

![image-20220214134023258](.\image-20220214134023258.png)



To keep things organized, reposition the Zombies in the Hierarchy view, so that they appear under the ==ENEMIES== heading.

![image-20220214134048997](.\image-20220214134048997.png)

### What Next?



There's plenty more to do here, for example...

- The multiple zombies are perfectly synchronized together: some small offets on their animation would make things feel a lot more realistic.
- They don't actually do any damage when they reach the player (remember that we adapted the Hover Bot enemy, which only had ranged weapons - which we disabled).
- The hit boxes for the zombies seem like they could do with some tuning.
- The zombie's sound effects aren't remotely zombie-like!



We'll look at some of these points, along with lots more in future lessons.  For now, enjoy populating the world with zombies, and shooting them up!

