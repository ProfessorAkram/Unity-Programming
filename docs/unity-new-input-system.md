# Unity's New Input System
Unity’s original input system is **polling-based**: every frame, your code checks whether input has happened. This works for simple projects, but it gets brittle as your game grows or targets multiple platforms with different control schemes.

Unity’s new Input System is **event-driven and action-based**. Instead of constantly checking for specific keys or buttons, you define **Actions** (_e.g, Move, Jump, Attack_) and bind them to any device. When input occurs, the system raises an event, and your code responds. (You can still poll an Action’s value when you need to.)

**Benefits:**

- Define actions like _Move, Jump, or Attack_ instead of hardcoding buttons.
- Seamlessly support **keyboard, mouse, gamepad, touch, and custom controllers** from one setup.
- Separate gameplay logic from input bindings, **keeping code cleaner and easier to scale**.
- Get built-in **support for rebinding, local multiplayer, and context-sensitive input**.

In this lesson, we’ll create a new Input Actions asset from scratch, configure action maps and bindings, and connect them to gameplay. By the end, you’ll have a flexible, extensible input framework—ready for quick prototypes or full-scale production.

---
## Input System Package
By default, the latest Unity 6.2 3D core template should come standard with the new input system. However, if it is not installed or if you are working with an older or different template, the **Input System Package** will first need to be installed. 

#### 1. **Install** the Input System Package:
- Open Unity Package Manager `Window > Package Manager`.
- Search for the **Input System package** and install it.
- After installation, Unity will prompt you to restart and enable the new input system. Choose **Yes** to enable it.
- Unity will need to **restart**.

### Enabling the New Input System
Even after installing the Input System package, your project may still be using Unity's old Input Manager by default. To make sure your scripts use the correct system, you’ll need to update the Active Input Handling setting.

#### 2. **Enable** the New Input System in Project Settings
- Go to `Edit > Project Settings > Player`
- Under **Other Settings**, find **Active Input Handling**
- Choose one of the following options:

| Option                         | Description                                                | When to Use                                                                                                           |
| ------------------------------ | ---------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| **Input Manager (Old)**        | Uses only the legacy `Input.GetKey`, `Input.GetAxis`, etc. | Only if you're working on a very old project and **don’t plan** to use the new system yet.                            |
| **Input System Package (New)** | Enables only the new action-based input system.            | Best choice for **new projects** or **full migrations** to the new system.                                            |
| **Both**                       | Enables both input systems at once.                        | Use this if you're **transitioning gradually**, or working with **assets/plugins** that still rely on the old system. |

- After selecting **Input System (New) or Both**, Unity may prompt you to **restart**; click **Yes**.

---
## Input Action Asset
We now have the new input system installed and turned on, but it doesn't actually do anything yet. Unity won't know what a _Jump_ or a _Move_ means until we define it. 

To make the new input system work, we need to create an **Input Actions Asset**. This asset can be thought of as an _Input Database_ storing all of the following information: 

- **Action Maps**: Group actions by context (Player, UI, Vehicle, etc.)
- **Actions**: Define what the player wants to do (Jump, Fire, Move)
- **Bindings**: Assign how those actions are triggered (Spacebar, Left Stick, Tap, etc.)
- **Control Schemes**: Decide which devices the player can use (keyboard, mouse, gamepad, etc.)

#### 1. **Create** a New Input Actions Asset
- In the **Project window**, right-click and create a new folder named **InputAssets**
- Right-click on the folder and choose `Create > Input Actions`.
- Name the asset **PlayerControls** or **InputActions**.

> [!WARNING]
> If you started with the 3D core template, a default **InputSystems_Actions** will already be included in the Assets folder. To avoid confusion, go ahead and **delete this asset**. Also, note that if you have imported the Unity Character Controller, it also has its own _StarteAssets_ input actions asset. We will leave this asset for now.

#### 2. **Open** the Input Action Asset
- Double-click on the newly created input asset
- In the **Input Actions Editor** dialog window, click **Auto-Save** on the right-hand side of the screen.
  - This ensures any changes made in the editor will auto-save if exited.

---

## Action Maps
An **Action Map** is essentially a container for related input actions. It lets you group actions based on the **context** in which they occur. For example, the controls for navigating a menu are very different from the controls used to move a character in gameplay. Action Maps allow you to separate these contexts cleanly.

Some common Action Maps include:
- **Player Action Map:** Move, Jump, Fire
- **Menu Action Map:** Navigate, Confirm, Cancel
- **Vehicle Action Map:** Steer, Accelerate, Brake

Action Maps can be enabled and disabled based on gameplay state. For example, a **UI Action Map** may be active when the player is in a menu, while the **Player Action Map** is enabled only during gameplay.

Using multiple Action Maps keeps your project organized, modular, and easy to extend. You can even combine them with **Control Schemes** so the same Action Map works across different devices.


#### 1. **Add** an Action Map
- In the left panel, of the **Input Actions Editor**, click the **`+`** button next to Action Maps.
- Name it **Player**
  - This action map will define all actions related to controlling the character.
 
### Actions

