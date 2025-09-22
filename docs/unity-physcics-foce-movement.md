# Force-Based Movement

While velocity-based movement is simple and effective, it has a drawback: directly setting the Rigidbody's velocity **overrides parts of Unity’s built-in physics system**. This can make objects feel less natural because collisions, momentum, and acceleration are bypassed.

When you want to simulate more **realistic physics-driven movement**, it’s recommended to use **forces** instead. Moving an object with force means Unity’s physics engine handles the **acceleration, deceleration, and interactions** with other objects. This gives objects a sense of **weight and inertia**, making gameplay feel more immersive.

Force-based movement is especially useful when:
- Objects accelerate or brake gradually.
- Collisions and impacts should realistically influence motion.
- Simulating vehicles, projectiles, or other physics-heavy interactions.


## Understanding Force

In Unity, **force-based movement** moves objects by applying forces to a Rigidbody instead of directly setting velocity. The physics behind this comes from **Newton’s second law**:

$$
F = m \cdot a
$$

Where:
- **$$F$$** = force applied (in Newtons)  
- **$$m$$** = mass of the object (kg)  
- **$$a$$** = acceleration (m/s²)

> [!NOTE]
> In Unity, mass is set on the Rigidbody in kilograms (kg), velocity is measured in meters per second (m/s), and acceleration is measured in meters per second squared (m/s²).

Let's say, for example, our object has a mass of 2kg and has a force of 10 N applied to the right:

$$
a = \frac{F}{m} = \frac{10 \, \text{N}}{2 \, \text{kg}} = 5 \, \text{m/s²}
$$

This means the object’s **velocity** increases by **5 meters per second** each second in the direction of the force; assuming **no other forces act on it**.

### Unity Physics Steps

Unity updates Rigidbody physics in `FixedUpdate`, not every frame. The physics time step is controlled by `FixedDeltaTime` (default: 0.02 s, or 50 physics steps per second).

Velocity is updated per physics step:

$$
v_{\text{new}} = v_{\text{current}} + a \cdot \text{FixedDeltaTime}
$$

Distance moved per step:

$$
d = v_{\text{current}} \cdot \text{FixedDeltaTime} + \frac{1}{2} a \cdot \text{FixedDeltaTime}^2
$$

Because `FixedDeltaTime` is much smaller than 1 second, velocity and distance increase gradually, not instantly. Other forces like **gravity**, **drag**, and **friction** also affect the object’s motion.



### When to use Force over Velocity 

Let's say we are creating a **racing game**, where vehicles gradually accelerate and brake rather than instantly reaching top speed. In this case, we want the movement to feel **realistic and physics-driven**, rather than simply starting and stopping abruptly.

Similarly, objects that roll or slide, like a ball down a slope or a crate pushed across the floor, respond naturally to collisions and other forces, rather than abruptly changing direction. Even characters can benefit from this approach, **feeling _“weighty”_** and obeying momentum, which gives movement a more lifelike and immersive quality. Unlike velocity-based movement, **force-based movement allows external forces** like explosions, wind, or pushes from other objects to influence motion in a consistent and believable way.

Implementing movement with **force** has both advantages and disadvantages: 

- ✅ Produces **natural, fluid** acceleration and deceleration.
- ✅ **Fully integrates with Unity physics** (collisions, mass, momentum).
- ✅ **Allows external forces** (wind, explosions, pushes) to influence movement.
- ❌ **Requires careful tuning** of mass, acceleration, and drag to feel right.
- ❌ **Slightly more complex** to implement and debug.

## Applying force in Unity

In Unity, the `Rigidbody` component offers multiple ways to **apply force** and influence an object's movement: 

- `Rigidbody.AddForce()` - Applies a linear force in **world space**; changing the object's velocity.
  - The ForceMode can be set to **Force**, **Acceleration**, **Impulse**, or **VelocityChange** depending on how you want the force to affect the object.
