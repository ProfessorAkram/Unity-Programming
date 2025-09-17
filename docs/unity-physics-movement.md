# Unity Physcis Movment

Previously, we created a [`MoveTransform Class`](unity-basic-movement.md) class that allowed for basic movement of a GameObject using transform.position. While functional, there’s one drawback: it doesn’t make use of **Unity’s physics system**. This means:

- The object does not naturally respond to **collisions**
- It won’t be affected by **gravity**, **friction**, or other **forces**
- It can pass straight through walls and other objects

In other words, while `MoveTransform` gives us direct control, it doesn’t make the object part of the physical world.

If we want our GameObjects to interact realistically, with walls, the ground, or other moving bodies, we need to move them using **`Rigidbody` physics**, not just by setting their  `Transform`.

---
## Introducing Rigidbody

A `Rigidbody` component allows a GameObject to be affected by Unity’s physics properties, such as **gravity**, **mass**, **drag**, and **collisions**. Once a `Rigidbody` is added, the object becomes part of the physics simulation and behaves **more realistically**.

The key difference is that movement should now **happen through the `Rigidbody`**, not by changing `transform.position` directly. If you continue to move an object by setting its `Transform`, you’ll bypass physics again, resulting in unrealistic behavior and broken collision detection.

> **🗡️ Side-Quest: Adding a Rigidbody**
>
> Test how game objects react when using physcis.
> 
> 1. Create a simple **Cube** in your scene.
> 2. Add a `Rigidbody` component from the Inspector.
> 3. Press **Play**; notice how gravity immediately pulls the Cube downward.
> 4. Place the **Cube** on top of another object (like a **Plane** or **Quad**) and watch it rest naturally on the surface instead of falling through.

### Types of Physics Objects

Now that we've added a `Rigidbody` and seen how it interacts with **gravity** and **collisions**, it’s important to understand the different **types of physics** objects in Unity. Each type behaves differently in the physics simulation:

**1. Static Objects**

Objects that never move should be set as `Static` in the **Inspector**. These objects **do not have a `Rigidbody`**. While they don’t move on their own, they can **still collide with dynamic objects**.
- **Use case:** Floors, walls, platforms—anything that stays in place but should block or support other objects.

**2. Kinematic Objects**
Objects whoes movement is controlled entirely through scripts, usually via `trasnform.position`, and **ignore physic forces** like gravity, but still need to collide with other objects. 
  - **Use case:** Moving platforms, doors, or any object that must be animated or moved by code while still interacting with dynamic objects.

> [!NOTE]
>
> If you only moved it with `transform.position` and no `Rigidbody`, fast-moving objects might clip through these objects. For examplea moving platform in a platformer where the player can land on it, and the platform moves via `trasnform.position`. Since the platform still needs interact with the player ( and possibly other objects), they still require a `Rigidbody` with `Is Kinematic` checked. This ensures **collisions** work correctly **without applying physics forces** to the object itself.

**3. Dynamic Objects**
 Objects that are fully affected by physcis (e.g. gravity, forces, drag, and collisions). 
  - **Use case:** Characters, balls, projectiles—anything that moves naturally in response to forces or collisions.

Choosing the right type is important. Use `Static` for immovable objects, `Kinematic` when you need scripted motion without full physics control and `Dynamic` for physics-driven objects.

By understanding these three types, you’ll know when to use `Rigidbody`, how your objects will interact, and how to design movement and collisions in your scene.

### Accessing Rigidbody
To move a GameObject with physics, you need to work with its `Rigidbody` component. Unlike the `Transform` (which every GameObject has by default), not all GameObjects automatically include a `Rigidbody`, you have to **add it yourself** in the Inspector.

That means you can't just call it directly like we do with:

`transform.position` 

Instead we have to make a **reference** to store the `Rigidbody` attached to this object:
```csharp
    // Reference to the object's Rigidbody component
    private Rigidbody _rigidBody;
```
Then we _set_ this variable to the actual `Rigidbody` on the GameObject:

```csharp
    //Set reference to the Rigidbody component
    _rigidBody = GetComponent<Rigidbody>(); 
```

#### Using `GetComponet<T>()`
Unity uses a **component-based system**, where each GameObject is made up of components (Transform, Rigidbody, Collider, etc.).

To access any of these components via script, you use the `GetComponent<T>()` method, which essentially tells Unity to **find the component of type _T_ attached to this GameObject**.

