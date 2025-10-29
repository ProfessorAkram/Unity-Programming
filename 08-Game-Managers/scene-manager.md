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


## Scene Dictionary
When managing scenes in a game, especially one with **additive loading and overlays**. We need a **centralized way to reference all scenes** and beable to differnate  about each scene:
1. **Layer** – is it a primary scene (main level, HUD, hub) or an overlay (Pause Menu, Inventory, Dialogue)?
2. **Category** – is it a menu, gameplay level, cutscene, or hub?

Instead of scattering scene names as strings throughout your code, we use a list of SceneEntry objects as a dictionary-like structure. Each SceneEntry holds the information about one scene: its name, layer, and category.

Using a class as a dictionary is appropriate here because:

It’s reference-based, so multiple parts of your code can reference the same scene entry without duplicating data.

It can be mutable, allowing us to add or remove scenes from lists at runtime if needed.

Unity can serialize it in the Inspector (with [System.Serializable]), so designers can edit the data once in the Inspector, without having to hardcode strings in scripts.

This approach provides a single source of truth for all scenes and makes it easy to automate scene loading/unloading based on game state.
```csharp
// Enums at the top of the file
public enum SceneLayer
{
    Primary,   // Main content: levels, HUD, hubs
    Overlay    // Temporary overlays like Pause Menu
}

public enum SceneCategory
{
    Menu,
    GameLevel,
    Cutscene,
    Hub,
    Other
}

// SceneEntry class
[System.Serializable]
public class SceneEntry
{
    public string sceneName;        // Scene name
    public SceneLayer layer;        // Primary or Overlay
    public SceneCategory category;  // Menu, GameLevel, Cutscene, etc.
}

```
Explain System.Serializable




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
The main responsibility for our **SceneFlowManager**, as the name implies, is managing scenes. As such, we need to create a reference to all the different types of scenes we want managed. 

```csharp
using UnityEngine;
using UnityEngine.SceneManagement;

public class SceneFlowManager : Singleton<SceneFlowManager>
{

    // Determines the “layer” of the scene
    public enum SceneLayer
    {
        Primary,   // Main content: levels, persistent menus like HUD
        Overlay    // Temporary overlays like Pause Menu, Inventory
    }

    // Determines the type/kind of the scene
    public enum SceneCategory
    {
        Menu,
        GameLevel,
        Cutscene,
        Hub,
        Other
    }

    [System.Serializable]
    [Tooltip("Dictionary of all scenes")]
    public class SceneEntry
    {
        public string sceneName;         // Name of the scene
        public SceneLayer layer;         // Primary or Overlay
        public SceneCategory category;   // Menu, GameLevel, Cutscene, etc.
    }




    [Header("Menu Scenes")]
    [SerializeField] private string _mainMenuScene = "MainMenu";
    [SerializeField] private string _pauseMenuScene = "PauseMenu";
    [SerializeField] private string _hudMenuScene = "HUDMenu";
    [SerializeField] private string _gameOverScene = "GameOver";

    [SerializeField]
    [Tooltip("Temporary overlay scenes, such as Pause Menu. \nExcludes persistent overlays scenes like the HUD.")]
    private List<string> _overlayScenes = new List<string>;

    // Main levels, HUD, menus that replace gameplay
    private List<string> _primaryScenes = new List<string>();  

    // Temporary overlays like Pause Menu
    private List<string> _secondaryScenes = new List<string>(); 

    [Header("Gameplay Levels")]
    [SerializeField]
    [Tooltip("List of all gameplay levels in the game.")]
    private List<string> _gameplayLevels = new List<string>();
    
    [SerializeField]
    [Tooltip("Set to true if levels are linear; false if next level will be passed explicitly.")]
    [SerializeField] private bool _levelsAreLinear = true;


}//end SceneFlowManager

```

### Handle Scene For State

```csharp
/// <summary>
/// Determines and loads the appropriate scene based on the current GameState.
/// This method is called by the GameManager whenever the game state changes.
/// </summary>
/// <param name="state">
/// The current GameState value representing the phase of the game 
/// (e.g., MainMenu, GamePlay, Pause, GameOver).
/// </param>
public void HandleSceneForState(GameState state)
{
    // Always clean up additive overlays first
    UnloadAllOverlays();

    switch (state)
    {
        case GameState.MainMenu
            LoadScene(mainMenuScene);
            break;
        case GameState.GamePlay:
            LoadLevel(currentLevelIndex);
            break;
        case GameState.Pause:
            LoadScene(pauseMenuScene);
            break;
        case GameState.GameOver:
            LoadScene(gameOverScene);
            break;
    }//end switch (state)
    
}//end HandleSceneForState()

```

### Loading Scenes

```csharp
/// <summary>
/// Loads a scene by name. 
/// - If the scene is listed in _overlayScenes and is not already loaded, it loads additively and tracks it.
/// - Otherwise, it loads the scene normally (replacing the current scene).
/// Null or empty scene names are ignored.
/// </summary>
/// <param name="sceneName">The name of the scene to load.</param>
private void LoadScene(string sceneName)
{
    if (string.IsNullOrEmpty(sceneName))
    {
        Debug.LogWarning("Attempted to load a scene with an empty or null name.");
        return;
    }

    // Load temporary overlay scene additively if not already loaded
    if (_overlayScenes.Contains(sceneName) && !_loadedOverlayScenes.Contains(sceneName))
    {
        SceneManager.LoadScene(sceneName, LoadSceneMode.Additive);
        _loadedOverlayScenes.Add(sceneName);
        Debug.Log($"Loaded overlay scene additively: {sceneName}");
    }
    // Otherwise, load normally, will still be additive
    else
    {
        SceneManager.LoadScene(sceneName, LoadSceneMode.Additive);
        Debug.Log($"Loaded scene: {sceneName}");
        
    }//end if(_overlayScenes)
    
}//end LoadScene()
```

### Unload Overlay Scenes on State Change 
Whenever there is a Game State change, any overlay scenes will need to be unloaded. 

```csharp
/// <summary>
/// Unloads all currently loaded temporary overlay scenes (e.g., Pause Menu, Inventory)
/// and clears the tracking list. Persistent overlay scenes like the HUD are not affected.
/// </summary>
private void UnloadAllOverlays()
{
    //For every overlay scene in loaded list
    foreach (string overlay in _loadedOverlayScenes)
    {
        UnloadScene(overlay);
        
    }//end foreach

    _loadedOverlayScenes.Clear();
    
}//end UnloadAllOverlays()

```

```csharp
```


Loading Game Level Scenes

```csharp

    /// <summary>
    /// Loads the next level (if levels are linear).
    /// </summary>
    public void LoadNextLevel()
    {
        if (_gameplayLevels.Count == 0)
        {
            Debug.LogWarning("No gameplay levels defined in SceneFlowManager.");
            return;
            
        }//end if (_gameplayLevels.Count == 0)

        if (_levelsAreLinear)
        {
            _currentLevelIndex++;
            if (_currentLevelIndex >= _gameplayLevels.Count)
            {
                Debug.Log("No more levels. Returning to Main Menu.");
                LoadScene("MainMenu");
                return;
                
            }//end if (_currentLevelIndex >= _gameplayLevels.Count)
            
        }//end if (_levelsAreLinear)

        string nextLevel = _gameplayLevels[_currentLevelIndex];
        SceneManager.LoadScene(nextLevel);

    }//end LoadNextLevel()



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