- `Rigidbody.AddTorque()` - Applies a **rotational force (torque)**, causing it to spin around its center of mass.
- `Rigidbody.AddExplosionForce() - **Simulates an explosion** by applying a force that **radiates outward** from a specific point, affecting nearby objects.
- `Rigidbody.AddRelativeForce()` - Applies a linear force **relative to the Rigidbody’s local axes**.
- `Rigidbody.AddRelativeTorque()` - Applies torque **relative to the Rigidbody’s local axes**.

Each of these methods has its own specific use cases, but for this lesson, we will keep things simple and focus only on using `Rigidbody.AddForce`.

### `AddForce()` Method

The `Rigidbody.AddForce()` method applies a force to a Rigidbody, causing it to accelerate in the direction of that force. Unlike directly setting the velocity, this allows Unity’s physics system to determine how the object actually moves, taking into account its mass, drag, and any other forces acting on it (like gravity or collisions).

The method takes two parameters:
- **Vector3 force** – a vector that specifies the direction and strength of the push.
- **ForceMode mode** – determines how the force is applied, such as gradually over time, instantly, or ignoring mass.

Using these parameters, you can make objects move more realistically, respond to collisions naturally, and create different types of movement behaviors depending on the chosen `ForceMode`.

### Force Modes
Each of the Rigidbody force methods allows you to influence an object's movement, but the exact effect depends on how the force is applied. Unity provides four distinct `ForceMode` parameters to control this behavior. 
- `ForceMode.Force` – **Continuous force over time**, mass affects acceleration; good for gradual, realistic motion (e.g., car accelerating).
- `ForceMode.Acceleration` – **Continuous acceleration ignoring mass**; good for uniform forces like wind or gravity.
- `ForceMode.Impulse` – **Instant force affected by mass**; good for sudden bursts like jumps or explosions.
- `ForceMode.VelocityChange` – **Instant force ignoring mass**; good for responsive or arcade-style movement.

It’s helpful to see how each ForceMode actually affects velocity and distance mathematically. The behavior of each mode depends on whether the force is applied continuously or instantly, and whether the object’s mass influences the resulting motion.

Below, we’ll break down the equations for each ForceMode and show how to calculate velocity and distance per physics step in Unity using FixedDeltaTime.

### Force Modes

Each of the Rigidbody force methods allows you to influence an object's movement, but the exact effect depends on how the force is applied. Unity provides four distinct `ForceMode` parameters to control this behavior:

- `ForceMode.Force` – **Continuous force over time**, mass affects acceleration; good for gradual, realistic motion (e.g., car accelerating).  
- `ForceMode.Acceleration` – **Continuous acceleration ignoring mass**; good for uniform forces like wind or gravity.  
- `ForceMode.Impulse` – **Instant force affected by mass**; good for sudden bursts like jumps or explosions.  
- `ForceMode.VelocityChange` – **Instant force ignoring mass**; good for responsive or arcade-style movement.

While the descriptions above give a conceptual overview, it’s helpful to see **how each ForceMode actually affects velocity and distance mathematically**. The behavior of each mode depends on whether the force is applied continuously or instantly, and whether the object’s mass influences the resulting motion.  

Below, we’ll break down the equations for each ForceMode and show how to calculate **velocity and distance per physics step** in Unity using `FixedDeltaTime`.

#### 1. ForceMode.Force
- **Continuous force over time**, mass affects acceleration.  
- Good for gradual, realistic motion (e.g., car accelerating).  

$$
a = \frac{F}{m}
$$

$$
v_{\text{new}} = v_{\text{current}} + a \cdot \text{FixedDeltaTime}
$$

$$
d = v_{\text{current}} \cdot \text{FixedDeltaTime} + \frac{1}{2} a \cdot \text{FixedDeltaTime}^2
$$

---

#### 2. ForceMode.Acceleration
- **Continuous acceleration ignoring mass**.  
- Useful for uniform forces like wind or gravity.  

$$
a = F_{\text{effective}}
$$

$$
v_{\text{new}} = v_{\text{current}} + a \cdot \text{FixedDeltaTime}
$$

$$
d = v_{\text{current}} \cdot \text{FixedDeltaTime} + \frac{1}{2} a \cdot \text{FixedDeltaTime}^2
$$

---

#### 3. ForceMode.Impulse
- **Instant force**, mass affects resulting velocity.  
- Good for sudden bursts like jumps or explosions.  

$$
\Delta p = F
$$

$$
\Delta v = \frac{\Delta p}{m} = \frac{F}{m}
$$

> [!NOTE]
>  The object’s velocity changes immediately, but distance is accumulated over subsequent physics steps.

---

#### 4. ForceMode.VelocityChange
- **Instant force**, ignores mass.  
- Good for responsive or arcade-style movement.  

$$
\Delta v = F_{\text{applied}}
$$

$$
d = v_{\text{new}} \cdot \text{FixedDeltaTime}
$$

> [!NOTE]
>  Mass does not reduce the velocity change; it is applied immediately.

---

## Refactoring `MoveRigidbody` with Force
Now that we understand how different `ForceMode` options affect movement, it makes sense to refactor our `MoveRigidbody` class to take advantage of them. Instead of directly setting `rigidbody.velocity` (which bypasses physics), we’ll shift to force-based movement. This allows Unity’s physics system to handle acceleration, drag, and collisions while still providing us with the flexibility to choose between instant, gradual, or mass-independent motion through the selected `ForceMode`.

### Updating Fields 
To leverage the different ways motion can be applied with force, we will need to be able to set what `ForceMode` we want applied. To do this, we will create a new `[SerlizedField]`:

```csharp
    [SerializeField]
    [Tooltip("Controls how movement forces are applied:\n" +
             "• Force – Gradual, realistic acceleration (affected by mass).\n" +
             "• Acceleration – Gradual but ignores mass (like wind).\n" +
             "• Impulse – Instant burst, affected by mass (like a jump or explosion).\n" +
             "• VelocityChange – Instant burst, ignores mass (arcade-style movement).")]
    private ForceMode _forceMode = ForceMode.Force;