The type isn't limited to built-in Unity components, you can also use it for any custom component (i.e., class) you create and attach to a GameObject.

Once you store the reference in a variable, you can use it to **access public properties** (like velocity, mass, or drag) or **call methods** on that component.

#### What if No Component is Found?

If `GetComponent<T>()` doesn’t find the requested component on the GameObject, it returns **null**.

This means any logic that tries to access the component (e.g., `_rigidBody.velocity = ...`) will throw a `NullReferenceException`.

To prevent this, it’s good practice to **check for null** before using the reference:

```csharp
if (_rigidBody == null)
{
    return; // Exit the method if the Rigidbody is missing
}
else
{
    _rigidBody.velocity = Speed * Direction;
} 
```

If the **component is essential for the class to work**, you can enforce its presence using Unity’s `[RequireComponent]` attribute. 
The attribute is placed before the class defintiion and the type of componet to be required is defined. Multiple `RequireComponent` attributes can be included.
By using Unity’s `[RequireComponent]` attribute ensures that the **component is always added** to the GameObject, so the reference will never be `null`:
```csharp
    [RequireComponent(typeof(Rigidbody))]
    public class MoveRigidbody : MonoBehaviour
    {
        // Your class logic here
    }
```

Using `[RequireComponent]` helps avoid null reference errors and ensures your class always has the components it needs to function properly.

---

## Create the `MoveRigidbody` Component

Now that we are familiar with Unity's `Rigidbody` component, it’s time to create a movement component that relies on Rigidbody instead of directly modifying `transform.position`.

1. **Create a new C# class** named `MoveRigidbody` and open it in your editor.

2. **Open your existing [`MoveTransform`](unity-basic-movement.md) class and copy**  everything inside the class into the new MoveRigidbody class.

**The foundation of both classes is the same:** properties for speed, direction, and movement flags. The key difference is that `MoveRigidbody` **will move the object through its `Rigidbody`**, allowing it to properly interact with gravity, collisions, and other physics forces.

This approach makes it easy to **reuse your existing logic** while upgrading your movement to use Unity's physics system.

### Get the `RigidBody` Component
Before we can move our game object we need to get access to the `Rigidbody` componet as mentioned above. 
To do this we need to update our `MoveRigidbody` class to the following: 

```csharp

[RequireComponent(typeof(Rigidbody))] 
public class MoveRigidbody: MonoBehaviour
{
    // Serialized fields for initial values

    // Reference to the object's Rigidbody component
    private Rigidbody _rigidBody;

    // ... rest of the class

    // Awake is called once on initialization         
    private void Awake()
    {
        // Validate initial speed and direction via properties
        Speed = _speed;
        Direction = _direction;

        //Set reference to the Rigidbody component
        _rigidBody = GetComponent<Rigidbody>(); 
        
        //Set GameObject's initial position
        transform.position = Vector3.zero;
        
        // Determine if the object should start moving
        _isMoving = _moveOnAwake;
    }//end Awake()

    // ... rest of the class

}
```
### Enforce Physics for Dynamic Objects
Earlier, we discussed the differences between the types of Unity physics objects. If we want this GameObject to interact properly with physics, it must be **dynamic** and not **kinematic**.

By default, the `isKinematic` property is set to `false` when a `Rigidbody` is added. However, since our functionality **depends on the `Rigidbody` being dynamic**, and we can't be sure that the level designer has not manually changed this value. Whether intentional or not, we can avoid issues by enforcing it in `Awake()` after obtaining the Rigidbody reference:

```csharp
 // Awake is called once on initialization         
    private void Awake()
    {
        // Validate initial speed and direction via properties
        Speed = _speed;
        Direction = _direction;

        //Set reference to the Rigidbody component
        _rigidBody = GetComponent<Rigidbody>();

        //Ensure that Rigidbody is dynamic
        _rigidBody.isKinematic = false; 
        
        //Set GameObject's initial position
        transform.position = Vector3.zero;
        
        // Determine if the object should start moving
        _isMoving = _moveOnAwake;
    }//end Awake()

    // ... rest of the class

}
```
Setting `isKinematic = false` in code ensures that your `Rigidbody` will always behave as a dynamic physics object, regardless of any changes made in the Inspector.

<br>

