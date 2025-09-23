# Force-Based Movement

While velocity-based movement is simple and effective, it has a drawback: directly setting the Rigidbody's velocity **overrides parts of Unity’s built-in physics system**. This can make objects feel less natural because collisions, momentum, and acceleration are bypassed.

When you want to simulate more **realistic physics-driven movement**, it’s recommended to use **forces** instead. Moving an object with force means Unity’s physics engine handles the **acceleration, deceleration, and interactions** with other objects. This gives objects a sense of **weight and inertia**, making gameplay feel more immersive.

Force-based movement is especially useful when:
- Objects accelerate or brake gradually.
- Collisions and impacts should realistically influence motion.
- Simulating vehicles, projectiles, or other physics-heavy interactions.


## Understanding Force
In Unity, when we move objects using **forces**, we aren’t simply telling them “move at this speed.” Instead, we **push or pull** them, just like in the real world. This is more realistic and allows objects to respond naturally to collisions, mass, and other forces in the environment.

The physics behind this comes from **Newton’s second law of motion**, which states:

$$
\vec{F} = m \cdot \vec{a}
$$

Where:

- $$\(\vec{F}\)$$ = force applied (how hard we push or pull, in Newtons)  
- $$\(m\)$$ = mass of the object (how heavy it is, in kilograms)  
- $$\(\vec{a}\)$$ = acceleration (how fast the object speeds up, in meters per second squared)

Consequently, acceleration can be calculated as: 

$$
\vec{a} = \frac{\vec{F}}{m}
$$

- If the object is **heavier**, it accelerates more slowly.  
- If the **force is bigger**, it accelerates faster.

### Instant Push 

When using **force-based movement** in Unity, we provide a **force vector**, and Unity calculates the resulting acceleration automatically based on the Rigidbody’s mass. You could apply a “make-believe” force factor to simulate full pushes or collisions.

Sometimes, for quick or arcade-style movement, we use a “force multiplier” to simulate an instant push. For example:

```csharp
Vector3 targetVelocity = Speed * Direction;
float acceleration = 2f; //Force multiplier for instant push (could be mass of object)
Vector3 appliedForce = targetVelocity * acceleration
```
- `Speed` is our target velocity, let's say 25f.
- `acceleration` scales the force vector so the object reaches the target speed quickly (instant push)
- `appliedForce` is the force vector we give to the Rigidbody. 

In this case, our `appliedForce` would be $$25 * 2 = 50$$, but this is a **gameplay value**, **not a real-world force** in Newtons.

### Target Speed

**However, in most gameplay situations**, we don't care about the raw force. Instead, we usually know **the speed we want the object to reach**. In these cases, we reverse-engineer the force: we calculate the **required acceleration** to reach the target speed and then multiply by mass to get the force to apply. This gives us precise, predictable movement while still using Unity’s physics engine.

If we have a **target `Speed`** (i.e., target velocity), the acceleration required to reach that velocity in a single physics step can be calculated as the **rate of change of velocity**:

$$
\vec{a}_{\text{required}} = \frac{\Delta \vec{v}}{\Delta t}
$$

Where:

$$
\Delta \vec{v} = \vec{v}_{\text{target}} - \vec{v}_{\text{current}}
$$

- $$\Delta \vec{v}$$ is the **change in velocity needed to reach the target**.
- $$\Delta t$$  is the physics timestep, which in Unity is `Time.fixedDeltaTime`.

>[!NOTE]
>`Time.fixedDeltaTime` is the fixed interval at which Unity updates the physics system. By default, it’s **0.02 seconds**, meaning Unity performs **50 physics steps per second**. Using this value ensures that acceleration and force are applied consistently, regardless of the frame rate.


This approach allows us to compute the required acceleration first, and then use it to determine the force to apply with Unity’s physics system, even if we don’t know the raw force beforehand.





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

Similarly, objects that roll or slide, like a ball down a slope or a crate pushed across the floor, respond naturally to collisions and other forces, rather than abruptly changing direction. Even characters can benefit from this approach, **feeling _“weighty”_** and obeying momentum, which gives movement a more lifelike and immersive quality. Unlike velocity-based movement, **force-based movement allows external forces**, such as explosions, wind, or pushes from other objects, to influence motion in a consistent and believable way.

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
- In Unity’s ForceMode.Force, we’re applying a **force that produces acceleration** over the physics step.
- (targetVelocity - _rigidbody.velocity) / Time.fixedDeltaTime → correct for ForceMode.Force because it produces the acceleration needed to reach the target velocity over the physics step.

