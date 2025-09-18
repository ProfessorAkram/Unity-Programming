# Unity Physics Movement

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
> Test how game objects react when using physics.
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
Objects whose movement is controlled entirely through scripts, usually via `transform.position`, and **ignore physical forces** like gravity, but still need to collide with other objects. 
  - **Use case:** Moving platforms, doors, or any object that must be animated or moved by code while still interacting with dynamic objects.

> [!NOTE]
>
> If you only moved it with `transform.position` and no `Rigidbody`, fast-moving objects might clip through these objects. For example, a moving platform in a platformer where the player can land on it, and the platform moves via `transform.position`. Since the platform still needs to interact with the player ( and possibly other objects), it still requires a `Rigidbody` with `Is Kinematic` checked. This ensures **collisions** work correctly **without applying physics forces** to the object itself.

**3. Dynamic Objects**
 Objects that are fully affected by physics (e.g. gravity, forces, drag, and collisions). 
  - **Use case:** Characters, balls, projectiles—anything that moves naturally in response to forces or collisions.

Choosing the right type is important. Use `Static` for immovable objects, `Kinematic` when you need scripted motion without full physics control, and `Dynamic` for physics-driven objects.

By understanding these three types, you’ll know when to use `Rigidbody`, how your objects will interact, and how to design movement and collisions in your scene.

### Accessing Rigidbody
To move a GameObject with physics, you need to work with its `Rigidbody` component. Unlike the `Transform` (which every GameObject has by default), not all GameObjects automatically include a `Rigidbody`; you have to **add it yourself** in the Inspector.

That means you can't just call it directly like we do with:

`transform.position` 

Instead, we have to make a **reference** to store the `Rigidbody` attached to this object:
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

The type isn't limited to built-in Unity components; you can also use it for any custom component (i.e., class) you create and attach to a GameObject.

Once you store the reference in a variable, you can use it to **access public properties** (like velocity, mass, or drag) or **call methods** on that component.

#### What if No Component is Found?

If `GetComponent<T>()` doesn’t find the requested component on the GameObject, it returns **null**.

This means any logic that tries to access the component (e.g., `_rigidBody.linearVelocity = ...`) will throw a `NullReferenceException`.

To prevent this, it’s good practice to **check for null** before using the reference:

```csharp
if (_rigidBody == null)
{
    return; // Exit the method if the Rigidbody is missing
}
else
{
    _rigidBody..linearVelocity = Speed * Direction;
} 
```