> [!IMPORTANT]
> Before continuing, make sure the `MoveRigidbody` component has been added to a GameObject in your Unity scene.
> To do this, you have a few options:
> - Drag and drop the **C# script** from the **Project Window** onto the GameObject in the **Hierarchy Window** or the **Scene Window**.  
> - With the GameObject selected, drag and drop the script into the **Inspector Window**.  
> - Alternatively, in the **Inspector Window**, click the **Add Component** button, then type the name of the component (`MoveTransform`) and select it.  
> Once added, the script becomes a component of the GameObject, allowing it to control its behavior during gameplay.

<br>

---

## Physics Movement 

By default, if two objects with physics are placed above each other in the scene, gravity will immediately take effect, and they will fall naturally. But Unity’s physics system does more than just apply **gravity**, it also handles **collisions, momentum, and other forces**:

- **Collisions:** When two Rigidbody objects collide, Unity calculates how they should bounce, slide, or stop based on their mass, velocity, and drag. For example, a heavy ball hitting a lighter cube will push the cube more than the other way around.

- **Momentum:** Moving objects carry momentum, which affects how they interact during collisions. A fast-moving object will have a stronger impact than a slow one.

- **Forces and Drag:** Objects can be affected by forces like wind, explosions, or friction. Drag (air resistance) slows them down naturally over time.

- **Continuous Interaction:** Physics objects automatically respond to each other and the environment. For instance, stacking boxes will cause them to settle realistically, topple over, or slide if one is pushed.

Once a Rigidbody is added, Unity simulates all of these physics behaviors automatically, based on the setting applied in the Rigidbody. This allows for realistic interactions between objects without you having to manually calculate collisions or momentum.

However, if you want **more control over how your GameObjects move**, Unity provides two main approaches using physics: **velocity-based movement** and **force-based movement**.

---

## Understanding Velocity

**Velocity** describes how fast and in which direction an object moves; essentially displacement over time, measured in meters per second **(m/s)**.
Its basic equation is:

$$
v = \frac{\Delta x}{\Delta t}
$$

Where:  

- **\( v \)** = velocity (vector, includes direction)  
- **\( \Delta x \)** = change in position (displacement)  
- **\( \Delta t \)** = change in time  

**For Example**

If a cube moves **5 meters to the right in 2 seconds**, its velocity is:

$$
v = \frac{5 \, \text{m}}{2 \, \text{s}} = 2.5 \, \text{m/s (to the right)}
$$

> [!NOTE]
> 
> In physics, **speed** is just **how fast an object moves**, while **velocity** includes both **speed and direction**.
> Unity’s `Rigidbody.velocity` follows this definition, as it is a `Vector3`. Storing both **how fast and which way the object is moving**.

### When to Use Velocity Over Tranform
Let's say that you are creating a **racing game**, whcih the players move at a **constant speed** along a track, smoothly responding to input.

If we were to use the standard `transform.position` , the object would move but it would **ignores collisions and physics**, so the vehcial could clip through walls or other racers.  

Using `Rigibody.velocity` allows us to: 
- Direct and consistent control of speed and direction.  
- Automatic physics interactions (collisions, slopes, friction).  
- Continuous motion without needing to update the position every frame manually.

<br>
> [!WARNING]
>
> The example above uses `Rigidbody.velocity` to demonstrate the syntax for accessing the property. However, in practice, you s**hould never use the class name directly**. You always access the velocity through a **reference to the component**.
> In our case, that reference is `_rigidBody.velocity`. The reference name could vary, some developers use `_rb`, but it’s **best practice to be explicit** in your variable names. For clarity, `_rigidBody` is preferred here.
<br>

This mirrors real-world physics: the veheical keeps moving at a set velocity until another force (collision, player input, braking) changes it.

---

## Update `Move()`
To move our objects with velocity we need to rewrite the `Move()` method to use the Rigidbody's velocity instead of `transform.position`.

```csharp
/// <summary>
/// Moves the object in a specified direction at a specified speed using Rigidbody velocity.
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
    
    // Move the GameObject using Rigidbody velocity
    _rigidBody.velocity = Speed * Direction;
}
```
#### Key Changes
**1. Remove the `_isMoving` flag** 
  - If the game object's velocity is greater than zero it will be moving, we do not need a flag to check this.
  - Instead we would check the velocity value :  `if (!_rigidbody.velocity == Vector3.zero)`
  - This check in nullfied here since we do not need to check the update for this condition any longer.

<br>
> [!NOTE]
>
> Remeber to remove the delecration for the flag `_isMoving` at the top of the class too.
<br>

