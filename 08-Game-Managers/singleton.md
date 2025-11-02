# Singleton Pattern

The **Singleton pattern** is a widely used design pattern for ensuring that a class has only one instance throughout the lifetime of the application. This pattern provides a way to control access to shared, global resources without the need for multiple instances, which can help avoid unexpected behavior or performance issues due to redundant processing

## Singleton in Game Development 

In game development, a common use case for a singleton is in the creation of a **single instance of a manager or controller class (e.g., AudioManager, GameManager, or PlayerController)** where creating multiple instances could lead to conflicts or inefficiencies. By implementing the Singleton pattern, you can ensure that no matter how many times the object is requested or instantiated, only one instance will exist.

## Singleton Base Class
Since there are many instances that require Singleton behavior, rewriting the behavior can be time-consuming, while copying and pasting the Singleton code can lead to redundancy and potential bugs. To increase efficiency, a base Singleton class that any class can inherit from. This simplifies implementation and makes the codebase more maintainable and scalable.

By using a base class for the Singleton pattern, you:
- **Reduce Code Duplication:** Avoid rewriting the Singleton logic for each class.
- **Improve Maintainability:** If you need to change the Singleton logic, you only modify the base class.
- **Enhance Readability:** By centralizing the Singleton logic, the inherited classes remain clean and focused on their specific responsibilities.

### Example Singleton base class with detailed comments:

```csharp
// Generic Singleton base class that any MonoBehaviour can inherit from
public class Singleton<T> : MonoBehaviour where T : MonoBehaviour
{
    // Static instance that holds the reference to the Singleton
    public static T Instance {get; private set;}

    // Unity's Awake method, called when the script instance is being loaded
    private void Awake()
    {
        // Check for singleton duplication
        CheckForSingleton();

    }//end Awake()

    // Ensures that only one instance of the Singleton exists
    void CheckForSingleton()
    {
        // If no instance exists, assign this instance
        if (Instance == null)
        {
            Instance = this as T;
        } 
        // If an instance already exists and it's not this one, destroy the new instance to maintain the Singleton
        else{
        // Ensure that only the original Singleton instance persists
            Destroy(gameObject); 
        }
               
        // Log the current instance for debugging purposes
        Debug.Log(Instance);

    }//end CheckForSingleton()


}//end Singleton
```
### How It Works:

**Generic Singleton Class:** The `Singleton<T>` class is a generic base class, where T is a placeholder for the specific type that will inherit from it (e.g., GameManager, AudioManager). In C#, generics allow you to define classes or methods with a placeholder for a data type, which is substituted at runtime. This enables you to reuse the same Singleton logic across multiple classes, ensuring that each class has a single instance without duplicating the same code. By using `Singleton<T>`, you centralize the Singleton functionality, making it easier to maintain and apply to various MonoBehaviour types.

**Static Instance:** The `Instance` property is `static`, ensuring that it holds the single instance across all scenes. The Instance property provides **global access** to this instance.

**Awake Method:** In Unity, the Awake method is called when the script is loaded. Here, it's used to trigger the `CheckForSingleton` method, which ensures only one instance of the class exists.

**CheckForSingleton Method:** This method checks whether the **Instance** is null.
- If it is null, it assigns the current object as the Singleton.
- If an instance already exists and it's not the current object, the new instance is destroyed to prevent multiple instances.

**Debug Logging:** The `Debug.Log(Instance)` statement logs the current instance, which can be useful for verifying the Singleton behavior during development.

### Checking for Persistence
In most cases, **Singletons** are *persistent*, meaning they remain active for the entire lifespan of the game — even when scenes are loaded or unloaded.  
This allows systems like the **GameManager**, **AudioManager**, or **UIManager** to maintain their state across different scenes.

However, there are situations where you may need a Singleton that is **not persistent**.  
For example, a **GameSpawner** might need to enforce a single instance within a scene but doesn’t need to exist across all scenes.

To keep our code flexible, we’ll add a field that determines whether the Singleton should be persistent.  
When the `Instance` is initialized, the script will check this field and decide whether to call `DontDestroyOnLoad()`.  
This approach allows us to reuse the same Singleton base class for both persistent and scene-specific managers.

#### Create a Boolean for Persistence

In our singleton base class, add the following field: 

```csharp
[SerlizedField]
[ToolTip("Is the game object persistent through scenes.)]
private bool _isPersistant = true;

```

Next, we will update our `CheckForSingleton()` method to call the method for persistence. 

``` csharp
    // Ensures that only one instance of the Singleton exists
    void CheckForSingleton()
    {
        // If no instance exists, assign this instance
        if (Instance == null)
        {
            Instance = this as T;
    
            CheckForPersistance();
        }

```

Now we will create the `CheckForPersistance()` method: 

```csharp
    void CheckForPersitance()
    {       
        // Check if persistence is required
        if (isPersistent)
        {
            // Detach from parent if there is one
            if (transform.parent != null)
            {
                transform.SetParent(null);
            }

            // Mark this GameObject as not to be destroyed
            DontDestroyOnLoad(gameObject);

        }//end CheckForPersitance()
```



>[!NOTE]
>The `DontDestroyOnLoad` method only works if the object is a root object in the scene (i.e., it has no parent). If your Singleton instance has a parent object, it will not persist as expected. To ensure persistence in such cases, you **must detach the object from its parent** before calling `DontDestroyOnLoad`, as we have done in the example above.


**[<< Return Lesson Contents](unity-managers.md)** | **[Continue to Game Manager tutorial >>](game-manager.md)**

