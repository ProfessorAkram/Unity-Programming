# Force-Based Movement
In Unity, a **force** is a push or pull applied to an object that makes it accelerate, just like in the real world. Using force-based movement allows Unity's **physics engine** to handle **acceleration**, **deceleration**, **collisions**, and **interactions** with other objects naturally. This creates a sense of weight and momentum, making the movement feel more realistic and immersive.

## Why Not Just Use Velocity?

Velocity-based movement is simple and often effective, but it comes with a limitation: directly setting a Rigidbody’s velocity overrides parts of Unity’s physics system. This can make objects feel unnatural because collisions, momentum, and external forces are ignored.

By using forces instead, your objects respond to the environment in a believable way. They can be pushed, pulled, slowed down, or influenced by explosions, wind, or collisions—all without extra code.

### Force vs. Velocity in Action

Here are a few examples where force-based movement shines:

- **Vehicles:** Cars in a racing game gradually accelerate and brake rather than instantly reaching top speed.
- **Rolling or sliding objects:** Balls or crates respond naturally to collisions and slopes.
- **Characters:** A player-controlled character can feel “weighty,” obeying momentum and gravity.
- **External forces:** Explosions, gusts of wind, or pushes from other objects affect motion realistically.

#### Pros and Cons of Force-Based Movement

- ✅ Produces natural, fluid acceleration and deceleration
- ✅ Fully integrates with Unity physics (collisions, mass, momentum)
- ✅ Allows external forces to influence movement

- ❌ Requires careful tuning of mass, acceleration, and drag
- ❌ Slightly more complex to implement and debug

Force-based movement is particularly useful when objects accelerate gradually, collisions matter, or you want to simulate vehicles, projectiles, or other physics-heavy interactions.

---

## Understanding Force
The foundation of force-based movement comes from **Newton’s Second Law of Motion**, which states:

$$
\vec{F} = m \cdot \vec{a}
$$

Here:

- $$\(\vec{F}\)$$ is the force applied (the push or pull)  
- $$\(m\)$$ is the object’s mass  
- $$\(\vec{a}\)$$ is the resulting acceleration  

This tells us that the same force applied to two different objects will produce different results:  
- A **light ball** accelerates quickly  
- A **heavy crate** barely budges  

Likewise, if we want the same object to accelerate faster, we must apply more force.

Another way to see this relationship is to solve for acceleration:

$$
\vec{a} = \frac{\vec{F}}{m}
$$

This is important in games: when you push a car, swing a bat, or roll a ball, the object doesn’t instantly reach its final speed. Using forces, Unity simulates this **gradual change in velocity**, giving movement a lifelike feel.

### Units of Force
Force is measured in **Newtons (N)**, which are defined as **kilograms·meters per second squared (kg·m/s²)**. This tells us how much force is required to accelerate a certain mass at a certain rate. In simpler terms, the Newton also represents the **magnitude** of the push or pull (i.e, how strong the force is). To fully describe a force, we also need to know its **direction**, because forces are **vectors**.

$$
\vec{F} = \text{magnitude} \times \text{direction}
$$


When working with forces in Unity, it’s important to understand the units being used. 
- Mass in kilograms (kg)
- Distance in meters (m)
- Time in seconds (s)
- Force in Newtons (N)

For example, applying 10 N of force to a 2 kg Rigidbody produces an acceleration of 5 m/s².

### Physics Steps

Understanding the units of **mass**, **distance**, and **force** is only part of the picture. In Unity, physics is simulated in **discrete time steps**, not continuously every frame. This means the timing of updates determines how forces change an object’s velocity and position over time, which in turn affects how movement feels in your game.

Unity calculates `Rigidbody` physics inside the `FixedUpdate()` method, rather than the usual `Update()`. This ensures consistent physics behavior regardless of frame rate. The interval between physics updates is defined by `Time.fixedDeltaTime`, which defaults to **0.02 seconds** (50 physics steps per second).

Because physics updates occur in steps, Unity computes the object’s motion incrementally. At each step:

- **Velocity increases linearly** and is updated according to acceleration (from all applied forces, including gravity, drag, or user input) incrementally each physics step:

$$
\vec{v}_{\text{new}} = \vec{v}_{\text{current}} + \vec{a} \cdot \text{FixedDeltaTime}
$$

- **Position** is updated based on the current velocity and acceleration. This **displacement grows quadratically**:

$$
\vec{d} = \vec{v}_{\text{current}} \cdot \text{FixedDeltaTime} + \frac{1}{2} \vec{a} \cdot (\text{FixedDeltaTime})^2
$$

![Velocity over time is linear; Distance over time is quadratic](../imgs/velocity-distance-graph.png)

Because `FixedDeltaTime` is small, velocity and position change gradually rather than instantly. This is why objects **accelerate smoothly**, giving movement a natural, lifelike feel.

### Other Forces Acting on Objects

Additionally, Unity automatically accounts for other forces such as **gravity**, **drag**, and **friction** at each physics step. Multiple forces combine to produce a **net force**, which determines acceleration and resulting motion. This makes force-based movement **predictable and physically consistent**, while still allowing for interactions like collisions, pushes, or environmental effects.

To determine how an object moves with multiple forces at work, we calculate the **net force**, which is simply the sum of all individual forces:

$$
\vec{F}_{\text{net}} = \vec{F}_1 + \vec{F}_2 + \dots
$$

Once we know the net force, the resulting acceleration is:

$$
\vec{a} = \frac{\vec{F}_{\text{net}}}{m}
$$

A common and important example is **gravity**. When enabled on a `Rigidbody`, Unity automatically applies gravitational force:

$$
F_g = m \cdot g
$$

Other forces can also influence objects:
- **Drag** (linear and angular damping) slows motion over time, simulating air resistance or rotational friction.
- **Physics Materials** can be applied to objects to control friction and bounciness, affecting how objects slide, roll, or bounce off surfaces.

By combining these forces—gravity, drag, user-applied pushes, and collisions—Unity calculates the net effect on each Rigidbody at every physics step. This allows movement to feel both dynamic and physically realistic, without requiring manual adjustments to positions or velocities.

> [!NOTE]
> In Unity, the default **gravity is set to -9.81 m/s²** on the **Y-axis**, matching Earth’s gravity.
> 
> This value can be changed under **`Edit` > `Project Settings` > `Physics`** if you want to simulate different environments (e.g., the Moon’s gravity).

---

## Applying force in Unity
Now that we understand **forces, vectors, mass, acceleration, and Unity’s physics steps**, it’s time to see how we can actually **apply forces** to objects in a Unity scene. The `Rigidbody` component provides several methods for influencing an object’s movement:

- `Rigidbody.AddForce()` - Applies a linear force in **world space**; changing the object's velocity.
  - The ForceMode can be set to **Force**, **Acceleration**, **Impulse**, or **VelocityChange** depending on how you want the force to affect the object.
- `Rigidbody.AddTorque()` - Applies a **rotational force (torque)**, causing it to spin around its center of mass.
- `Rigidbody.AddExplosionForce()` - **Simulates an explosion** by applying a force that **radiates outward** from a specific point, affecting nearby objects.
- `Rigidbody.AddRelativeForce()` - Applies a linear force **relative to the Rigidbody’s local axes**.
- `Rigidbody.AddRelativeTorque()` - Applies torque **relative to the Rigidbody’s local axes**.

Each of these methods has its own specific use cases, but for this lesson, we will keep things simple and focus on the most common method: `Rigidbody.AddForce`.

### `AddForce()` Method

The `Rigidbody.AddForce()` method applies a force to a Rigidbody, causing it to accelerate in the direction of that force. Unlike setting the Rigidbody’s velocity directly, this method allows Unity’s physics engine to calculate the resulting motion, taking into account:

- The object’s **mass**
- **Drag** and other damping effects
- Other forces such as **gravity** or **collisions**


The method takes two parameters:
- **Vector3 force** – a vector that specifies the direction and strength of the push.
- **ForceMode mode** – determines how the force is applied, such as gradually over time, instantly, or ignoring mass.

