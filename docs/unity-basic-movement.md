# Basic Movement in Unity 
When designing games, one of the most common tasks is moving an object from one place to another. For example, imagine you want a projectile, like a fireball, to travel across the screen from point A to point B. How can we make this happen in Unity?

The answer lies in one of the most important components every GameObject has: the **Transform** component.

## The Transform Component

Every GameObject in Unity automatically has a **Transform**. You can think of it as the object's "address" in the game world; it tells Unity where the object is, how it's rotated, and how large it is.

The Transform has three main properties:

- **Position** - where the object is located.  
- **Rotation** - the object's orientation (how it's turned).  
- **Scale** - the size of the object.  

## Understanding 3D Positions with Vector3

In Unity, every object’s position in 3D space is represented by a **Vector3**, which is a combination of three numbers: X, Y, and Z.

- **X** : Left and right
- **Y**: Up and down
- **Z** : Forward and backward
 
**Vector3** represents **direction** and **magnitude**. Positive and negative values on each axis determine **which way** and **how far** the object moves. 
 
### Vector3 Shorthand

Unity provides several built-in **Vector3** shortcuts that represent common directions:

| Vector3           | Meaning                    | Equivalent Vector |
|------------------|----------------------------|-----------------|
| `Vector3.zero`   | Center of the scene          | (0, 0, 0)       |
| `Vector3.right`   | Move to the right          | (1, 0, 0)       |
| `Vector3.left`    | Move to the left           | (-1, 0, 0)      |
| `Vector3.up`      | Move up                    | (0, 1, 0)       |
| `Vector3.down`    | Move down                  | (0, -1, 0)      |
| `Vector3.forward` | Move forward in Z axis     | (0, 0, 1)       |
| `Vector3.back`    | Move backward in Z axis    | (0, 0, -1)      |

These shortcuts make it easier to write movement code without remembering the exact numbers for each axis.

>[!TIP]
> Using `Vector3` shortcuts like `Vector3.up` or `Vector3.right` makes your code more readable and reduces mistakes when specifying directions manually.

---
## Create the `MoveTransform` Class
Before we can get started moving GameObjects via code, we need to create a class that moves a GameObject by directly manipulating its **Transform** component.

It’s important to clearly recognize the purpose of the class in its name, following proper naming conventions. Since this class is responsible for moving objects using the Transform system, we will name it: **`MoveTransform`**

This name makes the purpose of the class explicit:

- It indicates that the class is a behavior attached to a GameObject.  
- It differentiates it from future movement classes, like `MoveRigidbody`, which will use physics.  

By choosing clear and descriptive names, we make our code easier to understand, maintain, and extend.

### GameObject Componets
For a class to run in Unity, it must either be **attached to a GameObject** or be called from another class that is attached to a GameObject present in the scene at runtime.

When a class is attached to a GameObject, it is referred to as a **component**. Components allow GameObjects to have specific behaviors, such as movement, rendering, or collision detection.

<br>

> [!IMPORTANT]
> Before continuing, make sure the `MoveTransform` component has been added to a GameObject in your Unity scene.
> To do this, you have a few options:
> - Drag and drop the **C# script** from the **Project Window** onto the GameObject in the **Hierarchy Window** or the **Scene Window**.  
> - With the GameObject selected, drag and drop the script into the **Inspector Window**.  
> - Alternatively, in the **Inspector Window**, click the **Add Component** button, then type the name of the component (`MoveTransform`) and select it.  
> Once added, the script becomes a component of the GameObject, allowing it to control its behavior during gameplay.

<br>

---

## Setting Object's Transfom
GameObject's transform values can be set in the **Inspector Window** in the editor. More often than not, however, this transformation should be dynamic, through code. To do this, we need to reference the `transform` component and the property we want to manipulate, in this case `position`, followed by the value we want.

For example:
```csharp
transform.position = Vector3.zero;
```
Code Breakdown: 
- `transform` references the GameObject’s Transform component.  
- `position` is the property representing its current location.  
- `Vector3.zero` is shorthand for `(0, 0, 0)`, which sets its position to the center of the scene.

<br>

>[!NOTE]
> Notice how we don’t need to write `this.transform`. The script is attached to the GameObject, so Unity already knows we’re talking about *that object’s* transform.

<br>

---

## The Unity Lifecycle

To position a GameObject by script, the code needs to be placed inside one of Unity’s lifecycle methods. These are special methods Unity automatically calls at specific times during a GameObject’s existence.