**1. No `Time.deltaTime` Needed**
  - Rigidbody velocity is measured in **meters per second**, so Unity automatically moves the object each physics step.  
  - Unlike `transform.position += ...`, you **don’t manually scale by `deltaTime`**


### When and How to Call `Move()` with Velocity
Previously, the `_moveOnAwake` flag was used to **set the object as _“moving”_ when initialized**, but with Rigidbody-based movement, this approach no longer applies. Since velocity is handled by the physics engine, we now need to **explicitly call** `Move()` to set the Rigidbody’s velocity.

Instead, we should check this flag in `Start()`. Unity guarantees that the **physics system is ready** at this stage, so calling `Move()` will correctly set the Rigidbody’s velocity and the object will interact properly with gravity, collisions, and other forces.

To make this clearer, we’ll rename the flag to `_moveOnStart` and add a `Start()` method:

**1. Rename and update** the serialized field to:

```csharp
    [SerializeField]
    [Tooltip("Should the object be moving on start?")]
    private bool _moveOnStart = true;
```

**2. Create** the `Start()` method: 

```csharp
// Start is called once before the first Update
    private void Start()
    {
        if (_moveOnStart)
        {
            Move();

        }//end if(_moveOnStart)
        
    }//end Start()
    
```

Since Rigidbody velocity continues automatically once set, **we no longer need to call `Move()` in `Update()` every frame**. The Rigidbody will keep moving according to the assigned velocity, and Unity handles all the physics calculations during FixedUpdate steps.

In our current class, **calling `Move()` was the only thing happening in `Update()`**, so we can safely **remove the entire `Update()` method**.

**2. Remove** the `Update()` method: 

```csharp
    // Start is called once before the first Update
    private void Start()
    {
        if (_moveOnStart)
        {
            Move();

        }//end if(_moveOnStart)
        
    }//end Start()

    // ... Removed Update

```

<br>
> [!Caution]
> 
> If your `Update()` method contains other logic besides movement, you would only remove the `Move()` call, leaving the rest of the `Update()` intact.
<br>

---

## Updating Speed or Direction

At this point, whenever the `Move()` method is called, the object will continue moving as long as the Rigidbody’s velocity is not zero. Whenever `Move()` is called, we can pass new **speed** and **direction** values as arguments to update the Rigidbody’s velocity.

However, if **Speed** or **Direction** properties change during gameplay, for example, via a power-up or player input, the Rigidbody’s velocity wont automatically update, because `Move()` is no longer being called every frame.

Since `Move()` relies on physics, we could call it in `FixedUpdate()`, which is Unity’s **physics-timed update loop**. `FixedUpdate()` runs at a **consistent, fixed interval** (default 0.02 seconds, or 50 times per second), ensuring physics calculations like velocity, forces, and collisions remain stable and predictable, regardless of the frame rate.

That said, **we don't need to call `Move()` every physics step**, only when **Speed** or **Direction** actually change. To optimize this, we can create private fields to store the current speed and direction. Then, in `FixedUpdate()`, we check if either property has changed, and only call `Move()` when an update is necessary.

**1. Create fields** for storing the current speed and direction

```csharp
    // Store the last applied values for optimization
    private float _currentSpeed;
    private Vector3 _currentDirection;
```
**2. Set Current Values** for  speed and direction in the `Start()`
While our **Speed** and **Direction** properties are initialized in `Awake()`, we wait until `Start()` to copy those values into `_currentSpeed` and `_currentDirection`. The reason is order of execution:

- `Awake()` runs very early in the Unity lifecycle and is responsible for preparing the object. Here, we make sure the properties are set and validated against their serialized field values from the Inspector.
- By the time `Start()` runs, we can be confident those values are finalized. Assigning them to our "current" fields at this point gives us a reliable snapshot of the starting speed and direction as the game begins.

In short, `Awake()` ensures the properties are correct, and `Start()` records those validated values as the object’s initial state for gameplay.

```csharp
    // Start is called once before the first Update
    private void Start()
    {
        
        _currentSpeed = Speed;
        _currentDriection =Direction;

        if (_moveOnStart)
        {
            Move();

        }//end if(_moveOnStart)
        
    }//end Start()

```

**2. Create `FixedUpdate()`** and check for Speed and Direction changes

```csharp
    //Called at fixed intervals (i.e. physic steps) 
    private void FixedUpdate()
    {
        // Only update velocity if speed or direction changed
        if (Speed != _currentSpeed || Direction != _currentDirection)
        {
            Move(Direction, Speed);
        }

    }//end FixedUpdate()
```

