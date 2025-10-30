# Game Manager System

## Lesson Overview
This lesson introduces the Game Manager, a central system that controls global game logic such as state transitions, score management, and overall flow. You’ll learn how to implement a **Singleton** base class to ensure only one instance of key managers exists, then expand the system to handle game states, initialization, and persistent data across scenes.

## Tutorial Lessons

#### [01: Creating a Singleton Base Class](singleton.md)
Learn the purpose of the Singleton pattern and why it’s useful for global systems like game managers. Create a generic **Singleton base class** that ensures only one instance of a manager exists, and make it persistent across scenes using `DontDestroyOnLoad()`.

#### [02: Building Game Manager](game-manager.md)
Use the Singleton base class to create a **GameManager** that tracks the overall state of the game. Define a GameState enum (e.g., MainMenu, Playing, Paused, GameOver) and implement methods to transition between them. The GameManager will also manage core _"global"_ gameplay variables such as score and lives.

#### [03: Adding a Level Manager]()
The LevelManager is a separate system responsible for handling level-specific logic, such as win or loss conditions. 