Since this line directly sets the position once, it makes the most sense to place it in either `Awake()` or `Start()`. Both methods run once when the object is created, but there are subtle differences:

### 📌 Awake vs. Start

#### Awake()
- Called immediately when the script instance is loaded.  
- Runs before any `Start()` methods.  
- Good for setting up values or references that don’t depend on other objects.  

#### Start()
- Called before the first frame update, but only if the script instance is enabled.  
- Runs after all `Awake()` methods have finished.  
- Good for initialization that depends on other objects being ready.

<br>

> [!Tip]
> - Use `Awake()` for setting up your own object’s internal values.  
> - Use `Start()` if your setup depends on other objects (for example, if you need to reference another GameObject that might not exist yet in `Awake()`).

<br>

Therefore, in this example, we would set our position in the `Awake()` method as in the example below: 

```csharp
private void Awake()
{
    //Set GameObject's initial position
    transform.position = Vector3.zero;
}//end Awake()

```
<br>

> [!IMPORTANT]
> Remember to always **save your script** before testing it in the Unity Editor.  
> Press **Play** in the Editor to see the script in action.

<br>

---

## Moving GameObjects with Transform

To move a GameObject, we can manipulate its **Transform** component at different points in the GameObject’s lifecycle. For example:

- During a triggered event, like a key press.  
- Every frame of the game, using the `Update()` method.  

Instead of simply setting the position (which would teleport the object), we often want to **increment it**, so the object moves smoothly over time. We do this by adding to the current position.

For example, to move an object 1 unit to the right every frame:

```csharp
private void Update()
{
   //Move GameObject to the right
   transform.position += Vector3.right; 
} // end Update()
```

### Understanding Frame Rate
While the example for moving a GameObject works, there’s a big problem: the **object’s speed depends on the frame rate**.  

**Frame rate** is the number of frames (images) displayed per second in a game or animation. It is measured in **FPS** (*Frames Per Second*).  

Right now, our movement code simply adds a direction vector (e.g., `Vector3.right → (1, 0, 0)`) to the object’s position once per frame. That means:  

- At **60 FPS**, the object moves **60 units per second** (`1 unit each frame × 60 frames`).  
- At **120 FPS**, the object moves **120 units per second**.  

In other words, faster machines make the object move *twice as fast*, while slower machines make the movement feel *sluggish or unfair*.  

Frame rates can vary widely depending on hardware and performance, but most games target around **60 FPS** for smooth gameplay. 

To combat this issue, we need a method to make the object's movement **independent of the frame rate**. That’s where `Time` and `Time.deltaTime` come in.

### Using `Time.deltaTime`
Unity’s `Time` class keeps track of several aspects of game time, such as:

- `Time.time` → the total time since the game started  
- `Time.deltaTime` → the time elapsed since the last frame

  <br>

The word **delta (Δ)** is a mathematical symbol meaning *“change in”*.  
In this case, `deltaTime` is the change in time between frames, calculated as:

```math
Δt = t_current frame - t_last frame
``` 
<br>

Let's say that the game is running at 60 FPS, therefore each frame lasts about 1/60th of a second, so:

``` math
Time.deltaTime = 1 / 60 ≈ 0.016 (seconds-per-frame)
```
<br>

>[!NOTE]
> Unity handles this calculation automatically. You don’t need to worry whether the game is running at 30 FPS, 60 FPS, or any other frame rate; `Time.deltaTime` always gives you the correct value.

### Adding Speed
In the example above, if `Time.deltaTime` returns a value like `0.016`, which, when multiplied by a direction vector (like `Vector3.right`) would move the object just `0.016` units per frame, far too small to be meaningful in gameplay.  

What’s missing is **speed**, a variable that defines how fast the object should move, measured in *units per second*.  

By multiplying **speed × deltaTime × direction**, we calculate the actual distance the object should move **this frame**, making the movement **smooth**, **consistent**, and **meaningful** regardless of frame rate.



```csharp
private float _speed = 5f; // units per second

private void Update()
{
    //Move GameObject to the right
    transform.position += _speed * Time.deltaTime * Vector3.right;
}//end Update()

```
Let’s break this down:

- `_speed` defines how far the object should move per second.  
- `Time.deltaTime` scales that movement for the actual time elapsed since the last frame, keeping motion consistent across different frame rates.  
- `Vector3.right` specifies the direction of movement.  

This approach ensures that the object moves **smoothly**, **consistently**, and **efficiently**, even on machines with different frame rates.

<br>