**3. Update `Move()`** method to set the current speed and direction
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

        //Record the current direciton and speed
        currentDirection = Direction;
        currentSpeed = Speed;
        
        // Move the GameObject using Rigidbody velocity
        _rigidBody.velocity = Speed * Direction;

    }//end Move()

```
---
## Stoping Objects with Velocity 

To stop our game objects all we need to do is update our `Stop()` method to set the Rigidbod's velocity to zero. 

**1. Update `Stop()`** and set velocity

```csharp
/// <summary>
/// Stops the object's movement immediately by zeroing its Rigidbody velocity.
/// </summary>
public void Stop()
{
    _rigidBody.velocity = Vector3.zero;  // Immediately halts motion
}
```

This is an **immediate stop** and in some cases could be a bit jaring, especially in a racing style game. If you want the object to slow down naturally, you would need a different approach (like a gradual deceleration or “brake”).

---

## Gradual Stop with “Brake”
In Unity, we can make a GameObject slow down smoothly by **interpolating** its velocity toward zero. One common method is `Vector3.Lerp`.

**What Lerp does:**

- Short for **Linear Interpolation**, Lerp takes a **start value**, a **target value**, and a **factor** (usually between 0 and 1).

- It returns a value somewhere between the start and target, based on the factor.

- Repeatedly calling Lerp each physics step gradually moves a value toward the target.

### Adding Controls for Braking 
Now that we understand how Lerp works, we need a way to **control how quickly the object decelerates**. To do this, we declare a **`_deceleration` field**, which acts as the **“factor”** in our Lerp calculation. This value determines how fast the object slows down each physics step, and by exposing it in the Inspector, it’s easy to tweak for different gameplay behaviors.

```csharp
    [SerializeField]
    [Range(0f, MAX_SPEED)]
    [Tooltip("Speed of the object’s movement. Cannot exceed maxiumum speed.")]
    private float _speed = 5f;


    // Direction of movement
    [SerializeField]
    private Vector3 _direction = Vector3.right;

    [SerializeField]
    [Tooltip("How quickly the object slows down when braking.")]
    private float _deceleration = 5f;


```
Next, we need a **flag to indicate when braking is active**. This is a simple **boolean variable** that we set to true whenever we want the GameObject to begin decelerating:

```csharp
  //Is the object currently braking 
  private bool _isBraking = false;

```

Finally, we provide a method to **trigger braking**. Unlike `Stop()`, which immediately sets the Rigidbody’s velocity to zero, the `Brake()` method does not **directly modify velocity**. Its sole responsibility is to **set a flag** that tells the physics system to start gradually decelerating the object.

While _we could integrate_ this behavior into the `Stop()` method, keeping a separate `Brake()` method has a few advantages: it allows for **immediate stops when necessary**, provides **clearer, more explicit control**, and makes it easier to reason about the object’s behavior in different gameplay situations. 

<br>
> [!NOTE]
>
> Because the **gradual deceleration** using Lerp occurs over multiple physics steps, it must be handled in `FixedUpdate()`. The `Brake()` method’s role is simply to set the flag that signals when to start applying the Lerp-based slowdown.
<br>

```csharp

/// <summary>
/// Triggers gradual deceleration of the object.
/// </summary>
public void Brake()
{
    // Enable braking; FixedUpdate will handle slowing the Rigidbody
    _isBraking = true;

}//end Brake()
```

---

## Modifiying FixedUpdate() for Braking

To implement **gradual braking**, we first need to update `FixedUpdate()` to check if the object is currently braking by examining the `_isBraking` flag. This ensures that our physics calculations happen in sync with Unity’s physics system.

```csharp
    //Called at fixed intervals (i.e. physic steps) 
    private void FixedUpdate()
    {
        // Only update velocity if speed or direction changed
        if (Speed != _currentSpeed || Direction != _currentDirection)
        {
            Move(Direction, Speed);
        }

        // is the object currently braking 
        if (_isBraking)
        {
            
        }

    }//end FixedUpdate()
