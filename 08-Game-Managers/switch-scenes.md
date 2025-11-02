# Scene Management
In larger-scale games, scene loading and transitions are often managed by a dedicated **scene manager** to keep responsibilities clearly separated. However, for smaller projects or prototypes, it’s perfectly reasonable to consolidate this functionality within the **GameManager**, allowing it to control both the active game state and the flow between scenes. This not only simplifies our architecture for smaller projects but also provides a clear example of how state management and scene flow work together.

---

## Single Point of Entry
To manage multiple scenes efficiently and ensure that core systems remain active throughout the game, we use a **single persistent entry point**, a scene from which the game always starts and in which core systems persist for the lifetime of the application. All other scenes, such as levels, menus, or overlays, are loaded on top of this starting scene, ensuring the game always begins from a known, consistent location.

This foundational scene is typically called the **Bootstrap** scene. While players never see it directly, it provides a stable place to initialize critical systems such as the **GameManager**, audio managers, and other global components before any gameplay or UI appears.

Rather than completely unloading and replacing the scene each time the player moves between menus or levels, we'll take a more efficient approach, **loading scenes additively** on top of the Bootstrap scene. This setup allows us to stack and manage different layers of the game without disrupting the core systems.

Using a single point of entry is considered **good practice** because it:
- Guarantees that essential systems are initialized in the correct order.
- Reduces the risk of duplicate or missing managers across scenes.
- Makes it easier to maintain persistence and state consistency throughout the game.
- Simplifies debugging and future expansion, since all core systems start from a known, central location.

Even as the game grows more complex, maintaining a single entry point helps keep initialization predictable and reliable, which is especially important when working with multiple additive scenes, overlays, or persistent managers.

# 

>[!TIP]
> A single entry point is not the only way to achieve safe initialization. Other options include **lazy initialization** or adjusting the **script execution order** in Unity, which can also help to ensure that systems exist before being accessed.

#


## :hammer_and_wrench: Setting up Scenes

Before we can start switching between levels, menus, and overlays, we need to set up our project with a **Bootstrap scene**. In addition, we'll also organize our scene lists in the Build Settings. This ensures that all scenes we plan to load additively—such as menus, gameplay levels, and overlays—are properly recognized by Unity and can be loaded dynamically during runtime.

#### 1. Create a Bootstrap Scene

1. In Unity Editor, from the **project panel**
   - Select the **Scenes** folder and right-click 
   - Create a **New Scene** and name it **Bootstrap**
2. Delete everything from this scene
3. Add the **GameManager** prefab to the scene
4. **Save** the scene

#

#### 2. Update the Scenes List
 1. In the Unity Editor, choose **File > Build Profile** 
 2. Add the `Bootstrap` scene to the list
    - Ensure that the `Bootstrap` scene is the first scene in the list
 3. Add any additional scenes to the build list; the order is not important
    - Only add scenes that are required for playing the game (e.g., MainMenu, GameOver, Level-01, Level-02) 
    - Exclude test scenes.
   
---

## :hammer_and_wrench: Managing Scene Transitions with the GameManager
For the **GameManager** to control scene transitions, we need to define which scenes exist and how they are loaded or unloaded. This allows menus, gameplay levels, and overlays to appear seamlessly during gameplay.


#### 1. Using Scene Managment
To handle the transition scenes, the **GameManager** needs to interact with **Unity’s Scene Management system**.
We will also record these scenes into a list, which will require the C# **System.Collections.Generic** namespace.
**
1. Edit the **GameManager** class to include the following: 

```csharp
using System.Collections.Generic; 
using UnityEngine;
using UnityEngine.SceneManagement;

// The GameManager class is derived from the Singleton pattern to ensure there is only one instance of it in the game.
public class GameManager: Singleton<GameManager>
{ ...
```

#

####  2. Create Scene References
Now that our scenes have been created and set up in the editor, we will need our game manager to have a reference to each of them. 

1. Add the following fields to the GameManager