```


> [!NOTE]
> While we could keep the tooltip short (e.g., _“Controls how movement forces are applied”_), level designers may not be familiar with all the available ForceMode options. Expanding the tooltip into multiple lines gives them helpful context and reduces ambiguity.


## Speed vs Momentum vs Acceleration 
Before we continue, we need to discuss how each mode calculates the movement of an object. Currently, our code is set up with a `MAX_SPEED`, `Speed ', and `Direction`. 

### Speed

**`ForceMode.VelocityChange`**
This mode works just as `Rigidbody. Velocity`, in which `Speed` is calculated in meters per second in a given `Direction`, checking that our `Speed` can never go above `MAX_SPEED`. 

$$
v = \frac{\Delta x}{\Delta t} = \frac{5 \, \{m}}{1 \, \{s}} = 5 \, \{m/s}
$$

In this instance, the `Speed` value is just calculating the object's speed. 

### Momentum 

**`ForceMode.Impulse`**
This mode works just like `ForceMode.VelocityChange`, using momentum ($${p}$$), which is an object's velocity ($${v}$$) (i.e., `Speed` * `Direction`) multiplied by the object's mass ($${m}$$) measured in killograms (kg). 

$${p} = {v}*{m}$$

If we use the same example above and assume that our `Speed` is 5f, and our object's mass (set in the `Rigidbody` component) is 1kg, then our momentum is also 5f. The movement will be identical to **`ForceMode.VelocityChange`**. However, if our mass is 2kg, our momentum is now 10f. While the momentum is doubled, the velocity (meters/second) has not changed; the object will still move at the same rate of 5 meters per second. We just needed twice as much force to get the object moving due to its mass. 

## Acceleration 
We've already discussed how Force is calculated by Newton's second law

