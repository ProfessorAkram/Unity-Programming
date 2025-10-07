# Refactoring `MoveTransform`
Almost **all code benefits from refactoring** at some point, whether to improve readability, maintainability, or flexibility. By revisiting our MoveTransform component, we can make it **cleaner**, **more modular**, and capable of handling a wider range of scenarios without changing its core behavior.

In the case of our `MoveTransform` component, we address the features mentioned above. 

## Create Properties for Speed and Direction
Currently, our MoveTransform component stores data (variables) directly. In programming, this is known as a **field**.
```csharp
private float _speed = 5f;
private Vector3 _direction = Vector3.right;
```
These fields are marked **private**, which means other scripts **cannot directly access or override them**. This is good practice for **encapsulation**, but it also limits flexibility when we want to safely read or modify these values from other scripts.

To solve this, we use **properties**, special methods that behave like fields but allow us to **control reading**, **writing**, and **validation**.

Let’s **add these properties** to our class so that we can safely access and modify the speed and direction of our MoveTransform component without exposing the underlying fields:

```csharp
public float Speed
{
   get => _speed; 
   set => _speed = value;
}

public Vector3 Direction
{
   get => _direction; 
   set => _direction = value;
}
```
> [!NOTE]
> In C#, when a method, property, or other member only needs to execute a **single expression**, you can use **expression-bodied member syntax**. This lets you replace the usual `{ }` block with `=>` for a more concise style.
> For example, instead of writing: 
> ```
>  public int Square(int x)
> {
>    return x *x;
> }
> ```
>  You can write it as:
>  ```
> public int Square(int x) => x * x;
>  ```
> 
> It makes your code concise and readable, but if you need **multiple statements or more complex logic**, you still need the full `{ }` block. 


By adding these **properties**, other parts of our game can interact with the `MoveTransform` component **safely and dynamically**.

For example:

- The UI could read the current `Speed` and `Direction` to display them on-screen.  
- A boundary system could reverse the `Direction` when the object hits a wall.  
- A power-up could temporarily double the `Speed` without directly modifying the private field.

### Updating the `Move()`
Since we now have properties for `Speed` and `Direction`, we should also update the `Move()` method to use them instead of the private fields. This ensures that any validation or dynamic changes applied through the properties are respected during movement.

Currently, our `Move()` method implements the following:

```csharp
transform.position += _speed * Time.deltaTime * _direction;
```
**Update** the `Move()` method applying the properties instead of fields: 

```csharp
transform.position += Speed * Time.deltaTime * Direction;
```
> [!TIP]
> This is a small but important refactor that keeps your class **consistent**, **encapsulated**, and **flexible**.

This change guarantees that the object moves according to the **current property values**, not just the initial field values. It also makes the movement logic fully compatible with external scripts that might change the speed or direction at runtime. 

<br>

> **✔️ CHECK POINT**
> 
> Save your script, switch back to the Unity editor, and press **Play** to test the changes in action.

<br>

---

## Add Validation for Speed and Direction

Once we have **public properties** for **speed** and **direction**, we may want to **prevent invalid or problematic values** that could break gameplay or produce inconsistent movement.

### Normalizing Direction
At the moment, there are no constraints on the **direction** value that can be assigned. This means a designer or script could set a vector of any length, not just a unit vector.

For example, suppose the designer sets the value **in the editor** to
```
Direction [3, 0, 4]
```
Although our variable is named **Direction**, we are actually assigning a **vector**, which has both **direction** and **magnitude**. The magnitude of a vector is calculated as:

$\text{magnitude} = \sqrt{x^2 + y^2 + z^2}$

In this example, the magnitude would be:
$\sqrt{3^2 + 0^2 + 4^2} = 5$

If we multiply this vector directly by a speed value in the movement calculation:

```csharp
transform.position += Speed * Time.deltaTime * Direction;
```

The object would move **5 times farther in that frame than intended**, because the vector's length scales the movement. This makes movement inconsistent and breaks the assumption that `Speed` represents units per second.

This inconsistency can be avoided by **normalizing the direction vector** when it’s assigned. Normalization converts any vector to a **unit vector** (magnitude = 1) while preserving its direction.

