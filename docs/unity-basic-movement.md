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

---

## Setting Object's Transfom
GameObject's transform values can be set in the **Inspector Window** in the editor. More often than not, however, this transformation should be dynamic, through code. To do this, we need to reference the `transform` component and the property we want to manipulate, in this case `position`, followed by the value we want.

For example:
```csharp
transform.position = Vector3.zeero;
```
Code Breakdown: 
- `transform` references the GameObject’s Transform component.  
- `position` is the property representing its current location.  
- `Vector3.zero` is shorthand for `(0, 0, 0)`, which sets its position to the center of the scene.

>[!NOTE]
> Notice how we don’t need to write `this.transform`. The script is attached to the GameObject, so Unity already knows we’re talking about *that object’s* transform.

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

>[!Tip]
> - Use `Awake()` for setting up your own object’s internal values.  
> - Use `Start()` if your setup depends on other objects (for example, if you need to reference another GameObject that might not exist yet in `Awake()`).

Therefore, in this example, we would set our position in the `Awake()` method as in the example below: 

```csharp
private void Awake()
{
    //Set GameObject's inital position
    transform.position = Vector3.zero;
}//end Awake()

```
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

>[!Warning]
>This moves the object by 1 unit per frame, which can be very fast depending on your frame rate. To make movement **frame-rate** independent, multiply by `Time.deltaTime`.

### Smooth Transitions with Time.Delta Time
While the example for moving a GameObject works, there is a slight problem: the object’s speed now depends on the frame rate.
- On a fast computer running 120 FPS, it moves **twice as fast** as on a slower computer running 60 FPS.
- To fix this, we use `Time.deltaTime`, which represents the **time elapsed since the last frame**. Multiplying by `Time.deltaTime` ensures movement is consistent across all frame rates.

>[!NOTE]
> **Frame Rate** is the number of frames (images) displayed per second in a game or animation. It is measured in **FPS (Frames Per Second)**.
> Frame rates can vary widely depending on hardware and game performance, but most games target around **60 FPS** for smooth gameplay.
> - Lower frame rates, like **30 FPS**, can appear choppy.
> - Higher frame rates, such as **120 FPS**, feel very smooth and responsive.

While using `Time.deltaTime` fixes frame-rate dependency, we often want to **control how fast the object moves**, i.e., the rate of change of its position. We can do this by creating a **speed** variable that represents the distance the object should move per second.

```csharp
private float _speed = 5f; // units per second

private void Update()
{
    //Move GameObject to the right
    transform.position += Vector3.right * _speed * Time.deltaTime;
}//end Update()

```
**Code Breakdown:**
- `speed` determines how far the object moves each second.  
- Multiplying by `Time.deltaTime` scales this movement to the actual time elapsed per frame, resulting in smooth, consistent motion across all frame rates.  

This concept can be thought of as calculating the object’s **rate of change between frames**. The variable `speed` defines how much change should happen each second, and `Time.deltaTime` applies that change accurately for the current frame.

---
## Private Fields and `[SerializeField]`

In object-oriented programming, it’s best practice to keep class fields **private**. This ensures **encapsulation**, meaning that the internal state of an object is protected from unintended external changes.

```csharp
private float _speed = 5f; // Good practice: keep fields private
```

However, in Unity, **public fields** are automatically displayed in the Inspector, which makes it convenient to adjust values without touching the code. This is especially useful when:

- You want to tweak values in the Editor for testing or balancing.  
- You are working on a team, and a designer or level designer needs access to adjust variables like speed, jump height, or health.

```csharp
public float _speed = 5f; // Automatically shown in Inspector, but breaks encapsulation

```

To maintain encapsulation while still exposing the field in the Inspector, Unity provides the `[SerializeField]` attribute:

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
   transform.position += _direction * _speed * Time.deltaTime; 
}//end Update()

```
Now the designer can change the movement direction in the Inspector, for example, up, forward, or any custom vector, without touching the code.

This makes the movement **flexible and data-driven**, which is a core principle of good game design: allow designers to tweak behavior **without modifying scripts**.

---

## Using Properties for Speed and Direction

While **fields** store data, it’s often a good idea to create **properties** to access and modify them safely.  

Properties are **methods that act like fields**, allowing you to control how a value is read or written.

```csharp
 public float Speed 
{ 
    get { return _speed; } 
    set { _speed = value; } 
}

public Vector3 Direction
{
    get { return _direction; }
    set { _direction = value; }
}

