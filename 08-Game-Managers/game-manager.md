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
**Game States** represent the various stages of a game, such as whether the game is running, paused, or over, and the transitions between these states. They are crucial in complex games where different systems (AI, UI, physics, etc.) need to interact in a specific sequence depending on the game's current state.

#

>[!WARNING]
> For this example our **GameManager** will handles game states by transitioning between different phases of gameplay, such as "GamePlay," "Pause," and "Game Over." To manage these states, there are generally two common approaches: the **State Pattern** and the **Finite State Machine (FSM)**.
>
>In this case, we implement the **FSM** method, which uses a predefined set of states and transitions between them in a clear and structured manner. This is a simpler and more efficient way to manage state changes, especially in our game where the states are not complex enough to require fully encapsulated behaviors per state, as the State Pattern would offer.

#

Before we start development on the GM we need to have a rough idea of the game states we will have in the game and what takes place during that state. While these may vary, the most common states include: 
- **MainMenu**: The game starts here. The GM will load any necessary UI components for the main menu and wait for user input to either start the game, load a saved game, or exit.
- **GamePlay**: The core of the game, in which the player is actively playing. The GM continuously checks if the game conditions are met for a win or loss.
- **Pause**: When the game is paused, the GM freezes gameplay systems, disables player input for movement, and opens the pause menu.
- **GameOver**: When the player loses or when the game is over, the GM will display the Game Over screen and stopping all gameplay logic.

### Defining Game States

To manage the various phases of the game, we will define the **game states** as an **Enum**. Enums are ideal for this purpose, as they allow us to clearly define a set of named values representing the possible game states, such as "MainMenu," "Playing," "Paused," and "GameOver." Using an Enum makes the code more readable and reduces the risk of mistakes that could arise from using raw integers or strings.

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

```csharp showLineNumbers title="GameState.cs"
   public enum GameState
   {
       MainMenu,    // Game is in the main menu
       GamePlay,     // Game is actively being played
       Pause,      // Game is paused
       GameOver    // Game is over
   }
   ```

5. The class and return to Unity.

---

## :hammer_and_wrench: Basic Game Manager 

1.	In the **`Project`** window under **`Assets > Scripts > Managers`** right-click and create a new script.

2.	Name this new script **`GameManager`** and open the file in your preferred IDE. 

### Singleton Inheritance
As mentioned earlier a GM is typically designed using a **Singleton Design Pattern**. To implement this we will be making use of our **`Singleton`** base class we created previously. 

3. Type the following code below.

```csharp showLineNumbers title="GameManager.cs"

using UnityEngine;

// The GameManager class is derived from the Singleton pattern to ensure there is only one instance of it in the game.
public class GameManager : Singleton<GameManager>
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

            case GameState.Pause:
                // Paused Logic
                Debug.Log("Game State: Pause");
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

- **CurrentState (GameState)**: Holds the current state of the game (e.g., `MainMenu`, `Playing`, `Paused`, `GameOver`), determining which game logic to execute.

- **Start Method**: Initializes the game by calling the `ChangeGameState` method and passing the `GameState.MainMenu` before the first frame update.

- **ManageGameState Method**: Contains a `switch` statement that checks the `CurrentState` and executes the appropriate logic for each game state (e.g., `MainMenu`, `Playing`, `Paused`, `GameOver`).

- **ChangeState Method**: Changes the `CurrentState` to a new game state when called, allowing the game to transition between different states (e.g., from `MainMenu` to `Playing`).

### GM Game Object
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
>The `_gameManager` reference is set in the `Start()` method of the **Player** class to ensure the **GameManager** instance is fully instantiated before the Player tries to access it.
If you attempt to reference `GameManager.Instance` in `Awake()` or at declaration, there is a chance the **GameManager** has not yet been initialized, which could lead to null reference errors.
Initializing in `Start()` helps avoid this issue.

#

Now, when the player collides with the goal, you will see messages in the console:

```
Goal reached! Changing game state...
Game State: GameOver
```

The debug output on our `OnTriggerEnter()` in the **Player** class and the `ManageGameState()` method in the **GameManager** confirms that the **GameManager** is working and that state changes are being triggered.

#

>[!TIP]
> #### Use a Single Entry Point
>Another method to handle order of execution is to have a single entry point scene. Essentially, this means that the game starts with one main empty scene containing the **GameManager** and other global managers, such as a **SceneFlowManager**.
>All other gameplay, menu, or level scenes are then **loaded additively** on top of this entry scene. This ensures all managers are awake and ready to communicate before any other objects attempt to access them, avoiding timing and dependency issues.

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

Let's take a look at how we would implment some of these behaviors. 

---

## :hammer_and_wrench: Switching Scenes with GameStates 
```csharp
using UnityEngine;
using UnityEngine.SceneManagement;

// The GameManager class is derived from the Singleton pattern to ensure there is only one instance of it in the game.
public class GameManager : Singleton<GameManager>
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
                SceneManager.LoadScene("MainMenu");
                break;

            case GameState.GamePlay:
                // Playing Logic
                Debug.Log("Game State: GamePlay");
                SceneManager.LoadScene("Level01");
                break;

            case GameState.Pause:
                // Paused Logic
                Debug.Log("Game State: Pause");
                SceneManager.LoadScene("PauseMenu", LoadSceneMode.Additive);
                break;

            case GameState.GameOver:
                // GameOver logic
                Debug.Log("Game State: GameOver");
                SceneManager.LoadScene("GameOver");
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
#### How It Works
- **using UnityEngine.SceneManagment** provides access to the **SceneMangment** namespaces for loading scenes in Unity.
-  Each game state corresponds to a specific scene or menu.
- **Additive** loading is used for overlays like the pause menu, so gameplay continues underneath.

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