**Update** the `Direction` property to **normalize** the set value: 

```csharp
public Vector3 Direction
{
    get => _direction;
    set => _direction = value.normalized;
}
```

With this approach:

- The vector always has a **magnitude of 1**.
- `Speed` now fully controls **how far the object moves per frame**.
- The direction only determines the **path** of movement, not the distance traveled.

By enforcing normalization in the property setter, we ensure **consistent and predictable movement**, regardless of the original vector assigned.

<br>

> **✔️ CHECK POINT**
> 
> Save your script, switch back to the Unity editor, and press **Play** to test the changes in action.

<br>

### Capping Speed
Just as we normalize our **Direction** to ensure consistent movement, we also need to **validate the** `Speed` to prevent invalid or excessively high values that could break gameplay or feel unbalanced. For example, we might want to ensure that speed is **never negative and does not exceed a certain maximum**.

To enforce this, we define a **maximum allowed speed**. This should be a **private field** so it remains encapsulated, but we can **serialize** it to make adjustments easy in the Inspector:

```csharp
// Max speed set in the Inspector
[SerializeField] 
private float _maxSpeed = 10f; 
```
<br>

>[!NOTE]
> We don’t need a property for `_maxSpeed` because it's only used internally for validation. Other scripts don’t need to read or write it directly.

<br>

#### Testing Speed Value

Now that we have `_maxSpeed`, we **could** create a method to **validate or cap the speed** whenever it’s set. In this case, we want to ensure the speed stays within a valid range, **never below 0 and never above** `_maxSpeed`.

For example: 
```chsarp
// ❌ Less Efficient
    private float ValidateSpeed(float speed)
    {
       if (Speed < 0f) Speed = 0f;
       else if (Speed > _maxSpeed) Speed = _maxSpeed;

    }//end ValidateSpeed()

```

This approach works, but we can simplify it further. Since **speed validation only occurs when assigning the property**, we don’t need a separate method — the check can be performed **directly in the setter**.

Unity's `Mathf.Clamp()` method allows us to **condense the conditional logic into a single line**. It returns the value constrained within a specified range, using three parameters:

- The value to limit.
- The minimum allowed value.
- The maximum allowed value.

**Update** the `Speed` property to ensure our speed always stays between `0` and` _maxSpeed`.

```cshapr
 // ✅ More Efficient
public float Speed
{
   get => _speed; 
   set => _speed = Mathf.Clamp(value, 0f, _maxSpeed);
}
```
This ensures that `_speed` **always stays within the valid range**, and you don’t need a separate validation method or manual checks.

<br>

> **✔️ CHECK POINT**
> 
> Save your script, switch back to the Unity editor, and press **Play** to test the changes in action.

<br>

### Validating Field
Currently, our speed and direction validation only occurs when the **properties** are set. However, in the Unity Editor, a designer can modify the **fields** directly in the Inspector.

We can ensure proper validation at initialization by assigning the fields to their respective properties in the `Awake()` method. This triggers the property setters, applying any validation logic when the game object is created:

```csharp
private void Awake()
{
    // Validate initial speed and direction via properties
    Speed = _speed;
    Direction = _direction;

} // end Awake()
```

This approach guarantees that values set in the Inspector are **clamped and normalized** correctly at the start of the game.

<br>

> **✔️ CHECK POINT**
> 
> Save your script, switch back to the Unity editor, and press **Play** to test the changes in action.

<br>

## Visual Cues 
While we have implemented validation for our fields, these constraints may not be immediately obvious in the Unity Editor. For example, if a level designer sets `Direction` to `(3, 0, 4)` or `Speed` to `20`, the property setters will automatically correct the values when the game runs. However, the designer might not understand why the values suddenly changed.

One way to improve clarity is to use Unity's `[Tooltip]` attribute on fields in the Inspector. This displays a small piece of text when you hover your mouse over the field, providing information on its purpose, valid values, or expected behavior.

```csharp
     [SerializeField]
     [Tooltip("Direction of movement. Will be normalized automatically to ensure consistent movement.")]
     private Vector3 _direction = Vector3.right;

     [SerializeField]
     [Tooltip("Speed of the object’s movement. Cannot exceed maximum speed.")]
     private float _speed = 5f;
     
```