>[!NOTE]
>Notice that the direction vector appears at the end. This is deliberate: by **multiplying the two floats first** (`_speed * Time.deltaTime`), Unity performs **fewer vector multiplications**, which is slightly **more efficient** and avoids creating unnecessary temporary `Vector3` objects.

<br>

---

## Private Fields and `[SerializeField]`

In object-oriented programming, it’s best practice to keep class fields **private**. This ensures **encapsulation**, meaning that the internal state of an object is protected from unintended external changes.

```csharp
private float _speed = 5f; // Good practice: keep fields private
```

In Unity, **public fields** are automatically displayed in the Inspector, which makes it convenient to adjust values without touching the code. This is especially useful when:

- You want to tweak values in the Editor for testing or balancing.  
- You are working on a team, and a designer or level designer needs access to adjust variables like speed, jump height, or health.

```csharp
public float _speed = 5f; // Automatically shown in Inspector, but breaks encapsulation

```
The problem is that making a variable public just for Inspector access breaks **encapsulation**, other scripts can now freely change it.

To solve this, Unity provides **attributes** special markers, in square brackets `[]` that are placed above variables to change how they behave in the Inspector. One of the most useful is `[SerializeField]`. This attribute lets you keep a field private in code while still exposing it in the Inspector for easy editing:

```csharp
[SerializeField]
private float _speed = 5f; // Private field, but editable in Inspector
```

- The field remains private, so other scripts cannot access it directly.  
- Unity serializes it and displays it in the Inspector, allowing you to tweak it visually.
  - **Serialization** allows Unity to store the value between editor sessions and scene loads.

Even though the field is private in code, Unity can still read and write its value in the Editor.  

This is the recommended approach for variables you want to **expose to the editor** but **keep encapsulated in code**.

 `[SerializeField]` gives you the best of both worlds: proper encapsulation in code, and convenient editing in the Unity Inspector.


--- 

## Moving in Different Directions

So far, our object moves only to the right using `Vector3.right`. But what if we want it to move in another direction, or the level designer wants to control the direction **without modifying the code**?

Just as we created a field for **speed**, we can also create a field for **direction**:

```csharp
[SerializeField]
private float _speed = 5f; // Private field, but editable in Inspector

[SerializeField]
private Vector3 _direction = Vector3.right; // Default direction is right


private void Awake()
{
    //Set GameObject's inital position
    transform.position = Vector3.zero;
}//end Awake()

private void Update()
{
   //Move GameObject
   transform.position += _speed * Time.deltaTime * _direction ; 
}//end Update()

```
Now the designer can change the movement direction in the Inspector, for example, up, forward, or any custom vector, without touching the code.

This makes the movement **flexible and data-driven**, which is a core principle of good game design: allow designers to tweak behavior **without modifying scripts**.

---

## Optimizing `Update()` by Adding a `Move()` Method

Right now, all of the functionality for moving the GameObject happens directly inside the `Update()` method.

```csharp

private void Update()
{
    //Move GameObject
    transform.position += _speed * Time.deltaTime * _direction;  
}//end Update()

```

While this works, placing all logic in `Update()` can make it **cluttered**, **harder to manage**, and **less efficient**. Streamlining `Update()` by delegating tasks to dedicated methods helps **optimize performance**, **reduce errors**, and make your **code easier to read** and maintain. Here’s why this matters:

- **Potential for Errors**
    - `Update()` runs every frame, so any logic inside it executes constantly.  
    - If multiple actions are added here, it’s easy to introduce bugs or unintended behavior.

- **Reusability (DRY Principle)**
    - If we need the same movement logic in other places (e.g., triggered by an event or another script), duplicating code in multiple `Update()` methods violates the **Don’t Repeat Yourself (DRY)** principle.  
    - By creating a separate method, we can reuse the same movement logic whenever it’s needed.

- **Single Responsibility Principle**
    - Each method should have **one responsibility**.  
    - If `Update()` both moves the object and handles other logic (like animations, collisions, or input), it breaks this principle.

### Creating a `Move()` Method

To address the issues with placing all logic in `Update()`, we can move the movement logic into a dedicated method, like `Move()` and call the method from the `Update()`. This allows us to:

- Keep `Update()` clean and focused.  
- Trigger movement from other events or scripts without duplicating code.  
- Maintain a single responsibility for each method.

```csharp
private void Update()
{
    Move();
}//end Update()

///<summary>
/// Move the object by the transform position
///</summary>
private void Move()
{
    //Move GameObject
    transform.position += _speed * Time.deltaTime * _direction;

}//end Move()

```