$$
F = m \cdot a
$$

Conversely, we can calculate acceleration as 

$$
{a} = \frac{{F} \}{{m}\}
$$

In both these instances, force is measured in Newtons, but we need to convert this to kg * meters/seconds squared, which, if you recal, isl the mass in kg multiplied by meters/second, which is our momentum, which is squared. In addition, since we are using the phsycis time, we are not actually using seconds here but time. To be exact Unity's physics step, this step is uses FixedDeltaTime at a rate of Default Time.fixedDeltaTime = 0.02 s (50 physics steps per second). 

Therefore, our velocity becomes v = a*t , so our acceleration multiplied by time, this is linear, but while our velocity (think of it as speed) increases over time, the distance we move is not immediate. Instead, distance is d= 1/2 a * t squared. Remember that in unity we are not measuring time in seconds with physics by the physics step. 
So if our `Speed` is 5f and we use that as the calculation of our velocity and time is one second, velocity is 5f. Distance is 1/2 of 5f, making it 2.5f, time is physical step of 0.02 squared which is 0.0004 multiplied by 2.5f, which is 0.001. This acceleration increases over time, starting out very small in this instance. Still, there are other forces acting on this object, gravity, drag and friction can all play a factor in how fast an object moves. 








## Refactoring the Movement
Currently, the `Move()` takes care of all the calculations for speed and direction, then applies the values to the `_rigidBody.linearVelocity` to simulate motion. The calculations for setting speed and direction are still valid, but we no longer want to directly change the linear velocity of the Rigidboy component. Instead, we want to apply different 

We need to update the `Move()` method so it no longer contains the velocity calculations directly. Instead, `Move()` now sets the shared direction and speed values, and then calls `HandleMovementMode()`.

```csharp
    /// <summary>
    /// Moves the object in a specified direction at a specified speed.
    /// </summary>
    /// <param name="direction">The direction to move the object (optional).</param>
    /// <param name="speed">The speed at which the object should move (optional).</param>
    private void Move(Vector3? direction = null, float? speed = null)
    {
        // Resolve the effective values for this frame
        Vector3 moveDirection = direction ?? Direction;
        float moveSpeed = speed ?? Speed;

        // Update properties to ensure validation and internal consistency
        Direction = moveDirection;
        Speed = moveSpeed;

        //Record the current direction and speed
        _currentDirection = Direction;
        _currentSpeed = Speed;

        // Handle movement based on movement mode
        HandleMovementMode();
        
    }//end Move()












## Handling Acceleration and Deceleration
In games, movement rarely happens instantly; objects often **accelerate** to their target speed and **decelerate** when stopping. Velocity-based movement typically doesn’t need gradual acceleration, because we can instantly set the Rigidbody’s velocity. Force-based movement, on the other hand, **relies on acceleration and deceleration** to feel smooth and realistic.

While Unity’s physics system automatically calculates acceleration from applied forces, relying on it alone can make movement feel **unpredictable**, because mass, drag, collisions, and frame rate can all affect a Rigidbody’s behavior. In gameplay scenarios where precise control is important, like a racing game, we don’t want these interactions to interfere with the player’s experience. By adding **explicit acceleration and deceleration values**, we can fine-tune how quickly objects speed up and slow down, keeping movement smooth, responsive, and predictable, while still benefiting from Unity’s physics for collisions and realism.

To give us precise, predictable control, we introduce explicit fields for `_acceleration` and `_brakingForce`. Acceleration determines how quickly an object speeds up, while braking force controls how quickly it slows down when stopping. These values ensure movement feels smooth, responsive, and intuitive, while still taking advantage of Unity’s physics system.

**1. Add** the following fields

```csharp
[SerializeField, Tooltip("Acceleration applied per second when using force mode.")]
private float _acceleration = 10f;

[SerializeField, Tooltip("Deceleration applied per second when stopping in force mode.")]
private float _brakingForce = 10f;