If the **component is essential for the class to work**, you can enforce its presence using Unity’s `[RequireComponent]` attribute. 
The attribute is placed before the class definition, and the type of component to be required is defined. Multiple `RequireComponent` attributes can be included.
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
Before we can move our game object, we need to get access to the `Rigidbody` component as mentioned above. 
To do this, we need to update our `MoveRigidbody` class to the following: 

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
        
        // Determine if the object should start moving
        _isMoving = _moveOnAwake;
    }//end Awake()

    // ... rest of the class

}
```
### Enforce Physics for Dynamic Objects
Earlier, we discussed the differences between the types of Unity physics objects. If we want this GameObject to interact properly with physics, it must be **dynamic** and not **kinematic**.

By default, the `isKinematic` property is set to `false` when a `Rigidbody` is added. However, since our functionality **depends on the `Rigidbody` being dynamic**, we can't be sure that the level designer has not manually changed this value. Whether intentional or not, we can avoid issues by enforcing it in `Awake()` after obtaining the Rigidbody reference:

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

<br>

> **✔️ CHECK POINT**
> 
> Save your script, switch back to the Unity editor, and press **Play** to test the changes in action.

<br>

---

## Physics Movement 

By default, if two objects with physics are placed above each other in the scene, gravity will immediately take effect, and they will fall naturally. But Unity’s physics system does more than just apply **gravity**; it also handles **collisions, momentum, and other forces**:

- **Collisions:** When two Rigidbody objects collide, Unity calculates how they should bounce, slide, or stop based on their mass, velocity, and drag. For example, a heavy ball hitting a lighter cube will push the cube more than the other way around.

- **Momentum:** Moving objects carry momentum, which affects how they interact during collisions. A fast-moving object will have a stronger impact than a slow one.

- **Forces and Drag:** Objects can be affected by forces like wind, explosions, or friction. Drag (air resistance) slows them down naturally over time.

- **Continuous Interaction:** Physics objects automatically respond to each other and the environment. For instance, stacking boxes will cause them to settle realistically, topple over, or slide if one is pushed.

Once a Rigidbody is added, Unity simulates all of these physics behaviors automatically, based on the settings applied in the Rigidbody. This allows for realistic interactions between objects without requiring you to calculate collisions or momentum manually.

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

<br>

> [!NOTE]
> In physics, **speed** is just **how fast an object moves**, while **velocity** includes both **speed and direction**.
> Unity’s `Rigidbody.linearVelocity` follows this definition, as it is a `Vector3`. Storing both **how fast and which way the object is moving**.

<br>

<br>

>[!IMPORTANT]
>Starting with Unity 6.0, the `Rigidbody.velocity` property has been renamed to **`Rigidbody.linearVelocity`**. This rename is meant to clarify that it measures **linear motion** only, distinguishing it from **angularVelocity** and making the API more intuitive.

<br>
<br>

> [!WARNING]
>
> The example above uses `Rigidbody.linearVelocity` to demonstrate the syntax for accessing the property. However, in practice, you **should never use the class name directly**. You always access the velocity through a **reference to the component**.
> In our case, that reference is `_rigidBody.linearVelocity`. The reference name could vary; some developers use `_rb`, but it’s **best practice to be explicit** in your variable names. For clarity, `_rigidBody` is preferred here.
> 
<br>


### When to Use Velocity Over Transform
Let's say that you are creating a **arcade-style spaceship game**, , where players move at a **constant speed** and respond **instantly to input**.

If we were to use the standard `transform.position`, the object would move exactly as commanded, but it would **ignore collisions and physics**, allowing the object to clip through walls, obstacles, or other players.

<br>

> [!Note]
> **Gravity should be disabled** in this scenario. Otherwise, the object might accelerate downward or behave unpredictably due to other physics forces.

<br>

Using `Rigidbody.velocity` in this manner has both advantages and disadvantages:
- ✅ Directly control speed and direction in a **predictable, immediate** way.
- ✅ **Maintain automatic physics interactions**, like collisions and slope handling.
- ✅ Move objects **continuously** without manually updating the position every frame.
- ❌ **Ignored physics behaviors** (e.g. momentum, acceleration, and mass),  unless explicitly implemented.
- ❌ **External forces don’t automatically affect movement**, such as drag or push.
- ❌ Movement is **slightly less realistic** compared to full physics simulation.
  

Implmenting velocity in this manner provides players with a **responsive, snappy control experience**. Ideal for fast-paced, arcade-style gameplay in which you don’t need additional physics behaviors, like momentum, drag, or complex mass interactions, which can require more processing. This approach keeps **movement predictable, efficient, and easy to control**, while still benefiting from collisions and basic physics.

---

## Update `Move()`
To move our objects with velocity, we need to rewrite the `Move()` method to use the Rigidbody's velocity instead of `transform.position`.

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
    _rigidBody.linearVelocity = Speed * Direction;
}
```
#### Key Changes
**1. Remove the `_isMoving` flag** 
  - If the game object's velocity is greater than zero, it will be moving; we do not need a flag to check this.
  - Instead, we would check the velocity value:  `if (!_rigidBody.linearVelocity == Vector3.zero)`
  - This check was nullified here since we do not need to check the update for this condition any longer.

<br>
> [!NOTE]
>
> Remember to remove the declaration for the flag `_isMoving` at the top of the class too.
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
        //Check if the object moves on start
        if (_moveOnStart)
        {
            Move();

        }//end if(_moveOnStart)
        
    }//end Start()
    
```

Since Rigidbody velocity continues automatically once set, **we no longer need to call `Move()` in `Update()` every frame**. The Rigidbody will keep moving according to the assigned velocity, and Unity handles all the physics calculations during FixedUpdate steps.

However, the velocity is still subject to all other physics forces (e.g., gravity, drag, collisions, physics materials, joints). These can alter or cancel the motion over time.

<br> 

> [!IMPORTANT]
> Before continuing, try playing the scene in the Unity editor, with gravity enabled and disabled:
> - With gravity on, the object will be pulled downward, which may make it appear to “stop” in the original direction.
> - With gravity off, the object will continue moving indefinitely in the assigned direction, unless another force interferes.
>
> **KEEP GRAVITY OFF**
> 
> While this isn’t physically accurate, disabling gravity ensures that once we apply velocity, the object will continue moving without additional code. This makes testing and prototyping easier. Later, when we introduce a controller, it will take responsibility for applying input to adjust or stop the object’s velocity.

<br>

### Modify the `Update()`

If the call to `Move()` were the only thing happening in `Update()`, we could remove the method entirely. However, since `Update()` also performs the `_enableEditorTesting` check, we’ll keep it and simply remove the condition that calls `Move()`.

**2. Modify** the `Update()` method: 

```csharp
    private void Update()
    {
        
#if UNITY_EDITOR
        if (_enableEditorTesting)
        {
            RunMovementTest();

        } //end if(_enableEditorTesting)
#endif

    }//end Update()

