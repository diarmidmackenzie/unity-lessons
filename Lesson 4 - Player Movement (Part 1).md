# Lesson 4 - Player Movement (Part 1)

In this lesson, we're going to look at player movement controls.

After understanding the basics of how the controls work, we're going to make three enhancements to the game's control system.

- Sliding
- Rolling
- Wall-running.

This first part is going to cover an overview of the movement controls, and some updates for sliding.  Part 2 (next lesson) will tackle Rolling and Wall-running.

### Existing Movement Controls

The existing movement controls for the FPS sample game are nicely summarised in this menu screen, which you can access in-game.

![image-20220319115632089](.\image-20220319115632089.png)



Some key points to note:

- The game supports both keyboard/mouse and gamepad controls
- The main play can move, sprint, jump and crouch.  They can also fly using a jetpack (the jetpack pickup isn't in the game by default, but you can easily add it)
- There's an in-game controls display, so if we're going to update the controls, we should also update that!

### How do the Controls Work?

There's a few layers to the control system - we'll look at them bottom-up.

The first script to look at is PlayerCharacterController script on the Player GameObject.  There's about 400 lines of code here, so a fair bit of complexity - we'll come back and look at this later.

But there are other components that are important too:

- The "Character Controller" component on the Player GameObject. 

  - This is a unity built-in component, rather than a script. It is an alternative to a RigidBody, that is specifically optimized for character control, full details here: https://docs.unity3d.com/ScriptReference/CharacterController.html
  - You'll see this has some controls like "step offset" (which defines how high a step the character can climb) and "slope limit" (which defines how steep a slope the character can stand on without sliding off).

- The PlayerInputHandler script.

  - This script exists to handle input from *either* the gamepad or the keyboard & mouse.  It provides a set of functions to PlayerCharacterController that abstract away the details of which control system is being used.

- The GameConstants script.

  - If you look at PlayerInputHandler, you'll see that it doesn't directly reference controls.  Instead, it refers to things like `GameConstants.k_ButtonNameSprint`
  - These are defined in GameConstants... e.g. `public const string k_ButtonNameSprint          = "Sprint";`
  - But even this doesn't actually define the keys or buttons themselves...

- ... these are actually defined under Edit / Project Settings / Input Setting.

  ![image-20220319123016501](.\image-20220319123016501.png)

  - If you look closely here, you'll see that there are actually two entries for controls such as "Sprint", one for the keyboard control, and one for the gamepad control.
  - The code in PlayerInput Handler can access the state of either/both buttons through a single call to Input.GetButton("Sprint")
    - The Input interface documentation can be found here: https://docs.unity3d.com/ScriptReference/Input.html

- Finally, the in-game documentation for the controls consists of a PNG image at Assets/FPS/Art/Textures/UI/ControlScheme_PopUp.png

  - This will need to be manually updated to match any changes made to the control scheme (using image editing software such as [GIMP](https://www.gimp.org/))



### Digging Deeper - PlayerCharacterController

PlayerCharacterController has 400 lines of code.  Rather than provide commentary on all of them, I'll highlight here some key observations about the code that will be relevant as we add to it.

There are a few keys bits of state that are used through the function:

- isCrouching
- isGrounded (is the player on the ground, as opposed to being in the air)
- isSprinting

Note that because isCrouching and isGrounded are defined at the top-level of the class, they can be set or read within any function within the class.

Some important functions are the following:

- GroundCheck() - This checks whether the player is in contact with the ground, and sets "isGrounded" based on this,

- SetCrouchingState() - This sets the "isCrouching" variable based on the player's controls, but also checking available headroom in the case where a player is trying to stand up.

  - This also triggers the onStanceChanged Unity Action, which allows the HUD to keep the visual indication of the player's stance up--to-date.

    ![image-20220319130841698](.\image-20220319130841698.png)

- HandleCharacterMovement() - This function does the bulk of the work in updating the character's  position in 4 phases:
  1. Update the player's rotation, considering both horizontal and vertical rotation
  2. use the "isGrounded", "isCrouching" and "isSprinting" state, plus controller state to determine a specific 3D vector to move the character by.
  3. apply this movement
  4. watch out for any resulting collisions, and adjust characterVelocity for the next frame to account for collisions (so that e.g. the player can't run through walls)
- Update() - This function is the top-level function called every frame.  It's processing is fairly simple:
  1. check if the player has fallen too far into the void ("Y kill")
  2. update the player's grounded state, and apply any fall damage
  3. update the player's crouching state
  4. invoke HandleCharacterMovement() to take care of the movement this frame.



### Sliding

Before writing any code, we need to have a clear plan for how our sliding behaviour is going to work.  What are the controls?  How does sliding relate to crouching, sprinting and jumping?

Let's plan on the following:

- The control for sliding is to crouch while in mid-air.  On landing, the player goes into a slide.
- The player then slides for up to a second, until they lift the "crouch" button up.
- The player cannot steer while sliding - they continue to move in a fixed direction (unless they hit obstacles etc.)
- The player can jump while sliding - but when they do so, they they stop sliding.
- When sliding, the player's height is lower than when crouching.  Also, the player's camera view is tilted to the side, providing an additional visual cue that they are sliding.

To track this new state, we'll introduce a new variable "isSliding".

We need to clarify:

- When to set "isSliding" to true
- When to set "isSliding" to false
- What's different about how we handle character movement when "isSliding" is set?



Let's figure each of these in turn, and make some code updates....

**When to set "isSliding" to true?**

We set "isSliding" to true when the player lands, and "isCrouching" is set to true.

just after this line....

```
public bool isCrouching { get; private set; }
```

...add this line:...

```
public bool isSliding { get; private set; }
```



And just after these lines....

```
        // landing
        if (isGrounded && !wasGrounded)
        {
```

... add these lines...

```
            // if crouching, go into a slide...
            if (isCrouching) {
                isSliding = true;
            }
```



**When to set "isSliding" to false?**

We'll stop sliding in several circumstances

1. If the player stops crouching
2. If the player jumps
3. After the player has been sliding for a second.



Let's make some code updates to implement this...

1. In this code...

```
        // crouching
        if (m_InputHandler.GetCrouchInputDown())
        {
            SetCrouchingState(!isCrouching, false);        
        }
```



Add this additional line inside the braces (just after "SetCrouchingState")

```
            isSliding = false
```

This will ensure that whenever crouch or uncrouch, we'll strop sliding.



2. Let's leave this for now - we'll see why in a minute...



3. Just after this line...

```
       public bool isSliding { get; private set; }
```

   ... add this line....

```
       private float slideStartTime = 0;
```

   At the beginning of Update(), add these lines...   

```

        if (Time.time > slideStartTime + 1) {
            isSliding = false;
        }
```

... and after this line...

```
                isSliding = true;
```

... add this line...

```
                slideStartTime = Time.time;
```

The above code introduces a new private variable "slideStartTime" to track the time at which we started sliding.  Every time Update() is called, we check whether it's over a second since we started sliding.  If it is, we stop sliding.



**Different Character Movement when "isSliding" is set**

There's actually a really simple change we can make, which gives us a pretty good start in terms of sliding behaviour...

Take a look at the section marked `//character movement handling` in `HandleCharacterMovement()`

It splits the movement handling into two cases: grounded movement, and aerial movement.

In grounded movement, the player has full control of their direction, there are footstep sounds etc.

In aerial movement, the player has limited control over their movement (but still has some control, due to the "Acceleration Speed in Air" setting on the Player Character Controller - set this to 0 if you want to disable player movement in the air & when sliding...)

![image-20220320130947944](.\image-20220320130947944.png)



So all we need to do is update this code...

```
            // handle grounded movement
            if (isGrounded)
            {
```

... to this...

```// handle grounded movement
            // handle grounded movement
            if (isGrounded && !isSliding)
            {
```

.. and we'll have a basic "sliding" behaviour for our player character...

### Testing out Sliding

There's a large open area to the left of the second room, which is great for testing out sliding...

If we test out with the changes above, we should see:

- If you press crouch just before landing, the player character "slides" forward for about a second.

However there are still some issues.

1. The gun continues to wobble as if they are walking.
2. The player is still at crouching height - we wanted our sliding player to be lower.
3. We don't have the camera tilt that we suggested above.
4. The player can't jump while sliding.

Let's work through these issues one at a time...

#### The gun continues to wobble as if they are walking

This might be a bit of a surprise, as the gun doesn't wobble when the player is in the air.

Didn't we set the code up to treat sliding movement like aerial movement?

The issue here is that the weapon bobbing is implemented in a different component.

Take a look at `UpdateWeaponBob` in `PlayerWeaponManager`

We see this code:

```
// calculate a smoothed weapon bob amount based on how close to our max grounded movement velocity we are
            float characterMovementFactor = 0f;
            if (m_PlayerCharacterController.isGrounded)
            {
                characterMovementFactor = Mathf.Clamp01(playerCharacterVelocity.magnitude / (m_PlayerCharacterController.maxSpeedOnGround * m_PlayerCharacterController.sprintSpeedModifier));
            }
```

This is enabling a weapon bob as long as the player is grounded.  Fixing this so that the weapon doesn't bob when sliding is just a matter of updating this line...

```
            if (m_PlayerCharacterController.isGrounded)
            
```

...to this...

```
            if (m_PlayerCharacterController.isGrounded  &&
                !m_PlayerCharacterController.isSliding)
```

Note above that when we defined `isSliding`, we defined it as `public`.  That's what means it can be accessed directly in this way from outside the module where it is defined.  Since we defined `slideStartTime` as `private`, it wouldn't be accessible in the same way.

#### The player is still at crouching height - we wanted our sliding player to be even lower

There are a couple of parameters exposed by `PlayerCharacterController`, "Capsule Height Standing" and "Capsule Height Crouching"

#### ![image-20220320132445474](.\image-20220320132445474.png)

Let's add another...

Below this code...

```
    [Tooltip("Height of character when standing")]
    public float capsuleHeightStanding = 1.8f;
    [Tooltip("Height of character when crouching")]
    public float capsuleHeightCrouching = 0.9f;
```

...add these extra lines....

```
    [Tooltip("Height of character when sliding")]
    public float capsuleHeightSliding = 0.7f;
```



Now, when we set...

```
isSliding = true;
```

add this line...

```
m_TargetCharacterHeight = capsuleHeightSliding;
```



and when we set...

```
isSliding = false;
```

add this line...

```
m_TargetCharacterHeight = capsuleHeightCrouching;
```

This gives a slight additional reduction to the player's height while sliding vs. crouching.

It might be tempting to reduce this further, to say 0.4 or 0.5 (40cm or 50cm).  Unfortunately, if we do this we hit problems.  The player is modelled as a capsule with a radius of 0.35 (see the "Character Controller" component on the Player GameObject).  If you reduce the height of the capsule below twice the radius, this creates a bunch of problems for the physics engine (you'll see the player jerk around and take damage).

There are various ways this could potentially be fixed, but for now it's simplest just to leave the sliding height at 0.7.



#### We don't have the camera tilt that we suggested above

In our initial proposal for sliding, we'd proposed a tilt on the player's camera view while sliding, to provide an additional visual cue.

This is actually pretty simple to do.

Just after the place where we declared...

```
private float slideStartTime;
```

... add another variable...

```
private float slideAngle = 0;
```

Then, in `HandleCharacterMovement()` find this line of code...

```
            playerCamera.transform.localEulerAngles = new Vector3(m_CameraVerticalAngle, 0, 0);
            
```

... and update to this...

```playerCamera.transform.localEulerAngles = new Vector3(m_CameraVerticalAngle, 0, 0);
            float targetAngle = isSliding ? -30f : 0f;            
            slideAngle = Mathf.Lerp(slideAngle, targetAngle, 15 * Time.deltaTime);            
            playerCamera.transform.localEulerAngles = new Vector3(m_CameraVerticalAngle, 0, slideAngle);
```



To briefly explain these changes...

- We've added a new variable at the scope of the class, `slideAngle` so that we can maintain state over time about the angle we're at.

- Each frame, we set a target angle of either -30 or 0, depending on whether we are sliding.

  - `targetAngle = isSliding ? A : B` is a convenient one-line shorthand in C# for this...

```
    if (isSliding) {
        targetAngle = A;
    }
    else {
        targetAngle = B;
    }
```

- We use a Math function called "Lerp", which we use to linearly interpolate between the current value and the target value, based on the time passed since the last frame.

  - The Lerp function is explained here: https://docs.unity3d.com/ScriptReference/Mathf.Lerp.html
  - This is a common way to performan smooth transiitons, for example it's used in the existing code in `UpdateCharacterHeight` to handle smooth transitions between crouching and standing.
  - The value of 15 used in the 3rd parameter could be tuned to make the transition faster or slower, as desired.

  

#### The player can't jump while sliding

This is a consequence of the fact that we treat sliding the same as aerial movement, and the existing code doesn't allow the player to jump when they are in the air (no double jumping).

If we want to be able to jump while sliding, we need to add some jump logic into the  aerial branch.



The simplest change is to just take this section of code...

```
				// jumping
                if (isGrounded && m_InputHandler.GetJumpInputDown())
                {
                ... <approx 16 lines of code>
                }
```

... and move it out of the if branch that it's in to the next level.

So copy those ~20 lines of code, and paste them in just before this comment:

```
// apply the final calculated velocity value as a character movement
```

Change the line indenting to match the new position (in most editors you can reduce the indent of a group of lines by highlighting all lines, and pressing `Ctrl-[`).

In its new position, this code will do almost exactly what we want.  It wont' allow for double jumping, because it expliitly checks the `isGrounded` condition.

Just one small addition is to add this line of code at the end, so that jumping cancels out any sliding that's in progress.

```
                // end any sliding in progress
                isSliding = false;
```



### More Testing

We can now test out the above changes (or maybe you tested them one-at-a-time already?)

Things are looking pretty good now, but there's still some room for improvement.

Some things that I noticed.

- If you sprint (hold shift), you can't go into a slide.  This is because sprinting disables crouching, and we only initiate sliding if we are crouching when we land.  One way to fix this might be to adjust this bit of code.

```
            if (isSprinting)
            {
                isSprinting = SetCrouchingState(false, false);
            }
```

- Have we got sliding correct for slopes?

  - There's already a slope in the game, which you can test, and you could easily add a wider variety of slopes to test with.
  - If you leave "Acceleration Speed in the Air" set to a non-zero value, then holding down W will allow you to slide up slopes pretty fast in a way that feels highly unrealistic.  If you set this value to zero, the handling of sliding on slopes feels better, but I'm not sure it's completely correct yet.
  - Actually, the approach we took of using the aerial controls has worked out pretty well.  If you slide down a slope, you accelerate over time, and if you try to slide up a slope, you actually slide backwards down it, which is quite a nice result.
  - But I'm not sure about the duration of the slide.   Currently we always slide for 1 second.  But ideally how long we slide for would depend on the terrain - if we start sliding down a slope, maybe we should keep sliding, until we reach the bottom?
  
  

### Exercises

Based on the above, here's two suggested exercises to work on:

1. Fix the interaction between sprinting & sliding, so that you can go into a slide when landing, even when sprinting.

2. Add independent settings for "Acceleration Speed in the Air" and "Acceleration Speed while Sliding".

3. Devise a solution that allows a player to slide down a long slope until they reach the bottom.

4. Test & refine the realism of movement and acceleration when sliding on slopes.

5. Add a new "stance" icon that indicates that the player is sliding (rather than standing or crouching)

   ![image-20220320153515216](image-20220320153515216.png)



Hints:

- For sprinting and sliding, there's a very simple fix (can you see why this works?)

  `if (isSprinting)` -> `if (isSprinting  && isGrounded)`

- For sliding on slopes, it could be helpful to build a range of different slopes of different lengths & gradients for testing.  Try sliding up, down and across the slopes.

- `characterVelocity` is a `Vector3` that contains the current velocity of the player's character.

  - You can get the magnitude of the vector (i.e. the player's current speed) via `characterVelocity.magnitude`
  - You could multiply the velocity every frame by a number slightly smaller than 1, so simulate the effects of friction in slowing down sliding.
  - More documentation on Unity `Vector3`s here: https://docs.unity3d.com/ScriptReference/Vector3.html

- you can also determine the slope of the surface that the character is standing on by looking at the "normal" from the collision of the player with the ground.  The "normal" of a surface is a vector that is perpendicular to that surface (so the normal of flat ground is directly up).  Take a look at this code from `GroundCheck()`which uses these normals to determine whether a collision should be considered a collision with the floor (rather than e.g. a wall).

  ```
                  // storing the upward direction for the surface found
                  m_GroundNormal = hit.normal;
  
                  // Only consider this a valid ground hit if the ground normal goes in the same direction as the character up
                  // and if the slope angle is lower than the character controller's limit
                  if (Vector3.Dot(hit.normal, transform.up) > 0f &&
                      IsNormalUnderSlopeLimit(m_GroundNormal))
  ```

  

- 