```
<br>

> **✔️ CHECK POINT**
> 
> Save your script, switch back to the Unity editor, and press **Play** to test the changes in action.

<br>

### Organizing Fields in the Inspector
When we look in the Editor, the **_GENERAL SETTINGS_** section contains several fields that apply to both velocity and force movement. However, our new fields for acceleration and braking force are only relevant when using force mode. To make this clear and keep the Inspector organized, we update and add `[Header]` attributes to separate fields logically.

```csharp

   
    [SerializeField]
    [Tooltip("Choose between velocity-based and force-based movement")]
    private MovementMode _movementMode = MovementMode.Velocity;

    [Header("MOVEMENT SETTINGS")]

    [SerializeField]
    [Tooltip("Should the object be moving on start?")]
    private bool _moveOnStart = true;

    // Direction of movement
    [SerializeField]
    private Vector3 _direction = Vector3.right; 
    
    [SerializeField]
    [Range(0f, MAX_SPEED)]
    [Tooltip("Speed of the object’s movement. Cannot exceed maxiumum speed.")]
    private float _speed = 5f;

    [Header("FORCE MODE SETTINGS")]

    [SerializeField, Tooltip("Acceleration applied per second when using force mode.")]
    private float _acceleration = 10f;
    
    [SerializeField, Tooltip("Deceleration applied per second when stopping in force mode.")]
    private float _brakingForce = 10f;

```

The fields in the Inspector are arranged to help level designers **quickly understand and set up movement**:

- **Movement mode** — choose whether the object uses velocity or force.
- **MOVEMENT SETTINGS** - explicitly indicates these settings are for movement
  - **Move on start** — Should the object begin moving immediately?
  - **Direction** — which way the object moves.
  - **Speed** — how fast the object moves; this naturally **connects** to the force-specific settings.
- **FORCE MODE SETTINGS** - used only by Force
  - **Acceleration** — determines how quickly the object reaches the target **speed**.
  - **Braking force** — determines how quickly the object slows down from that **speed**.

This order makes it easy to think step by step: first decide if the object moves, then which way and how fast, and finally, if using force, how it accelerates and brakes.

<br>

> **✔️ CHECK POINT**
> 
> Save your script, switch back to the Unity editor, and press **Play** to test the changes in action.

<br>

---

## Refactoring Movement: Handling Modes Separately

Up until now, our `Move()` method handled the calculation for moving objects with velocity. The calculations for adding force will be different, and we will also need to check which mode (velocity or force) will be used. Putting all this logic in one method can 
messy and makes it hard to read and debug. 

To make the code **cleaner and easier to maintain**, we can keep the shared logic in the `Move()` method and dedicated methods: 

- `HandleMovementMode()` - handles the check for movement mode, calls the  appropriate `MoveWith..()` method
- `MoveWithVelocity()` — handles instant, precise movement by setting the Rigidbody’s velocity.
- `MoveWithForce()` — handles physics-driven movement by applying forces, taking acceleration and braking into account.

  


    
``` 

### Handle Movement based on Mode 
The `HandleMovementMode()` method checks the current movement mode and calls the appropriate method to move the Rigidbody.

```cshaprp
/// <summary>
/// Determines which movement mode is currently selected and 
/// calls the corresponding method to move the object.
/// </summary>
private void HandleMovementMode()
{
    switch (_movementMode)
    {
        case MovementMode.Velocity:
            Debug.Log("Move with Velocity");
            MoveWithVelocity();
            break;

        case MovementMode.Force:
            Debug.Log("Move with Force");
            MoveWithForce();
            break;

        default:
            Debug.LogWarning("Unhandled movement mode: " + _movementMode);
            break;

    }//end switch(_movementMode)

}//end HandleMovementMode()

```

### Move with Velocity 
Previously, the velocity calculation was performed directly inside the `Move()` method. To clean up the code and separate concerns, we now place this calculation in its own dedicated method, `MoveWithVelocity()`.

