# Game Manager
The **GameManager (GM)** is a key component of a game's architecture. It's responsible for overseeing the **flow of the game**, managing **game states**, controlling **major systems**, and maintaining **global functionality**. Often seen as the **central controller** of a game, the GM ensures that gameplay is cohesive and functions smoothly.  It plays such a **central role** in orchestrating game logic that it is considered a **core game system**. Nearly all games, from simple prototypes to large-scale productions, rely on some form of **GameManager** to **coordinate gameplay mechanics and state transitions**, making it essential to the game's overall structure. 

>[!NOTE]
>In many cases Game Manager (GM) will implement a **Singleton Pattern** to maintain a single point of control over the game's flow and systems. This ensures that no matter where you are in the game, whether in a scene, menu, or during gameplay, the GM remains the same instance, providing a consistent and unified experience.  
>
> **Why Use a Singleton for the Game Manager?**  
> 1. **Global Access**: The GM needs to be easily accessible from UI, player input, AI, and other systems without manually passing references.  
> 2. **Consistency**: Prevents multiple instances with conflicting states, ensuring game-wide synchronization.  
> 3. **Efficiency**: Reduces memory usage and improves performance by eliminating redundant instances.

## Game Manager Functionality
Game Managers (GMs) handle a variety of responsibilities, including **game state management, system coordination, and global functionality**. While a GM may also manage elements like score tracking, scene transitions, or event handling, in this section, we will focus specifically on how it manages **game states** and ensures smooth transitions between them. To demonstrate how this works in practice, we will be building the GameManager for our example game *One Wild Night*.  

### Game States
**Game States** represent the various stages of a game, such as whether the game is running, paused, or over, and the transitions between these states. They are crucial in complex games where different systems (AI, UI, physics, etc.) need to interact in a specific sequence depending on the game's current state.

>[!WARNING]
> For this example our **GameManager** will handles game states by transitioning between different phases of gameplay, such as "GamePlay," "Pause," and "Game Over." To manage these states, there are generally two common approaches: the **State Pattern** and the **Finite State Machine (FSM)**. In this case, we implement the **FSM** method, which uses a predefined set of states and transitions between them in a clear and structured manner. This is a simpler and more efficient way to manage state changes, especially in our game where the states are not complex enough to require fully encapsulated behaviors per state, as the State Pattern would offer.

Before we start development on the GM we need to have a rough idea of the game states we will have in the game and what takes place during that state. While these may vary, the most common states include: 
- **MainMenu**: The game starts here. The GM will load any necessary UI components for the main menu and wait for user input to either start the game, load a saved game, or exit.
- **GamePlay**: The core of the game, in which the player is actively playing. The GM continuously checks if the game conditions are met for a win or loss.
- **Pause**: When the game is paused, the GM freezes gameplay systems, disables player input for movement, and opens the pause menu.
- **GameOver**: When the player loses or when the game is over, the GM will display the Game Over screen and stopping all gameplay logic.

### Defining Game States

To manage the various phases of the game, we will define the **game states** as an **Enum**. Enums are ideal for this purpose, as they allow us to clearly define a set of named values representing the possible game states, such as "MainMenu," "Playing," "Paused," and "GameOver." Using an Enum makes the code more readable and reduces the risk of mistakes that could arise from using raw integers or strings.

We can declare our **Enum** within the **GameManager** class or in its own separate class file. To keep things modular and flexible, we will declare the **GameState Enum** in its own class, ensuring that each state is decoupled from the core logic of the **GameManager**.

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
        // Ghange the game state to the main menu
        ChangeState(GameState.MainMenu);
    
    }//end Start()

    //Manages the logic for each game state 
    void ManageGameState(){

        // Checks the current game state and performs the appropriate actions.
        switch (CurrentState)
        {
            case GameState.MainMenu:
                // MainMenu Logic
                break;

            case GameState.GamePlay:
                // Playing Logic
                break;

            case GameState.Pause:
                // Paused Logic
                break;

            case GameState.GameOver:
                // GameOver logic
                break;
        }//end  switch(CurrentState)

    }//end ManageGameState

 // Changes the current state to a new game state
    public void ChangeState(GameState newState)
    {
        CurrentState = newState;

    }//end ChangeState

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
5. Press **Play** from the Unity Editor. In the **`Hierarchy`** window you should notice that **`GameManager`** object is now listed under **`DoNotDesotry`**. The **`Console`** window should also display a message that outputs the GameManager and the object name (in this case "GameManager") that it is an instance of. These are all behaviors that were setup in the **`Singleton`** base class. 
6. Exit **Play** mode and save the scene.





