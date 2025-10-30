# Game Manager
The **GameManager** is a key component of a game's architecture. It's responsible for overseeing the **flow of the game**, managing **game states**, controlling **major systems**, and maintaining **global functionality**. Often seen as the **central controller** of a game, the Game Manager ensures that gameplay is cohesive and functions smoothly.  It plays such a **central role** in orchestrating game logic that it is considered a **core game system**. Nearly all games, from simple prototypes to large-scale productions, rely on some form of **GameManager** to **coordinate gameplay mechanics and state transitions**, making it essential to the game's overall structure. 

#

>[!NOTE]
> In many cases, Game Manager will implement a **Singleton Pattern** to maintain a single point of control over the game's flow and systems. This ensures that no matter where you are in the game, whether in a scene, menu, or during gameplay, the GM remains the same instance, providing a consistent and unified experience.  
>
> **Why Use a Singleton for the Game Manager?**  
> 1. **Global Access**: The GM needs to be easily accessible from UI, player input, AI, and other systems without manually passing references.  
> 2. **Consistency**: Prevents multiple instances with conflicting states, ensuring game-wide synchronization.  
> 3. **Efficiency**: Reduces memory usage and improves performance by eliminating redundant instances.

# 

## Game States
One of the core responsibilities of a Game Manager is **game state management**. GameStates define **what the game is doing right now**; for example, `MainMenu`, `GamePlay`, or `GameOver`. They represent mutually exclusive, player-visible modes. Systems like scoring, level progression, or UI updates are **not GameStates**; instead, they are **actions or systems that respond to the current state**.

For instance, the scoring system only updates during `GamePlay` and ignores input in other states. Similarly, level progression triggers only while the player is actively playing. By centralizing the current mode in a GameState, all other systems can **check the state and behave appropriately**, making the game logic easier to manage, more predictable, and scalable as new states are added.

### State Pattern vs Finite State Machine (FSM)
Beyond **GameStates**, states are used in many other contexts in games. For example, a character could have states like `Idle`, `Walk`, `Run`, and `Jump`, while an AI NPC might have states such as `Patrol`, `Attack`, or `Sleep`. Whether managing individual object states or overarching game states, there are two common approaches: the **Finite State Machine (FSM)** and the **State Pattern**. 

**Finite State Machine (FSM):** is are predefined states, typically represented as an enum. Transitions between states are controlled in a centralized system, such as the GameManager or the object the states relate to. Each state triggers the relevant logic when entered, but the behavior is handled in one place.

**✅ Pros:**
- Simple to implement and easy to read
- Efficient for games with a small number of states
- Centralized control makes debugging straightforward

**❌ Cons:**
- Can become cluttered if there are many states or complex behaviors
- Less flexible for objects with unique, independent state logic

# 

**State Pattern:** involves encapsulating each state as its own class with its own behavior and logic. Transitions between states are handled through methods within these state objects.

**✅ Pros:**
- Very flexible and modular — each state can define its own behavior independently
- Easier to scale for complex objects or systems with many unique states
- Reduces the risk of a centralized controller becoming too large

**❌ Cons:**
- More complicated to set up initially
- Can be overkill for simple games with only a few states
- More classes and files to manage, which may confuse beginners

#

For our GameManager, we will be **implementing the FSM approach**. This approach is beginner-friendly, simple to implement, and ideal for small-scale prototype game projects. Using an FSM allows us to clearly manage state transitions, load and unload scenes, and control gameplay behavior without the extra complexity of creating separate classes for each state, as the State Pattern would require.

#

### Defining Game States

Since we are implementing an **FSM** for our **game states**, we will want to define each state as an **enum**. Enums are ideal for this purpose, as they allow us to clearly define a set of named values representing the possible game states. Using an Enum makes the code more readable and reduces the risk of mistakes that could arise from using raw integers or strings.

Before we start development on the Game Manager, we need to have a rough idea of the game states we will have in the game and what takes place during each state. While these may vary, the most **common core game states** include: 
- **MainMenu**: The game starts here. The GM will load any necessary UI components for the main menu and wait for user input to either start the game, load a saved game, or exit.
- **GamePlay**: The core of the game, in which the player is actively playing. The GM continuously checks if the game conditions are met for a win or a loss.
- **GameOver**: When the player loses or when the game is over, the GM will display the Game Over screen and stop all gameplay logic.

We can declare our **Enum** within the **GameManager** class or in its own separate class file. To keep things modular and flexible, we will declare the **GameState Enum** in its own class, ensuring that each state is decoupled from the core logic of the **GameManager**.

# 

>[!TIP]
> While we refer to **game states** in the plural, it's important to note that we're essentially defining a new **data type**, called **`GameState`** (singular), which will have multiple potential values, each representing a different phase of the game.
---

