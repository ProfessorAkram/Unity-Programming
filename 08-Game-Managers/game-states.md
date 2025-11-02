# Understanding States in Games 
In game development, a **state** represents a specific mode or condition that defines what an object or system is currently doing. At any moment, a game, character, or system exists in exactly one state, such as **Running**, **Paused**, or **Attacking**. States help organize logic by ensuring that only the appropriate behaviors run at the appropriate times.

For example, a player character might have states like `Idle`, `Walk`, `Run`, and `Jump`, each controlling animations and input responses differently. Similarly, an enemy AI could have states like `Patrol`, `Chase`, or `Attack`, changing how it behaves based on the situation. By using states, we make our code easier to follow, more modular, and less prone to bugs caused by overlapping logic.

## State Pattern vs Finite State Machine (FSM)
There are two common approaches to managing states in games: the **Finite State Machine (FSM)** and the **State Pattern**. Both aim to structure how states are defined and how transitions between them occur, but they differ in complexity and flexibility.

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

---

## Game States
One of the most common applications of states is in managing the **overall flow of a game**, often referred to as _Game States_. GameStates define **what the game is doing right now**; for example, `MainMenu`, `GamePlay`, or `GameOver`. They represent mutually exclusive, player-visible modes. Systems like scoring, level progression, or UI updates are **not GameStates**; instead, they are **actions or systems that respond to the current state**.

By centralizing this information in a Game State, all other systems can check the current state and respond accordingly. This structure makes the overall game logic more organized, predictable, and scalable as new states are added.

### Using Game States with a Game Manager

In many projects, game states are managed by a **Game Manager**; a centralized system responsible for tracking the current state of the game and handling transitions between them. Game state management can be implemented using either the **State Pattern** or a **Finite State Machine (FSM)**, as discussed earlier. For our Game Manager, we’ll be** using the FSM approach**, as it’s beginner-friendly, straightforward to implement, and well-suited for small-scale or prototype projects. An FSM makes it easy to manage state transitions, load and unload scenes, and control gameplay flow without the added complexity of creating separate classes for each state, which the State Pattern typically requires.

#

## Defining Game States

Since we are implementing an **FSM** for our **game states**, we will want to define each state as an **enum**. Enums are ideal for this purpose, as they allow us to clearly define a set of named values representing the possible game states. Using an Enum makes the code more readable and reduces the risk of mistakes that could arise from using raw integers or strings.

Before we start development on the Game Manager, we need to have a rough idea of the game states we will have in the game and what takes place during each state. While these may vary, the most **common core game states** include: 
- **Bootstrap**: The game will boot here and initialize the GameManager and other core systems before any gameplay or menus appear. 
- **MainMenu**: The game starts here. The GameManager will load any necessary UI components for the main menu and wait for user input to either start the game, load a saved game, or exit.
- **GamePlay**: The core of the game, in which the player is actively playing. The GM continuously checks if the game conditions are met for a win or a loss.
- **GameOver**: When the player loses or when the game is over, the GM will display the Game Over screen and stop all gameplay logic.

#

>[!NOTE]
> The **Bootstrap** game state corresponds to a persistent **Bootstrap scene**, which ensures the game starts from a consistent foundation. We'll discuss the details of this scene and how it manages core systems later, when we build the GameManager framework.

#

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
       BootStrap,  // Initial boot state
       MainMenu,   // Main menu screen
       GamePlay,   // Active gameplay
       GameOver    // Game over screen
   }
```   
5. Save the class and return to Unity.

---

Now that we have our game states established, we can start to build out our Game Manager in the next section. 

**[<< Return Singelton tutorial](singleton.md)** | **[Continue to Game Manager tutorial >>](game-manager.md)**