```csharp
   [Header("Scene Management")]
   [SerializeField]
   [Tooltip("The main menu scene that loads when the game starts.")]
   private string _mainMenuScene;
   
   [SerializeField]
   [Tooltip("The HUD overlay that appears during gameplay.")]
   private string _hudScene;
   
   [SerializeField]
   [Tooltip("The pause menu overlay that appears when the game is paused.")]
   private string _pauseMenuScene;
   
   [SerializeField]
   [Tooltip("The Game Over scene that loads when the player loses or finishes the game.")]
   private string _gameOverScene;
   
   [SerializeField]
   [Tooltip("All the level scenes in the game, in the order they should be played.")]
   private List<string> _gameLevels = new List<string>();
   
   // Index of the currently active level in the levelScenes list
   private int _currentLevelIndex = 0;
   
   // Tracks the currently loaded primary scene (menu or level)
   private string _currentScene;
   
   //List of all loaded scenes
   private List<string> _loadedScenes = new List<string>();

```
Next, we need to create dedicated methods for loading and unloading scenes.

While we could call `SceneManagement.Load()` directly in each `ManageGameState()` case, this approach quickly becomes repetitive and hard to maintain—especially when multiple cases perform similar operations with different scene names. By creating a centralized `LoadScene()` method, we can not only load a scene but also track it in the `_loadedScenes` list for easier management.

Similarly, we will create an `UnloadScene()` method to safely remove individual scenes when needed, as well as an `UnloadAllScenes()` method to quickly clear all currently loaded scenes. This approach keeps scene management organized, consistent, and easy to maintain as the game grows.

#

#### 3. Create the `LoadedScene()` method
This method handles **loading a new scene additively** and keeping track of it in the `_loadedScenes` list. It also allows you to designate whether the scene should be considered the **current primary scene**.

The method will:
- Load the scene passed to it
- Add the scene to the `_loadedScenes` list for tracking
- Optionally set the scene as the `_currentScene` if it represents a primary scene (like a level or menu)
  
```csharp
/// <summary>
/// Loads a scene additively and tracks it in the _loadedScenes list.
/// </summary>
/// <param name="sceneName">The name of the scene to load.</param>
/// <param name="setAsCurrent">
/// If true, sets this scene as the current primary scene (e.g., main level or menu).
/// If false, the scene is treated as an overlay (e.g., HUD, pause menu) and does not become the current scene.
/// </param>
private void LoadScene(string sceneName, bool setAsCurrent = true)
{
    // Load the scene additively so existing scenes are preserved
    SceneManager.LoadScene(sceneName, LoadSceneMode.Additive);
    
    // Track the loaded scene
    _loadedScenes.Add(sceneName);

    // Optionally set as current scene
    if (setAsCurrent)
    {
        _currentScene = sceneName;
    }

} // end LoadScene()

```
#
> [!NOTE]
> The `setAsCurrent` boolean is useful for distinguishing primary scenes (levels, menus) from secondary overlay scenes (HUD, PauseMenu). Overlay scenes remain loaded but do not replace the `_currentScene`.

#

#### 4. Create the `UnloadedScene()` method
This method handles **unloading a single scene** and ensures it is removed from the `_loadedScenes` list. It safely checks if the scene is loaded before attempting to unload it and avoids errors if the scene is not tracked.

The method will:
- Get a reference to the scene being passed
  - Check if the scene is currently loaded
  - Unload the scene if it is loaded
- Check if the scene exists in _loadedScenes
  - Remove it from the list if present
  
```csharp
/// <summary>
/// Unloads a single scene and removes it from the _loadedScenes list if present.
/// </summary>
/// <param name="sceneName">The name of the scene to unload.</param>
private void UnloadScene(string sceneName)
{
    //Reference to "this" scene being passed
    Scene thisScene = SceneManager.GetSceneByName(sceneName);

    // Checks if "this" scene is loaded and unloads
    if (thisScene.isLoaded)
    {
        SceneManager.UnloadSceneAsync(sceneName);

    }//end if (scene.isLoaded)
    

    // Safely remove scene from list if it exists
    if (_loadedScenes.Contains(sceneName))
    {
        _loadedScenes.Remove(sceneName);
        
    }//end if(_loadedScenes.Contains(sceneName))
    
}//end UnloadScene()
```
#
> [!NOTE]
> The `UnloadScene()` method is ideal for removing temporary or overlay scenes, like the PauseMenu, without affecting other active scenes.