An **Action** represents a single player intention or **event**, like _Jump, Fire, or Move_. Think of it as a **named command** that your gameplay code will respond to. Every _Action_ belongs to a specific _Action Map_, which provides context. For example, a **Confirm** action might belong to a **UI Action Map**, while **Jump** would belong to the **Player Action Map**.

> [!NOTE]
> When naming Actions, use **one-word verbs** that clearly describe the player’s intention (e.g., Jump, Fire, or Move). Clean, concise names make it easier to **reference Actions in code**, read the Input Actions Asset at a glance, and **maintain consistency** across Action Maps.

Each **Action** has a **type** that defines how input is processed:
- **Button:** Triggered when pressed (e.g., Jump, Fire)
- **Value:** Returns continuous input (e.g., Move returns a Vector2 for WASD or joystick)
- **Pass-Through:** Similar to Value, but does not block other actions

#### 2. **Rename** new Action
- In the center panel, of the **Input Actions Editor**, right-click on the default **New action** and chose **Rename**
- Rename this action to **Move**

#### 3. **Set** Action Type
- Select the **Move** Action in the **Input Actions Editor**
- In the **Action Properties** panel, click the **Action Type** drop down and choose **Value**

After defining the** Action Type**, the next step is to choose a **Control Type**. The **Control Type** determines what kind of input data the Action expects. It helps Unity know how to interpret the binding for this Action.

**Common Control Types:**
- **Button**: For single presses (e.g., Jump, Fire)
- **Vector2**: For two-dimensional input like movement or camera look (e.g., WASD keys or a joystick)
- **Vector3**: For three-dimensional input (less common for standard player input)
- **Axis**:  For single-axis input, such as triggers or sliders
- **Quaternion**: For rotation input, mainly used with motion controllers

#### 4. **Set** Control Type
- Select the **Move** Action in the **Input Actions Editor**
- In the **Action Properties** panel, click the **Control Type** drop down and choose **Vector2**

### Bindings
The newly created **Move** action represents the player's intention; what they want to do, but not how they do it. **Bindings** are the actual keys, buttons, or sticks that trigger each Action. Bindings translate **abstract actions** (like _Jump or Fire_) into **real input devices**.

#### 5. **Add** bindings
- With the **Move** Action selected in the **Input Actions Editor**
- Click on the **`<No Binding>`** and delete, this default binding
- Select **Move** Action and click on the `+` next to the right of the Action.
  -  Select **`Add Up/Down/Left/Right Composite`**
 
A **composite binding** allows for multiple physical inputs to be combined into a **single logical value** for an Action. This is especially useful for movement because a character usually moves in two directions at once (horizontal and vertical), and players might use different devices to control it.

Think of a composite binding like a music keyboard, where each key (Up, Down, Left, Right) makes a sound on its own. But you can **press two keys at the same time** to play a chord. The composite binding “hears” all the keys pressed at once and combines them into **one output** (like a chord) that your game can understand.

#### 6. **Setup** the Keyboard composite bindings
- Click **2D Vector Composite** dropdown arrow
  - This expands the Up/Down/Left/Right composites
- Click on the **Up** composite
  - In the **Bindinds Properites** click on **Path**
  - Choose **`Keyboard > By Character Mapped to Key > W [Keybboard]`**
- **Repeat** this process for the rest of the composite settings.

#### 7. **Add** Gamepad bindings
- Select **Move** Action and click on the `+` next to the right of the Action.
  -  Select and add another new **`Add Up/Down/Left/Right Composite`**
- Click **2D Vector Composite** dropdown arrow, expand the settings
- Click on the **Up** composite
  - In the **Bindinds Properites** click on **Path**
  - Choose **`Gamepad > Left Stick/Up`** 
- **Repeat** this process for the rest of the composite settings.

**Use the following bindings for the Move Action:**

| Direction | Keyboard | Gamepad          |
| --------- | -------- | ---------------- |
| Up        | W        | Left Stick Up    |
| Down      | S        | Left Stick Down  |
| Left      | A        | Left Stick Left  |
| Right     | D        | Left Stick Right |


When the player presses W + D, the composite binding outputs Vector2(x=1, y=1). Your movement code can then read this vector and move the character diagonally.

### Control Schemes 
As mentioned earlier, Unity's new input system is **event-driven**, which means it only reacts when input happens instead of constantly checking for it like the older **polling-based** system. This is more efficient, but if your project has bindings for multiple devices, Unity still needs to **listen to all those bindings**, even for devices that aren’t connected. For example, if a player is using a keyboard and mouse on a PC, there’s no reason to listen for gamepad inputs.

This is where **Control Schemes** come in. A Control Scheme is a group of devices that a player can use together to control the game. It tells Unity **which devices' bindings should be active**. Using Control Schemes helps optimize input handling, keeps your project organized, and makes it easier to support multiple devices or players. While Control Schemes aren’t strictly required, **they make your input setup cleaner, more flexible, and easier to scale**.

For Unity to know which bindings belong to which device group, you **must assign each binding to one or more Control Schemes** using the “Use in Control Scheme” option.