```

<br>

> **✔️ CHECK POINT**
> 
> Save your script, switch back to the Unity editor, and press **Play** to test the changes in action.

<br>
---
## Updating Speed or Direction

---

## Updating Speed or Direction

At this point, whenever the `Move()` method is called, the object will continue moving as long as the Rigidbody’s velocity is not zero. Whenever `Move()` is called, we can pass new **speed** and **direction** values as arguments to update the Rigidbody’s velocity.

However, if **Speed** or **Direction** properties change during gameplay, for example, via a power-up or player input, the Rigidbody’s velocity won't automatically update, because `Move()` is no longer being called every frame.

Since `Move()` relies on physics, we could call it in `FixedUpdate()`, which is Unity’s **physics-timed update loop**. `FixedUpdate()` runs at a **consistent, fixed interval** (default 0.02 seconds, or 50 times per second), ensuring physics calculations like velocity, forces, and collisions remain stable and predictable, regardless of the frame rate.

That said, **we don't need to call `Move()` every physics step**, only when **Speed** or **Direction** actually change. To optimize this, we can create private fields to store the current speed and direction. Then, in `FixedUpdate()`, we check if either property has changed, and only call `Move()` when an update is necessary.

**1. Create fields** for storing the current speed and direction

```csharp
    // Store the last applied values for optimization
    private float _currentSpeed;
    private Vector3 _currentDirection;
```
**2. Set Current Values** for  speed and direction in the `Start()`
While our **Speed** and **Direction** properties are initialized in `Awake()`, we wait until `Start()` to copy those values into `_currentSpeed` and `_currentDirection`. The reason is the order of execution:

- `Awake()` runs very early in the Unity lifecycle and is responsible for preparing the object. Here, we make sure the properties are set and validated against their serialized field values from the Inspector.
- By the time `Start()` runs, we can be confident those values are finalized. Assigning them to our "current" fields at this point gives us a reliable snapshot of the starting speed and direction as the game begins.

In short, `Awake()` ensures the properties are correct, and `Start()` records those validated values as the object’s initial state for gameplay.

```csharp
    // Start is called once before the first Update
    private void Start()
    {
        
        //Store current speed and direction
        _currentSpeed = Speed;
        _currentDriection =Direction;
        
        //Check if the object moves on start
        if (_moveOnStart)
        {
            Move();

        }//end if(_moveOnStart)
        
    }//end Start()

```

**2. Create `FixedUpdate()`** and check for Speed and Direction changes

```csharp
    //Called at fixed intervals (i.e., physic steps) 
    private void FixedUpdate()
    {
        // Only update velocity if speed or direction changed
        if (Speed != _currentSpeed || Direction != _currentDirection)
        {
            Move(Direction, Speed);

        }//end if(current speed/direction)

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

        //Record the current direction and speed
        currentDirection = Direction;
        currentSpeed = Speed;
        
        // Move the GameObject using Rigidbody velocity
        _rigidBody.linearVelocity = Speed * Direction;

    }//end Move()

```

<br>

> **✔️ CHECK POINT**
> 
> Save your script, switch back to the Unity editor, and press **Play** to test the changes in action.

<br>

---
## Stopping Objects with Velocity 

To stop our game object, all we need to do is update our `Stop()` method to set the Rigidbody's velocity to zero. 

**1. Update `Stop()`** and set velocity

```csharp
    /// <summary>
    /// Stops the object's movement immediately by zeroing its Rigidbody velocity.
    /// </summary>
    public void Stop()
    {
        // Immediately halts motion
        _rigidBody.linearVelocity = Vector3.zero;  
        
    }//end Stop()