> [!TIP]
> Because `[Tooltip]` already describes the variable, adding an inline comment would be redundant.


<br>

> **✔️ CHECK POINT**
> 
> Save your script, switch back to the Unity editor, and press **Play** to test the changes in action.

<br>


### Tighter constraints
While our `[Tooltip]` provides some visual context about the constraints on the `_speed` variable, and the mere inclusion of `_maxSpeed` should suggest a limit, these cues may not be **immediately obvious to the level designer**.

With that said, we might want to enforce **tighter constraints** while simultaneously making valid speed options more apparent to the designer.


To make valid speed options clearer and enforce tighter constraints, we can convert `_maxSpeed` to a **constant** and apply Unity's `[Range(min, max)]` attribute to `_speed`.

> [!NOTE]
> We can set `MAX_SPEED` as the upper limit in `[Range(0f, MAX_SPEED)]` because it is a **constant**. Unity requires `[Range]` values to be fixed at compile time, so only constants (not regular variables) can be used here.

**Update** the following fields: 
```csharp
// Maximum speed allowed
private const float MAX_SPEED = 10f;

[SerializeField]
[Range(0f, MAX_SPEED)]
[Tooltip("Speed of the object’s movement. Cannot exceed maximum speed.")]
private float _speed = 5f;

```

> [!CAUTION]
> **DO NOT** forget to also update the `Speed` property, after switching `_maxSpeed` to the `MAX_SPEED` **constant**.
> `set => _speed = Mathf.Clamp(value, 0f, MAX_SPEED);`

---

## Extending Move()

In our current design, the `Move()` method is **private**, meaning only the script itself can call it. This is a safe default because it enforces **encapsulation**: the object controls how it moves, without interference from outside scripts.

However, in many games, other objects or events need to control movement. For example:

- A player pulls a lever that triggers a platform to move.  
- A power-up temporarily boosts an enemy's speed.  
- A boundary or collision triggers a change in direction.  

Keeping `Move()` private avoids tight dependencies by default. However, to allow external scripts to influence movement safely and dynamically, we can make the method **public**.

> [!TIP]
>  Start with **private** methods to **maintain self-contained behavior**. When there’s a clear need for **external control**, you can expose the method through parameters or carefully designed **public** access.

### Accepting Parameters
Updating `Move()` to be public makes it accessible to other scripts. While our public properties allow external scripts to modify `Speed` and `Direction`, there are situations where it’s more convenient or efficient to pass temporary values directly to the method—such as responding to an event, applying a one-off speed adjustment, or triggering movement with a specific direction.

To support this, we can modify the original private `Move()` method to accept `direction` and `speed` as parameters:

```csharp
/// <summary>
/// Moves the object in a specified direction at a specified speed.
/// </summary>
/// <param name="direction">The direction to move the object.</param>
/// <param name="speed">The speed at which the object should move.</param>
public void Move(Vector3 direction, float speed)
{
    // Assign parameters to properties
    Direction = direction;
    Speed = speed;

    //Debug.Log("Direction: " + Direction);
    //Debug.Log("Speed " + Speed);

    // Move the GameObject
    transform.position += Speed * Time.deltaTime * Direction;

}//end Move()

```
> [!TIP]
> When dynamically setting variables, as in this case, it’s a good idea to verify that the **values are updating correctly**. Using `Debug.Log` here can help confirm that the `direction` and `speed` passed to the method are being applied as expected. Once you’ve verified that the values are correct and the movement behaves as intended, you should **remove or comment out the** `Debug.Log` **statements** to avoid unnecessary console output and potential performance overhead.


#### Why assign parameters to the properties?

Even though the values are temporary inputs for this method call, assigning them to the properties ensures that:

- Validation logic in the property setter is automatically applied (e.g., `Speed` cannot exceed `_maxSpeed`).

- The object’s internal state is updated, so future `Move()` calls reflect the most recent values.

- The class remains consistent, keeping temporary changes in sync with the persistent properties without introducing additional state variables.

<br>