## :hammer_and_wrench: Creating Game States

1.	Open the **example project** in Unity
2.	In the **`project panel`** under **`Assets > Scripts >`** right-click and create a new folder named **`Managers`**
3.	In the newly created **`Managers`** folder, right-click and **create a new script**.
4.	Name this new script **`GameState`** and open the file in your preferred IDE. 
5. Clear any default class information and instead define the **`enum`** as shown below.

```csharp
   public enum GameState
   {
       MainMenu,   // Main menu screen
       GamePlay,   // Active gameplay
       GameOver    // Game over screen
   }
```   
5. Save the class and return to Unity.

---

## :hammer_and_wrench: Basic Game Manager 
Now that we have our game states established, we can start to build out our Game Manager, which inherits from our **Singleton** base class. This guarantees that there is **only one instance** of the GameManager at any time. Singleton-based managers are ideal for **global systems** like game state management, scoring, or persistent data.

#

#### 1. Create the GameManager Class

1.	In the **`Project`** panel under **`Assets > Scripts > Managers`** right-click and create a new script.
   - Name this new script **`GameManager`** and open the file in your preferred IDE.
2. Edit the script as follows: 

```csharp

using UnityEngine;

// The GameManager class is derived from the Singleton pattern to ensure there is only one instance of it in the game.
public class GameManager: Singleton<GameManager>
{
    // Reference to the current state of the game
    public GameState CurrentState {get; private set;};

```
Here, we implement the **Singleton base class** to ensure there is **only one instance** of the GameManager in the game. At the same time, we define a reference to the **current game state** using a property with a `private set`.

The `private set` ensures that while other scripts can **read** the value of `CurrentState`, **only the GameManager itself can modify it**, maintaining control over the game's state transitions.

#

#### 3. Create a `ChangeGameState()` method
Because the GameManager is **responsible for state transitions**, it needs a method to **change the current game state**. This method acts as the central point for initiating any state change, making it easy to ensure that all associated logic and scene management occur in a controlled and consistent way. Without such a method, state changes could become scattered and harder to maintain.

```csharp
   /// <summary>
   /// Changes the current game state and triggers corresponding logic
   /// only if the new state is different from the current state.
   /// </summary>
   /// <param name="newState">The new game state to switch to.</param>
   public void ChangeGameState(GameState newState)
   {
       // Early exit if already in this state
       if (newState == CurrentState)
           return;
   
       // Update the current state and manage scenes
       CurrentState = newState;

       ManageGameState();

   }//end ChangeGameState

```
To prevent unnecessary calls, we first check whether the new state is different from the current state. Only if it is different do we update `CurrentState` and proceed.

After updating the state, our method calls the `ManageGameState()` method to **manage the behaviors of the state change**.

# 

#### 4. Create a `ManageGameState()` method
This method ensures that every time the game state changes, the GameManager **consistently executes the relevant logic** for that state. As the game grows, you can expand each case to include scene management, UI updates, and other state-specific behaviors.

```csharp
/// <summary>
/// Executes the logic associated with the current game state.
/// This method is called whenever ChangeGameState() updates the state.
/// </summary>
private void ManageGameState()
{
    switch (CurrentState)
    {
        case GameState.MainMenu:
            Debug.Log("Game State: MainMenu");
            break;

        case GameState.GamePlay:
            Debug.Log("Game State: GamePlay");
            break;

        case GameState.GameOver:
            Debug.Log("Game State: GameOver");
            break;

        default:
            Debug.LogError($"[GameManager] Unknown GameState: {CurrentState}. No scenes loaded.");
            break;

    }//end switch(CurrentState)

}//end ManageGameState()

```
#

#### 5. Add a `Start()` method
Since our game uses a **single entry point** with a **bootstrap scene**, we need to ensure that the game is properly initialized before any gameplay or menu logic runs. The `Start()` method executes **before the first frame update**, making it the perfect place to set up the initial game state.

In this case, we **initialize the game** by setting the starting state to `MainMenu` using our `ChangeGameState()` method. This ensures that all scene loading and state-specific logic is executed consistently right at the beginning of the game.

```csharp
// Start is called before the first frame update
void Start()
{
    // Set the initial game state to Main Menu
    ChangeGameState(GameState.MainMenu);

}//end Start()
```

By calling `ChangeGameState()` here, we make sure the GameManager takes control of **all necessary scene and state setup** before the player sees anything on screen.

#

#### 6. Create a GamManager Prefab
Now that we have created our **GameManager** class, we will return to the Unity Editor setup or project.
1. In the **project panel** , under **Scenes** right-click and create a **New Scene**
   - Name this scene **Bootstrap**
   - Delete everything from this scene

#

>[!NOTE]
> The **Bootstrap scene** serves as a **single point of entry** for the game. It ensures that the GameManager and other core systems are initialized before any other scenes or gameplay elements load. Players never directly see this scene, as it immediately transitions to the first real game state.