The 0.5 * a dt^2 term is implicitly handled by Unity’s physics engine when it integrates acceleration into displacement.

You don’t need to manually include that term unless you want to predict exact displacement before applying the force.
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
#### 1. ForceMode.VelocityChange
- Unity expects $$\Delta \vec{v}$$, the **change in velocity** applied each physics step.
- **Mass is ignored**; it simply adds this $$Δv$$ to whatever the Rigidbody is currently doing.
- Calculating for $$Δv$$ ensures the Rigidbody ends up **exactly at the target velocity**, even if it was **already moving**.

$$
\Delta \vec{v} = \vec{v}_{\text{target}} - \vec{v}_{\text{current}}
$$

- $\Delta \vec{v}$ → change in velocity  
- $\vec{v}_{\text{target}}$ → target velocity  
- $\vec{v}_{\text{current}}$ → current velocity

When we set `_rigidbody.linearVelocity`, we are essentially telling the object, “move at this exact speed.” However, with `ForceMode.VelocityChange`, we are saying “Add this much speed to your current velocity to reach the desired target.” 

```csharp
// ❌ Directly set Rigidbody velocity
_rigidbody.linearVelocity = Speed * Direction;

// ✅ Calculate the delta velocity
Vector3 targetVelocity = Speed * Direction;
Vector3 currentVelocity = _rigidbody.velocity; 
Vector3 deltaVelocity = targetVelocity - currentVelocity;

// Apply using VelocityChange
_rigidbody.AddForce(deltaVelocity, ForceMode.VelocityChange);
```

---

#### 2. ForceMode.Impulse
- Unity expects an **instantaneous force** applied as a **one-time push**.  
- Mass **affects the resulting velocity**: heavier objects accelerate less for the same impulse.  
- Good for sudden bursts like jumps, explosions, or collisions.

$$
\Delta \vec{v} = \frac{\text{appliedImpulse}}{m}
$$

- $\Delta \vec{v}$ → change in velocity  
- $m$ → mass of the Rigidbody  
- `appliedImpulse` → vector passed to `AddForce`  

Since we already have a target speed and direction, we can calculate the impulse directly from them:

```csharp
// Target velocity we want this push to produce
Vector3 targetVelocity = _direction * _speed;

// Impulse required to achieve this velocity change
Vector3 appliedImpulse = targetVelocity * _rigidbody.mass;

// Apply the impulse as a one-time push
_rigidbody.AddForce(appliedImpulse, ForceMode.Impulse);
```

> [!NOTE]
> This will **instantly change the Rigidbody's velocity** by the target amount. No _delta_ calculation is needed unless you want to reach a specific velocity regardless of current motion.

---



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

In this instance, the `Speed` value calculates the object's speed. 

### Momentum (Impulse)

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

In both these instances, force is measured in Newtons, but we need to convert this to kg * meters/seconds squared, which, if you recall, is the mass in kg multiplied by meters/second, which is our momentum, then squared. In addition, since we are using the physics time, we are not actually using seconds here but Unity's physics step. This step uses `FixedDeltaTime`, whose default value is **0.02 s** (50 physics steps per second). 

Since the velocity becomes:

$${v} = {a}*{t}$$

Our acceleration is multiplied by time (physics step), which is linear. However, while our velocity (think of it as speed) increases over time, the distance we move is not immediate. 

![Velocity over time is linear; Distance over time is quadratic](../imgs/velocity-distance-graph.png)

Instead, distance over time is calculated as:

$${d} = \frac{1}{2} {a} * {t}^2$$

Therefore, let's say that our speed is **5f** and time is a physics step, which equates to **0.02**. Our resulting equation would be: 

$${0.001} = \frac{1}{2} {5} * {0.02}^2$$

This acceleration increases over time, starting very small in this instance. Still, other forces are acting on this object; gravity, drag, and friction can all play a factor in how fast an object moves. 

----
## Variables and Naming Conventions

Before we move forward, let’s clarify the terms we’ll be using in our codebase. These terms are grounded in physics, but we’re choosing names that are both accurate and approachable for anyone reading or using the system.

### Speed
`Speed` defines the target magnitude of velocity. Essentially, the rate of movement over units per second, or in this case, physics steps. It's not the current velocity/speed of the object, but our goal.

