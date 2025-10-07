# Vehicle Controller
If our game requires precise **physics-based movement** while still responding to player input, like in a racing game where we want to control a vehicle that obeys physics, we need a way to **leverage both input and physics updates**.

To do this, we will create a `VehicleController` that collects input via the new Input System and feeds it into our `MoveRigidbody` component, ensuring smooth, consistent movement while respecting Unity’s physics update cycle.

#### 1. **Create** the class
- **Create a new C# class** named `VehicleController` and open it in your editor.
- **Double-click on `VehicleController`** to open it in your editor.

#### 2. **Import** the Namespace

```csharp
using UnityEngine.InputSystem;
```

#### 3. Get a Reference for the `MoveRigidbody` component
```csharp
    //Reference to the MoveTransform component 
    private MoveRigidbody _moveRigidbody;

    // Start is called once before the first Update
    private void Start()
    {
        // Check if MoveRigidbody component DOES NOT EXIST
        if (!TryGetComponent<MoveRigidbody>(out _moveRigidbody))
        {
            Debug.LogError("MoveRigidbody component missing!");

        }//end if (!TryGetComponent<MoveRigidbody>(out _moveRigidbody))
      
    }//end Start()
```
---

## Getting Vector Input
Just like our previous `PlayerController`, the _Move_ action provides a **Vector2** representing the X and Y input from the keyboard, joystick, or gamepad. Since our `MoveRigidbody` component uses **physics**, all movement updates must happen in `FixedUpdate()`, including calls to `Move()`.

This means we **cannot **calculate and apply movement directly inside `OnMove()`. Instead, we need to:
- **Record the input vector** when the action is triggered.
- **Convert it to a 3D direction** inside `FixedUpdate()`.
- **Call** `Move()` on the `MoveRigidbody` with that calculated direction.

To accomplish this, the first step is to create a **field to store the input vector**. This field will be updated in `OnMove()` whenever the player provides input, and then `FixedUpdate()` will use it to calculate the direction and move the object smoothly with physics.

#### 4. Create a field for Input Vector

```csharp
    // Stores the current 2D input from the SendMessage system
    private Vector2 _inputVector;
```

## Creating the `OnMove()` method
Next, we will create the `OnMove()` method, which simply updates the `_inputVector` field with the value passed.

#### 5. Create the `OnMove()` method

```csharp
    /// <summary>
    /// Triggered by the Move input Action. Converts the 2D input from the player
    /// (keyboard, joystick, or gamepad)
    /// </summary>
    /// <param name="value">
    /// The InputValue wrapper passed automatically by the Player Input component. 
    /// </param>
    public void OnMove(InputValue value)
    {
        // Extract Vector2 from InputValue
        _inputVector = value.Get<Vector2>();
        Debug.Log("OnMove called: " + _inputVector);

    }//end OnMove()

```
---

## Applying Movement in FixedUpdate

Since our `MoveRigidbody` component relies on physics, we need to update the movement inside `FixedUpdate()`. Physics updates in Unity are frame-independent and occur at consistent intervals, which ensures smooth, predictable motion regardless of frame rate. 



#### 6. Create a `FixedUdpate()`
To use the input collected from `OnMove()`, we need to create the following in the `FixedUpdate()`:

```csharp
    private void FixedUpdate()
    {
        // Code goes here
    }//end FixedUpdate()
```

- **Convert the 2D input to 3D**
  - Take the `_inputVector` from `OnMove()` (Vector2).
  - Map X to X and Y to Z to move on the XZ plane, using `direction`

```csharp
    Vector3 moveDirection = new Vector3(_inputVector.x, 0f, _inputVector.y);
```

- Check if there is **significant input**
  – If the vector magnitude is very small (near zero), we treat it as no input.

```csharp
  if (moveDirection.sqrMagnitude > 0.01f)
  {
      // Code for movement goes here
  }
  else
  {
      // Code for braking goes here
  }//end if (moveDirection.sqrMagnitude > 0.01f)

```
**- If there is Input**
  - Normalize the direction
    - Ensures consistent movement speed, even for diagonal input.
    - Optional, since MoveRigidbody already normalizes the direction internally, but it is still considered good practice.
  - Call `Move()` on `MoveRigidbody`
    - Pass the normalized `direction` to `Move()`
    - `MoveRigidbody` handles updating the internal Direction and applying smooth acceleration.
      
  ```csharp
          if (moveDirection.sqrMagnitude > 0.01f)
        {
             //Optional, but good practice to normalize direction before passing it
             moveDirection.Normalize();

            // Move() updates Direction internally
            _moveRigidbody.Move(moveDirection); 
        }
```   
**- else** (No input detected)
    - **Brake when no input is present**
      – If the player isn’t pressing any keys or joystick directions, we call Brake() to gradually stop the object.


 ```csharp
            else
        {
            _moveRigidbody.Brake();
            
        }//end if (moveDirection.sqrMagnitude > 0.01f)
```
---

## Handling Turns

In many games, especially racing or driving games, **how an object turns is just as important as how it moves**. Smooth turning makes the vehicle feel responsive and realistic, while instant rotation can feel stiff or unnatural.

### Who Handles Turning

Turning is a **decision about movement direction**, not about raw physics. Therefore, it belongs in the **controller** rather than the `MoveRigidbody` component:

- **Controller Responsibility:** Converts player input into a target direction and rotation.
- **MoveRigidbody Responsibility:** Moves the object in a given direction using forces, velocity, and acceleration.

Keeping these responsibilities separate makes your code cleaner, easier to maintain, and more flexible for different movement behaviors.

