# Scene Flow Manager
While the **GameManager** is responsible for overseeing the flow of the game, managing game states, controlling major systems, and maintaining global functionality, a common trap for beginners is trying to make the **GameManager** do everything. Just as we shouldn’t place every task in a single `Update()` method, the GameManager should only handle global functions that don’t fall under a more specific category.

While using the **GameManager** for all tasks works for very simple projects, it mixes responsibilities, making your code harder to maintain and scale. Following this principle reinforces the **Single Responsibility Principle (SRP)**; each class should have a clear, focused purpose.

One responsibility we can extract from the **GameManager** is **scene management**. By creating a **SceneFlowManager**, we give a dedicated system that specializes in scene-related operations, separating that responsibility from general game logic.

## What a SceneFlowManager Does
Our **SceneFlowManager** will be is responsible for handling everything related to the scenes in your game. While the **GameManager** knows what state the game is in (Playing, Paused, GameOver, etc.), the **SceneFlowManager** knows which scene should be active for that state.

Following are some of the key responsibilities of the **SceneFlowManager**:
- Loads and reloads scenes (levels, menus, or results screens).
- Manages transitions between scenes, such as menus → gameplay → results.
- Works in tandem with game states to ensure the correct scene reflects the current state.
- Optional: handles transition effects (fade-in/out, loading screens).

By keeping scene management separate from game state logic, your project becomes more modular, easier to debug, and easier to expand with new scenes or transitions.

>[!IMPORTANT]
> Unity already has a class called **SceneManager** in **UnityEngine.SceneManagement**. Naming a custom script **SceneManager** will create conflicts and compiler errors.
> To avoid any ambuity or conflicts we will name our scene manager, **SceneFlowManager** as this also implies that we are not only manging the different scenes but the flow (transition and sequencing) between the scenes.

### Implmenting the Singleton Pattern
Just like our **GameManager** implments the **singleton pattern** so will our **SceneFlowManager**. This will ensure that there is ever only one instance of the **SceneFlowManager** throughout the entire game. 
The **SceneFlowManager** will also need referece to all Unity's Scene Management libraries.

```csharp

using UnityEngine;
using UnityEngine.SceneManagement;

public class SceneFlowManager : Singleton<SceneFlowManager>
{

}

```

---

## Managing Scenes
The main responsiblity for our **SceneFlowManager** as the name implies is managing scenes. As, such we need to create a reference to all the different types of scenes we want managed. 

```csharp
using UnityEngine;
using UnityEngine.SceneManagement;

public class SceneFlowManager : Singleton<SceneFlowManager>
{
    [Header("Gameplay Levels")]

    [SerializeField]
    [Tooltip("List of all gameplay levels in the game.")]
    private List<string> _gameplayLevels;

    [Tooltip("Set to true if levels are linear; false if next level will be passed explicitly.")]
    [SerializeField] private bool _levelsAreLinear = true;

    private int currentLevelIndex = 0;

    [Header("Additive Menu Scenes")]

    [SerializeField]
    [Tooltip("Menus that load on top of the current gameplay scene.")]
    private List<string> additiveMenus = new List<string>;

    private List<string> loadedAdditiveScenes = new List<string>();

    [Header("Full Replacement Scenes")]

    [Tooltip("Menus or hubs that replace gameplay completely.")]
    [SerializeField] private List<string> fullMenus = new List<string>;


}//end SceneFlowManager

```

<!--
1. Create a list for all game levles
2. Create a counter for indexing the game levles

```csharp
//List of all playable game levels (scenes)
private List<string> _gamelevels = new List<string> { "Level01", "Level02" };

//Index coutner for gameplay levels
private int _currentLevelIndex = 0;
```

3. Create a `LoadGameLevel()` method

```csharp

/// <summary>
/// Loads the current gameplay level without reloading it unnecessarily.
/// </summary>
private void LoadGameLevel()
{
    // Check if the current level index is within the range of available levels
    if (_currentLevelIndex < _gamelevels.Count)
    {
        // Determine which level to load based on the current index
        string levelToLoad = _gameplaylevels[_currentLevelIndex];

        // Get a reference to the currently active scene
        Scene activeScene = SceneManager.GetActiveScene();

        // Only load if it's not already the active level
        if (activeScene.name != levelToLoad)
        {
            SceneManager.LoadScene(levelToLoad);

         }//end if(activeScene)

    }//end if (_currentLevelIndex < _gamelevels.Count)

}//end LoadGameLevel()

```
#### How it works
- Checks the **current level index** to ensure it is within the range of available game levels.
  - Prevents trying to load a non-existent level if the player has completed all levels.
- Determines the **level to load** based on the `_currentLevelIndex` in `_gamelevels`.
- Gets a reference to the currently **active scene**.
  - This prevents reloading the same level unnecessarily, which could reset player progress or gameplay.
- Compares the active scene name to the level to load.
  - Only loads the scene if it is not already active.
- Uses **SceneManager.LoadScene()** to load the new level when required.


4. 
-->