```
If the object is **braking**, we need to implement `Vector3.Lerp`. In doing so, it’s important to understand how Lerp is calculated:

$$
\text{newValue} = \text{currentValue} + (\text{targetValue} - \text{currentValue}) \times t
$$

Where:

- **newValue** = the updated value  
- **currentValue** = the current value  
- **targetValue** = the value you want to approach  
- **t** = interpolation factor (usually between 0 and 1)

In this case our **\( t \)** is (decleartion value * Time.FixedDeltaTime).

<br>

> [!NOTE]
>
> When we set a Rigidbody’s velocity directly (e.g., `_rigidBody.velocity = Speed * Direction`), Unity applies it in meters per second and not by varing framerate, so there’s no need to multiply by `Time.deltaTime`.
>
> However, `Vector3.Lerp` moves a value by a **fraction per call**, not a real-world speed. To make the deceleration consistent over time, we multiply by `Time.fixedDeltaTime`, which **represents the duration of a single physics step**. This ensures that braking behaves consistently, regardless of the physics update rate.
<br>

Since the velocity is multiplied by a fraction each frame, it gets closer and closer to zero but **will never reach exactly zero**. Therfore, we need to define a **threshold** to decide when the object is “effectively stopped.”  

A common approach is:

$$
\text{if } |\text{velocity}| < \epsilon, \text{ then consider velocity zero.}
$$

For our use case,  **ε** (epsilon) would be something like **0.01 m/s**.

This means we not only need to perform the Lerp calculation, but also determine when the velocity is close enough to zero. Since this involves multiple calculations, placing all of this directly in `FixedUpdate()` would violate the **Single Responsibility Principle**, making the method harder to read and maintain.

Instead, we can extract the braking logic into its own method—e.g., `ApplyBraking()`, and call it from `FixedUpdate()`. This keeps `FixedUpdate()` clean, containing only the minimal logic of **checking the braking flag and delegating the work**.

**Modify `FixedUpdate()`** to call `ApplyBraking()`
```csharp
    //Called at fixed intervals (i.e. physic steps) 
    private void FixedUpdate()
    {
        // Only update velocity if speed or direction changed
        if (Speed != _currentSpeed || Direction != _currentDirection)
        {
            Move(Direction, Speed);
        }

        // is the object currently braking 
        if (_isBraking)
        {
            ApplyBraking();
        }

    }//end FixedUpdate()
```

---
## Create Method to Apply Braking 
The `ApplyBrake()` method will be responsible for gradually reducing the Rigidbody’s velocity toward zero while ensuring that we stop once the object is effectively at rest.

### Checking Velocity with Magnitude

As mentioned above, Lerp gradually reduces the Rigidbody’s velocity toward zero but never reaches exactly zero. To determine when a GameObject is **effectively stopped**, we check the **magnitude** of its velocity vector.

The **magnitude** represents the length of a vector, which in this case corresponds to the object’s overall speed:

$$
|\mathbf{v}| = \sqrt{v_x^2 + v_y^2 + v_z^2}
$$

Where:

- \(v_x, v_y, v_z\) are the components of the velocity along each axis.  

By comparing this magnitude to a small threshold (e.g., 0.01 m/s) as in:

```csharp
_rigidBody.velocity.magnitude < 0.01f
```
We can decide when to consider the object stopped and take any necessary actions, such as setting velocity explicitly to zero or disabling braking. 

### Create `ApplyBraking()`

Our `ApplyBraking()` method is called internally by `FixedUpdate()` and triggered via the public `Brake()` method. Since it is used only within the class, it can remain **private**.

This method gradually interpolates the Rigidbody’s velocity toward zero and checks when it is close enough to stop. Once the object is effectively at rest, it calls `Stop()` and clears the `_isBraking` flag.

```csharp
    /// <summary>
    /// Gradually reduces the Rigidbody's velocity toward zero using linear interpolation (Lerp).
    /// </summary>
    private void ApplyBrake()
    {
        // Gradually reduce velocity
        _rigidBody.velocity = Vector3.Lerp(_rigidBody.velocity, Vector3.zero, _deceleration * Time.fixedDeltaTime);

        // Stop braking once velocity is close enough to zero
        if (_rigidBody.velocity.magnitude < 0.01f)
        {
            Stop();
            _isBraking = false;
        }

    }//end Brake()

```

> [!NOTE]
>
> The object must have zero velocity to fully stop. While we could directly set `_rigidBody.velocity = Vector3.zero` here, we already handle that in the `Stop()` method. To follow the **DRY (Don't Repeat Yourself)** principle, we simply call `Stop()` instead of duplicating the code.




----



Force-based movement

rb.AddForce(direction * forceAmount);


Feels more “realistic.”

Acceleration, momentum, drag apply naturally.

Good for cars, projectiles, floating objects.