Using these parameters, you can make objects move more realistically, respond to collisions naturally, and create different types of movement behaviors depending on the chosen `ForceMode`.

## Force Modes
Each of the Rigidbody force methods allows you to influence an object's movement, but the exact effect depends on how the force is applied. Unity provides four distinct `ForceMode` parameters to control this behavior. 
- `ForceMode.Force` – **Continuous force over time**, mass affects acceleration; good for gradual, realistic motion (e.g., car accelerating).
- `ForceMode.Acceleration` – **Continuous acceleration ignoring mass**; good for uniform forces like wind or gravity.
- `ForceMode.Impulse` – **Instant force affected by mass**; good for sudden bursts like jumps or explosions.
- `ForceMode.VelocityChange` – **Instant force ignoring mass**; good for responsive or arcade-style movement.

Mathematically, these modes determine **how the force contributes to velocity and displacement each physics step**. Continuous forces accumulate gradually through multiple `Fixed` steps, while instantaneous forces change the Rigidbody’s velocity immediately. Choosing the correct ForceMode is essential for achieving the **desired feel of movement** in your game.

---

### 1. ForceMode.VelocityChange
Previously, when we implemented `_rigidbody.velocity = Speed * Direction;` we were **ignoring Unity’s physics system**, effectively telling the Rigidbody, _“Your velocity is exactly this,”_ without considering mass, gravity, drag, or collisions. While this may seem simple, it often produces **unnatural movement** in dynamic scenes.

Using `AddForce()` with `ForceMode.VelocityChange` works differently. Instead of directly setting the Rigidbody’s velocity, we **apply a change in velocity**, calculated as the **delta velocity** ($$\Delta \vec{v}$$) , to the Rigidbody:

$$
\Delta \vec{v} = \vec{v}_{\text{target}} - \vec{v}_{\text{current}}
$$

- $$\Delta \vec{v}$$ → change in velocity  
- $$\vec{v}_{\text{target}}$$ → target velocity  
- $$\vec{v}_{\text{current}}$$ → current velocity

This mode **ignores mass**, meaning the Rigidbody will reach the target velocity immediately, while still respecting other forces like gravity and collisions.

---

### 2. ForceMode.Impulse
`ForceMode.Impulse` is conceptually very similar to `ForceMode.VelocityChange`, but with one key difference: **it accounts for the Rigidbody's mass**. An impulse applies a sudden force that produces an immediate change in velocity proportional to the object’s mass:

$$
\text{appliedImpulse} = \Delta \vec{v} * {m}
$$

- `appliedImpulse` → vector passed to `AddForce`  
- $$\Delta \vec{v}$$ → change in velocity  
- $$m$$ → mass of the Rigidbody  

By scaling the delta velocity by mass, Impulse produces an **instantaneous, realistic response** that interacts naturally with Unity’s physics, including collisions and drag. Think of it as giving a heavier object a proportionally stronger push to achieve the same change in motion as a lighter object.

---

### 3. ForceMode.Force

`ForceMode.Force` applies a continuous force to the Rigidbody. Unlike `ForceMode.VelocityChange` or `ForceMode.Impulse`, this force is applied gradually over time, and the Rigidbody’s mass directly influences the resulting acceleration:

$$
\vec{a} = \frac{\vec{F}_{\text{net}}}{m}
$$

In many tutorials, `ForceMode.Force` is implemented with an **arbitrary acceleration magnitude** multiplied by a **direction vector**. That acceleration is then **scaled by the Rigidbody's mass** to produce the **applied force**:

```csharp
Vector3 acceleration = 5f * direction;
Vector3 appliedForce = acceleration * _rigidbody.mass;
_rigidbody.AddForce(appliedForce, ForceMode.Force);

```

> [!WARNING]
> Many tutorials don't even take this extra step;  they simply plug in a large, arbitrary _“magic number”_ as the applied force. While this can work in practice, it often requires trial and error to _“feel right”_ and doesn’t reflect a physics-based approach.

The scalar that determines the strength of the applied acceleration can be a constant, like `5f` in the example above, or a variable commonly called `accelerationMultiplier`. It scales the direction (or delta velocity) to control the overall force applied.

