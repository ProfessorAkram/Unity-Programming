# Game Manager
The **GameManager (GM)** is a key component of a game's architecture. It's responsible for overseeing the **flow of the game**, managing **game states**, controlling **major systems**, and maintaining **global functionality**. Often seen as the **central controller** of a game, the GM ensures that gameplay is cohesive and functions smoothly.  It plays such a **central role** in orchestrating game logic that it is considered a **core game system**. Nearly all games, from simple prototypes to large-scale productions, rely on some form of **GameManager** to **coordinate gameplay mechanics and state transitions**, making it essential to the game's overall structure. 

#

>[!NOTE]
>In many cases Game Manager (GM) will implement a **Singleton Pattern** to maintain a single point of control over the game's flow and systems. This ensures that no matter where you are in the game, whether in a scene, menu, or during gameplay, the GM remains the same instance, providing a consistent and unified experience.  
>
> **Why Use a Singleton for the Game Manager?**  
> 1. **Global Access**: The GM needs to be easily accessible from UI, player input, AI, and other systems without manually passing references.  
> 2. **Consistency**: Prevents multiple instances with conflicting states, ensuring game-wide synchronization.  
> 3. **Efficiency**: Reduces memory usage and improves performance by eliminating redundant instances.

# 

## Game Manager Functionality
Game Managers (GMs) handle a variety of responsibilities, including **game state management, system coordination, and global functionality**. While a GM may also manage elements like score tracking, scene transitions, or event handling, in this section, we will focus specifically on how it manages **game states** and ensures smooth transitions between them. To demonstrate how this works in practice, we will be building the GameManager for our example game *One Wild Night*.  

### Game States
GameStates define **what the game is doing right now**; for example, `MainMenu`, `GamePlay`, or `GameOver`. They represent mutually exclusive, player-visible modes. Systems like scoring, level progression, or UI updates are **not GameStates**; instead, they are **actions or systems that respond to the current state**.

For instance, the scoring system only updates during `GamePlay` and ignores input on other states. Similarly, level progression triggers only while the player is actively playing. By centralizing the current mode in a GameState, all other systems can **check the state and behave appropriately**, making the game logic easier to manage, more predictable, and scalable as new states are added.

#### State Pattern vs Finite State Machine (FSM)
Beyond **GameStates**, states are used in many other contexts in games. For example, a character could have states like `Idle`, `Walk`, `Run`, and `Jump`, while an AI NPC might have states such as `Patrol`, `Attack`, or `Sleep`. Whether managing individual object states or overarching game states, there are two common approaches: the Finite State Machine (FSM) and the State Pattern. 

- **Finite State Machine (FSM):** are predefined states, typically represented as an enum. Transitions between states are controlled in a centralized system, such as the GameManager or the object the states relate to. Each state triggers the relevant logic when entered, but the behavior is handled in one place.

**✅ Pros:**
- Simple to implement and easy to read
- Efficient for games with a small number of states
- Centralized control makes debugging straightforward

**❌ Cons:**
- Can become cluttered if there are many states or complex behavior
- Less flexible for objects with unique, independent state logic

- **State Pattern:** invovles encapsulating each state as its own class with its own behavior and logic. Transitions between states are handled through methods within these state objects.

**✅ Pros:**
- Very flexible and modular — each state can define its own behavior independently
- Easier to scale for complex objects or systems with many unique states
- Reduces the risk of a centralized controller becoming too large

**❌ Cons:**
- More complicated to set up initially
- Can be overkill for simple games with only a few states
- More classes and files to manage, which may confuse beginners

For our GameManager, we will be **implmenting the FSM approach**. This approach is beginer friendly, simple to implment and ideal for smalle scale prototype game projects. Using an FSM allows us to clearly manage state transitions, load and unload scenes, and control gameplay behavior without the extra complexity of creating separate classes for each state, as the State Pattern would require.

### Defining Game States

Since we are implment a **FSM** for our **game states**, we will wnat to define each state as an **enum**. Enums are ideal for this purpose, as they allow us to clearly define a set of named values representing the possible game states. Using an Enum makes the code more readable and reduces the risk of mistakes that could arise from using raw integers or strings.

Before we start development on the GM we need to have a rough idea of the game states we will have in the game and what takes place during that state. While these may vary, the most **common core game states** include: 
- **MainMenu**: The game starts here. The GM will load any necessary UI components for the main menu and wait for user input to either start the game, load a saved game, or exit.
- **GamePlay**: The core of the game, in which the player is actively playing. The GM continuously checks if the game conditions are met for a win or loss.
- **GameOver**: When the player loses or when the game is over, the GM will display the Game Over screen and stopping all gameplay logic.