```

This is an **immediate stop** and in some cases could be a bit jarring, especially in a racing-style game. If you want the object to slow down naturally, you would need a different approach (like a gradual deceleration or “brake”).

<br>

> **✔️ CHECK POINT**
> 
> Save your script, switch back to the Unity editor, and press **Play** to test the changes in action.

<br>

---
# 🎉 New Achievement: Rigidbody Movement

We’ve now created a MoveRigidbody component that moves our GameObject with velocity.

```csharp
using UnityEngine;

[RequireComponent(typeof(Rigidbody))] 
public class MoveRigidbody : MonoBehaviour
{
    
    // Reference to the object's Rigidbody component
    private Rigidbody _rigidBody;
    
    // Maximum speed allowed
    private const float MAX_SPEED = 10f;
    
    // Store the last applied values for optimization
    private float _currentSpeed;
    private Vector3 _currentDirection;

    
    // Serialized fields for initial values
    
    [Header("GENERAL SETTINGS")]
    
    [SerializeField]
    [Range(0f, MAX_SPEED)]
    [Tooltip("Speed of the object’s movement. Cannot exceed maxiumum speed.")]
    private float _speed = 5f;
    
    [SerializeField]
    [Tooltip("How quickly the object slows down when braking.")]
    private float _deceleration = 5f;
   
    // Direction of movement
    [SerializeField]
    private Vector3 _direction = Vector3.right; 
    
    [SerializeField]
    [Tooltip("Should the object be moving on start?")]
    private bool _moveOnStart = true;
    
    
   
    
#if UNITY_EDITOR
    [Header("FOR TESTING ONLY")]
    [SerializeField] 
    private bool _enableEditorTesting = false;

    //Enum for list of possible actions to test 
    private enum TestAction
    {
        None,   // No action selected
        Move,   // Run Move() test
        Stop    // Run Stop() test

    }

    [SerializeField]
    [Tooltip("Select which action to test in the Editor.")]
    private TestAction _testAction = TestAction.None;

#endif
    
    
    // Public properties with encapsulation
    
    public float Speed
    {
        get => _speed; 
        set => _speed = Mathf.Clamp(value, 0f, MAX_SPEED);
    }
    
    public Vector3 Direction
    {
        get => _direction;
        set => _direction = value.normalized;
    }
    
    

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

    }//end Awake()
    
    // Start is called once before the first Update
    private void Start()
    {
        //Store current speed and direction
        _currentSpeed = Speed;
        _currentDirection = Direction;
        
        //Check if object moves on start
        if (_moveOnStart)
        {
            Move();

        }//end if(_moveOnStart)
        
    }//end Start()
    

    // Update is called once per frame
    private void Update()
    {
        
#if UNITY_EDITOR
        if (_enableEditorTesting)
        {
            RunMovementTest();

        } //end if(_enableEditorTesting)
#endif
    }//end Update()
    
    //Called at fixed intervals (i.e. physic steps) 
    private void FixedUpdate()
    {
        // Only update velocity if speed or direction changed
        if (Speed != _currentSpeed || Direction != _currentDirection)
        {
            Move(Direction, Speed);
        }


    }//end FixedUpdate()
    
    
     
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
        _currentDirection = Direction;
        _currentSpeed = Speed;

        // Move the GameObject using Rigidbody velocity
        _rigidBody.linearVelocity = Speed * Direction;

    }//end Move()
    
    /// <summary>
    /// Stops the object's movement immediately by zeroing its Rigidbody velocity.
    /// </summary>
    public void Stop()
    {
        // Immediately halts motion
        _rigidBody.linearVelocity = Vector3.zero;  
        
    }//end Stop()
    

    
    
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
                _testAction = TestAction.None; // Reset after running
                break;

            case TestAction.Stop:
                Debug.Log("Testing Stop");  
                Stop();
                _testAction = TestAction.None; // Reset after running
                break;


            case TestAction.None:
            default:
                // Do nothing
                break;

        }//end switch(_testAction)

    }//end RunMovementTest()
#endif
    
}//end MoveRigidbody

```

While `Rigidbody.linearVelocity` gives us **snappy, arcade-style controls**, using **forces** (`Rigidbody.AddForce`) becomes useful when we want **gradual acceleration and deceleration, momentum**, or more **realistic interactions** with collisions and slopes. Velocity works well for **fast, predictable movement**, while **force-based movement** allows for **emergent, physics-driven behavior**.

---