```csharp
/// <summary>
/// Applies velocity-based movement to the Rigidbody.
/// </summary>
private void MoveWithVelocity()
{
    // Directly set Rigidbody velocity for immediate, responsive movement
    _rigidBody.velocity = Direction * Speed;

}//end MoveWithVelocity()
```

The `Move()` method handles any arguments for direction and speed, assigning them to the `Direction` and `Speed` properties. By the time `MoveWithVelocity()` is called, these properties have already been updated, so no additional parameters are needed.


### Move with Force

The calculation for force-based movement is different from velocity-based movement because we **don’t directly set the Rigidbody’s velocity**. Instead, we calculate the **force needed to gradually move the object toward a target velocity**, allowing physics to handle acceleration, deceleration, and interactions with other objects.

In velocity-based movement, the Rigidbody instantly reaches the target speed and direction. With force-based movement:

- A **desired velocity** determines the speed and direction you want the Rigidbody to move **each frame**, based on the `Speed` and `Direction` properties.
- **Acceleration** limits how much the Rigidbody’s velocity can change each physics update with `Time.fixedDeltaTime`.
- The **velocityChange** is the difference between the **current velocity and desired velocity**.
  - If it is too large, it is **clamped to** `_acceleration * Time.fixedDeltaTime`.
  - `Vector3.ClampMagnitude` ensures that the velocity change is **limited in any direction**, not just per axis.
  - This effectively means the Rigidbody **accelerates toward the target velocity at a controlled rate**, instead of instantly reaching it.
- **Applied force** is the **mass** of the Rigidbody multiplied by the **clamped velocity change**, applied as an impulse.

**Create the MoveWithForce() method**

```csharp
    /// <summary>
    /// Applies force-based movement to the Rigidbody using acceleration and braking.
    /// </summary>
    private void MoveWithForce() 
    {
        // Calculate the desired velocity to move by for each frame based on Direction and Speed
        Vector3 desiredVelocity = Direction * Speed;

        // Calculate the difference between the current velocity and the target velocity
        Vector3 velocityChange = desiredVelocity - _rigidBody.velocity;

        // Limit the velocity change to the maximum acceleration allowed this frame (physics step)
        float maxChange = _acceleration * Time.fixedDeltaTime;

        // Keep the velocity change within the allowed limit, for gradual acceleration
        velocityChange = Vector3.ClampMagnitude(velocityChange, maxChange);

        // Push the Rigidbody toward the target velocity by adding force
        _rigidBody.AddForce(velocityChange * _rigidBody.mass, ForceMode.Impulse);

    }//end MoveWithForce()


```

This method gives objects a **more natural, weighty feel**, makes movement responsive to collisions or external forces, and allows for realistic acceleration and braking.


## Defining Movement Actions

Currently, the `Move()` method delegates to `HandleMovementMode()`, to decide whether to use velocity-based or force-based calculations. The `Stop()` logic will also depend on the object’s current movement mode. To avoid duplicating code, we’ll extend `HandleMovementMode()` so it can check both the movement mode and the type of action (move or stop).

In order to do this, we will need an additional **enum** for **movement actions**, similar to how we defined the **movement modes**. With this in place, `HandleMovementMode()` can branch on both the mode and the action, keeping all decision-making in a single, reusable method.

**Create MoveAction Enum** 

```csharp

    /// <summary>
    /// Defines the type of movement action to perform.
    /// </summary>
    private enum MovementAction
    {
        Move,
        Stop
    }
```

### Refactor `HandleMovementMode()` method
The `HandleMovementMode()` method currently decides which movement calculation to use based on the object’s movement mode. To extend it so that it can also handle stopping, we need to know **whether the action is a Move or a Stop**. We can do this by adding a MoveAction parameter. When either `Move()` or `Stop()` calls `HandleMovementMode()`, this parameter will let the method not only check the movement mode, but also choose the correct action for each case.


