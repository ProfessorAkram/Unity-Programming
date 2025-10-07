# Moving GameObjects with Transform

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

<br>

> **✔️ CHECK POINT**
> 
> Save your script, switch back to the Unity editor, and press **Play** to test the changes in action.

<br>

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

<br>

> **✔️ CHECK POINT**
> 
> Save your script, switch back to the Unity editor, and press **Play** to test the changes in action.

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
The problem is that making a variable public just for Inspector access breaks **encapsulation**, and other scripts can now freely change it.

To solve this, Unity provides **attributes**, special markers, in square brackets `[]` that are placed above variables to change how they behave in the Inspector. One of the most useful is `[SerializeField]`. This attribute lets you keep a field private in code while still exposing it in the Inspector for easy editing:

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
    //Set GameObject's initial position
    //transform.position = Vector3.zero;

}//end Awake()

private void Update()
{
   //Move GameObject
   transform.position += _speed * Time.deltaTime * _direction ;

}//end Update()

```
Now the designer can change the movement direction in the Inspector, for example, up, forward, or any custom vector, without touching the code.

This makes the movement **flexible and data-driven**, which is a core principle of good game design: allow designers to tweak behavior **without modifying scripts**.

<br>

> **✔️ CHECK POINT**
> 
> Save your script, switch back to the Unity editor, and press **Play** to test the changes in action.

<br>


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

> [!NOTE]
> The `Move()` method will be **public** as it most likely will not only be called on this component, but also triggered by other events, like player input from a `CharacterController` class. 

```csharp
private void Update()
{
    Move();

}//end Update()

///<summary>
/// Move the object by the transform position
///</summary>
public void Move()
{
    //Move GameObject
    transform.position += _speed * Time.deltaTime * _direction;

}//end Move()

```
<br>

> **✔️ CHECK POINT**
> 
> Save your script, switch back to the Unity editor, and press **Play** to test the changes in action.

<br>


---
## Using Debug.Log for Debugging

So far, we’ve focused on getting our object to move correctly. Up to this point, we haven’t needed to check the internal state of our code because the movement is simple and visible in the Scene view.

However, as projects get more complex, it’s often not enough to just watch objects move. Sometimes you need to know what values your variables hold at runtime, or whether your methods are being called correctly. This is where `Debug.Log` comes in.

`Debug.Log` is a simple way to print messages to the Unity Console, helping you understand what’s happening in your code as the game runs.

For example, you can log the object’s position every frame:
```csharp
public void Move()
{
    // Move GameObject
    transform.position += _speed * Time.deltaTime * _direction;

    // Log the new position to the Console
    Debug.Log("Current Position: " + transform.position);
}

```

<br>

> **✔️ CHECK POINT**
> 
> Save your script, switch back to the Unity editor, and press **Play** to test the changes in action.

<br>

### Debug.Log Variants and Viewing Messages
Above, we made use of Unity's generic `Debug.Log`, which is used to track normal runtime values and flow. However, Unity provides two other types of console messages for debugging:

1. **Debug.LogWarning** – Highlights unusual situations that aren’t necessarily breaking the game, but might need attention. For example, if an object’s speed exceeds a threshold:
```csharp
Debug.LogWarning("Object is moving unusually fast!");
```
Warnings appear in the Console with a **yellow caution** icon before the message, helping them stand out.

2. **Debug.LogError** – Marks critical problems or errors that need immediate fixing. For instance, if a position becomes invalid:

```csharp
Debug.LogError("Object position is invalid!");
```
Errors appear in the Console with a **yellow explanation** icon before the message, helping them stand out.

### Viewing Messages in the Editor

Since `Move()` runs every frame (inside `Update()`), using `Debug.Log` will print a new line to the Console each frame, which can quickly become overwhelming.

> [!TIP]
> Unity collapses repeated identical messages automatically. This keeps your Console readable by showing _“x repeated messages”_ instead of spamming new lines.

Additionally, you can click the **Collapse** button at the top of the Console to combine repeated messages, making it much easier to track changes over time.

### Best Practices for Debugging

`Debug.Log`, as the name implies, is for **debugging**. Once you’ve used it to verify that your code is working correctly and your variables are behaving as expected, you typically no longer need these messages. At that point, it’s a good idea to **comment out or remove them**, especially inside methods like Update() that run every frame, to avoid unnecessary performance overhead.

```csharp
// Debug.Log("Current Position: " + transform.position);
```
#### Using `#if UNITY_EDITOR` for Editor-Only Debugging
Sometimes, you want debug messages to **remain visible while working in the Unity Editor**, but ensure they are **excluded from the final build**. The `#if UNITY_EDITOR` preprocessor directive makes this easy.

Code placed inside a `#if UNITY_EDITOR` block runs **only in the Editor**:

```csharp
#if UNITY_EDITOR
Debug.Log("Current Position: " + transform.position);
#endif
```
When you build the game for PC, mobile, or console, the compiler **completely removes this code**, so it has **no impact on performance or memory** in the final game. This allows you to keep helpful debug messages while testing, without worrying about them affecting your players or the shipped build.

Using `#if UNITY_EDITOR` provides several advantages. It’s **safe for production builds**, letting you leave debug statements in your code without slowing down the game or cluttering output. It also helps produce **cleaner, more efficient builds**, since messages inside these blocks are automatically excluded, eliminating the need to manually remove or comment them out before shipping.

> [!TIP]
> Use `#if UNITY_EDITOR` whenever you have debug code that is helpful during development but not needed in the released game. This keeps your workflow efficient and ensures your builds remain optimized.

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
         //transform.position = Vector3.zero;

     }//end Awake()

     // Update is called once per frame
     private void Update()
     {
         Move();

     }//end Update()
     
     ///<summary>
     /// Move the object by the transform position
     ///</summary>
     public void Move()
     {
         //Move GameObject
         transform.position += _speed * Time.deltaTime * _direction;

        // Debug.Log("Current Position: " + transform.position); 
     
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

**[<< Return to Transform Component tutorial](transform-component.md)** | **[Continue to Refactor Movement tutorial >>](refactor-movement.md)**