### Acceleration
Normally in physics, _acceleration_ is the **rate of change of velocity over time**.
In our code, we will think of this as a **force multiplier** that determines how strongly we push an object toward its Speed.

Why? Because the word _acceleration_ is easier to understand than “force multiplier.” So even though it’s not a perfect physics definition, we will need to create an `_acceleration` field as this force multiplier.

### Acceleration Time
Acceleration time is the duration it takes to reach the target Speed, controlling the smoothness of motion. If we want to allow for gradual acceleration, we will need a field named `_accelerationTime`. This naming matches the real-world context, for example, if a car goes from 0 to 60 in 5 seconds, that “5 seconds” is the _acceleration time_.

#### Acceleration Time Flag
The method by which we calculate force using acceleration vs acceleration time will differ, and in efforts to simplify our code, we will use a **flag** to check which to calculate. This will be a simple **bool** field to check if we want to  `_useAccelarationTime`. By default, we will assume this is false for a more instantaneous force application.

### Reference to Mass
To properly calculate _Impulse_ and to ensure the gradual acceleration has enough force to move an object, we need to reference the object's **mass**. This can easily be done using the `_rigidBody.mass` property. While this is pretty easy to write, we can simplify it by creating a field for `_mass` and then in `Awake()` set it to the Rigidbody's mass value. It might only be a few characters shorter, but even little things like this can speed up the process. 

**Create** the following fields: 

```csharp
    //Reference to the object's Mass
    private float _mass;

    [SerializeField]
    [Tooltip("Acceleration of speed; force multiplier.\n" +
             "• The larger the multiplier, the quicker the object will move.")]
    private float _acceleration = 100f;

    [SerializeField]
    [Tooltip("Use acceleration time for controlled acceleration")]
    private bool _useAccelerationTime = false;

    [SerializeField] [Tooltip("Time it takes to go from a stop to a desired speed \n" +
             "• The smaller the number, the faster the object will get up to speed")]
    private float _accelerationTime = 0.5f;
```
**Set** the reference for `_mass` in the `Awake()` method

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

## Update `Move()` Calculations

Previously, our `Move()` method was directly changing the `linearVelocity` of the object's Rigidbody. However, we now want to update this to calculate the different forces we may want to apply, depending on our force mode. Since we need to perform calculations and check for the force mode, we will refactor the `Move()` method to only run the calculations and then call the `ApplyForce()` method to apply the correct force calculation depending on the force mode. 

**Update** the `Move()` method

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

        // Move the GameObject using Rigidbody velocity
        //_rigidBody.linearVelocity = Speed * Direction;
        
        Vector3 appliedVelocity = Speed * Direction;
        Vector3 appliedImpluse = appliedVelocity * _mass;
        Vector3 appliedForce = appliedVelocity * _acceleration;
        
        //If using accelerated time for more control
        if (_useAccelerationTime)
        {
            float appliedAcceleration = Speed / _accelerationTime;
            Vector3 appliedAcceleratedForce = appliedAcceleration * _mass * Direction;
            
            appliedForce = appliedAcceleratedForce;
            
        }//end if (_useAccelerationTime)
        
        // Flags the object as moving
        //_isMoving = true;
        
       //Call ApplyForce method

    }//end Move()

```
### How it works 
 1.  Removed the direct setting of the Rigibody's `linear velocity`.
 2.  An `appliedVelocity` calculation replaces the calculation for linear velocity and will be used when using `ForceMode.VelocityChange`.
 3.  `appliedImpulse` is the `appliedVelocity` multiplied by mass and will be the calculation for using `ForceMode.Impulse`
 4.  When using `ForceMode.Force` or `ForceMode.Acceleration`, we calculate the `appliedForce`, which is the `appliedVelocity` multiplied by our `_acceleration` multiplier.
 5.  However, if `_useAccelerationTime` is true, we need to calculate:
     - The `appliedAcceleration`, which is the rate of `Speed` over the `_accelerationTime`
     - Determine the `appliedAccelerationForce`, which is the `appliedAcceleration` multiplied by both the object's `_mass` and `Direction`
     - Set `appliedForce` equal to the new `appliedAcceleratedForce` value
   
> [!NOTE]  
> In the `if(_useAccelerationTime)` condition, we assign the resulting value back to `appliedForce`. This allows us to pass the `appliedForce` to our next method (i.e., `ApplyForce`) without having to check a second time. 









If spped is too fast include Continuous collision detection 

Clamp at max speed 


Stop






