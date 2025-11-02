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
[Tooltip("Reference to the PlayerController Input Action Asset used to handle pause and other input.")]
private PlayerController _inputActions;

```

2. In the Unity Editor, select the **GameManager** prefab from the **Bootstrap** scene, in the **hierarchy panel** 
3. In the **Inspector panel**, Drag your **PlayerController Input Action Asset** into the **Input Actions** field.
   - The GameManager can now enable the action map and subscribe to the Pause action.
4. **Applie Overides** to the **GameManager** prefab

#

Using the Observer Pattern to Listen for Pause

Rather than checking input every frame, we use the Observer pattern to react to input events:

The Input Action is the subject: it emits an event whenever the player performs the action.

The GameManager is the observer: it subscribes to the event and reacts automatically.

Subscription:
```csharp
private void OnEnable()
{
    InputActions.Player.Enable();
    InputActions.Player.Pause.performed += OnPausePressed;
}

private void OnDisable()
{
    InputActions.Player.Pause.performed -= OnPausePressed;
    InputActions.Player.Disable();
}

private void OnPausePressed(InputAction.CallbackContext context)
{
    TogglePause();
}
```

Whenever the player presses the pause button, the Input System triggers the event, and the GameManager’s OnPausePressed() is called.

TogglePause() then pauses or resumes the game, loads/unloads the pause menu, and updates the game state.

✅ Key takeaway: This decouples input handling from gameplay logic, works across all additive scenes, and is a clean, event-driven way to manage game actions.