Using a simple direction vector works for straightforward movement, but it doesn’t account for the Rigidbody's current velocity. If the object is sliding, drifting, or partially aligned with the desired direction, blindly applying force along the input direction can result in overshoot or inconsistent acceleration. A more **physics-aware approach** uses **delta velocity** instead of the input direction:

```csharp
float accelerationMultiplier = 5f;
Vector3 deltaVelocity = targetVelocity - _rigidbody.velocity;
Vector3 acceleration = accelerationMultiplier * deltaVelocity;
Vector3 appliedForce = acceleration * _rigidbody.mass;
_rigidbody.AddForce(appliedForce, ForceMode.Force);

```
While this approach works well for intuitive movement,


While the previous approach, scaling the direction or delta velocity by a constant acceleration multiplier, works well for **intuitive, responsive movement**, it does not guarantee precise control over how quickly the Rigidbody reaches the target velocity. A more **predictable, physics-based approach** calculates the exact force required from the **target change in velocity** (i.e., $${\Delta \vec{v}}$$) over a desired **acceleration time** (i.e., $${t_{\text{acceleration}}}$$):

$$
\vec{a} = \frac{\Delta \vec{v}}{t_{\text{acceleration}}}
$$

Where:

- $$\(\Delta \vec{v}\)$$ → change in velocity you want  
- $$\(t_{\text{acceleration}}\)$$ → duration in **seconds** over which the object should reach the target velocity  

Then the force is calculated as:

$$
\vec{F} = m \cdot \vec{a}
$$

#### Acceleration Over Time

