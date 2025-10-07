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
**Vector3** represents **direction** and **magnitude**. Positive and negative values on each axis determine **which way** and **how far** the object moves. 

It's important to note that Unity's coordinate system differs from some other 3D applications. In Unity:

- **X** remains the horizontal axis (left and right).
- **Y** is the vertical axis (up and down).
- **Z** is the depth axis (forward and backward).

In 3D programs like Blender, or in other game engines such as Unreal, the Z axis is used for vertical movement. This difference can be confusing when switching between tools, so keeping Unity’s coordinate system in mind will help avoid mistakes when importing models or reasoning about movement.

 
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

### GameObject Components
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

> [!CAUTION]  
> Unless you are instantiating a new GameObject, you generally should not set its initial position in code; that is typically determined by the game designer in the scene. This line is included only to demonstrate how `transform.position` works. After testing, comment it out or delete it.

---

**[<< Return to Lesson Contents](transform-movement.md)** | **[Continue to Basic Movement tutorial >>](basic-movement.md)**