```csharp
/// <summary>
    /// Determines which movement mode is currently selected and 
    /// calls the correct method for either moving or stopping.
    /// </summary>
    ///     /// <param name="action">The type of MovementAction to perform (e.g., Move/Stop) </param>
    private void HandleMovementMode(MovementAction action)
    {
        switch (_movementMode)
        {
            case MovementMode.Velocity:
                if (action == MovementAction.Move)
                {
                    Debug.Log("Move with Velocity");
                    MoveWithVelocity();
                }
                else if (action == MovementAction.Stop)
                {
                    Debug.Log("Stop with Velocity");
                    StopWithVelocity();
                }
                break;

            case MovementMode.Force:
                if (action == MovementAction.Move)
                {
                    Debug.Log("Move with Force");
                    MoveWithForce();
                }
                else if (action == MovementAction.Stop)
                {
                    Debug.Log("Stop with Force");
                    StopWithForce();
                }
                break;

            default:
                Debug.LogWarning("Unhandled movement mode: " + _movementMode);
                break;
        }//end switch
    }//end HandleMovementMode()
```
### Update calls to `HandleMovementMode()`

Now that `HandleMovementMode()` requires the move action argument, we will need to update the `Move()` that calls this method. 

```csharp
    /// <summary>
    /// Moves the object in a specified direction at a specified speed.
    /// </summary>
    /// <param name="direction">The direction to move the object (optional).</param>
    /// <param name="speed">The speed at which the object should move (optional).</param>
    private void Move(Vector3? direction = null, float? speed = null)
    {
        // Resolve the effective values for this frame
        Vector3 moveDirection = direction ?? Direction;
        float moveSpeed = speed ?? Speed;

        // Update properties to ensure validation and internal consistency
        Direction = moveDirection;
        Speed = moveSpeed;

        //Record the current direction and speed
        _currentDirection = Direction;
        _currentSpeed = Speed;

        // Handle MOVE action, based on movement mode
        HandleMovementMode(MovementAction.Move));
        
    }//end Move()
```

Additionally, we will need to **update the `Stop()`** method

```csharp
    /// <summary>
    /// Stops the object's movement by updating the movement flag.
    /// </summary>
    public void Stop()
    {
        // Handle STOP action, based on movement mode
        HandleMovementMode(MovementAction.Stop));

    }//end Stop()
```

---

## Creating Stop Methods for Velocity and Force

Now that we've updated our `Stop()` method to delegate movement mode selection to `HandleMovementMode()`, we need to create the specific stop methods for velocity and force movement. These methods define exactly how an object should stop depending on its movement type.

### Velocity-Based Stop

Stopping a velocity-based object is straightforward. Since velocity movement instantly sets the Rigidbody's velocity, stopping is simply a matter of **zeroing out the velocity**. 

```csharp

    /// <summary>
    /// Instantly stops the Rigidbody by setting its velocity to zero.
    /// Used for velocity-based movement.
    /// </summary>
    private void StopwithVelocity()
    {
        // Instantly stop the Rigidbody by setting velocity to zero
        _rigidBody.linearVelocity = Vector3.zero;

    }//end StopWithVelocity

```
## Force-Based Stop

Instead of instantly halting the object, force-based movement gradually decelerates it using physics-based forces.


To calculate the gradual slowdown of the object using force, we first need to check if the object is still moving. One way to do this is by looking at the magnitude of the Rigidbody’s current velocity. The magnitude tells us how fast the object is moving.

However, calculating the magnitude requires taking a square root, which is slightly more expensive for the computer to compute. Since we don’t actually need the exact speed, just whether the object is effectively moving, we can use sqrMagnitude instead. This gives the squared length of the velocity vector, avoiding the square root calculation and making the check a little faster.

For example:

Suppose the object’s velocity vector is (0.1, 0, 0):

Magnitude: √(0.1² + 0² + 0²) = 0.1

Squared magnitude: 0.1² + 0² + 0² = 0.01