We can declare our **Enum** within the **GameManager** class or in its own separate class file. To keep things modular and flexible, we will declare the **GameState Enum** in its own class, ensuring that each state is decoupled from the core logic of the **GameManager**.

# 

>[!TIP]
> While we refer to **game states** in the plural, it's important to note that we're essentially defining a new **data type**, called **`GameState`** (singular), which will have multiple potential values,each representing a different phase of the game.
---

## :hammer_and_wrench: Creating Game States

1.	Open the **example project** in Unity
2.	In the **`project window`** under **`Assets > Scripts > Managers`** right-click and create a new script.
3.	Name this new script **`GameState`** and open the file in your preferred IDE. 
4. Clear any default class information and instead define the **`enum`** as shown below.

```csharp
   public enum GameState
   {
       MainMenu,   // Main menu screen
       GamePlay,   // Active gameplay
       GameOver    // Game over screen
   }
   ``   
5. Save the class and return to Unity.

---

## :hammer_and_wrench: Basic Game Manager 

1.	In the **`Project`** window under **`Assets > Scripts > Managers`** right-click and create a new script.
2.	Name this new script **`GameManager`** and open the file in your preferred IDE.
3.	Convert the **`GameManager`** game object into a **prefab**. 

### Singleton Inheritance
As mentioned earlier, a **GameManager** is typically designed using a **Singleton Design Pattern**. To implement this we will be making use of our **`Singleton`** base class we created previously. 

3. Type the following code below.

```csharp showLineNumbers title="GameManager.cs"

using UnityEngine;

// The GameManager class is derived from the Singleton pattern to ensure there is only one instance of it in the game.
public class GameManager: Singleton<GameManager>
{
    // Reference to the current state of the game
    public GameState CurrentState;

    // Start is called before the first frame update
    void Start()
    {
        // Change the game state to the main menu
        ChangeGameState(GameState.MainMenu);
    
    }//end Start()


    /// <summary>
    /// Executes logic for the current game state.
    /// </summary>
    private void ManageGameState(){

        // Checks the current game state and performs the appropriate actions.
        switch (CurrentState)
        {
            case GameState.MainMenu:
                // MainMenu Logic
                Debug.Log("Game State: MainMenu");
                break;

            case GameState.GamePlay:
                // Playing Logic
                Debug.Log("Game State: GamePlay");
                break;

            case GameState.GameOver:
                // GameOver logic
                Debug.Log("Game State: GameOver");
                break;
        }//end  switch(CurrentState)

    }//end ManageGameState()

    /// <summary>
    /// Changes the current game state and triggers corresponding logic.
    /// </summary>
    /// <param name="newState">The new game state to switch to.</param>
    public void ChangeGameState(GameState newState)
    {
        CurrentState = newState;

    }//end ChangeState()

}//end GameManager