> **✔️ CHECK POINT**
> 
> Save your script, switch back to the Unity editor, and press **Play** to test the changes in action.

<br>  
  

### Handling Default Values

The method above works, but there’s a drawback: every time we call `Move()`, we must supply both parameters, even if we just want to use the current `Speed` or `Direction`. This can become repetitive and violates the KISS principle (“Keep It Simple, Stupid”).

To make the method more flexible, we can use nullable parameters (`Vector3?` and `float?`) and fall back to the object’s properties if no value is provided. We also introduce two **local variables** (`moveDirection` and `moveSpeed`) to store the actual values being used in this frame:

```csharp
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

    //Debug.Log("Direction: " + Direction);
    //Debug.Log("Speed " + Speed);

   // Move the GameObject using the resolved frame values
    transform.position += Speed * Time.deltaTime * Direction;

}//end Move()
```

#### How it Works 

- **Nullable Parameters (`Vector3?` and `float?`)**
   - The `?` allows the parameter to be nullable, so the caller can omit it.
   - If `null` is passed (or the argument is omitted), the method uses the current property value.
   - The `??` null-coalescing operator selects the parameter if it exists; otherwise, it falls back to the property.

- **Local Variables**
   - `moveDirection` and `moveSpeed` store the resolved values for this frame.
   - This ensures the movement calculation is clear, readable, and frame-consistent.

- **Updating the Properties**
   - Assigning the resolved values to `Direction` and `Speed` applies validation automatically.
   - Keeps the object’s state consistent for subsequent method calls.

- **Movement Calculation**
   - Uses the **local variables** (`moveDirection` and `moveSpeed`) to guarantee that the movement reflects exactly the values we resolved for this frame.
   - This ensures **frame-rate independent movement** and avoids unintended side effects from property access.

This approach gives us the best of both worlds:

- External scripts can influence movement without bypassing validation.  
- Movement can respond to temporary or dynamic inputs without affecting the object’s persistent state unexpectedly.  
- The class remains modular, flexible, and easy to maintain.

<br>

> **✔️ CHECK POINT**
> 
> Save your script, switch back to the Unity editor, and press **Play** to test the changes in action.

<br>

---

## Adding Movement Control Flags
In many games, we don’t always want an object to start moving immediately or move continuously without restriction. For example:

- An enemy might wait before chasing the player.
- A moving platform might start only when a button is pressed.
- A power-up might temporarily freeze player movement.

To manage these situations, we can introduce **control flags** — simple **Boolean variables** that act as switches to turn certain behaviors **on or off**. Flags provide flexible control without rewriting code. Instead of hard-coding when movement should occur, we can toggle these flags based on game logic.

In our `MoveTransform` component, we will add two flags:

- **_moveOnAwake** → Determines whether the object starts moving automatically when initialized.
- **_isMoving** → Tracks whether the object is currently moving. This is updated whenever `Move()` or `Stop()` is called.

```csharp
[SerializeField]
[Tooltip("Should the object be moving on initialization?")]
private bool _moveOnAwake = true;

// Runtime movement flag
private bool _isMoving;

```
> [!NOTE]
> We do not set a value for `_isMoving` here. Instead, we will determine if the object should start moving at initialization using the value of `_moveOnAwake`.

### Testing Movement Control Flags
Now that we have added our flags, we can use them to control when the object is allowed to move.

```csharp
   private void Awake()
    {
        // Validate initial speed and direction via properties
        Speed = _speed;
        Direction = _direction;
        
        // Determine if the object should start moving
       _isMoving = _moveOnAwake;
      
    }//end Awake()

    private void Update()
    {
       if(_isMoving)
       {
         Move();

       }//end if(_isMoving)

    }//end Update()


```

<br>

> **✔️ CHECK POINT**
> 
> Save your script, switch back to the Unity editor, and press **Play** to test the changes in action.

<br>
  
### Updating `Move()` with `_isMoving`
With the current flags implemented, `_isMoving` is only true if `_moveOnAwake` is `true`.

But what if we want the object to remain still at the start and have an external script trigger movement? In that case, we need to ensure that whenever `Move()` is called, `_isMoving` is set to `true`.

