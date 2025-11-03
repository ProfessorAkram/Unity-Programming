# Pausing the Game 

Another common function in many games is implementing a **pause menu**. Pausing usually involves freezing gameplay and displaying a menu where players can resume, restart, or exit.

When the `PauseMenu` is triggered, the menu will load as an **additive** scene, and we will stop all gameplay, adjusting the `Time.timeScale` property. When exiting pause, we will do the reverse, and because the same button or trigger will turn on and off the pause, we can do all this in a single `TogglePause ()` method that can check against a single bool. 

---
> ### ⏱️ TIME.timeScale
> In Unity, `Time.timeScale` is a **global multiplier** for how fast time progresses in the game. It affects **all time-dependent systems**, including:
> - `Update()` loops (for movement, animations, etc.)
> - Physics simulations (`Rigidbody` updates)
> - Coroutines that use `WaitForSeconds`
>
> **Key points:**
> - `Time.timeScale = 1` → Normal time progression (default).
> - `Time.timeScale = 0` → Time is frozen, effectively pausing the game.
> - `Time.timeScale < 1` → Time moves slower than normal (slow motion).
> - `Time.timeScale > 1` → Time moves faster than normal (fast forward).
>
> **Important Notes:**
> UI elements and input are **not automatically affected by `Time.timeScale`**. This is why pause menus can still respond to clicks even when the game is frozen.
> Coroutines using `WaitForSecondsRealtime` are **unaffected by timeScale**, while `WaitForSeconds` is scaled by it.

---

## :hammer_and_wrench: Toggling Pause 

#### 1. Create a Boolean to Track Pause

To determine if the game is currently paused, we define a **read-only boolean property**:

```csharp
// Read-only property that returns true if the game is currently paused
public bool IsPaused => Time.timeScale == 0;
```

>[!NOTE]
> `IsPaused` is declared using shorthand for a getter using the **expression-bodied syntax** that is equivalent to: 
> ```csharp
>  public bool IsPaused
> {
>    get
>    {
>        return Time.timeScale == 0;
>    }
>}
>```

This property is **read-only**, meaning its value cannot be set directly. Instead, it automatically reflects the current pause state: it returns true when `Time.timeScale` is 0 (paused) and `false` otherwise. Changing the `Time.timeScale` elsewhere in the game will automatically update the value of `IsPaused`.

#
 
#### 1. Create a TogglePause() method

```csharp
    /// <summary>
    /// Toggles the pause state of the game.
    /// Freezes or resumes gameplay using Time.timeScale,
    /// and loads/unloads the PauseMenu scene.
    /// </summary>
    public void TogglePause()
    {
        // Only allow pausing/unpausing during gameplay
        if (CurrentState != GameState.GamePlay && !IsPaused)
            return;

        //If the game is not paused
        if (!IsPaused)
        {
            // Pause the game
            Time.timeScale = 0f;
            LoadScene(_pauseMenuScene);
        }
        else
        {
            // Resume the game
            Time.timeScale = 1f;
            UnloadScene(_pauseMenuScene);

        }//end if (!IsPaused)

    }//end TogglePause()

```
**How it Works**
- `TogglePause()` only allows pausing if the game is in the `GamePlay` state, preventing accidental pauses in menus or GameOver screens.
- When the game is **not paused**, `TogglePause()` freezes gameplay (`Time.timeScale = 0`) and loads the pause menu additively.
- When the game is **already paused**, `TogglePause()` resumes gameplay (`Time.timeScale = 1`) and unloads the pause menu.
- This approach ensures a smooth transition between gameplay and pause without reloading the level or losing game state.

---

## :hammer_and_wrench: Triggering Pause via Player Input

Now that we have created the  `TogglePause()` method in the GameManager, the next step is to allow players to trigger it using input. This ensures the game can pause or resume dynamically without requiring any manual calls in code.

We'll allow the player to pause the game by adding a **Pause action** to our existing **PlayerController Input Action Asset**. This **action is independent of any specific object in the scene**, so the persistent GameManager in the Bootstrap scene can reference it and listen for input at any time. Using the **Observer pattern**, the GameManager will subscribe to the Pause action event, meaning it will automatically react whenever the player presses the assigned button—triggering TogglePause() without any constant polling or manual checks.

