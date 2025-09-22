# CHARACTER CONTROLLER CHALLENGE

Previously, we created a `MoveTransform` Class, which handles moving an object in a given `Direction` at a set 'Speed`. This component was designed to be **modular** as many objects could use the same behaviors. For example, we could attach it to enemies, moving platforms, projectiles, or other objects that need consistent motion. Each object can move independently without knowing why it’s moving—its movement is handled entirely by the component.

But what if we want to **control an object using player input**? To handle **player-driven movement**, we create a controller class, often called a `CharacterController` or `PlayerController`. Its job is: 
- Check for input **every frame**.
- Determine the direction the player wants to move.
- Tell the MoveTransform component to move the object in that direction.

This keeps **movement logic separate from input logic**, which is cleaner, more maintainable, and allows us to reuse MoveTransform for objects that don’t require player control.

---

## Checking for Input
Unity allows us to use a variety of input checks. To keep things very simple, we will implement Unity's basic input system and check against the keyboard keys. 
In this case, we will be checking using `W`, `A`, `S`, `D`, the arrow keys, and space for jump. 
Since our current scene is in a 3D perspective, we need to remember Unity's X, Y, and Z axes, and therefore: 
- `W` / `Up` Arrow → Move forward (Z-axis)
- `S` / `Down` Arrow → Move backward (-Z-axis)
- `A` / `Left` Arrow → Move left (-X-axis)
- `D` / `Right` Arrow → Move right (X-axis)
- `Space` → Move up (y-axis)

In Unity, to check for keyboard input, we can use the `Input.GetKey(KeyCode.KEY)` method, where **KEY** is the key we want to use. For example, we wanted to check if we were moving forward: 

```csharp
     if (Input.GetKey(KeyCode.W) || Input.GetKey(KeyCode.UpArrow))
    {
      //call to move forward
    
    }
```

> [!IMPORTANT]
> Checking for input directly with `KeyCode (e.g., Input.GetKey(KeyCode.W))` is part of **Unity's old Input System**. While it is recommended to use **Unity's new Input System** for modern projects, we'll cover that later. For now, we will allow for both the old and new systems in the same project:
> 1. Go to `Edit` > `Project Settings` > `Player` > `Other Settings`.
> 2. Find **Active Input Handling**.
> 3. Change it from Input System Package (New) → **Both**.
> 4. Press the `Okay` button in the pop-up dialog, Unity will then reload the project.> . 

---

## Get the `MoveTransform` Component
The actual methods to move our game object are in the `MoveTransform` component, so we will need to get a reference to that component. To access any components via script, we can use Unity's `GetComponent<T>()` method, which essentially tells Unity to **find the component of type _T_ attached to this GameObject**.

The type isn't limited to built-in Unity components; you can also use it for any custom component (i.e., class) you create and attach to a GameObject.

Once you store the reference in a variable, you can use it to **access public properties** (like velocity, mass, or drag) or **call methods** on that component.

### What if No Component is Found?

If `GetComponent<T>()` doesn’t find the requested component on the GameObject, it returns **null**.

This means any logic that tries to access the component (e.g., `_rigidBody.linearVelocity = ...`) will throw a `NullReferenceException`.

To prevent errors, it’s good practice to **check for null** before using a component reference. Unity provides an easy way to do this with `TryGetComponent<T>(out var)`. This method returns true if the component exists and automatically assigns the reference to the variable; if the component does not exist, it returns false.

In this example, we first declare a private field to hold a reference to our `MoveTransform`. Then, inside `Start()`, we call `TryGetComponent` to safely check if the component is present. If it’s missing, we log an error so the issue can be caught quickly.

```csharp
    //Reference to the MoveTransform component 
    private MoveTransform _moveTransform;

    // Start is called once before the first Update
    private void Start()
    {
        // Check if MoveTransform component DOES NOT EXIST
        if (!TryGetComponent<MoveTransform>(out _moveTransform))
        {
            Debug.LogError("MoveTransform component missing!");

        }//end if(!TryGetComponent<MoveTransform>(out _moveTransform))
      
    }//end Start()

```
> [!NOTE]
> When `TryGetComponent` succeeds, it automatically assigns the found component to our `_moveTransform` field. We don’t need to manually set it. In this example, we only handle the false case, logging an error to help us catch problems early in development.

---


# Challenge: Build Your Own Character Controller
Using the skills we've already learned, create your own `CharacterController` class. 

- Create a new `CharacterController` class.
- Add both the `CharacterController` and `MoveTransform` components to the object to control.
- Use `TryGetComponent` to reference `MoveTransform`.
  - Set reference to the field `_moveTransform`
- Read input from `W`, `A`, `S`, `D`, the arrow keys for **every frame**
- Call `_moveTransform.Move()` with the correct **direction**.
- Use proper commenting and `Debug.Log` for error checking.

## Bonus Challenge

Think about how to add jumping along the Y-axis? 

Without using gravity, how can we get the object to jump up and then return to its original Y position? 



