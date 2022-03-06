# Lesson 3 - Weapon Systems GO!

After a couple of lessons working on Zombies, my students are asking for a greater variety of weapons to shoot them with...

### Accessing the Default Weapons

The FPS sample game actually comes with three different weapons, but a few tweaks are needed to provide access to them.

To enable the other two, open up the "Player" Game Object in the inspector, and open up the "Starting Weapons" sub-section of the "Player Weapons Manager" component.

![image-20220305124821166](.\image-20220305124821166.png)



Increase Size from 1 to 3, and you'll find you have 3 slots to put weapons into

Click on "Weapon_Blaster (WeaponController)" and in the Project view, you'll see the 3 available weapons.

You can drag the other 2 weapons into the 2nd & 3rd slots.

![image-20220305125026064](.\image-20220305125026064.png)



Alternatively, you can leave the starting weapons as they are, go to FPS / Prefabs / Pickups / in the Project view, and drag weapon pickup game objects into the scene.

![image-20220305125132101](.\image-20220305125132101.png)



There's a Jetpack pickup as well, if you want to try that out - we won't be looking at the Jetpack this lesson, though.



### Understanding the Blaster

The first weapon is the Blaster - we'll start by taking a look at this, and understanding how it works.  This will set us up for understanding the other weapons, and for creating new ones.



Open the Weapon_Blaster Prefab (in Prefabs / Weapons), and you'll see it's composed of a lot of Game Objects:

![image-20220305125631028](.\image-20220305125631028.png)



Actually, this isn't as complicated as it might look.

- All the code for the weapon is in components at the top-level (Weapon_Blaster), in 3 scripts - which we'll look at in a moment.
- The rest of the object tree defines the geometry of the gun, including the various moving parts (each moving part is a separate game object).



Let's start by looking at the structure of the gun itself:

![image-20220305154153790](.\image-20220305154153790.png)


The gun is composed of:

- The main body of the gun ("Primary_Weapon")
- The two front pieces of the gun (left and right) (Primary_Weapon_Front_Piece_01 and _02)
- The 4 moveable cylinders ("Primary_Weapon_Container_01, _02, _03 and _04")



If you look at these elements in the Project view (Models folder) (click through from the references to Meshes in the Inspector)...

![image-20220305155945772](.\image-20220305155945772.png)

...you'll see a view like this:
![image-20220305155459200](.\image-20220305155459200.png)

The grey outlined area, and small arrow ![image-20220305155530494](.\image-20220305155530494.png)show that these elements are all sub-elements of a single uploaded model (click the right-facing arrow![image-20220305155636541](.\image-20220305155636541.png) on the Zombie Walk FBX model and you'll see it has sub-elements as well)

The difference is that with the Zombie Walk FBX model, our Game Object referred directly to the top-level model.  With this gun, there are multiple GameObjects, and each refers to a distinct sub-element.

The key benefit of this is that it allows the parts of the model to be moved independently of each other - which is needed for the gun's cylinder animations.



So, in summary, the FBX model for the gun gets broken down into sub-element, and each element is instantiated as a separate Game Object, which allows for them to each be animated.



Beyond that, there's just a couple of other things worth highlighting in the hierarchy for the Primary Weapon:

- Gun Muzzle.  This consists of just a Transform - i.e. a specific point in space relative to the gun.  It's used as an anchor point for visual effects (gun muzzle flashing)
- Aim Point.  Another Transform.  This defines where the gun is pointing - it's used to determine the trajectory of projectiles.
- Various VFX prefabs
  - 4 x VFX prefabs for the cylinders - these generate the bubbles effect that you see in the cylinders after firing
  - 1 x VFX prefab for overheating - this generates the clouds of orange smoke you see coming out of the gun when it overheats.



### Understanding the Blaster Code

Now we've understood the structure of the gun Game Objects, let's look at the code that manages the behaviour of the gun.

There are 3 scripts:

- Weapon Controller (Script)
  - Most of the core logic for controlling the gun - we'll go into this in detail shortly.
- Weapon Fuel Cell Handler (Script)
  - Controls movement of "Fuel Cells" (the cylinders that rise out of the gun as you fire it)
  - This reads a single value `currentAmmoRatio` from the Weapon Controller, and animates the fuel cells based on that.
- Overheat Behaviour (Script)
  - Controls the overheat behaviour of the gun.
  - This reads a few variables from the Weapon Controller, and implements overheating behaviour based on them:
    - `currentAmmoRatio`
    - `isWeaponActive`
    - `isCooling`

So to get the Fuel Cell and Overheat behaviours to operate correctly, Weapn Controller just has to set these variables correctly, and these other scripts will take care of the rest.



Now let's look at the Weapon Controller script itself.

It has a lot of configuration - we won't cover every single field, but we'll go through most of it...

- Definitions of what crosshairs look like, and how they change when a target is in sight.

  ![image-20220305162052615](.\image-20220305162052615.png)

- Various parameters that control show shooting works, recoil etc.
  - Shoot type Automatic = continuous shooting, Manual = one shot per click, Charging = hold down click to build up power of shot).

