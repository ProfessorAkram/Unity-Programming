# Handling UI Functionality

Once you have your UI set up, the next step is to make it functional. This involves creating a script to handle what happens when buttons are clicked and connecting it to your UI elements. In this tutorial, we’ll create a MainMenu script that controls the main menu behavior.

#### 1. Creating the MainMenu Script

1. In your Scripts folder, create a new C# script called `MainMenu`.

2. Open the script in your editor.

The script will include the following methods:
- `PlayGame()` for loading the first game level
- `ExitGame()` for quitting the application

```csharp

   using UnityEngine;
   
   public class MainMenu : MonoBehaviour
   {
       // Loads the first level of the game
       public void PlayGame()
       {
   
       }//end PlayGame()
   
      // Quits the game application
       public void ExitGame()
       {
   
       }//end ExitGame()
   
   }//end MainMenu

```

#### 2. Unity Scene Management

To make PlayGame() work, you need to know a bit about scene management:

- **Build Settings:** Each scene in your project must be added to the Build Settings (`File` > `Build Settings`).
- **Scene Order:** Scenes have a **build index**, starting at **0**.
   - Typically, the **main menu is 0**, and the first level is 1.
   - The scene order can be changed in the build setting

In order to manage scenes via script Unity's **SceneManagment** namespace needs to be included: 
```csharp
using UnityEngine.SceneManagement;
```
To load a scene, we use the SceneManager.LoadScene(); method. This method takes two parameters:

- **Scene:** This can be specified by **build index** or **scene name** (in quotes).
   - If your scenes are in a known order, the build index can be used.
   - Since there is no guarantee of scene order, it is **often safer to use the scene name**.
   - The scene name must be:
      - **Written in quotes**
      - **Spelled exactly as it appears in your project**
      - **Included in the Build Settings**

- **Load Mode:** Determines how the new scene is loaded. Unity provides two types:
   - **Single (default):** Replaces the current scene with the new one.
   - **Additive:** Loads the new scene on top of the current scene.

> [!NOTE]
> Single load mode is the default, so this parameter does not need to be set when calling the method.

Example: Load scene with **build index** 
```csharp

     // Loads the first level of the game
       public void PlayGame()
       {
           // Load scene by build index (first level is usually index 1)
           SceneManager.LoadScene(1);

       }//end PlayGame()
```

Example: Load scene with **scene name**
```csharp

     // Loads the first level of the game
       public void PlayGame()
       {
           // Or you could load by scene name:
           SceneManager.LoadScene("Level_1");

       }//end PlayGame()
```