As explained in the [Physics Steps](#physics-steps) section, Unity processes physics in **discrete steps** (`FixedUpdate`) rather than continuously. This means that when you apply a continuous force or acceleration (`ForceMode.Force` or `ForceMode.Acceleration`), Unity **doesn't update velocity all at once**. Instead, each physics step applies only a fraction of the total acceleration. 

For example, if acceleration is $$\(5 \ \text{m/s}^2\)$$ and `FixedDeltaTime = 0.02` s:

$$
\vec{d} = \frac{1}{2} \cdot 5 \cdot 0.02^2 = 0.001 \ \text{m}
$$

At the first physics step, the displacement is very small. Over multiple steps, **velocity accumulates**, and the object moves farther. Other forces, such as **gravity**, **drag**, and **friction**, also influence the motion.

The **acceleration time** ( $$\(t_{\text{acceleration}}\)$$ )is the **total real-world time** you want the object to take to reach the target velocity.
- Larger acceleration time → slower acceleration, more gradual movement
- Smaller acceleration time → faster acceleration, more abrupt movement

> [!IMPORTANT]
> We **do not need to calculate the per-step displacement**. Once you determine the acceleration from $${\Delta \vec{v}}/{t_{\text{acceleration}}}$$ , Unity automatically applies it incrementally each physics step (based on `FixedDeltaTime`). In other words, Unity _“spaces out”_ the acceleration over the chosen time, ensuring the Rigidbody reaches the target velocity smoothly and predictably, while still respecting forces like gravity, drag, and collisions.


---

#### 4. ForceMode.Acceleration
`ForceMode.Acceleration` applies a continuous acceleration to the Rigidbody, ignoring the object’s mass. Unlike `ForceMode.Force`, where heavier objects require proportionally more force to achieve the same acceleration, `ForceMode.Acceleration` lets you directly set the acceleration:

$$
\vec{a} = \frac{\Delta \vec{v}}{t_{\text{acceleration}}}
$$

**No calculation for force is needed**; just the desired acceleration, and Unity updates the Rigidbody’s velocity each physics step.

This mode is particularly useful when you want **uniform effects**, such as:
- Gravity-like forces applied manually
- Wind or current affecting objects regardless of mass
- Environmental effects like conveyor belts or moving platforms

Because mass is ignored, the same acceleration is applied to all objects, whether they are light or heavy.

---

## Refactoring `MoveRigidbody` with Force
Now that we understand how different `ForceMode` options affect movement, it makes sense to refactor our `MoveRigidbody` class to take advantage of them. Instead of directly setting `rigidbody.velocity` (which bypasses physics), we’ll shift to force-based movement. This allows Unity’s physics system to handle acceleration, drag, and collisions while still providing us with the flexibility to choose between instant, gradual, or mass-independent motion through the selected `ForceMode`.

### Updating Fields 
To leverage the different ways motion can be applied with force, we will need to be able to set what `ForceMode` we want applied. To do this, we will create a new `[SerializeField]`:

#### Step 1: **Create** the following field

```csharp
    [SerializeField]
    [Tooltip("Controls how movement forces are applied:\n" +
             "• Force – Gradual, realistic acceleration (affected by mass).\n" +
             "• Acceleration – Gradual but ignores mass (like wind).\n" +
             "• Impulse – Instant burst, affected by mass (like a jump or explosion).\n" +
             "• VelocityChange – Instant burst, ignores mass (arcade-style movement).")]
    private ForceMode _forceMode = ForceMode.Force;

```

> [!TIP]
> While we could keep the tooltip short (e.g., _“Controls how movement forces are applied”_), level designers may not be familiar with all the available ForceMode options. Expanding the tooltip into multiple lines proides helpful context and reduces ambiguity.

#### Speed and Acceleration 
To implement **force-based movement**, we first need to define how **speed** and **acceleration** will work. Unlike directly setting the Rigidbody's velocity, we now treat `Speed` as the **target speed** we want the object to reach.

When using `ForceMode.Force` or `ForceMode.Acceleration`, there are two ways to control acceleration:
- Acceleration multiplier (`_accelerationMultiplier`) – a user-defined value applied each physics step to scale the force or acceleration. This is simple to implement and tweak, but it is an arbitrary _"magic number"_ and must be tuned carefully for smooth motion.
- Acceleration time (`_accelerationTime`) – calculate the force needed to reach the target speed over a desired duration, producing smoother, more predictable motion.

We use a flag (`_useAccelerationTime`) to determine which method to apply, giving us flexibility to choose between a simple multiplier or time-based acceleration for different gameplay situations.

#### Step 2: **Creae** the additional fields

```csharp
    [SerializeField]
    [Range(0f, MAX_SPEED)]
    [Tooltip("Target speed for the object’s movement. Clamped to MAX_SPEED.")]
    private float _speed = 5f;
    
    [SerializeField]
    [Tooltip("Acceleration magnitude applied toward the target velocity.\n" +
         "• Controls how strong the applied force/acceleration is when using ForceMode Force Acceleration.\n" +
         "• Larger values = stronger acceleration (reaches target speed faster).\n" +
         "• Smaller values = weaker acceleration (reaches target speed more gradually).")]
    private float _accelerationMultiplier = 100f;
    
    [SerializeField]
    [Tooltip("Enable to calculate force based on acceleration time instead of using a fixed acceleration multiplier. \n" +
             "• Produces smoother, time-based ramp-up to the target speed.")]
    private bool _useAccelerationTime = false;
    
    [SerializeField]
    [Tooltip("Duration (in seconds) over which the object accelerates to its target speed.\n" +
         "• Unity spreads the acceleration across physics steps automatically.\n" +
         "• Smaller values = quicker, more abrupt acceleration.\n" +
         "• Larger values = slower, smoother acceleration.")]
    private float _accelerationTime = 0.5f;

```

#### Reference to Mass
To properly calculate _Impulse_ and to ensure the gradual acceleration has enough force to move an object, we need to reference the object's **mass**. This can easily be done using the `_rigidBody.mass` property. While this is pretty easy to write, we can simplify it by creating a field for `_mass` and then in `Awake()` set it to the Rigidbody's mass value. It might only be a few characters shorter, but even little things like this can speed up the process. 

#### Step 1: **Create** the following fields: 

```csharp
    //Reference to the object's Mass
    private float _mass;
```
#### Step 2: **Set** the reference for `_mass` in the `Awake()` method

```csharp
 private void Awake()
    {
        // Validate initial speed and direction via properties
        Speed = _speed;
        Direction = _direction;
        
        //Set reference to the Rigidbody component
        _rigidBody = GetComponent<Rigidbody>(); 
        
        //Ensure that Rigidbody is dynamic
        _rigidBody.isKinematic = false; 
        
        //Set reference to the object's mass
        _mass = _rigidBody.mass;

    }//end Awake()
```

---

## Refactoring `Move()` Method

Previously, the `Move()` method directly modified the Rigidbody's linearVelocity. With **force-based movement**, the Rigidbody's velocity is no longer set directly, so that line should be removed or commented out. 

Additionally, the `_isMoving` flag should also be removed from `Move()`, because setting it there only indicates that movement was **requested**, not that the object is actually moving. For accurate movement detection, _isMoving should instead be determined by checking the Rigidbody’s velocity in `Update()` or `FixedUpdate()`.

#### Step 1: **Remove or Comment out** the following fileds
```chsharp
        // Move the GameObject using Rigidbody velocity
        //_rigidBody.linearVelocity = Speed * Direction;

        // Flags the object as moving
        //_isMoving = true;

```

### Calculating the values needed for movement

To implement force-based movement using `AddForce()`, the movement logic is split into **two responsibilities**:

1. Calculating the values needed for movement.
2. Applying the calculated forces.

Since `Move()` already updates and tracks the object's speed and direction, it is a natural place to perform the additional calculations required for force-based movement. These calculations include:
- **Target velocity:** the velocity the object should aim for based on speed and direction.
- **Delta velocity:** the difference between the current velocity and target velocity.
- **Impulse:** instantaneous change in momentum.
- **Acceleration:** either scaled by a multiplier or calculated over an acceleration time for gradual changes.

`Move()` **does not directly apply force** to the Rigidbody. Instead, it prepares the values needed and passes them to **a new method, `ApplyForce()`**, which handles the actual application of force.


#### Step 2: **Update** the `Move()` method with the following: 

```chsharp
    /// <summary>
    /// Moves the object in a specified direction at a specified speed.
    /// </summary>
    /// <param name="direction">The direction to move the object (optional).</param>
    /// <param name="speed">The speed at which the object should move (optional).</param>
    public void Move(Vector3? direction = null, float? speed = null)
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
        
        // Desired velocity based on current speed and direction
        Vector3 targetVelocity = Speed * Direction; 
        
        // Velocity difference to reach target
        Vector3 deltaVelocity = targetVelocity - _rigidBody.linearVelocity; 
        
        // Instant force (like a sudden push)
        Vector3 impulse = deltaVelocity * _mass; 

        Vector3 acceleration; 

        //If using accelerated time for more control
        if (_useAccelerationTime)
        {
            // Gradually reach target velocity over a specified acceleration time
            acceleration = deltaVelocity / _accelerationTime;
        }
        else
        {
            // Scale delta velocity by multiplier for responsive movement
            acceleration = deltaVelocity * _accelerationMultiplier;
        }//end if (_useAccelerationTime)
        

        ApplyForce(deltaVelocity, impulse, acceleration);

    }//end Move()

```
---

##  Applying Physics Forces 
The **new** `ApplyForce()` method **handles physics execution** by taking the pre-calculated values from `Move()` and applying them to the Rigidbody according to the selected ForceMode:
- **ForceMode.Force:** Continuous force scaled by mass.
- **ForceMode.Acceleration:** Continuous acceleration ignoring mass.
- **ForceMode.Impulse:** Instantaneous force scaled by mass.
- **ForceMode.VelocityChange:** Instantaneous change in velocity, ignoring mass.

#### Step 1: **Create** the `ApplyForce()` method: 

```csharp
    /// <summary>
    /// Applies force to the Rigidbody based on the selected ForceMode.
    /// </summary>
    /// <param name="deltaVelocity">
    /// The difference between the target velocity and the current Rigidbody velocity.
    /// Used for ForceMode.VelocityChange to instantly adjust velocity.
    /// </param>
    /// <param name="impulse">
    /// The instantaneous change in momentum (mass × delta velocity).
    /// Used for ForceMode.Impulse to apply a sudden push.
    /// </param>
    /// <param name="acceleration">
    /// The acceleration vector to apply, either scaled by a multiplier or calculated over an acceleration time.
    /// Used for ForceMode.Force and ForceMode.Acceleration.
    /// </param>
    private void ApplyForce(Vector3 deltaVelocity,  Vector3 impulse, Vector3 acceleration)
    {
        switch (_forceMode)
        {
            case ForceMode.Force:
                // Applies a continuous force based on acceleration and mass 
                _rigidBody.AddForce(acceleration * _mass, ForceMode.Force);
                break;

            case ForceMode.Acceleration:
                // Applies a continuous acceleration
                _rigidBody.AddForce(acceleration, ForceMode.Acceleration);
                break;

            case ForceMode.Impulse:
                // Applies an instant force (like a sudden push), mass affects velocity change
                _rigidBody.AddForce(impulse, ForceMode.Impulse);
                break;

            case ForceMode.VelocityChange:
                // Instantly changes velocity, ignores mass (like directly setting velocity)
                _rigidBody.AddForce(deltaVelocity, ForceMode.VelocityChange);
                break;

            default:
                Debug.LogWarning("Unhandled ForceMode: " + _forceMode);
                break;
        }//end switch(_forceMode)
        

    }//end ApplyForce()
```
With this separation, `Move()` focuses on calculating the necessary movement values, while `ApplyForce()` handles the actual physics execution. This **improves readability, maintainability, and flexibility**, making it easy to adjust movement behavior or add new force modes in the future.

<br>

> **✔️ CHECK POINT**
> 
> Save your script, switch back to the Unity editor, and press **Play** to test the changes in action.

<br>

---
## Detecting Movement
In the previous implementation, movement was tracked using a manual `_isMoving` flag set inside `Move()`. However, with force-based physics, **requesting movement is not the same as actually moving**. An object might:
- Receive a movement input but be blocked by friction or collision,
- Still be sliding due to inertia even after the input stops,
- Be pushed externally by another force.

Therefore, **movement should be determined by the Rigidbody’s actual velocity**, not by manually toggling a flag. 

#### Step 1: **Remove or comment out** the field for `_isMoving`. 

```csharp
    // Runtime movement flag
    //private bool _isMoving;
```

A more reliable way to detect motion is to check whether the Rigidbody’s velocity magnitude exceeds a small threshold (to account for floating-point noise and near-zero drift).

#### Step 2: **Create** a new method named `IsMoving()`

```csharp
  /// <summary>
  /// Returns true if the Rigidbody is currently moving above a small threshold.
  /// </summary>
  public bool IsMoving(float threshold = 0.01f)
  {
      return _rigidBody.linearVelocity.sqrMagnitude > threshold * threshold;
  }

```

#### When to Use IsMoving()

The `isMoving()` method becomes most useful when you need to react to real movement, not just input.
For example, a braking system could check `IsMoving()` before applying deceleration, ensuring that forces are only applied when necessary:

```csharp
if (IsMoving())
    ApplyBraking();
```

---


## Gradual Stopping with Physics

In our current `MoveRigidbody()` class, we already have a `Stop()` method that immediately halts movement by resetting velocity:

```csharp
/// <summary>
/// Stops the object's movement immediately by zeroing its Rigidbody velocity.
/// </summary>
public void Stop()
{
    // Immediately halts motion
    _rigidBody.linearVelocity = Vector3.zero;  
} // end Stop()

```
This kind of instant stop**** is useful for:
- Resetting an object during gameplay or respawn.
- Snapping characters into place for cutscenes.
- Preventing sliding when precision is required.

> [!NOTE]
> The `Stop()` method directly sets the Rigidbody’s `linearVelocity` to zero. While using `AddForce()` with `ForceMode.VelocityChange` could also stop the object, directly setting `linearVelocity` is simpler and more reliable for an **immediate, guaranteed stop**. `VelocityChange` is better suited for applying sudden velocity adjustments dynamically, but for a full, instant halt, directly modifying linearVelocity ensures the object stops exactly when and how we want.

If our goal, however, is to create a more realistic **physics-based movement**, then it would benefit from a **gradual stopping instead of an immediate halt**. Just like we **accelerate smoothly** toward a target velocity, we may also want to slow down smoothly, especially in systems such as **racing games, character inertia, or heavy physics objects**.

### Apply Braking

Instead of setting velocity to zero instantly, we will set a **braking force** in the opposite direction of movement. Once we **apply braking**, it will reduce velocity over time. 

#### Step 1: **Create** a field for the **braking force**

```csharp
    [SerializeField]
    [Tooltip("Multiplier controlling how strongly the object slows when braking.")]
    private float _brakingMultiplier = 10f;
```

#### Step 2: **create** the method to **`ApplyBraking()`**

```csharp
    /// <summary>
    /// Gradually slows the object by applying a consistent braking force
    /// regardless of mass.
    /// </summary>
    public void ApplyBraking()
    {
        // Return immediately if the object is already stopped 
        if (!IsMoving())
            return;

        // Get the Rigidbody's current velocity
        Vector3 currentVelocity = _rigidBody.linearVelocity;

        // Apply braking force opposite to the movement direction
        Vector3 brakeForce = -currentVelocity.normalized * _brakingMultiplier;
    
        // Use Acceleration for consistent braking regardless of mass
        _rigidBody.AddForce(brakeForce, ForceMode.Acceleration);
    }
```
Notice that in this implementation of `ApplyBraking()`, we specifically use `ForceMode.Acceleration`. Using `Impulse` and `VelocityChange` would be too abrupt, causing the object to stop suddenly rather than gradually. Since we want **smooth, gradual braking**, these modes are not suitable.

When deciding between `ForceMode.Acceleration` and `ForceMode.Force` for braking, the difference comes down to **how mass affects deceleration**:

- **Acceleration**
  - **Pros:** Consistent braking regardless of object mass, good for characters, hovercrafts, or other objects where predictable deceleration is important.
  - **Cons:** Ignores mass, so braking may feel less “realistic” for very heavy objects.

- **Force**
  - **Pros:** Mass affects deceleration, so heavier objects naturally slow more slowly. Suitable for physics-based vehicles or objects with mass.
  - **Cons:** Braking is not consistent across objects of different masses, which can feel unpredictable in gameplay.

Using `ForceMode.Acceleration` in this case ensures that the **braking feels consistent and predictable** for gameplay, regardless of the Rigidbody's mass. If a more realistic, mass-dependent braking behavior is desired, **Force** could be used instead.

---

## Updating Testing

Since we do not yet have input set up for our `MoveRigidbody()` class, we previously implemented a `RunMovementTest()` method to simulate various movement scenarios in the Editor. Currently, the test runs `Move()` and `Stop()`, but we now want to include testing for `ApplyBraking()`.

#### Step 1: Update the `TestAction` **enum** to include a `Brake` option:

```csharp
    //Enum for list of possible actions to test 
    private enum TestAction
    {
        None,   // No action selected
        Move,   // Run Move() test
        Stop,   // Run Stop() test
        Brake  //Run ApplyBraking() test

    }
```

#### Step 2: **Update** `RunMovementTest()` to handle the new braking test:

```csharp
#if UNITY_EDITOR    
    /// <summary>
    /// Runs the selected movement test in the Editor based on the _testAction enum.
    /// </summary>
    private void RunMovementTest()
    {
        switch (_testAction)
        {
            case TestAction.Move:
                Debug.Log("Testing Move");
                Move();
                break;

            case TestAction.Stop:
                Debug.Log("Testing Stop");  
                Stop();
                break;
            
            case TestAction.Brake:
                Debug.Log("Testing Brakes");  
                ApplyBraking();
                break;
            
            case TestAction.None:
                Debug.Log("Testing None");
                //Do nothing
                break;
                
            default:
                Debug.Log("Unhandled TestAction: " + _testAction); 
                break;

        }//end switch(_testAction)

    }//end RunMovementTest()
#endif
```
<br>

> **✔️ CHECK POINT**
> 
> Save your script, switch back to the Unity editor, and press **Play** to test the changes in action.

<br>

---



If spped is too fast include Continuous collision detection 

Clamp at max speed 


















