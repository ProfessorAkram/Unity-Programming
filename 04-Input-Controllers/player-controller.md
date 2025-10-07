# Player Controller 
Now that we have our input system set up, we can start work on the `PlayerController` class that defines the methods triggered by the **Player Input component**. 

**1. Create a new C# class** named `PlayerController` and open it in your editor.

**2. Double-click on `PlayerController`** to open it in your editor.

## Get the Move Component
In our previous lessons, we created `MoveTransform` and `MoveRigidbody` classes. Each of these moves the GameObject in a different way. When setting up our `PlayerController`, we need to decide which movement style we want to use and get a reference to the appropriate component. For this lesson, we will keep things simple and use the `MoveTransform` component. In the end the `PlayerController` should work exactly like the `CharacterController` just with the new Input System

#### 3. Get a Reference for the `MoveTransform` component

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

## Creating Triggered Methods
  
Now that we have a reference to our `MoveRigidbody` component, we need to create the **methods that are triggered by the input actions** from the Player Input component. These methods will receive the input data and tell the movement component what to do.

### Observer Pattern
The way in which the **Player Input component** and `PlayerController` interact is an example of the **Observer pattern** (sometimes called the **subscription pattern**). In this pattern, an **event source notifies subscribers whenever something happens**.

In this case: 
- The Player Input component is the event source; it detects input actions like Move or Jump.
- Your associated methods _(e.g, `OnMove()`)_ is the subscriber; it “registers interest” in that input.
- When the input Action occurs, the **Player Input component** automatically **triggers the corresponding method**.

> [!IMPORTANT]
> Unlike in a traditional Observer pattern implementation, **you do not need to manually subscribe to these events**. Unity automatically connects Actions to methods on the GameObject based on the method naming convention _(See below)_.

### Method Naming Conventions for Input Actions

A key part of using the Player Input component is understanding **how it finds and calls your methods**. Every time an Input Action fires, the Player Input component looks for a method on the same GameObject that **matches the Action name, prefixed with `On`**.

**Example:**
- Action name: `Move` → Method name: `OnMove`
- Action name: `Jump` → Method name: `OnJump`

This naming convention is essential because it allows Unity to **automatically trigger the correct method** without manually wiring each Action. If the method is missing or the name doesn’t match, the Action will fire, but nothing will happen.

> [!TIP]
> Using **short, descriptive, verb-based** names for Actions and their corresponding methods. This will ensure that your code is easier to read and maintains a consistent pattern for all input actions.

### Create `OnMove()` Method

Before creating the method that will be triggered by the Move Action, make sure your script **includes the necessary namespace** for the new Input System.

#### 1. **Import** the Namespace

```csharp
using UnityEngine.InputSystem;
```

This namespace gives you access to `InputValue`, `PlayerInput`, and other classes needed to work with the new Input System. Without it, the compiler will not recognize `InputValue` or related types.

#### 2. **Define** the `OnMove()` method

```csharp
    /// <summary>
    /// Triggered by the Move input Action. Converts the 2D input from the player
    /// (keyboard, joystick, or gamepad) into a 3D direction and passes it to the 
    /// MoveTransform component to move the player.
    /// </summary>
    /// <param name="value">
    /// The InputValue wrapper passed automatically by the Player Input component. 
    /// </param>
    public void OnMove(InputValue value)
    {

    } //end OnMove()
```
The parameter `InputValue` **is not the actual** `Vector2`, but a generic container that can hold any type. Since our _Move_ action is set to a **Vector2** control type, we need to extract the `Vector2` value from it. This value can then be converted into a `Vector3` for movement in 3D space.

```csharp
    /// <summary>
    /// Triggered by the Move input Action. Converts the 2D input from the player
    /// (keyboard, joystick, or gamepad) into a 3D direction and passes it to the 
    /// MoveTransform component to move the player.
    /// </summary>
    /// <param name="value">
    /// The InputValue wrapper passed automatically by the Player Input component. 
    /// </param>
    public void OnMove(InputValue value)
    {
        // Extract the Vector2 from the InputValue
        Vector2 inputVector = value.Get<Vector2>();
        Debug.Log("OnMove called: " + inputVector);

        // Convert the 2D input into a 3D direction on the XZ plane
        Vector3 direction = new Vector3(inputVector.x, 0f, inputVector.y);
        Debug.Log("OnMove direction: " + direction);
        

        // Call the movement component with the calculated direction
        _moveTransform.Move(direction);

    }//end OnMove()
```

> [!TIP]
> Adding a `Debug.Log()` is a quick way to verify that the correct input value is being passed to your method.

#### 4. **Call** the `Move()` method
- Call the `Move()` method on the `MoveTransform` component
- Pass the `direction` into the method


```csharp
    /// <summary>
    /// Triggered by the Move input Action. Converts the 2D input from the player
    /// (keyboard, joystick, or gamepad) into a 3D direction and passes it to the 
    /// MoveTransform component to move the player.
    /// </summary>
    /// <param name="value">
    /// The InputValue wrapper passed automatically by the Player Input component. 
    /// </param>
    public void OnMove(InputValue value)
    {
        // Extract the Vector2 from the InputValue
        Vector2 inputVector = value.Get<Vector2>();
        Debug.Log("OnMove called: " + inputVector);

        // Convert the 2D input into a 3D direction on the XZ plane
        Vector3 direction = new Vector3(inputVector.x, 0f, inputVector.y);
        Debug.Log("OnMove direction: " + direction);

        // Call the movement component with the calculated direction
        _moveTransform.Move(direction);

    }//end OnMove()
```

<br>

> **✔️ CHECK POINT**
> 
> Save your script, switch back to the Unity editor, and press **Play** to test the changes in action.

<br>

---

# 🎉 New Achievement: Player Controller

We now have a `PlayerContoller` that works exactly like our `CharacterController` but implements **Unity's New Input System** 

```csharp
using UnityEngine;
using UnityEngine.InputSystem;
 
public class PlayerController : MonoBehaviour
{
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
 
    
    /// <summary>
    /// Triggered by the Move input Action. Converts the 2D input from the player
    /// (keyboard, joystick, or gamepad) into a 3D direction and passes it to the 
    /// MoveTransform component to move the player.
    /// </summary>
    /// <param name="value">
    /// The InputValue wrapper passed automatically by the Player Input component. 
    /// </param>
    public void OnMove(InputValue value)
    {
        // Extract the Vector2 from the InputValue
        Vector2 inputVector = value.Get<Vector2>();
        Debug.Log("OnMove called: " + inputVector);

        // Convert the 2D input into a 3D direction on the XZ plane
        Vector3 direction = new Vector3(inputVector.x, 0f, inputVector.y);
        Debug.Log("OnMove direction: " + direction);
        

        // Call the movement component with the calculated direction
        _moveTransform.Move(direction);

    }//end OnMove()
 
 
 
}//end PlayerController
```

It’s important to note that the new Input System, just like the old one, **reacts every frame** of the game. This means input values are only recorded during `Update()`.

But what if we want to use our `MoveRigidbody` component to control the object’s movement? Since this component uses **physics**, any changes to velocity or forces need to be applied in `FixedUpdate()`, which runs at a consistent interval in sync with the physics engine.

What would a controller that bridges frame-based input and physics-based movement look like? 
When would we use this type of controller instead of moving an object directly?

---

**[<< Return to New Input System tutorial](unity-new-input-system.md)** | **[Continue to Vehicle-Controller tutorial >>](vehicle-controller.md)**