### Implementing Smooth Turns 
To implement smooth turning, we need to control how fast the vehicle rotates. Increase it for snappier turns, decrease it for more gradual turns. We can do this by defining a turning speed. 

#### 7. **Create** a Turning Speed field

 ```csharp
    [SerializeField]
    [Tooltip("How quickly the vehicle rotates toward the input direction.")]
    private float _turnSpeed = 5f;
```

### Calculating Turns

To make our vehicle rotate smoothly toward the input direction, we use **interpolation techniques**. Interpolation lets us gradually move from a **current value** to a **target value** over time, instead of snapping instantly.

The most common way to interpolate values is **Linear Interpolation**, or **Lerp**.
- **Lerp** takes a **start value**, a **target value**, and a **factor** (usually between 0 and 1).
- It returns a value **somewhere between the start and target** based on that factor.
- Repeatedly calling Lerp each frame or physics step gradually moves a value toward the target.
- Lerp works great for simple scalars like speed, opacity, or health bars.

However, when dealing with **3D rotations or directions**, Lerp has a limitation: it interpolates in a straight line. This can produce **unnatural turning** behavior for objects moving in 3D space, because directions live on a **sphere**, not along a straight line.

For smooth 3D turns, we use **Spherical Linear Interpolation**, or **Slerp**.
- Slerp operates along the **surface of a sphere**, smoothly rotating one direction toward another.
- It ensures the object takes the **shortest rotational path**, creating natural, realistic turns.
- In Unity, `Vector3.Slerp(currentDirection, targetDirection, time)` or `Quaternion.Slerp(currentRotation, targetRotation, time)` is commonly used.
- By repeatedly applying Slerp in `FixedUpdate()`, we can rotate a vehicle gradually while still respecting physics-based movement.

Before we interpolate, we need to know **where the object is facing**. In a vehicle or character controller:
- The **target direction** is the **player input**, calculated from the input vector (e.g., _inputVector converted to a Vector3).
- This target vector represents the **desired forward direction** in world space.

Slerp will then **rotate the object’s current forward** toward this target vector over time, producing smooth turns instead of instant snapping. 

In Unity, the forward direction is accessed using **`transform.forward`**:
- Every GameObject has a **Transform**, which defines its **position**, **rotation**, and **scale**.
- `transform.forward` is a **vector pointing along the object’s Z-axis** in world space, taking into account its current rotation.
- By assigning a new value to `transform.forward` using `Slerp`, we **gradually update the object’s facing direction**, moving smoothly from the current orientation toward the target direction.


#### 8. **Update** Rotation on `FixedUpdate()`
- Implement `Slerp` to create smooth turning
```csharp
          if (moveDirection.sqrMagnitude > 0.01f)
        {
             //Optional, but good practice to normalize direction before passing it
             moveDirection.Normalize();

            // Smoothly rotate the vehicle toward the input direction
            transform.forward = Vector3.Slerp(transform.forward,moveDirection, turnSpeed 
            * Time.fixedDeltaTime);

            // Move() updates Direction internally
            _moveRigidbody.Move(moveDirection); 
        }
```


---

# 🎉 New Achievement: Vehicle Controller

Now that we’ve connected player input to physics-based movement and added smooth turning, we have a working `VehicleController` that leverages the new Input System alongside our `MoveRigidbody` component.

```csharp
using UnityEngine;
using UnityEngine.InputSystem;
 
public class VehicleController : MonoBehaviour
{
    //Reference to the MoveTransform component 
    private MoveRigidbody _moveRigidbody;

    // Stores the current 2D input from the SendMessage system
    private Vector2 _inputVector;

    [SerializeField]
    [Tooltip("How quickly the vehicle rotates toward the input direction.")]
    private float _turnSpeed = 5f;

    // Start is called once before the first Update
    private void Start()
    {
        // Check if MoveRigidbody component DOES NOT EXIST
        if (!TryGetComponent<MoveRigidbody>(out _moveRigidbody))
        {
            Debug.LogError("MoveRigidbody component missing!");

        }//end if (!TryGetComponent<MoveRigidbody>(out _moveRigidbody))
      
    }//end Start()


    private void FixedUpdate()
    {
        Vector3 moveDirection = new Vector3(_inputVector.x, 0f, _inputVector.y);

        if (moveDirection.sqrMagnitude > 0.01f)
        {
             //Optional, but good practice to normalize direction before passing it
             moveDirection.Normalize();

            // Smoothly rotate the vehicle toward the input direction
            transform.forward = Vector3.Slerp(transform.forward,moveDirection, turnSpeed 
            * Time.fixedDeltaTime);

            // Move() updates Direction internally
            _moveRigidbody.Move(moveDirection); 
        }else
        {
            _moveRigidbody.Brake();
            
        }//end if (moveDirection.sqrMagnitude > 0.01f)

    }//end FixedUpdate()
 

    /// <summary>
    /// Triggered by the Move input Action. Converts the 2D input from the player
    /// (keyboard, joystick, or gamepad)
    /// </summary>
    /// <param name="value">
    /// The InputValue wrapper passed automatically by the Player Input component. 
    /// </param>
    public void OnMove(InputValue value)
    {
        // Extract Vector2 from InputValue
        _inputVector = value.Get<Vector2>();
        Debug.Log("OnMove called: " + _inputVector);

    }//end OnMove()

 
}//end VehicleController 
```
With this setup, you have a **modular controller system**: the input logic is separated from the movement logic, and both are tied into the physics system correctly.

---

**[<< Return to Player-Controller tutorial](player-controller.md)**