```
#### **How It Works:**
- **Singleton Pattern**: The `GameManager` class inherits from `Singleton<GameManager>`, ensuring only one instance of the `GameManager` exists in the game, commonly used for game-wide management like state control.

- **CurrentState (GameState)**: Holds the current state of the game (e.g., `MainMenu`, `GamePlay`, `GameOver`), determining which game logic to execute.

- **Start Method**: Initializes the game by calling the `ChangeGameState` method and passing the `GameState.MainMenu` before the first frame update.

- **ManageGameState Method**: Contains a `switch` statement that checks the `CurrentState` and executes the appropriate logic for each game state (e.g., `MainMenu`, `Playing`, `Paused`, `GameOver`).

- **ChangeState Method**: Changes the `CurrentState` to a new game state when called, allowing the game to transition between different states (e.g., from `MainMenu` to `Playing`).

### GamManager Game Object
Now that we have created our GM class, we will return to Unity to actually apply the class to a GM object in our scene. 
1. Open the included **`SampleScene`** in the project. 
2. In the hierarchy under the **Managers** folder right-click and choose **`Create > Create Empty`** 
3. Name the new empty game object **GameManager**
4. Add the GameManager component (i.e. class) from the **`Project`** window to the **`GameManager`** game object. 
In the inspector you will see that the **`Is Persistent`** property. This property was inherited from the base **Singleton** class. Make sure that the **`Is Persistent`** property is set to true, doing so will ensure the GM will be persistent throughout the game. 
5. Press **Play** from the Unity Editor.
   - In the **`Hierarchy`** window you should notice that **`GameManager`** object is now listed under **`DoNotDesotry`**.
   - The **`Console`** window should also display the **debug message "Game State: MainMenu"**, which is the message for the **GameState.MainMenu** which we setup in the `Start()` mehtod.
7. Exit **Play** mode and save the scene.
   
---

## :hammer_and_wrench: Implement the Player Call to GameManager
Before adding additional logic to the **GameManager**, it’s important to verify that state changes are being properly managed. One way to do this is by having the **Player** trigger an event that tells the **GameManager** to change states.

While technically any object could communicate with the **GameManager**, it’s important to limit which objects do.
By limiting communcations with the **GameManager** we: 
- Reduce potential **conflicts and hidden dependencies**, because not every object is trying to control global state.
- Maintain a **clean and logical line of communication**, making it **easier to track how and why state changes occur**.

In many instances the **Player** class is a natural point of interaction in the game. Most events that affect game state — reaching a goal, taking damage, or completing a level — originate from the player's actions.
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

public class Player : MonoBehaviour
{
    private GameManager _gameManager;

    void Start()
    {
        // Reference the GameManager Instance
        _gameManager = GameManager.Instance;

     }//end Start()


    private void OnTriggerEnter(Collider other)
    {
        if (other.CompareTag("Goal"))
        {
            Debug.Log("Goal reached! Changing game state...");
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

Rather than completely replacing the scene each time the player moves between menus or levels, we will **load all scenes additively**, stacking them on top of a single, empty **Bootstrap (or Idle) scene**.

### Single Point of Entry
The **Bootstrap scene** serves as a **single point of entry** for the game, initializing the GameManager and other core systems before any gameplay begins. It is always loaded in the background and ensures everything is ready for the player. **We don’t need a “Bootstrap” GameState**, because GameStates represent what the player experiences, menus, gameplay, or game over; while the Bootstrap scene is purely infrastructure. Once initialization is complete, the GameManager immediately transitions to the first real GameState, so players never notice the Bootstrap scene.

#

>[!NOTE]
> Using a Bootstrap scene as a single point of entry ensures that scripts like the Player can safely access the GameManager. Other options, such as **lazy initialization** or **setting script execution order**, achieve the same goal: guaranteeing that all core systems exist before anything tries to use them.

#

>[!NOTE]
> **Lazy initalization** ensures safe access to objects by creating or assigning them only when they’re needed; not before. Best for secondary systems and not the core game loop. 

#

#### 1. Create a Bootstrap Scene
 1. In the Unity Editor Create a new scene named `Bootstrap`
 2. Place the **GameManager** prefab in the scene and save the scene.
 3. Choose **File > Build Profile** and add the `Bootstrap` scene to the list
    - Ensure that the `Bootstrap` scene is the first scene in the list
    - Add any additional scenes to the build list the order is not important
    - Only add scenes that are required for playing the game. Exclude test scenes.

#

>[!NOTE]
> The **Bootstrap scene** exists to **initialize the GameManager** and other core systems before any gameplay begins. It is always loaded in the background and ensures everything is ready for the player. **We don’t need a “Bootstrap” GameState** because GameStates represent what the player experiences—menus, gameplay, pause, or game over; while the **Bootstrap scene is just infrastructure**. Once initialization is complete, the GameManager immediately transitions to the first real GameState, so players never actually notice the Bootstrap scene.

#

This approach gives the GameManager complete control over the game’s flow and ensures consistent behavior across every scene.

Loading scenes additionally allows for:

- **Single Point of Entry:** All gameplay begins from one persistent scene that contains the GameManager and other core systems.
This makes the game easier to manage and debug, because every other scene—menus, levels, overlays—loads into a known, controlled environment.

- **Guaranteed Initialization Order:** The GameManager loads before any other gameplay objects.
This ensures that global systems like scoring, state tracking, or event management are ready and running before other scripts start referencing them.
Without this structure, different scenes might initialize their objects first, leading to null reference errors or inconsistent game states.


### Scene References
Our first step is to create proper references to each of our scenes and a list for all gameplay levels (scenes). 

####  1. Add the following fields to the GameManager

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
We have now created the primary fields for accessing and keeping track of all scenes in the game. 

#### 2. Load and Unload Scenes
While we could simply call the `SceneManagment.Load()` method in the `ManageGameState()` cases, if we add additional logic it will quickly get bloated. Especially if each case does the same thing, with different values. In this instance we not only want to load the scene, but also keep track of it in the `_loadedScenes` list.  Similarly we will need to create a method for unloading a scene when a new scene is loaded, as well as removing it from the `_loadedScenes` list.

1. Create the `LoadedScene()` method that
   - Loads the scene passed to it
   - Adds that scene to the `_loadedScenes` list
  
```csharp
/// <summary>
/// Loads a scene additively and tracks it in the _loadedScenes list.
/// Helps manage active scenes cleanly across different game states.
/// </summary>
/// <param name="sceneName">The name of the scene to load.</param>