#

2. In the **hierarchy panel**, right-click and choose **`Create > Create Empty`** 
   - Name the new empty game object **GameManager**
   - Add the **GameManager** class to the **GameManager** game object
   - In the Inspector, ensure the **`Is Persistent`** property is set to true.

#

>[!NOTE]
> Setting the GameManager as persistent isn’t strictly necessary in this setup because all scenes will load on top of the bootstrap scene. Since the bootstrap scene will never unload, the GameManger will always exist. However, **enabling persistence acts as a fail-safe** in our Singleton base class, ensuring that the GameManager exists even if this scene is accidentally unloaded. 

#

7. Press **Play** from the Unity Editor.
   - In the **`Hierarchy`** window you should notice that **`GameManager`** object is now listed under **`DoNotDesotry`**.
   - The **`Console`** window should also display the **debug message "Game State: MainMenu"**, which is the message for the **GameState.MainMenu** which we setup in the `Start()` mehtod.
     
8. Exit **Play** mode and save the scene.
   
---

## :hammer_and_wrench: Implement the Player Call to GameManager
Before adding additional logic to the **GameManager**, it’s important to verify that state changes are being properly managed. One way to do this is by having the **Player** trigger an event that tells the **GameManager** to change states.

While technically any object could communicate with the **GameManager**, it’s important to limit which objects do.
By limiting communications with the **GameManager**, we: 
- Reduce potential **conflicts and hidden dependencies**, because not every object is trying to control global state.
- Maintain a **clean and logical line of communication**, making it **easier to track how and why state changes occur**.

In many instances, the **Player** class is a natural point of interaction in the game. Most events that affect game state — reaching a goal, taking damage, or completing a level — originate from the player's actions.
By restricting communication to objects like the Player, the GameManager remains responsible for managing states without becoming entangled with unnecessary object interactions.

#

#### 1. Add A Goal
 1. In the Unity Editor Hierarchy window
     - Add a trigger object in the scene (like a goal) 
     - Ensure it has a Collider and that it is set to **is trigger**
     - Tag this object as "Goal".
   
#

#### 2. Update/Create the Player Class 
1. Make the Player detect the trigger and tell the GameManager to change the state.

```csharp
using UnityEngine;

public class Player: MonoBehaviour
{
    // Reference to the singleton GameManager instance
    private GameManager _gameManager;

    // Start is called before the first frame update
    void Start()
    {
        // Get the GameManager instance to access global game state methods
        _gameManager = GameManager.Instance;

    }//end Start()


    private void OnTriggerEnter(Collider other)
    {
        // Check if the player collides with an object tagged as "Goal"
        if (other.CompareTag("Goal"))
        {
            // Log for debugging
            Debug.Log("Goal reached! Changing game state...");

            // Trigger a game state change to GameOver
            _gameManager.ChangeGameState(GameState.GameOver);

        }//end if("Goal")

    }//end OnTriggerEnter()

}//end Player

```
#

>[!IMPORTANT]
> In the **Player** class we set the reference to the `_gameManager` in `Start()` to ensure the **GameManager** exists before the Player accesses it. Referencing it in `Awake()` or at declaration may cause null errors. Other safe methods exist, which we’ll cover later.

#

Now, when the player collides with the goal, you will see messages in the console:

```
Goal reached! Changing game state...
Game State: GameOver
```

The debug output on our `OnTriggerEnter()` in the **Player** class and the `ManageGameState()` method in the **GameManager** confirms that the **GameManager** is working and that state changes are being triggered.

---

## Common GameManager Logic
As mentioned previously, the **GameManager** primarily oversees the **flow of the game**, managing **game states** and controlling **major systems**. 

**In a small-scale game**, this typically includes:
- **Starting and stopping gameplay** – initiating or ending a level.
- **Displaying menus or overlays** – such as main menu, pause, or results screens.
- **Pausing and resuming the game** – controlling game time and input.
- **Switching scenes** – loading and unloading levels, menus, or results screens.
- **Handling game over or level completion** – triggering transitions to results or next levels.
- **Basic global rules or systems** – things that don’t belong to a single object but affect the entire game.

Let's take a look at how we would implement some of these behaviors. 

---

## :hammer_and_wrench: Switching Scenes with GameStates 
Next, we will extend the **GameManager** to handle scene transitions and manage overall game flow through defined GameStates.

Rather than completely replacing the scene each time the player moves between menus or levels, we will **load all scenes additively**, stacking them on top of our **Bootstrap scene**.

### Single Point of Entry
The **Bootstrap scene** acts as the game's **single point of entry**, providing a consistent place to initialize the GameManager and other core systems. While players never directly see this scene, it ensures that all critical systems are ready before any gameplay or menus appear.

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