--- 
#### 1. Add a Pause Action to the PlayerController Input Action Asset
1. In the Unity Editor, **projoect panel**, double-click on the **PlayerController Input Action Asset**
2. In the **Input Action Window**, add a new action
   - (e.g., Pause, and bind it to a key or button such as Escape, Start, or P.)
3. This action will represent the player's intent to pause the game.

>[!WARNING]
> Even though the asset is named **PlayerController**, it doesn’t have to live on the player object;  it’s just **a container for input definitions**.

#

#### 2. Input Action Assets Are Independent

1. Our **GameManager** will need a reference to the **PlayerController Input Action Asset**

```csharp
[Header("Input")]
[SerializeField]
[Tooltip("Reference to the Input Action Asset used to handle pause and other input.")]
private InputActionAsset _inputActions;

```

2. In the Unity Editor, select the **GameManager** prefab from the **Bootstrap** scene, in the **hierarchy panel** 
3. In the **Inspector panel**, Drag your **Input Action Asset** into the **Input Actions** field.
   - The GameManager can now enable the action map and subscribe to the Pause action.
4. **Applie Overides** to the **GameManager** prefab

#

## Using the Observer Pattern to Listen for Pause

As we discussed in the previous lesson, the [Observer pattern](observer-pattern.md) allows one object to broadcast a notification while other objects independently react. This decouples systems and avoids hard-coded dependencies.

In Unity, the **new Input System** provides a perfect example of this in action. Instead of constantly checking for input in every `Update()` loop, we can **subscribe to input events** and react only when they occur:
- The **Input Action** acts as the **Subject**, emitting an event whenever the player performs an action (e.g., pressing the Pause button).
- The **GameManager** acts as the **Observer**, subscribing to the event and responding automatically.


#### 1. Subscribe and Unsubscribe to the Pause
In order for the **GameManager** to respond to player input for pausing the game, we need to **subscribe to the Pause action event**. Unity’s **new Input System** uses events, so we don’t check input manually every frame. Instead, we react whenever the event fires.
   
```csharp
private void OnEnable()
{
    // Enable the Player action map so input events can be triggered
    _inputActions.FindActionMap("Player").Enable();

    // Subscribe to the Pause action event
    _inputActions.FindAction("Pause").performed += OnPausePressed;

}//end OnEnable()

private void OnDisable()
{
    // Unsubscribe from the event
    _inputActions.FindAction("Pause").performed -= OnPausePressed;

    // Disable the Player action map to stop listening
    _inputActions.FindActionMap("Player").Disable();

}//end OnDisable()
```
### How it Works
1. Enable the Player action map
  - Input Action Assets are divided into **action maps** (groups of related actions).
  - Enabling the `Player` action map allows all actions defined under it—like Pause—to start listening for input.

2. Subscribe to the Pause action event
  - `FindAction("Pause").performed` is the **event that fires when the Pause action occurs** (e.g., pressing Escape or P).
  - `OnPausePressed` is the **callback method** that Unity calls whenever the event is triggered.

3. Unsubscribe in OnDisable
  - It’s important to remove the subscription when the GameManager is disabled.
  - This prevents memory leaks and avoids accidentally calling methods on destroyed objects.

4. Disable the Player action map
  - Stops listening to input events for this action map.
  - Helps keep input management predictable, especially if multiple action maps exist.

By enabling the action map and subscribing to the event, the GameManager can respond to the Pause input **without tightly coupling to the Player object**, keeping the code modular and using the Observer pattern in practice.

#

#### 2. Create a `OnPausePressed()` method
In Unity’s **new Input System**, each **input action triggers an event**. These events **require a method with a specific signature** to know what to call when the action occurs. That’s why we define `OnPausePressed()`.
 - The event expects a method that takes a single `InputAction.CallbackContext` parameter.
 - Even if you don’t need input data, the **method must match the signature**; otherwise, the compiler will complain.

Inside `OnPausePressed()`, we call `TogglePause()`. This keeps input handling separate from gameplay logic, letting the GameManager respond to the pause action **without the Player needing to know about it**.

```csharp
/// <summary>
/// Called when the Pause input action is performed.
/// Connects the input event to the GameManager's pause logic.
/// </summary>
/// <param name="context">Input callback info (not used here).</param>
private void OnPausePressed(InputAction.CallbackContext context)
{
    // Toggle the game's pause state
    TogglePause();

}//end OnPausePressed
```