private void LoadScene(string sceneName)
{
    SceneManager.LoadScene(sceneName, LoadSceneMode.Additive);
    
    //Tracks loaded scenes
    _loadedScenes.Add(sceneName); 
    
}//end LoadedScene()
```
# 

2. Create the `UnloadedScene()` method that
   - Set's a referene to teh actual scene
     - Checks if the scene is loaded
     - Unloads the scene
   - Check if the scene is in the loaded scenes list
     - Removes the scene from the `_loadedScenes` list
  
```csharp
/// <summary>
/// Unloads a single scene and removes it from the _loadedScenes list if present.
/// </summary>
/// <param name="sceneName">The name of the scene to unload.</param>
private void UnloadScene(string sceneName)
{
    //Refrence to "this" scene being passed
    Scene _thisScene = SceneManager.GetSceneByName(sceneName);

    // Checks if "this" scene is loaded and uloads
    if (scene.isLoaded)
    {
        SceneManager.UnloadSceneAsync(sceneName);

    }//end if (scene.isLoaded)
    

    // Safely remove scene from list if it exists
    if (_loadedScenes.Contains(sceneName))
    {
        _loadedScenes.Remove(sceneName);
        
    }if (_loadedScenes.Contains(sceneName))
    
}//end UnloadScene()
```






#### 2. Update ManageGameState()


```csharp
    /// <summary>
    /// Executes logic for the current game state.
    /// </summary>
    private void ManageGameState(){

// Early exit if the state is already active
    if (newState == CurrentState)
        return;

    // Unload all previously loaded scenes
    UnloadAllScenes();

    // Update the current state
    CurrentState = newState;

    // Load the appropriate scenes for the new state
    switch (newState)
    {
        case GameState.MainMenu:
            LoadScene(mainMenuScene);
            break;

        case GameState.GamePlay:
            LoadScene(levelScenes[currentLevelIndex]);
            LoadScene(hudScene); // HUD overlay
            break;

        case GameState.GameOver:
            LoadScene(gameOverScene);
            break;
    }//end  switch(CurrentState)

    }//end ManageGameState()
```
#### How It Works
- **using UnityEngine.SceneManagment** provides access to the **SceneMangment** namespaces for loading scenes in Unity.
-  Each game state corresponds to a specific scene or menu.
- **Additive** loading is used for overlays like the pause menu, so gameplay continues underneath.

#### 3. Unloading all Scenes
We now have implmented scene loading, but we will eventually want to unload these scene. To do this we will want to call a method to unload all additive scenes that have loaded on top of the Bootstrap scene. 

```csharp
/// <summary>
/// Unloads all currently loaded scenes except for the Bootstrap scene.
/// </summary>
public void UnloadAllScenes()
{
    // Loop through all currently loaded scenes
    for (int i = 0; i < SceneManager.sceneCount; i++)
    {
        Scene scene = SceneManager.GetSceneAt(i);

        // Only unload if it's not the Bootstrap scene
        if (scene.name != "Bootstrap" && scene.isLoaded)
        {
            SceneManager.UnloadSceneAsync(scene);
            
        }//end If
        
    }//end for sceneCount
    
}//end UnloadedAllAdditiveScenes()

```

#

>[!TIP]
>For states like **GamePlay**, you might eventually want to create a `NextLevel()` method to determine which scene to load. In a simple one-level game, directly loading **"Level01"** works fine. However, as your game grows, separating the level-loading logic into its own method makes the code cleaner and easier to maintain.
>
>If there are many scenes to manage, delegating this responsibility to a dedicated **SceneFlowManager** can help keep the project organized and prevent the **GameManager** from becoming overloaded.

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