#### 1. Update the Scenes List
 1. In the Unity Editor, choose **File > Build Profile** 
 2. Add the `Bootstrap` scene to the list
    - Ensure that the `Bootstrap` scene is the first scene in the list
 3. Add any additional scenes to the build list; the order is not important
    - Only add scenes that are required for playing the game (e.g., MainMenu, GameOver, Level-01, Level-02) 
    - Exclude test scenes.
   
#

#### 2. Using Scene Managment
To handle the transition scenes, the **GameManager** needs to interact with **Unity’s Scene Management system**.

1. Edit the **GameManager** class to include the following: 

```csharp
using UnityEngine;
using UnityEngine.SceneManagement;

// The GameManager class is derived from the Singleton pattern to ensure there is only one instance of it in the game.
public class GameManager: Singleton<GameManager>
{ ...
```

#

####  3. Create Scene References
Now that our scenes have been created and set up in the editor, we will need our game manager to have a reference to each of them. 

1. Add the following fields to the GameManager

```csharp
   [Header("Scene Management")]
   [SerializeField]
   [Tooltip("The main menu scene that loads when the game starts.")]
   private string _mainMenuScene = "MainMenu";
   
   [SerializeField]
   [Tooltip("The HUD overlay that appears during gameplay.")]
   private string _hudScene = "HUD";
   
   [SerializeField]
   [Tooltip("The pause menu overlay that appears when the game is paused.")]
   private string _pauseMenuScene = "PauseMenu";
   
   [SerializeField]
   [Tooltip("The Game Over scene that loads when the player loses or finishes the game.")]
   private string _gameOverScene = "GameOver";
   
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

#### 4. Create the `LoadedScene()` method
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

#### 5. Create the `UnloadedScene()` method
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
    Scene _thisScene = SceneManager.GetSceneByName(sceneName);

    // Checks if "this" scene is loaded and unloads
    if (scene.isLoaded)
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

#### 6. Create the `UnloadAllScenes()` method
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

#### 7. Update ManageGameState()
Now that we have dedicated methods for loading and unloading scenes, we can simplify the GameManager’s state handling.

The `ManageGameState()` method will be updated with the following features
- Unloading all previously loaded scenes to reset the game environment
- Updating the CurrentState to the new state
- Loading the appropriate scenes for the new state, including overlays like the HUD
- Handling any unexpected or invalid states with a default error check

By centralizing scene management here, we keep the game flow clean, predictable, and easy to extend as more states or levels are added.

```csharp
    /// <summary>
    /// Executes logic for the current game state.
    /// </summary>
    private void ManageGameState(){
       // Unload all previously loaded scenes
       UnloadAllScenes();

       // Load the appropriate scenes for the new state
       switch (newState)
       {
           case GameState.MainMenu:
               LoadScene(mainMenuScene);
               break;
   
           case GameState.GamePlay:
               //Load game level
               LoadScene(levelScenes[currentLevelIndex]);
   
                // Load the HUD as an overlay without setting it as the current scene
               LoadScene(hudScene, false); 
               break;
   
           case GameState.GameOver:
               LoadScene(gameOverScene);
               break;
   
           default:
              Debug.LogError($"[GameManager] Unknown GameState: {newState}. No scenes loaded.");
           break;
   
       }//end  switch(CurrentState)

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

Another common function is implementing a **pause menu** in the game. Pausing typically involves opening a menu and stopping gameplay.

In our current setup, the `"PauseMenu"` scene is loaded **additively**, meaning it appears on top of the current gameplay scene. However, this alone **does not actually pause the game**. Additionally, if we were to rely only on switching states in the `ManageGameState()` method, returning from pause to gameplay could unintentionally reload the current level, which is not what we want.

To handle this correctly, we need to:
- Avoid reloading the current gameplay scene when toggling pause.
- Stop or resume gameplay, usually by adjusting `Time.timeScale`.
- Manage additive loading/unloading of the pause menu as needed.

#
 
#### 1. Create a TogglePause() method

```csharp
/// <summary>
/// Toggles between GamePlay and Pause states.
/// Freezes or resumes gameplay using Time.timeScale
/// and loads/unloads the additive pause menu.
/// </summary>
public void TogglePause()
{
    if (CurrentState == GameState.GamePlay)
    {
        ChangeGameState(GameState.Pause);
        // Freeze gameplay 
        Time.timeScale = 0f; 
    }
    else if (CurrentState == GameState.Pause)
    {
        ChangeGameState(GameState.GamePlay);
        // Resume gameplay 
        Time.timeScale = 1f; 

        // Unload the pause menu
        SceneManager.UnloadSceneAsync("PauseMenu");

   }//end if(GamePlay/Pause)

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
      // Flag to tracks whether gameplay has started
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

---












