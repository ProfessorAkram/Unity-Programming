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

