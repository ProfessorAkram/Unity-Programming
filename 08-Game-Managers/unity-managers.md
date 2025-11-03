# Game Manager System

## Lesson Overview
This lesson explores how to build a flexible and maintainable **Game Management System** in Unity. You’ll start by learning how to create persistent manager classes, then expand them to handle **game states**, **scene transitions**, and **player input** using clean, event-driven design. By the end, you’ll have a GameManager capable of controlling your game’s entire flow, from loading levels to pausing gameplay.

## Tutorial Lessons

#### [01: Creating a Singleton Base Class](singleton.md)
Learn why the Singleton pattern is ideal for systems that must have only one instance—like game managers. You’ll create a reusable, generic Singleton base class and make it persistent across scenes using DontDestroyOnLoad().

#### [02: Implementing Game States](game-states.md)
Define a clear GameState enum (e.g., Bootstrap, MainMenu, Gameplay, Pause, GameOver) and understand how it helps organize and track the overall game flow. You’ll also connect this system to the GameManager for centralized state control.

#### [03: Building the Game Manager](game-manager.md)
Use your Singleton base class to build the GameManager, the hub that coordinates the game’s state transitions, initialization, and overall logic. The GameManager will act as the single point of entry and communication between major systems.

#### [04: Managing Scenes](switch-scenes.md)
Set up clean scene switching between menus and gameplay using Unity’s SceneManager API. Learn how to load and unload scenes additively, manage active scenes, and connect your scene logic with the GameManager.

#### [05: Decoupling Logic with the Observer Pattern](observer-pattern.md)
Explore how the Observer pattern (implemented with C# events) enables flexible communication between systems. You’ll create a fun analogy-based example, then apply the concept to make game objects react without direct references.

#### [06: Implementing Game Pause](game-pause.md)
Tie everything together by creating an event-driven pause system. Use Unity’s Input System to subscribe to the pause event, and have the GameManager handle the pause and resume logic across all active scenes.
