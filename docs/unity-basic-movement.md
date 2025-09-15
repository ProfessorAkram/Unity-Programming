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
private float speed = 5f; // units per second

private void Update()
{
    transform.position += Vector3.right * speed * Time.deltaTime;
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
private float speed = 5f; // Good practice: keep fields private
```

However, in Unity, **public fields** are automatically displayed in the Inspector, which makes it convenient to adjust values without touching the code. This is especially useful when:

- You want to tweak values in the Editor for testing or balancing.  
- You are working on a team, and a designer or level designer needs access to adjust variables like speed, jump height, or health.

```csharp
public float speed = 5f; // Automatically shown in Inspector, but breaks encapsulation

```

To maintain encapsulation while still exposing the field in the Inspector, Unity provides the `[SerializeField]` attribute:

```csharp
[SerializeField]
private float speed = 5f; // Private field, but editable in Inspector
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
private float speed = 5f; // Private field, but editable in Inspector

[SerializeField]
private Vector3 direction = Vector3.right; // Default direction is right


private void Awake()
{
    transform.position = Vector3.zero; //set inital position
}//end Awake()

private void Update()
{
    transform.position += direction * speed * Time.deltaTime; //Move GameObject 
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
    get { return speed; } 
    set { speed = value; } 
}

public Vector3 Direction
{
    get { return direction; }
    set { direction = value; }
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


**Code breakdown:**
- `get { return speed; }` → Returns the current speed value.  
- `set { ... }` → Runs whenever another script or method tries to change the speed.  
- The `if` statement ensures speed never exceeds 10, no matter who tries to set it.  
- `Debug.LogWarning` informs us in the console if someone tried to set an invalid value.  

> [!NOTE]
> `Debug.Log` is a Unity method used to **print messages to the Console window**. It’s extremely useful for **testing**, **debugging**, and **tracking** what your code is doing at runtime.