---

# 🎉 New Achievement: Basic Movement

We’ve now created a **MoveTransform component** that moves our GameObject on the screen! 
```csharp

using UnityEngine;

public class MoveTransform : MonoBehaviour
{
      // Serialized fields for initial values

     // Current speed of the object (units per second)
     [SerializeField]
     private float _speed = 5f; 


     // Direction of movement
     [SerializeField]
     private Vector3 _direction = Vector3.right; 

     // Awake is called once on initialization         
     private void Awake()
     {
         //Set GameObject's initial position
         transform.position = Vector3.zero;
     }//end Awake()

     // Update is called once per frame
     private void Update()
     {
         Move();
     }//end Update()
     
     ///<summary>
     /// Move the object by the transform position
     ///</summary>
     private void Move()
     {
         //Move GameObject
         transform.position += _speed * Time.deltaTime * _direction;
     
     }//end Move()

}//end MoveTransform
```
This component can be attached to **any GameObject**, and it works the same way while still allowing us to **control the speed and direction** of each object individually.

But wait… what if we wanted more control over how our object moves? Consider these possibilities:
- What if we wanted to safely change speed or direction from another script?
- What if we wanted to cap the object’s maximum speed?
- What if we didn’t want the object to move immediately on game start?
- What if we wanted to stop or resume movement dynamically?
- What if we wanted to override speed or direction for a single move call?

These questions highlight ways we can extend the capabilities of our Move component. Next, we’ll refactor our class to implement these features, making it more flexible, modular, and easier to control during gameplay.

--- 

# Refactoring `MoveTransform`
Almost **all code benefits from refactoring** at some point, whether to improve readability, maintainability, or flexibility. By revisiting our MoveTransform component, we can make it **cleaner**, **more modular**, and capable of handling a wider range of scenarios without changing its core behavior.

In the case of our `MoveTransform` component we address the features mentioned above. 

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

Currently our `Move()` method implments the following:

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

**Update** the `Direction` property to **normilize** the set value: 

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

### Validating Field
Currently, our speed and direction validation only occurs when the **properties** are set. However, in the Unity Editor, a designer can modify the **fields** directly in the Inspector.

We can ensure proper validation at initialization by assigning the fields to their respective properties in the `Awake()` method. This triggers the property setters, applying any validation logic when the game object is created:

```csharp
private void Awake()
{
    // Validate initial speed and direction via properties
    Speed = _speed;
    Direction = _direction;

    // Set the GameObject's initial position
    transform.position = Vector3.zero;
} // end Awake()
```

This approach guarantees that values set in the Inspector are **clamped and normalized** correctly at the start of the game.

## Visual Cues 
While we have implemented validation for our fields, these constraints may not be immediately obvious in the Unity Editor. For example, if a level designer sets `Direction` to `(3, 0, 4)` or `Speed` to `20`, the property setters will automatically correct the values when the game runs. However, the designer might not understand why the values suddenly changed.

One way to improve clarity is to use Unity's `[Tooltip]` attribute on fields in the Inspector. This displays a small piece of text when you hover your mouse over the field, providing guidance on its purpose, valid values, or expected behavior.

```csharp
     [SerializeField]
     [Tooltip("Speed of the object’s movement. Cannot exceed maxiumum speed.")]
     private float _speed = 5f;

     [SerializeField]
     [Tooltip("Direction of movement. Will be normalized automatically to ensure consistent movement.")]
     private Vector3 _direction = Vector3.right;
     
```

> [!TIP]
> Because `[Tooltip]` already describes the variable, adding an inline comment would be redundant.

### Tighter constraints
While our `[Tooltip]` provides some visual context about the constraints on the `_speed` variable, and the mere inclusion of `_maxSpeed` should suggest a limit, these cues may not be **immediately obvious to the level designer**.

With that said, we might want to might want to enforce **tighter constraints** while simultaneously making valid speed options more apparent to the designer.


To make valid speed options clearer and enforce tighter constraints, we can convert `_maxSpeed` to a **constant** and apply Unity's `[Range(min, max)]` attribute to `_speed`.

> [!NOTE]
> We can set `MAX_SPEED` as the upper limit in `[Range(0f, MAX_SPEED)]` because it is a **constant**. Unity requires `[Range]` values to be fixed at compile time, so only constants (not regular variables) can be used here.

