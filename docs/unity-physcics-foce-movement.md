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

In both these instances, force is measured in Newtons, but we need to convert this to kg * meters/seconds squared, which, if you recall, is the mass in kg multiplied by meters/second, which is our momentum, then squared. In addition, since we are using the phsycis time, we are not actually using seconds here but Unity's physics step. This step is uses FixedDeltaTime at a rate of Default Time.fixedDeltaTime = 0.02 s (50 physics steps per second). 

Therefore, our velocity becomes v = a*t , so our acceleration multiplied by time, this is linear, but while our velocity (think of it as speed) increases over time, the distance we move is not immediate. Instead, distance is d= 1/2 a * t squared. Remember that in unity we are not measuring time in seconds with physics by the physics step. 
So if our `Speed` is 5f and we use that as the calculation of our velocity and time is one second, velocity is 5f. Distance is 1/2 of 5f, making it 2.5f, time is physical step of 0.02 squared which is 0.0004 multiplied by 2.5f, which is 0.001. This acceleration increases over time, starting out very small in this instance. Still, there are other forces acting on this object, gravity, drag and friction can all play a factor in how fast an object moves. 

----


If spped is too fast include Continuous collision detection 

Clamp at max speed 


Stop