![image-20220305162017980](.\image-20220305162017980.png)

- Settings for ammo and charging

  ![image-20220305162135300](.\image-20220305162135300.png)

- Configuration of various Audio & Visual Effects

![image-20220305162213436](.\image-20220305162213436.png)


In terms of the logic of the script itself, it consists of fairly straightforward implementation of all these settings.  To find the code that implements the logic of a particular config option, just search at the top of a file for the declaration of the corresponding public variable, and then search the module for references to this variable.

- Most of these variables are only referenced in 2 or 3 places, and the logic for any one of them is fairly simple.

![image-20220305162507013](.\image-20220305162507013.png)



A few concepts are worth explaining:

- `Awake()` is effectively the same as `Start()` that we saw before.  There are some subtle differences (e.g. `Awake()` is called before `Start()` and is called even if a Game Object is disabled), but not worth worrying about too much as a beginner.
  - Details here if needed: https://docs.unity3d.com/ScriptReference/MonoBehaviour.Awake.html
- All the weapons in the game reload Ammo automatically based on the configured "Ammo Reload Rate".
  - This means there is no logic to handle Ammo Pickups - we'll look at fixing this later!
- The Weapon Controller implements Manual, Automatic & Charging Weapons.  The Blaster is Automatic, but other guns use different Shoot Types. (the shotgun is Manual, and the Disc Launcher is a Charging Weapon)
- The Weapon Controller supports an "Animator" that is triggered (with an "Attack" action) when it is fired.
  - None of the weapon types in the game use this, but it could be useful if you wanted to have a sword or a club as a weapon.

#### Exercises - Blaster

- Modify the fuel cells so that they start positioned outwards, and move inwards as the ammo is used up.

![image-20220305184755469](.\image-20220305184755469.png)



- Change the blaster so that it shoots red lasers rather than green

![image-20220306085629468](.\image-20220306085629468.png)





### The Disc Launcher

Let's look briefly at the differences there are with the Disc Launcher.

- It has a different 3D model
- It only has 2 Fuel Cells, rather than 4.  Other than that, these are handled the same way.
- It uses a different projectile, a disc
  - We didn't look at Projectiles yet, but in "Projectile Standard (Script)" there is a parameter "Gravity Down Acceleration" which is set to 10, resulting in the projectiles being affected by gravity, and firing in an arc.
- It has a "Shoot Type" of "Charge" - we talked about this above: it builds up firepower while you hold down the mouse button, then fires on release.  The amount of ammo consumed depends on the charge time.
- Different images used for crosshairs, various other different visual and sound effects, and differently tuned values for various other parameters.
- It has one additional component on the top-level game object: "Charged Weapon Effects Handler (Script)"
  - This manages the rotation of the front part of the Disc Launcher
  - This uses the `currentCharge` variable on the Weapon Controller component to control the speed & radius of the rotation of the front disc section, and a sound effect which increases in pitch as the charge increases.