**Update** the following fields: 
```csharp
[SerializeField]
[Range(0f, MAX_SPEED)]
[Tooltip("Speed of the object’s movement. Cannot exceed maximum speed.")]
private float _speed = 5f;

// Maximum speed allowed
private const float MAX_SPEED = 10f;
```

> [!CAUTION]
> **DO NOT** forget to also update the `Speed` property, after switching `_maxSpeed` to the `MAX_SPEED` **constant**.
> 'set => _speed = Mathf.Clamp(value, 0f, MAX_SPEED);'

---

## Extending Move()

In our current design, the `Move()` method is **private**, meaning only the script itself can call it. This is a safe default because it enforces **encapsulation**: the object controls how it moves, without interference from outside scripts.

However, in many games, other objects or events need to control movement. For example:

- A player pulls a lever that triggers a platform to move.  
- A power-up temporarily boosts an enemy's speed.  
- A boundary or collision triggers a change in direction.  

Keeping `Move()` private avoids tight dependencies by default.However, to allow external scripts to influence movement safely and dynamically, we can make the method **public**.

> [!TIP]
>  Start with private methods to maintain self-contained behavior. When there’s a clear need for external control, you can expose the method through parameters or carefully designed public access.

### Accepting Paramters
Updating `Move()` to be public makes it accessible to other scripts. While our public properties allow external scripts to modify `Speed` and `Direction`, there are situations where it’s more convenient or efficient to pass temporary values directly to the method—such as responding to an event, applying a one-off speed adjustment, or triggering movement with a specific direction.

To support this, we can modify the original private `Move()` method to accept `direction` and `speed` as parameters:

```csharp
/// <summary>
/// Moves the object in a specified direction at a specified speed.
/// </summary>
/// <param name="direction">The direction to move the object.</param>
/// <param name="speed">The speed at which the object should move.</param>
private void Move(Vector3 direction, float speed)
{
    // Assign parameters to properties
    Direction = direction;
    Speed = speed;

    // Move the GameObject
    transform.position += Speed * Time.deltaTime * Direction;

}//end Move()

```
#### Why assign parameters to the properties?

Even though the values are temporary inputs for this method call, assigning them to the properties ensures that:

- Validation logic in the property setter is automatically applied (e.g., `Speed` cannot exceed `_maxSpeed`).

- The object’s internal state is updated, so future `Move()` calls reflect the most recent values.

- The class remains consistent, keeping temporary changes in sync with the persistent properties without introducing additional state variables.
  

### Handling Default Values

The method above works, but there’s a drawback: every time we call `Move()`, we must supply both parameters, even if we just want to use the current `Speed` or `Direction`. This can become repetitive and violates the KISS principle (“Keep It Simple, Stupid”).

To make the method more flexible, we can use nullable parameters (`Vector3?` and `float?`) and fall back to the object’s properties if no value is provided. We also introduce two **local variables** (`moveDirection` and `moveSpeed`) to store the actual values being used in this frame:

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
Now that we added our flags, we can use them to control when the object is allowed to move.

```csharp
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

    private void Update()
    {
       if(_isMoving)
       {
         Move();

       }//end if(_isMoving)

    }//end Update()


```
  
### Updating `Move()` with `_isMoving`
With the current flags implemented, `_isMoving` is only true if `_moveOnAwake` is `true`.

But what if we want the object to remain still at the start and have an external script trigger movement? In that case, we need to ensure that whenever `Move()` is called, `_isMoving` is set to `true`.

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

        // Flags the object as moving
        _isMoving = true;

        // Move the GameObject using the resolved frame values
        transform.position += Speed * Time.deltaTime * Direction;

    }//end Move()
```
This ensures that any external call to `Move()` triggers the object to **keep moving every frame**.

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

---
# 🎉 New Achievement: Refactored Movement

Congratulations! You've **refactored** the `MoveTransform` component and with these improvements, the object now:

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

    [SerializeField]
    [Range(0f, MAX_SPEED)]
    [Tooltip("Speed of the object’s movement. Cannot exceed maxiumum speed.")]
    private float _speed = 5f;


    // Maximum speed allowed
    private const float MAX_SPEED = 10f;

    
    // Direction of movement
    [SerializeField]
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
    private void Move(Vector3? direction = null, float? speed = null)
    {
        // Resolve the effective values for this frame
        Vector3 moveDirection = direction ?? Direction;
        float moveSpeed = speed ?? Speed;

        // Update properties to ensure validation and internal consistency
        Direction = moveDirection;
        Speed = moveSpeed;
        
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