#

#### 5. Create the `UnloadAllScenes()` method
This method allows you to **unload all currently loaded scenes** at once. It ensures the `_loadedScenes` list is cleared afterward. `UnloadAllScenes()` will be used when transitioning between major game states (like MainMenu → GamePlay) to reset the game environment cleanly.

The method will:
- Iterate through all scenes in `_loadedScenes`
  - Call `UnloadScene()` on each scene
- Clear the `_loadedScenes` list to remove any remaining references
  
```csharp

/// <summary>
/// Unloads all currently loaded scenes except persistent ones
/// and clears the _loadedScenes list.
/// </summary>
private void UnloadAllScenes()
{
    foreach (string sceneName in new List<string>(_loadedScenes))
    {
        // Unload each scene safely
        UnloadScene(sceneName);
    }

    // Clear the list to remove any remaining references
    _loadedScenes.Clear();
    
}//end UnloadAllScenes()
```

# 

#### 6. Update ManageGameState()
Now that we have dedicated methods for loading and unloading scenes, we can simplify the GameManager’s state handling.

The `ManageGameState()` method will be updated with the following features
- Unloading all previously loaded scenes to reset the game environment
- Updating the CurrentState to the new state
- Loading the appropriate scenes for the new state, including overlays like the HUD
- Handling any unexpected or invalid states with a default error check

By centralizing scene management here, we keep the game flow clean, predictable, and easy to extend as more states or levels are added.

```csharp
  /// <summary>
    /// Executes the logic associated with the current game state.
    /// This method is called whenever ChangeGameState() updates the state.
    /// </summary>
    private void ManageGameState()
    {
        // Unload all previously loaded scenes
        UnloadAllScenes();
        
        switch (CurrentState)
        {
            case GameState.MainMenu:
                Debug.Log("Game State: MainMenu");
                LoadScene(_mainMenuScene);
                break;

            case GameState.GamePlay:
                Debug.Log("Game State: GamePlay");
                
                //Load game level
                LoadScene(_gameLevels[_currentLevelIndex]);
   
                // Load the HUD as an overlay without setting it as the current scene
                LoadScene(_hudScene, false); 
                break;

            case GameState.GameOver:
                Debug.Log("Game State: GameOver");
                LoadScene(_gameOverScene);
                break;

            default:
                Debug.LogError($"[GameManager] Unknown GameState: {CurrentState}. No scenes loaded.");
                break;

        }//end switch(CurrentState)

    }//end ManageGameState()
```

#

### Managing Level Progression

While `ManageGameState()` handles transitions between major game states like `MainMenu`, `GamePlay`, and `GameOver`, many games include multiple levels or stages within the same gameplay session. Level progression differs from changing the game state: the player remains in `GamePlay`, but the environment, challenges, and level-specific content need to update.

To handle this, we introduce the `LoadNextLevel()` method. This method will:
- Check if a current level scene is loaded, and unload it **as a safety measure** if necessary.
- Increment the level index to determine the next level
- Check if the level index exceeds the available levels
   - if so, resets the level counter
   - Switch to GameOver
   - Exits the method
- Loads the next level scene and sets it as the current scene

By using `LoadNextLevel()`, we can smoothly progress through levels without affecting the overall game state or disrupting UI and gameplay flow.

#### 1. Create the `LoadNextLevel()` method