#### Exercises - Disc Launcher

- Reverse the rotation of the spinning part at the front of the Disc Launcher
  - (there are 2 parts to this - the outer case, and the orbiting particles)
- Modify the disk projectiles so that they generate pink sparks, instead of green triangles, as shrapnel, on impact.

### The Shotgun

Finally, let's take a look at the Shotgun...

- It uses the same 3D model as the Blaster for the base - but instead of the Front Pieces, it has three cylinders as barrels (2 large, one small below)

- 4 Fuel Cells, again the handling is much the same.

- Another different projectile, with a wider spread angle, and many bullets per shot.

- It has a "Shoot Type" of "Manual" - one click per shot.

- Different images used for crosshairs, various other different visual and sound effects, and differently tuned values for various other parameters.

- It doesn't have any Overheat Behaviour (Script)

  

#### Exercises - Shotgun

- Change the color of the fuel cells from blue to pink.
- Change the fuel cell



### Adding a New Weapon

We're now going to add a new weapon.

It's going to be a very simple pistol, but unlike all the existing weapons, it's not going to recharge ammo automatically, but only when we pick up ammo from within the scene.  To make that work, we're going to have to implement a new kind of Pickup, and write a bit of code.

We'll start by duplicating the Blaster in the Project Window Assets / FPS / Prefabs / Weapons /  - just select the Blaster, hit Ctrl-D, and rename to Weapon_Pistol.

![image-20220306093124861](.\image-20220306093124861.png)



Now, in the prefab editor, we can delete a lot of the copied weapon - delete all the PistonSockets, the Front Pieces, and the Overheat VFX...

![image-20220306093243391](.\image-20220306093243391.png)



So we'll end up with something like this:

![image-20220306093355446](.\image-20220306093355446.png)

Now, on the top-level Weapoon_Pistol Game Object, delete the Fuel Cell Handler & Overheat Behavior Scripts (right click / "Remove Component")

![image-20220306093438436](.\image-20220306093438436.png)



Save the Prefab, and (for easy testing), make sure this weapon is set as a Starting Weapon in the Player's "Player Weapons Manager (Script)" component.

![image-20220306093624382](.\image-20220306093624382.png)



If we playtest now, we have a simplified gun, with none of the fancy fuel cells, and no overheat effects (though it still need time to recharge ammo when it runs out).

![image-20220306093917494](.\image-20220306093917494.png)



### Out of Ammo!

Our next change is to get rid of the auto-recharge of ammo.

A very simple change - set Ammo Reload Rate to 0.  And while we're at it, why not limit the gun to 6 shots?

![image-20220306094210395](.\image-20220306094210395.png)

(remember to save the Prefab before testing)

We now have a gun with 6 shots, and no way to reload it.  That definitely is titling things in the Zombies' favour!

Time to add some Ammo Pickups...

If we look in Prefabs / Pickups, we'll see a few pickups that we can use as a base to build from.

![image-20220306094707888](.\image-20220306094707888.png)

Let's look at Pickup_Health.  On the top-level Game Object, we have the following:

![image-20220306094753613](.\image-20220306094753613.png)



What we have here is:

- A Transform, which controls the pickup position
- A Box Collider & Rigid Body - these ensure we can do collision detection on the pickup
- Pickup (Script) - this handles the generic aspects of pickups, co-ordinating sound & visual effects, and bobbing & rotation of the pickup.
- Health Pickup (Script) - this handles the specific effects of the pickup - for the Health Pickup, that's increasing Player Health by a set amount.  For our Ammo Pickup, it will be boosting our Ammo.

Below the top-level object, there's a few other Game Objects

