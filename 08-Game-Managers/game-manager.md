# Game Manager
The **GameManager** is a key component of a game's architecture. It's responsible for overseeing the **flow of the game**, managing **game states**, controlling **major systems**, and maintaining **global functionality**. Often seen as the **central controller** of a game, the Game Manager ensures that gameplay is cohesive and functions smoothly.  It plays such a **central role** in orchestrating game logic that it is considered a **core game system**. Nearly all games, from simple prototypes to large-scale productions, rely on some form of **GameManager** to **coordinate gameplay mechanics and state transitions**, making it essential to the game's overall structure. 

#

>[!NOTE]
> In many cases, Game Manager will implement a **Singleton Pattern** to maintain a single point of control over the game's flow and systems. This ensures that no matter where you are in the game, whether in a scene, menu, or during gameplay, the GM remains the same instance, providing a consistent and unified experience.  
>
> **Why Use a Singleton for the Game Manager?**  
> 1. **Global Access**: The GameManager needs to be easily accessible from UI, player input, AI, and other systems without manually passing references.  
> 2. **Consistency**: Prevents multiple instances with conflicting states, ensuring game-wide synchronization.  
> 3. **Efficiency**: Reduces memory usage and improves performance by eliminating redundant instances.

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
    public GameState CurrentState {get; private set;} = GameState.BootStrap;

```
Here, we implement the **Singleton base class** to ensure there is **only one instance** of the GameManager in the game. 

At the same time, we define a reference to the **current game state** using a property with a `private set`.

By using a private set, we allow other scripts to **read** the game state, but only the GameManager itself can change it. This ensures that all state transitions are centralized, controlled, and predictable, preventing other scripts from inadvertently modifying the game flow.

The property is initialized to **GameState.Bootstrap**, representing the game's single entry point where core systems are set up before any menus or gameplay scenes are loaded.

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

#### 6. Create a GameManager Prefab
Now that we have created our **GameManager** class, we will return to the Unity Editor to implement it in our project.

1. In the Unity Editor, return to one of the _sample scenes _

2. In the **hierarchy panel**, right-click and choose **`Create > Create Empty`** 
   - Name the new empty game object **GameManager**
   - Add the **GameManager** class to the **GameManager** game object
   - In the Inspector, ensure the **`Is Persistent`** property is set to true.

3. Convert the **GameManager** game object to a prefab
   _(i.e., Drag and drop it from the **hierarchy panel** to the **project panel**)_

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

# 🎉 New Achievement: GameManager Framework
We now have a successful **GameManager** framework that we can build upon. 

```csharp
using UnityEngine;

public class GameManager: Singleton<GameManager>
{
    // Reference to the current state of the game
    public GameState CurrentState {get; private set;}
    
    // Start is called before the first frame update
    void Start()
    {
        // Set the initial game state to Main Menu
        ChangeGameState(GameState.MainMenu);

    }//end Start()
    
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
 
}//end GameManager

```
---
## Extending the GameManager Framework

While the GameManager framework provides a foundation for handling game state and overall flow, it can be expanded to manage a wide variety of game-specific logic. Depending on the scope and design of your project, in a **small-scale game**, this typically includes:
- **Starting and stopping gameplay** – initiating or ending a level.
- **Displaying menus or overlays** – such as main menu, pause, or results screens.
- **Pausing and resuming the game** – controlling game time and input.
- **Switching scenes** – loading and unloading levels, menus, or results screens.
- **Handling game over or level completion** – triggering transitions to results or next levels.
- **Basic global rules or systems** – things that don’t belong to a single object but affect the entire game.

Building on our current framework, we will next implement scene switching.


**[<< Return Game States tutorial](game-states.md)** | **[Continue to Switching Scenes tutorial >>](switch-scenes.md)**