#### 8. **Add** a Control Scheme
- In the **Input Actions Editor**, in the left pane at the top
  - click the **All Control Scheme** drop down
  - Choose **Add Control Scheme**

 #### 9. **Create** a Control Scheme
- In the **Add Conotrl Scheme** dialog
  - Set the name to **Keyboard & Mouse**
 
 #### 10. **Add** Device Type
- Under **Device Type**, press the **`+`**
  - Add **Keyboard**
- Repeat and Add **Mouse**

#### 11. **Create** Gamepad Scheme
 - Repeat steps 9 and 10, creating a control scheme for **Gamepad**
 - Set the **Device Type** to **Gamepad**

#### 12. **Assign** the Keyboard Control Scheme 
- Select the **Up** binding for the `W` Key
- In the **Binding Properties** in the right pane
  - Check the **KeyBoard & Mouse** under the **Use in Control Scheme** setting
- Repeat for each Keyboard binding

#### 13. **Assign** the Gamepad Control Scheme 
- Select the **Up** binding for the `Left Stick /Up` gamepad
- In the **Binding Properties** in the right pane
  - Check the **Gamepad** under the **Use in Control Scheme** setting
- Repeat for each Gamepad binding

#### 14. **Save** Action Settings
- Click **Save** or ensure that **Auto-Save** is enabled on the **Input Actions Editor**
- Exit the **Input Actions Editor**

---

## Player Input Component
Now that we have our **Input Actions Asset** and **Action Map** set up, the next step is to attach them to our Player GameObject using a **Player Input component**. This component acts as a bridge between the input system and your scripts: it listens for Actions and automatically sends the input values or triggers functions in your scene.

Think of the Player Input component as a **manager**: it doesn’t handle the gameplay itself, but it tells your game **when and how input happens**, so your scripts can respond.

#### 1. **Add** Player Input Component 
- **Select** your Player GameObject in the **Hierarchy**.
- In the **Inspector Window**, click `Add Component > Player Input`

#### 2. **Apply** Player Input Settings
- Under the **Player Input** settings in the **Inspector Window**
  - Drag and Drop the **PlayerControls** Input Action to the **Action**
  - Assign the **Default Scheme** to **Keyboard & Mouse** 
  - Assign the **Defaulat Map** to **Player**
 
### Choosing a Behavior
The last **Player Input Setting** that needs to be set is the **behavior setting**, which determines how the Player Input component sends input to your game. 

There are three main **Behavior options**, and each one uses a different **method signature** when calling your scripts:

 - **Invoke Unity Events:**
    - Triggers UnityEvents when Actions occur.
    - Visual workflows; functions are assigned directly in the Inspector.
    - Unity calls your method with a **CallbackContext** that provides **full event details**, such as:
      - Whether the input **started**, **performed**, or **canceled**
      -  _Ex._ `OnMove(InputAction.CallbackContext context)`
    - Useful for **tap vs hold**, **charging attacks**, or **timing-based actions**

- **Send Messages**
    - Calls methods on the GameObject that have the **same name as the Action**.
    - Clean and simple for script-based setups.
    - Passes an **InputValue** which only gives the **final input data**, such as:
      - A `Vector2` for movement
      - A `float` for trigger pressure
      - A `bool` for button press
      - _Ex._ `OnMove(InputValue value)`
    - Best for **simple actions** like movement, jump, or fire

- **Broadcast Messages**
    - Works like Send Messages, but **sends the method call to all child objects** as well.
    - Uses the same **Input Value** as **Send Message**
    - Useful when multiple components need to respond to the same Action.

> [!NOTE]
> A **method signature** is the combination of a **function’s name and the type of parameter it expects**. Two methods can have the same name but be treated as different methods if their parameter types are different. 

#### Which Behavior to Choose?

- **Invoke Unity Events** is strict and intentional. Since you manually connect methods in the Inspector, Unity expects the function to exist and match perfectly. If something is missing or mismatched, the input won’t fire. This makes it reliable for controlled setups, but a bit rigid.

- **Send Messages** are more flexible. Unity simply asks, “Does any script care about this Action?” If a matching method exists, it gets called. If not, nothing breaks. This is ideal for script-based workflows and rapid iteration.


In this lesson, we’ll be using **Send Messages** for its simplicity and flexibility.

#### 3. **Set** Behavior
- Under the **Player Input** settings in the **Inspector Window**
  - Set the **Behavior** property to **Send Message**

---

# Player Controller 
Now that we have our input system set up, we can start work on the `PlayerController` class that defines the methods triggered by the **Player Input component**. 

**1. Create a new C# class** named `PlayerController` and open it in your editor.

**2. Double-click on `PlayerController`** to open it in your editor.

## Get the Move Component
In our previous lessons, we created `MoveTransform` and `MoveRigidbody` classes. Each of these moves the GameObject in a different way. When setting up our `PlayerController`, we need to decide which movement style we want to use and get a reference to the appropriate component. For this lesson, we will keep things simple and use the `MoveTransform` component. In the end the `PlayerController` should work exactly like the `CharacterController` just with the new Input System

#### 3. Get a Reference for the `MoveTransform` component

```csharp
    //Reference to the MoveTransform component 
    private MoveRigidbody _moveRigidbody;

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
