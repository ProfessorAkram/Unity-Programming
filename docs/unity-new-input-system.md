# Unity's New Input System
Unity’s original input system is **polling-based**: every frame your code checks whether input has happened. This works for simple projects, but it gets brittle as your game grows or targets multiple platforms with different control schemes.

Unity’s new Input System is **event-driven and action-based**. Instead of constantly checking for specific keys or buttons, you define **Actions** (_e.g., Move, Jump, Attack_) and bind them to any device. When input occurs, the system raises an event and your code responds. (You can still poll an Action’s value when you need to.)

**Benefits:**

- Define actions like _Move, Jump, or Attack_ instead of hardcoding buttons.
- Seamlessly support **keyboard, mouse, gamepad, touch, and custom controllers** from one setup.
- Separate gameplay logic from input bindings, **keeping code cleaner and easier to scale**.
- Get built-in **support for rebinding, local multiplayer, and context-sensitive input**.

In this lesson, we’ll create a new Input Actions asset from scratch, configure action maps and bindings, and connect them to gameplay. By the end, you’ll have a flexible, extensible input framework—ready for quick prototypes or full-scale production.

---
## Input System Package
By default the latest Unity 6.2 3D core template should come standard with the new input system. However, if it is not installed or if you are working with an older, or different template the **Input System Package** will first need to be installed. 

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
We now have the new input system installed and turned on, but it doesn't actually do anything yet. Unity won't know what a _Jump_ or a _Move_ means untile we define it. 

To make the new input system work, we need to create an **Input Actions Asset**. This asset can be thought as a _Input Database_ stroing all of the following information: 

- **Action Maps**: Group actions by context (Player, UI, Vehicle, etc.)
- **Actions**: Define what the player wants to do (Jump, Fire, Move)
- **Bindings**: Assign how those actions are triggered (Spacebar, Left Stick, Tap, etc.)
- **Control Schemes**: Decide which devices the player can use (keyboard, mouse, gamepad, etc.)

#### 1. **Create** a New Input Actions Asset
- In the **Project window**, right-click and create a new folder named **InputAssets**
- Right-click on the folder and choose `Create > Input Actions`.
- Name the asset **PlayerControls** or **InputActions**.

> [!WARNING]
> If you started with the 3D core template a default **InputSystems_Actions** will already be included in the Assets folder. To avoid confusion, go ahead and **delete this asset**. Also, note that if you have imported the Unity Character Controlle, it also has it's own _StarteAssets_ input actions asset. We will leave this asset for now.

#### 2. **Open** the Input Action Asset
- Double click on the newly created input asset
- In the **Input Actions Editor** dialog window, click **Auto-Save** on the right hand side of the screen.
  - This ensures any changes made in the editor will auto save if exited.

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
> When naming Actions, use **one-word verbs** that clearly describe the player’s intention (e.g., Jump, Fire or Move). Clean, concise names make it easier to **reference Actions in code**, read the Input Actions Asset at a glance, and **maintain consistency** across Action Maps.

Each action Action has a **type** that defines how input is processed:
- **Button:** Triggered when pressed (e.g., Jump, Fire)
- **Value:** Returns continuous input (e.g., Move returns a Vector2 for WASD or joystick)
- **Pass-Through:** Similar to Value but does not block other actions

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
The newly created **Move** action represents the payer intention; what they want to do; but not how they do it. **Bindings** are the actual keys, buttons, or sticks that trigger each Action. Bindings translate **abstract actions** (like _Jump or Fire_) into **real input devices**.

#### 5. **Add** bindings
- With the **Move** Action selected in the **Input Actions Editor**
- Click on the **`<No Binding>`** and delete, this default binding
- Select **Move** Action and click on the `+` next to the right of the Action.
  -  Select **`Add Up/Down/Left/Right Composite`**
 
A **composite binding** allows for multiple physcial inputs to be comibined into a **single logincal value** for an Action. This is especially useful for movement because a character usually moves in two directions at once (horizontal and vertical), and players might use different devices to control it.

Think of a composite binding like a music keyboard, where each key (Up, Down, Left, Right) makes a sound on its own. But you can **press two keys at the same time** to play a chord. The composite binding “hears” all the keys pressed at once and combines them into **one output** (like a chord) that your game can understand.

#### 6. **Setup** the Keyboard composite bindings
- Click **2D Vector Composite** dropdown arrow
  - This exapnds the Up/Down/Left/Right composites
- Click on the **Up** composite
  - In the **Bindinds Properites** click on **Path**
  - Choose **`Keyboard > By Character Mapped to Key > W [Keybboard]`**
- **Repeat** this process for the rest of the composites settings.

#### 7. **Add** Gamepad bindings
- Select **Move** Action and click on the `+` next to the right of the Action.
  -  Select and add another new **`Add Up/Down/Left/Right Composite`**
- Click **2D Vector Composite** dropdown arrow, expand the settings
- Click on the **Up** composite
  - In the **Bindinds Properites** click on **Path**
  - Choose **`Gamepad > Left Stick/Up`** 
- **Repeat** this process for the rest of the composites settings.

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

Importantly, **setting up a Control Scheme doesn’t require any changes to individual bindings**. You just define the group of devices. For example, if you create a Control Scheme for Keyboard & Mouse, Unity will automatically only listen to the bindings for those devices, you don’t need to do any extra work.

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
 - Repeat steps 9 and 10 creating a control scheme for **Gamepad**
 - Set the **Device Type** to **Gamepad** 

#### 12. **Save** Action Settings
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

The Behavior setting determines how the Player Input component sends input to your game. There are three main options:

 - **Invoke Unity Events:**
  - Triggers UnityEvents when Actions occur.
  - Easy for beginners or visual workflows because you can assign functions directly in the Inspector.

- **Send Messages**
  - Calls methods on the GameObject that have the same name as the Action.
  - Lightweight, good if you want to keep things organized in scripts without using UnityEvents.

- **Broadcast Messages**
  - Sends input messages to all child objects of the Player GameObject.
  - Less common, but useful if multiple components need to react to the same Action.
 

 
  
