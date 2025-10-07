# Velocity-Base Movement
Before we dive into implementing movement, it's important to understand what **velocity** actually means. Velocity tells us both **how fast an object is moving** and **in which direction**, making it a fundamental concept for controlling movement in games. By grasping this idea, we can see why simply setting a Rigidbody’s velocity can instantly move an object in the desired direction.

## Understanding Velocity

**Velocity** describes how fast and in which direction an object moves; essentially displacement over time, measured in meters per second **(m/s)**.
Its basic equation is:

$$
v = \frac{\Delta x}{\Delta t}
$$

Where:  

- **$$v$$** = velocity (vector, includes direction)  
- **$$\Delta x$$** = change in position (displacement or speed)  
- **$$\Delta t$$** = change in time   

**For Example**

If a cube moves **5 meters to the right in 1 seconds**, its velocity is:

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
Let's say that you are creating an **arcade-style spaceship game**, where players move at a **constant speed** and respond **instantly to input**.

If we were to use the standard `transform.position`, the object would move exactly as commanded, but it would **ignore collisions and physics**, allowing the object to clip through walls, obstacles, or other players.

<br>

> [!Note]
> **Gravity should be disabled** in this scenario. Otherwise, the object might accelerate downward or behave unpredictably due to other physics forces.

<br>

Using `Rigidbody.velocity` in this manner has both advantages and disadvantages:
- ✅ Directly control speed and direction in a **predictable, immediate** way.
- ✅ **Maintain automatic physics interactions**, like collisions and slope handling.
- ✅ Move objects **continuously** without manually updating the position every frame.
- ❌ **Ignored physics behaviors** (e.g., momentum, acceleration, and mass),  unless explicitly implemented.
- ❌ **External forces don’t automatically affect movement**, such as drag or push.
- ❌ Movement is **slightly less realistic** compared to full physics simulation.
  

Implementing velocity in this manner provides players with a **responsive, snappy control experience**. Ideal for fast-paced, arcade-style gameplay in which you don’t need additional physics behaviors, like momentum, drag, or complex mass interactions, which can require more processing. This approach keeps **movement predictable, efficient, and easy to control**, while still benefiting from collisions and basic physics.

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

    // Flags the object as moving
    //_isMoving = true;
    
    // Move the GameObject using Rigidbody velocity
    _rigidBody.linearVelocity = Speed * Direction;
}
```
#### Key Changes
**1. Comment out the `_isMoving` flag** 
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
        
        //Check if the object moves on start
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
    
    //Called at fixed intervals (i.e., physic steps) 
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

        //Record the current direction and speed
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
                break;

            case TestAction.Stop:
                Debug.Log("Testing Stop");  
                Stop();
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
    
}//end MoveRigidbody

```

While `Rigidbody.linearVelocity` gives us **snappy, arcade-style controls**, using **forces** (`Rigidbody.AddForce`) becomes useful when we want **gradual acceleration and deceleration, momentum**, or more **realistic interactions** with collisions and slopes. Velocity works well for **fast, predictable movement**, while **force-based movement** allows for **emergent, physics-driven behavior**.

---

**[<< Return Rigidbody tutorial](rigidbody.md)** | **[Continue to Force Movement tutorial >>](force-movement.md)**