```
### Why Use Properties?

- **Encapsulation:** Keep the underlying field private while still giving controlled access.  
- **Validation:** Add logic to check values before changing them (e.g., clamp speed to a maximum).  
- **Consistency:** Other scripts can read and modify the property without directly accessing the private field.  
- **Flexibility for future changes:** Add animations, triggers, or events whenever the value changes, without rewriting code elsewhere.

### Implementing Validation
While exposing fields directly (even with `[SerializeField]`) works, **properties** give us more control over how values are set or retrieved. One key benefit is **validation**. 

For example, imagine we want to prevent the object from moving too fast. Let’s say a speed higher than **10 units per second** is too fast. With a property, we can enforce this rule:

```csharp
[SerializeField]
private float _speed = 5f; // Default speed

public float Speed
{
    get { return _speed; }
    set
    {
        if (value > 10f)
        {
            _speed = 10f; // Clamp speed to 10
            Debug.LogWarning("Speed too high! Clamped to 10.");
        }
        else
        {
            _speed = value; // Accept valid value

        }//end if (value > 10f)
    }
}//end Speed
```

**Code breakdown:**
- `get { return speed; }` → Returns the current speed value.  
- `set { ... }` → Runs whenever another script or method tries to change the speed.  
- The `if` statement ensures speed never exceeds 10, no matter who tries to set it.  
- `Debug.LogWarning` informs us in the console if someone tried to set an invalid value.  

> [!NOTE]
> `Debug.Log` is a Unity method used to **print messages to the Console window**. It’s extremely useful for **testing**, **debugging**, and **tracking** what your code is doing at runtime.

### Accessing Properties vs. Fields in Methods

When we define **properties** for fields, especially with validation or additional logic, it’s important that all methods in the class use the properties rather than directly accessing the private fields.

#### Why Use Properties Internally?

- **Validation is Applied** – If the property limits values (e.g., `Speed` cannot exceed 10), using the property ensures this rule is enforced consistently.  
- **Consistency Across the Class** – All internal and external accesses go through the same logic, reducing bugs caused by bypassing validation.  
- **Future-proofing** – If you add side effects to the property later (such as triggering events or updating UI), every method that uses the property automatically benefits.

#### When Not to Use Properties

- During object construction or initialization, where the property logic may not yet be ready.  
- Performance-critical situations where the property logic introduces measurable overhead, and you are certain bypassing it is safe.

With these coniderations in mind we need to update our `transform.position` in the `Update()` method. 

```csharp
transform.position += Direction * Speed * Time.deltaTime;
```

--- 
## Refactoring the `Update()` Method

Right now, all of the functionality for moving the GameObject happens directly inside the `Update()` method.

```csharp

private void Update()
{
    //Move GameObject
    transform.position += Direction * Speed * Time.deltaTime;  
}//end Update()

```

While this works, placing everything in `Update()` is not best practice. Here’s why:

### Potential for Errors
- `Update()` runs every frame, so any logic inside it executes constantly.  
- If multiple actions are added here, it’s easy to introduce bugs or unintended behavior.

### Reusability (DRY Principle)
- If we need the same movement logic in other places (e.g., triggered by an event or another script), duplicating code in multiple `Update()` methods violates the **Don’t Repeat Yourself (DRY)** principle.  
- By creating a separate method, we can reuse the same movement logic whenever it’s needed.

### Single Responsibility Principle
- Each method should have **one responsibility**.  
- If `Update()` both moves the object and handles other logic (like animations, collisions, or input), it breaks this principle.

--- 

## Creating a `Move()` Method

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
    transform.position += Direction * Speed * Time.deltaTime; 
}//end Move()

```

### Public or Private?
In our example, the `Move()` method is **private**, enforcing encapsulation. Deciding whether a method should be public or private can be tricky, so here are some guiding questions to consider:

- Does any external class truly need to call this method directly?  
- Would exposing this method create unnecessary dependencies between objects?  
- Could the behavior be triggered in a safer or more modular way, such as through events, signals, or internal condition checks?  

### Example

Imagine a player pulls a lever that causes platforms to move. Does the lever need direct access to the platform’s `Move()` method? Not necessarily. Instead:

- The platform could listen for an event triggered by the lever.  
- Or it could check conditions internally before moving.  

By keeping the method private, we avoid creating tight dependencies between unrelated objects, making the system easier to maintain and less error-prone.

> [!Tip]
> Use **private methods** to keep behavior self-contained, and carefully consider external access to avoid unnecessary dependencies.

### Dynamic Inputs
While we usually keep the `Move()` method **private** to encapsulate behavior, there are times when we want the same movement logic to work with dynamic inputs—for example, applying a speed boost or changing direction.

In such cases, we can make the method **public** and allow other scripts to pass parameters. This approach lets us:

- **Reuse logic without duplication** – no need to write separate movement code for each object.  
- **Control behavior dynamically** – adjust speed, direction, or other movement properties on the fly.  
- **Maintain modular design** – the object still controls how it moves, but external scripts can influence it safely through parameters.

---
## Dynamic Movement Using Parameters

In the instance of making our method **public**, we might also want to extend its capabilities so that it can handle dynamic inputs—for example, changing speed, direction, or other movement properties at runtime. 

This allows multiple objects or events to use the same movement logic without directly accessing internal fields or properties, keeping the design **modular** and **reusable**.

To do this we can modify the original private `Move()` method to accept **direction** and speed as **parameters**:

```csharp
/// <summary>
/// Moves the object in a specified direction at a specified speed.
/// </summary>
/// <param name="direction">The direction to move the object.</param>
/// <param name="speed">The speed at which the object should move.</param>
private void Move(Vector3 direction, float speed)
{
   //Set Properites
   Direction = direction;
   Speed = speed

  //Move GameObject 
  transform.position += Direction * Speed * Time.deltaTime; 
}

```
**Code Breakdown**
The `<param>` comments document each parameter, making it clear to anyone reading the code what values the method expects:

- **direction** – lets external scripts specify which way the object moves.  
- **speed** – lets external scripts adjust the object’s movement speed.  

Since the parameters provide temporary input to the method, we now assign these values to the corresponding properties. This is important because:

- It ensures any validation logic in the property setter is applied automatically.  
- It updates the object’s internal state, so future movement calls reflect the new direction or speed.  
- It keeps the class consistent, letting the object “remember” the most recent movement values rather than only using temporary variables.

### Handling Default Values

While the above works, there’s a downside: every time we call `Move()`, we **must provide both parameters**, even if we want the default behavior.

- If we only care about the object’s current **Direction** or **Speed**, passing parameters becomes repetitive.

- This adds unnecessary steps and goes against the **KISS principle** (“Keep It Simple, Stupid”).

To solve this, we can allow the parameters to be **nullable** (`Vector3?` and `float?`) and fall back to the object’s properties if no value is provide. In this instance, we need a **local method variable** to store the actual values being used, either the passed paramter or fall back default value. 

```csharp
/// <summary>
/// Moves the object in a specified direction at a specified speed.
/// </summary>
/// <param name="direction">The direction to move the object.</param>
/// <param name="speed">The speed at which the object should move.</param>
private void Move(Vector3? direction = null, float? speed = null)
{
   //Set local values
   moveDirection = direction ?? Direction;
   moveSpeed = speed ?? Speed;

  //Move GameObject 
  transform.position += moveDirection * moveSpeed * Time.deltaTime; 
}

```
**Code Breakdown**

**1. Nullable Parameters (`Vector3?` and `float?`)**

  - The `?` after the type allows the parameter to be **nullable**, meaning it can hold a `null` value.  
  - If the caller doesn’t provide a value, `direction` or `speed` will be `null`.  
  - Using the `??` **null-coalescing operator**, which checks if the value on the left-hand side is null.
     - If it is not null, the left-hand value is used.
     - If it is null, the right-hand value is used as a fallback.

**2. Local Variables**
Since we have check for null paramters we need be able to store the resulting value, 
These local variables store the **resolved values** that the method will use for this frame.  

- They ensure clarity, readability, and consistency when calculating the movement.

**3.  Naming the Local Variables**

  - **moveDirection** – clearly indicates the direction applied this frame.  
  - **moveSpeed** – clearly indicates the speed applied this frame.  

This naming distinguishes temporary, per-call values from the object’s persistent properties.

**4. Updating the Movement Calculation**

 - Uses the resolved local variables for **frame-rate independent movement**.

--- 

## Extending functionality
## Adding Higher-Level Control

So far, we’ve given our `Move()` method flexibility by allowing parameters for speed and direction, along with sensible fallbacks to default values. This gives programmers more dynamic control at runtime.

But what about control at a higher level? Imagine situations where we don’t want the object to move at all—regardless of speed or direction. For example:

- A player might be frozen by a trap or spell.  
- An NPC could be paused during a cutscene.  
- A platform might be inactive until the player triggers it.  

In these cases, passing parameters isn’t enough. We need a way to enable or disable motion entirely—something that gives level designers and game systems a simple “on/off switch” for movement.

## Why Not Just Set Values to Zero?

It’s important to note that simply setting the values (like speed or direction) to zero isn’t an optimal solution.  

Right now, our `Move()` method is being called in `Update()`, so even if the object isn’t moving, the method is still **wasting processing time every frame**.  

Instead, we want a true **movement gatekeeper** that prevents unnecessary calculations altogether when motion is disabled.


