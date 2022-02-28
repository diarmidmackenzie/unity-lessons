# Lesson 2 - Zombie Improvements



In lesson 1, we added some zombies to the Unity FPS sample game.

In this lesson we're going to fix up some of the issues we had with the zombies...

- The multiple zombies are perfectly synchronized together: some small offets on their animation would make things feel a lot more realistic.
- They don't actually do any damage when they reach the player (remember that we adapted the Hover Bot enemy, which only had ranged weapons - which we disabled).
- The hit boxes for the zombies seem like they could do with some tuning.
- The zombie's sound effects aren't remotely zombie-like!



### De-Synchronization

The first thing we want to do is stop the zombies from being perfectly synchronized, which makes them look less realistic.

We're actually going to de-synchronize them in two different ways:

- Give each zombie a different cycle offset, so they start at different points in their animation cycle
- Give each zombie a different speed.

First, we'll take care of the cycle offset...

In the Project view, navigate to Animation / Controllers, and select the Zombie Animator Controller.

![image-20220228104703572](.\image-20220228104703572.png)



Now, in the Animatior view, choose the Parameters tab, and add a Float Parameter to Animator

![image-20220221130739856](.\image-20220221130739856.png)

Call it "CycleOffset" - don't worry about setting a value, the default of 0.0 is fine.

Viewing the "Walk" state in the Inspector, find "CycleOffset", check the "Parameter" box, and choose the CycleOffset parameter you just created.

![image-20220221135505464](.\image-20220221135505464.png)

Now, we need to set these values randomly on each Zombie.  To do this, we'll need to write out first script - but don't worry, it's a simple one.

If you haven't set up an editor for editing scripts yet, you'll need to do this.