```csharp
/// <summary>
/// Loads the next level in the levelScenes list while keeping the player in the GamePlay state.
/// Unloads the current level, updates the current level index, and tracks the newly loaded scene.
/// </summary>
public void LoadNextLevel()
{
    // Unload the current level scene if one is loaded
    if (_currentScene != null && _loadedScenes.Contains(_currentScene))
    {
        UnloadScene(_currentScene);

    }//end if(_currentScene)

    // Increment level index
    currentLevelIndex++;

    // If no more levels, reset index, switch to GameOver, and exit
    if (currentLevelIndex >= levelScenes.Count)
    {
        currentLevelIndex = 0;
        ChangeGameState(GameState.GameOver);
        return;

    }//end

    // Load the next level and set it as the current scene
    LoadScene(levelScenes[currentLevelIndex]);

    Debug.Log($"Loaded Level: {levelScenes[currentLevelIndex]}");

}//end LoadNextLevel()

```

---

## :hammer_and_wrench: Manging Pause State 

Another common function in many games is implementing a **pause menu**. Pausing usually involves freezing gameplay and displaying a menu where players can resume, restart, or exit.

When the `PauseMenu` is triggered, the menu will load as an **additive** scene, and we will stop all gameplay, adjusting the `Time.timeScale` property. When exiting pause, we will do the reverse, and because the same button or trigger will turn on and off the pause, we can do all this in a single `TogglePause ()` method that can check against a single bool. 


#### 1. Create a Boolean to Track Pause

To determine if the game is currently paused, we define a **read-only boolean property**:

```csharp
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

.

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
        LoadScene(_pauseMenu);
    }
    else
    {
        // Resume the game
        Time.timeScale = 1f;
        unloadScene(_pauseMenu);

    }//end if (!IsPaused)

}//end TogglePause()

```

#### How it Works
- Checks the **current game state** to determine whether to pause or resume gameplay.
   - If the game is running (`GamePlay`), it switches to Pause and freezes the game by setting Time.timeScale = 0.
   - If the game is already paused (`Pause`), it switches back to GamePlay and resumes time (Time.timeScale = 1).
- Uses **additive scene loading** for the pause menu, allowing the menu to overlay the gameplay without unloading the level.
- **Unloads the pause men**u when returning to gameplay, ensuring the overlay is removed.
- Keeps pause logic **isolated** so other game state changes don’t unintentionally affect the time scale or UI.

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

#### 2. Handle Game Level Loads
When returning from a **Pause** state back to **GamePlay**, we want to **avoid reloading the current level**. Reloading unnecessarily could reset the player’s progress or interrupt gameplay.

One way to handle this is by tracking whether the gameplay has already started.
- If it has, we simply resume gameplay without reloading the level.
- If not, we load the first level only once.

#

1. Create a flag to check if gameplay has started
```csharp
      // Flag to track whether gameplay has started
      private void _hasGameplayStarted() = false; 
```

2. Create a StartGameplay() method

```csharp

/// <summary>
/// Starts gameplay by loading the first level and setting the gameplay flag.
/// This method should only be called the first time the game enters the GamePlay state.
/// </summary>
private void StartGameplay()
{
    //Load the first game level
    SceneManager.LoadScene("Level01");

    //Set the started flag to true
    _hasGameplayStarted = true;

}//end StartGameplay()

```

3. Update the GamePlay case in the state switch

```csharp
case GameState.GamePlay:
    // Gameplay logic
    Debug.Log("Game State: GamePlay");

    // Load the first level only if gameplay hasn't started
    if (!_hasGameplayStarted)
    {
        StartGameplay();
    }

    break;
```

#### How it Works
- The `_hasGameplayStarted` flag prevents unnecessary reloading when switching between Pause and GamePlay.
- `StartGameplay()` loads the first level and marks the flag as true.
- Once gameplay is active, returning from Pause simply resumes the game instead of restarting the scene.

#

>[!CAUTION]
> This simple setup only handles loading the first gameplay level.
>
> Additional logic would be required to:
> - **Load the next level** when a goal is reached.
> - **Reset the current level** after a player loss or restart.
> - **Detect when there are no more levels** and switch to the GameOver state.
>   
> These features can be added later through a `NextLevel()` or **SceneFlowManager**.