```csharp
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

        //Debug.Log("Direction: " + Direction);
        //Debug.Log("Speed " + Speed);

        // Flags the object as moving
        _isMoving = true;

        // Move the GameObject using the resolved frame values
        transform.position += Speed * Time.deltaTime * Direction;

    }//end Move()
```
This ensures that any external call to `Move()` triggers the object to **keep moving every frame**.

<br>

> **✔️ CHECK POINT**
> 
> Save your script, switch back to the Unity editor, and press **Play** to test the changes in action.

<br>

---
## Implement a Stop() method
Once an object is moving, there is no way to stop it. At first, you might think we could simply set the speed to zero to halt movement. However, this approach has a drawback: if `Move()` is called again later, any new speed values would immediately override the zero, and the object would start moving again unintentionally.

Instead, we can use our `_isMoving` flag to control movement. By setting `_isMoving` to `false`, we tell the object to **stop updating its position**, regardless of its speed. This approach separates the concept of _“how fast the object moves” _ from _“whether it is moving at all,”_ giving us cleaner and more predictable control over movement.

```csharp
    /// <summary>
    /// Stops the object's movement by updating the movement flag.
    /// </summary>
public void Stop()
{
    // Flags the object as stopped
    _isMoving = false;

}//end Stop()
```

<br>

> **✔️ CHECK POINT**
> 
> Save your script, switch back to the Unity editor, and press **Play** to test the changes in action.

<br>

---
# 🎉 New Achievement: Refactored Movement

Congratulations! You've **refactored** the `MoveTransform` component, and with these improvements, the object now:

- Implements properties with validation for `Speed` and `Direction`.

- Moves automatically if `_moveOnAwake` is `true`.

- Can be started and stopped dynamically via `Move()` and `Stop()`.

- Maintains `_speed` and `_direction` limits to prevent unintended behavior.

These changes make the movement system more **robust**, **predictable**, and **easier to extend** in future gameplay scenarios.

```csharp
using UnityEngine;

public class MoveTransform : MonoBehaviour
{
    // Serialized fields for initial values

    // Maximum speed allowed
    private const float MAX_SPEED = 10f;

    [SerializeField]
    [Range(0f, MAX_SPEED)]
    [Tooltip("Speed of the object’s movement. Cannot exceed maximum speed.")]
    private float _speed = 5f;

    [SerializeField]
    [Tooltip("Direction of movement. Will be normalized automatically to ensure consistent movement.")]
    private Vector3 _direction = Vector3.right;
    
    [SerializeField]
    [Tooltip("Should the object be moving on initialization?")]
    private bool _moveOnAwake = true;

    // Runtime movement flag
    private bool _isMoving;
    
    
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
        
        //Set GameObject's initial position
        transform.position = Vector3.zero;
        
        // Determine if the object should start moving
        _isMoving = _moveOnAwake;
    }//end Awake()

    // Update is called once per frame
    private void Update()
    {
        if(_isMoving)
        {
            Move();

        }//end if(_isMoving)
    }//end Update()
     
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

        //Debug.Log("Direction: " + Direction);
        //Debug.Log("Speed " + Speed);
        
        // Flags the object as moving
        _isMoving = true;

        // Move the GameObject using the resolved frame values
        transform.position += Speed * Time.deltaTime * Direction;

    }//end Move()
    
    /// <summary>
    /// Stops the object's movement by updating the movement flag.
    /// </summary>
    public void Stop()
    {
        // Flags the object as stopped
        _isMoving = false;

    }//end Stop()

}//end MoveTransform
```
### ⚠️ But wait! 

While our `MoveTransform` is complete, we don’t actually have a way to test the `Stop()` functionality. 

The `MoveTransform` component is only responsible for movement behavior, not controlling the object. A controller class (like PlayerController or EnemyAI) would normally handle when to call `Move()` or `Stop()`.

So what if you’re working with a team, and the controller isn’t built yet? How do you test your component without writing unnecessary extra code?

---

**[<< Return to Basic Movement tutorial](basic-movement.md)** | **[Continue to Editor-Only Debugging tutorial >>](editor-only-debugging.md)**