- [VSCode](https://code.visualstudio.com/download) is the recommended text editor for Unity.
- If you have problems with VSCode (even though it is free, we had some license issues due to being in a school setting), [Atom](https://atom.io/) is another excellent alternative.

Whatever editor you choose, configure it under Project / Preferences / External Tools, so that you can click directly on an script to open it in your editor.

![image-20220228105309739](.\image-20220228105309739.png)



OK - now let's create our first script...

First, we need to open the Zombie Prefab Asset.  This means that we'll be modifying out generic Zombie prefab, rather than any individual zombie - so any changes we make will immediately apply to all Zombies that use this prefab.

![image-20220221133623403](.\image-20220221133623403.png)



Go to the "Zombie Walk" object.

In the Inspector, at the bottom, click "Add Component", then choose "New script".

Call the new script "ZombieRandomizeAttributes"

Double click on the script, and you'll see it in the Project Folder.  By default, this is created in the root Assets folder, which isn't a great place in terms of keeping things organizaed, so let's move it to Assets / FPS / Scripts.



Now, open the new script in your editor by double-clicking on it.

By default we'll get some code like this...

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class ZombieRandomizeAttributes : MonoBehaviour
{
    // Start is called before the first frame update
    void Start()
    {
        
    }
        
    // Update is called once per frame
    void Update()
    {
        
    }
}
```

Delete the `Update()` function, and modify `Start()` to this:

```
    void Start()
    {
        Animator animator = GetComponent<Animator>();
        animator.SetFloat("CycleOffset", Random.Range(0.0f, 1.0f));
    }
```



This code is fairly straightforward.  What it does is:

- create a local variable of the "Animator" type, and use it to get a reference to the "Animator" component on this object
  - (`GetComponent<T>()` is a "generic" method, which can take a type in angle brackets as a parameter)
- set the "CycleOffset" to a random number, from 0 to 1 (which will pick a random start-point i the walk cycle)

Once this script is complete, exit the Prefab Asset by clicking on the < in the top left, and choose to Save your changes.

![image-20220228105940312](.\image-20220228105940312.png)



Now, when you run the game, you should see Zombies that are de-synchronized from each other.

If the game doesn't run, check the console output for errors, and double check the code in the script is all typed in correctly.



### De-synchronizing speeds

To give the Zombies a bit more individuality, we can also give them each a custom speed.

To ensure that the zombies' footfalls match their speed, we need to apply a consistent adjustment to both the Zombie's animation speed (on the "Zombie Walk" entity), and the NavMeshAgent speed on the parent "Zombie" entity.

We can do this with a bit more code in `ZombieRandomizeAttributes`

At the top of the file, add the following line (just below the line that says `using UnityEngine`)

```
using UnityEngine.AI;
```



Then modify the `Start()` function to the following:

```
    void Start()
    {        
        Animator animator = GetComponent<Animator>();
        animator.SetFloat("CycleOffset", Random.Range(0.0f, 1.0f));

        float speedMultiplier = Random.Range(0.5f, 1.5f);
        animator.speed = speedMultiplier;
        GetComponentInParent<NavMeshAgent>().speed = 0.35f * speedMultiplier;
    }
```



This code will run on initialization of each Zombie. It:

- Picks a random speed multiplier between 0.5  and 1.5.
- Sets the animator speed to this speed
- Also sets the NavMeshAgent speed on the parent to 0.35 x the same value (remember that 0.35 was the speed that we set that matched an animation speed of 1)



If you play the game again, you'll see that the zombies now move at differing speeds - with the animations synchronized to their movement, so that their feet stay stable on the floor at any speed.



### Fix the Hit Boxes

It's common, for performance reasons, to use a simple geometry for an entity's hit box, rather than using the entity's mesh itself.

Unfortunately our zombies still have hit boxes left over from the Hover Bot which are completely wrong.

We'll fix this now...



Open the Zombie Prefab (right-click / Open Prefab Asset)

Navigate in the Hierarchy to find the entity called HitBox

In the Inspector, select "SphereCollider", and you'll see the current Hit Box for the Zombie - a sphere centered around it's chest.  This made sense for the HoverBot, but it's all wrong for the Zombie.

![image-20220228110331032](.\image-20220228110331032.png)



Sphere Colliders can't be stretched, so instead, remove it and add a Capsule Collider...

- Right Click / Remove Component on the Sphere Collider

- The Add Component, and choose "Capsule Collider"

  ![image-20220228110816296](.\image-20220228110816296.png)

I set height to 1.5, radius to 0.4, and I had to adjust the rotation so that the capsule was vertical (by default it was rotated by 90 degrees), and the position so that it was centered correctly on the Zombie.

![image-20220221145624985](.\image-20220221145624985.png)

Use this view controller to view the zombie from different directions (top, side, front, to confirm correct positioning)

![image-20220228110725232](.\image-20220228110725232.png)



It's not perfect, but it should be a lot better than the sphere.

Exit Prefab Mode (back button, top-left corner) and choose to Save the Prefab.

Now, replay the game, and you should be able to shoot the zombies in the ankles!



### Inflicting Damage

Another problem we had was that the Zombie can't hurt us, which makes things a bit one-sided.

The issue here is that the Hover Bot we started only had ranged attacks, and we disabled those.

So we want to make a change so that the Zombies inflict damage on our player character, when they get within a close range.

Again, we start by opening the Zombie Prefab (right click / Open Prefab Asset)

Now, under WeaponRoot

- Create a GameObject "AttackZone" (right click / Create Empty, then right click / Rename)
- On this GameObject, add Sphere Collider component and position at Zombie's hands (I used radius = 0.2)



![image-20220221153402637](.\image-20220221153402637.png)



We must also add a non-Kinematic RigidBody for collision detection to work...

- Add Component / RigidBody

However, we want to keep this in a fixed position relative to the Zombie, so on the RigidBody, freeze Position & Rotation - and set physics attributes Drag and Angular Drag to zero.

![image-20220221161916413](.\image-20220221161916413.png)



Now we need to add a script to handle damage... We create this in the same way as the last script.  Again, you should move it from the root folder where it is created, to the Scripts folder.

![image-20220228112049811](.\image-20220228112049811.png)



Here's the script to inflict damage...

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class ZombieAttackCollisionDetection : MonoBehaviour
{
    [Header("Damage")]
    [Tooltip("Damage of the zombie attack")]
    public float damage = 20f;

    void OnCollisionEnter(Collision collision)
    {        
        Collider hitCollider = collision.collider;

        // Zombies only damage the plyayer, not themselves or each other.        
        if (hitCollider.GetType() == typeof(CharacterController)) {
            Damageable damageable = hitCollider.GetComponent<Damageable>();
            if (damageable)
            {
                damageable.InflictDamage(damage, false, gameObject);
            }
        }
    }
}
```



A brief explanation of this script:

- It exposes a public variable that allows for control of the damage that the zombie does
- Whenever the sphereCollider collides with anything, the script:
  - Checks whether the object it collided with is the type of Game Object controlled by the player
  - If it is, it extracts the "Damageable" component from the player, and calls a function there to inflict the specified amount of damage on the player.

A few things to note here:

- This script means that zombies will only damage Game Objects of type "Character Controller".  We might want to change that in future if we wanted Zombies to be able to do damage to NPCs, or if we wanted Zombies to damage each other.
- By inflicting damage through the Damageable component, we ensure that any effects like shields, invulnerability etc. that might be in place on the player will be handled correctly - tis script doesn't need to worry about them - that will all be handled inside the Damageable component.
- The Damageable component will also take care of any other side-effects of damage - e.g. red flashes on the screen, death / game over etc.
- The OnCollisionEnter function is only triggered at the *start* of a collision with the Zombie's AttackZone sphere collider.  This means that each time you come close to a Zombie, it will only inflict damage once.  If you step away, and then it comes close again, it will inflict damage again.  If desired, the script could be modified so that sustained contact results in extra damage - e.g. once every 0.5 seconds...
  - One way to do this would be to add an OnCollisionExit() and a variable to track collision state...
  - ... and then an Update() function that fires every frame, which can check how long the collision state has persisted, and inflict additional damage as desired.



When you have entered, and saved the script above, if you go back to the Inspector, you'll see that a "Damage" parameter has appeared on the new script component.  You can use this to control the amount of damage that the Zombie does.

![image-20220221163539466](.\image-20220221163539466.png)

Although the script sets this to a default value of 20, any value that is specified in the inspector will override this default.



### SFX

In the top-level Zombie object, we can see it still refers to the Hoverbot audio.

![image-20220221150637112](.\image-20220221150637112.png)

​		

There's actually a total of four HoverBot audio clips, three that we care about...

- Alert (played when it detects our presence) - configured on the "Enemy Mobile" Script Component
- Moevement Sound (played as it moves) - also configured on the "Enemy Mobile" Script Component
- Death (played on death) - configured in "VFX_HoverBot_Explosion", which is itself configured on the "Enemy Controller" Component

and two we don't...

- Attack (played when it shoots lasers - the ones we disabled)

- There is also an "Audio Source" Component configured on the Zombie Game Object, but as far as I can tell this doesn't seem to have any effect...
  - (*I'd like to investigate this further*).

If you don't want to record your own zombie sounds,[Freesound.org](https://freesound.org/search/) is a great resource for royalty-free sound effects.

I found this pack of 4 Zombie sounds, which seem like a good fit for this project.

https://freesound.org/people/gneube/packs/17699/



To add these to the game, just download them, rename them with simpler names (I used "zombie-moan.wav" etc.), and drag them into Assets / FPS / Audio / SFX / Enemies /

![image-20220228114226421](.\image-20220228114226421.png)

Once you have these in your Project folder, you can just drag them in place of the various HoverBot sounds.  Remember that if you want to modify all Zombies, you should open the Zombie Prefab Asset first.

I replaced:

- Hoverbot--Alert with zombie-snarl (Zombie / Enemy Mobile / On Detect SFX)
- Hoverbot_Movement with zombie-moan (Zombie / Enemy Mobile / Movement Sound)

For the Death sound, I duplicated the VFX_HoverBot_Explosion, renamed to VFZX_Zombie_Explosion.

![image-20220228132134451](.\image-20220228132134451.png)



Then I updated Zombie Death VFX to point to the new VFX prefab, and updated the Audio Source Audio Clip from HoverBot_Death to zombie-roar.

Doing all this means we still have a usable set of HoverBot assets, in case we wanted to re-introduce the HoverBot to the game at some point.

### Exercises

If you've worked through the above, here are some further suggestions for exercises to test what you have learned.  These will all require some level of additional scripting.

- Extend the ZombieRandomizeAttributes so that it also randomizes the amount of damage a Zombie does, to a random value between 10 and 30.

- Modify ZombieAttackCollisionDetection so that every 0.5 seconds you stay in contact with a Zombie, it inflicts additional damage.

- Modify ZombieAttackCollisionDetection so that there is a Zombie sound effect when the Zombie does damage to the player.

  - Hint: take a look at the EnemyMobile script for an example of code that exposes a sound effect in the inspector, and plays it under certain conditions.

    

### What Next?

Apart from the exercises above, there's some more work we could do to tidy up our Zombie prefab.  It still has a few leftovers from the HoverBot which it would be useful to clean up.  Even if they aren't impacting function, it will make things simpler to maintain.

It would also be nice to add some more varieties of zombies to the game.

But it also might be time to move on from the zombies, and look at enhancements to the map, the player's weapons, or to player movement...