- Some Visual Effects that make the Health pickup glow, and emit a stream of upwards particles
- The Mesh for the Health Pickup, which specifies what it looks like (it's shape and colour)

![image-20220306094858853](.\image-20220306094858853.png)



### Creating an Ammo Pickup

In the Project window, Prefabs folder, select Pickup_Health, duplicate it (Ctrl-D), rename to Pickup_Ammo, and open up the Prefab.

We need to make it look different from the Health Pickup.  Here's a suggested set of  changes:

- Mesh -> Cylinder

- Materials / Element0 -> Floor_DarkGrey

- Set rotation & scale like this:

  ![image-20220306100823498](.\image-20220306100823498.png)

- Finally, it seems to be too high up, so modify Transform / Y on VFX_Health to 0.

Also, rename the game objects in the prefab to reflect the fact that they are part of an Ammo pickup, not a Health Pickup

![image-20220306101026849](.\image-20220306101026849.png)

Now drag an instance of the Prefab into the scene, and test it out.  It should look something like this.  Not beautiful, but we're focussing on functionality for now - we can fix up the looks later.  At least it looks different from the Health Pickup.

![image-20220306101127983](.\image-20220306101127983.png)



Looking better - now we need to update the scripts so that this pickup controls our Ammo levels, not our Health.



### Updating the Script

The HealthPickup script is a good place to start from.

In the scripts folder, duplicate the Script, and rename the copy to AmmoScript.

Edit this line in the new sscript:

````
public class HealthPickup : MonoBehaviour
````

to this:

```
public class AmmoPickup : MonoBehaviour
```

Now, remove the "HealthPickup (Script)" component from the Pickup_Ammo Game Object, and add an "AmmoPickup (Script)" component in its place (e.g. by dragging the new script onto the Game Object in the Inspector).



To be able to add Ammo, we need to make some changes to the Weapon Controller Script.

Open the Weapon Controller script, and add the following function.

This could go anywhere in the file, but just before `UpdateAmmo()` is a good place.

```
    public void AddAmmo(float ammoIncrease) {
        
        m_CurrentAmmo += ammoIncrease;
        m_CurrentAmmo = Mathf.Clamp(m_CurrentAmmo, 0, maxAmmo);
        
        if (maxAmmo == Mathf.Infinity)
        {
            currentAmmoRatio = 1f;
        }
        else
        {
            currentAmmoRatio = m_CurrentAmmo / maxAmmo;
        }
    }
```



This script exposes a new *public* function (i.e. one that can be called from outside the component).  The function

- Adds a specified amount of ammo to the weapon's ammo level
- Makes sure that ammo level does not exceed the weapon's max ammo level
- Updates the currentAmmoRatio variable to the correct value.

The code for the 2nd and 3rd parts of this function are copied from the pre-existing  `UpdateAmmo()` function (which handles automatic ammo recharging).



Now, we need to set up the AmmoPickup Script to invoke this.

Modify the script to match the following:

```
using UnityEngine;

    public class AmmoPickup : MonoBehaviour
{
    [Header("Parameters")]
    [Tooltip("Amount of ammo to add on pickup")]
    public float ammoAmount = 6.0f;

    Pickup m_Pickup;

    void Start()
    {
        m_Pickup = GetComponent<Pickup>();
        DebugUtility.HandleErrorIfNullGetComponent<Pickup, AmmoPickup>(m_Pickup, this, gameObject);

        // Subscribe to pickup action
        m_Pickup.onPick += OnPicked;
    }

    void OnPicked(PlayerCharacterController player)
    {
        PlayerWeaponsManager playerWeaponsManager = player.GetComponent<PlayerWeaponsManager>();
        WeaponController weaponController = playerWeaponsManager.GetActiveWeapon();
        
        weaponController.AddAmmo(ammoAmount);
        
        m_Pickup.PlayPickupFeedback();

        Destroy(gameObject);
    }
}
```

If you compare this to the Health Pickup script, you'll see that it's pretty similar.  The key differences are:

- We've changed the name & default value of the parameter exposed in the Inspector, to set the amount of ammo available in the pickup
  - The default value is set to "6.0f" (the f is a bit of C# syntax that's needed when you specify a floating point number).
- Rather than modifying the player's health, we get hold of the player's active weapon (via the player's Weapons Manager), and add Ammo to it, using the new addAmmo function.

Before you test this, check that the "Ammo Amount" on your pickup is set to a suitable value (it should be set to 6 by default, but depending on the order you do things in, it might have ended up set to 0).

We should now have a functioning basic ammo pickup.



### Improving the Pickup

There's a couple of things we might want to improve about how the Ammo Pickup works.

- It always adds the ammo to the currently weapon.  That's not ideal.  We'd like ammo to be of a specific type, and reload the ammo on a specific weapon (even if it's not the active one)
- Its always picked up, even if your ammo was already full
- There's a particle effect on pickup left over from the Health Pickup, where there's a set of "+" objects that float upwards on pickup - we should remove this (or create a more suitable visual effect).

These can all be fixed, with a little more work.



### Reload Ammo for the Correct Weapon

To do this, we need to:

- Specify in the Ammo Pickup which weapon it's intended for
- Get old of this weapon, rather than the active weapon, from the PlayerWeaponsManager.  This is going to require some minor changes to the PlayerWeaponsManager script.



We'll make the changes to PlayerWeaponsManager first.  Find this existing function in PlayerWeaponsManager...

```
    public bool HasWeapon(WeaponController weaponPrefab)
    {
        // Checks if we already have a weapon coming from the specified prefab
        foreach(var w in m_WeaponSlots)
        {
            if(w != null && w.sourcePrefab == weaponPrefab.gameObject)
            {
                return true;
            }
        }

        return false;
    }
```



... and replace it with this:

```
    public bool HasWeapon(WeaponController weaponPrefab)
    {
        return (HeldWeapon(weaponPrefab) != null);        
    }

    public WeaponController HeldWeapon(WeaponController weaponPrefab)
    {
        // Checks if we already have a weapon coming from the specified prefab
        foreach(var w in m_WeaponSlots)
        {
            if(w != null && w.sourcePrefab == weaponPrefab.gameObject)
            {
                return w;
            }
        }

        return null;
    }
```

This gives us a new public function "HeldWeapon", which will provide access to a weapon if the player has it (and will return null otherwise).

To avoid unecessary code duplication, we have refactored the pre-existing HasWeapon function to use ReturnWeapon.



Now we have what we need to make the changes to AmmoPickup

We'll start by adding a new parameter at the top of the script:

```
    [Tooltip("Weapon this ammo is usable for")]
    public WeaponController ammoWeapon;
```

After saving the script, we can set Ammo Weapon for our Ammo Pickup to "Weapon_Pistol" in the inspector.

We update the OnPicked function as follows:

```
void OnPicked(PlayerCharacterController player)
    {
        PlayerWeaponsManager playerWeaponsManager = player.GetComponent<PlayerWeaponsManager>();
        WeaponController weaponController = playerWeaponsManager.HeldWeapon(ammoWeapon);

        if (weaponController) {
            weaponController.AddAmmo(ammoAmount);
        
            m_Pickup.PlayPickupFeedback();

            Destroy(gameObject);
        }
    }
```

This tries to get a specific type of Weapon that may be held by the player.  If the player has the weapon, the Ammo is picked up.  If not, the Ammo pickup is left in place.

Test this out by:

- Starting with the Pistol, and checking you can pick up the ammo.
- Starting without the Pistol (e.g. set starting weapon to Blaster), and check you don't pick up the ammo.
- (you could also test out scenarios with 2 / 3 / 4 weapons, but if the above two test cases work, the rest ought to work also)

### Don't Pick Up Unneeded Ammo

Next fix is to leave the pickup if we are already at full ammo.

Having made the changes above, this is really quite simple.

Recall that Weapon Controller exposes a public variable`currentAmmoRatio` (we saw this beign used by WeaponFuelCellHandler to control fuel cell positions).

So we just need to change this line above:

```
        if (weaponController) {
```



To this...

```
        if (weaponController && weaponController.currentAmmoRatio < 1) {
```



This is easy to test:

- Start the game with the Pistol, and try to pick up the Ammo without having fired a shot.   You won't pick it up.
- Then fire a shot and confirm that you can pick it up.



If you preferred to change the Ammo Pickup so you could only pick it up with < 75% Ammo (say), you could update the line to:

```
        if (weaponController && weaponController.currentAmmoRatio < 0.75) {
```



### Fix Particles on Pickup

The "+" particles that float upwards on pickup are specified as parameters on the Pickup script.

We can switch the SFX and VFX that occur on pickup to more suitable values for this pickup, - for example these:

![image-20220306134845921](.\image-20220306134845921.png)



### Balancing the Game

OK - the Ammo Pickup is now ready to go.

Now, we need to get the game balanced

- We need enough Ammo drops to make it possible to finish off all the Enemies
- But not so many, to make it easy.
- You might want to adjust various parameters to get the game correctly balanced, e.g.
  - The Ammo capacity of the gun (6 bullets isn't a lot!)
  - The damage that a single bullet does, and/or the HP that a Zombie has
  - How much additional Ammo you get from an Ammo pickup.

How you set these values will be down to the kind of game experience you want to create, and the level of challenge you want to set.  Have fun experimenting with this!

### Further Exercises

I've left the following as exercises for the learner...

- Improve the graphics for the Pistol and the Ammo Pickup.  We already saw how to import Models and set up Textures and Materials in lesson 1.  The same techniques can be used to import new models for the Pistol and the Ammo Pickup.

- If we only give the player the Pistol, we've created a "soft lock" situation, where they can run out of the ammo they need to complete the level.  How could you avoid that?  Can you implement the code for a solution?

- Although we've called our new gun a "Pistol", it is still shooting lasers.  Can you modify the projectiles that it shoots, so they look more like bullets?

  

### What Next?

Some ideas for where we could go next with this

- As mentioned in Lesson 2, there's still some clean-up work to do on the Zombie prefab
- We'd like more Zombies, and an extended map
- We also want to add more flexibility to player movement.

We'll look at one of these in the next lesson...



### Inline Exercises: Answers

Answers to the exercises that I included inline above, in case you get stuck.  Please don't check these until you have made a serious attempt to solve the problems by yourself, or you will miss out on learning....

#### Blaster

Modify Fuel Cell Movement (on Weapon Fuel Cell Handler)

![image-20220306083053222](.\image-20220306083053222.png)

Red lasers (on Mesh_Projectile, in the Projectile_Blaster prefab )

![image-20220306085434243](.\image-20220306085434243.png)



#### Disc Launcher

Reverse rotation (container) (in Charged Weapon Effects Handler)

![image-20220306082948184](.\image-20220306082948184.png)

Reverse rotation (orbiting particles) (also in Charged Weapon Effects Handler)

![image-20220306084937623](.\image-20220306084937623.png)

Pink sparks on impact (beneath VFX_DiskRange)

![image-20220306090018955](.\image-20220306090018955.png)



#### Shotgun

Fuel Cell Color
- Option 1: Duplicate the WeaponShotgun material, rename to WeaponShotgun Pink, change color, and update references to this material in each cylinder
![image-20220306091731487](.\image-20220306091731487.png)

- Option 2: Just modify the WeaponShotgun material Emission value directly from Blue to Pink
![image-20220306091936517](.\image-20220306091936517.png)



Fuel Cell Color on Blaster

- Because the Blaster uses the Overheat Behaviour script, the base color of the cylinders is overwritten by the colors from the Overheat Gradient.  Modify these to control the color of the Blaster's Fuel Cells.

![image-20220306092308901](.\image-20220306092308901.png)



#### Further Exercises

I haven't provided any answers for the "Further Exercises" - those are left for you to figure out for yourself.


