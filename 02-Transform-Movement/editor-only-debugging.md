# Editor-only Debugging Tools

Sometimes you need a quick way to **test your component** before the rest of the game systems are ready. But you don’t want that temporary code sneaking into the final build! This is where **Editor-only Debugging Tools** come in handy.

Earlier, we mentioned [`#if UNITY_EDITOR`](#using-if-unity_editor-for-editor-only-debugging), a special compiler directive in Unity. This directive enables us to enclose test code, allowing it to run only within the Unity Editor. Anything inside these blocks will never be included in a build, keeping your gameplay code clean while still providing powerful testing options during development.

---

## Enable Editor Testing 

To safely test our `MoveTransform` component in the Editor, we need a master switch, a flag that asks the question, _“Do you want to run tests in the Editor?”_

The flag should be a `[SerializeField]` to allow for **Inspector** modification and should be set to **false**, so nothing will run accidentally while you’re working on other parts of your game. Since this flag is not needed in the game build, we can place it inside a `#if UNITY_EDITOR` wrapper:

```csharp
#if UNITY_EDITOR
    [SerializeField] 
    private bool _enableEditorTesting = false;
#endif

```
This makes testing optional and keeps the code clean, ensuring that temporary test logic only runs when explicitly enabled in the Editor.

<br>

> [!TIP]
> The code inside a `#if UNITY_EDITOR` wrapper only runs in the Unity Editor. Many IDEs will render it as a comment, which can make it harder to read while typing. To avoid mistakes, write your code **first**, then wrap it with `#if UNITY_EDITOR`.

### Organizing Fields in the Inspector with Headers

When working with multiple `[SerializeField]` fields, it’s helpful to visually organize them in the Inspector. This makes it easier for designers or team members to understand which collection of fields does what. In our case, we have general behavior and ones meant only for testing.

In Unity, you can use the `[Header("Header Text")]` attribute to add a clear label above related fields:

```csharp
  // Serialized fields for initial values
    
    [Header("MOVEMENT SETTINGS")]
    
    [SerializeField]
    [Range(0f, MAX_SPEED)]
    [Tooltip("Speed of the object’s movement. Cannot exceed maximum speed.")]
    private float _speed = 5f;

.....
#if UNITY_EDITOR
    [Header("FOR TESTING ONLY")]
    [SerializeField] 
    private bool _enableEditorTesting = false;
#endif

```


## Testing Actions with Enums

When testing our `MoveTransform` component, we want to choose one action at a time, either `Move()` or `Stop()`.

Using two separate booleans (`_testMove` and `_testStop`) could work, but it has some drawbacks:

- Both could accidentally be true at the same time.
- It’s harder to scale if we add more test actions later.
- The inspector would get messy with multiple toggles.

What we need is an easy way to select between multiple options. Instead of asking a simple yes/no question like _“Should I move?”_ or _“Should I stop?”_ we ask, _“Which one of these actions should I run?"_ An **enum** is the solution. 

### What is an enum?

An **enum (short for "enumeration")** is a special type in C# that lets you define a named set of constant values. Instead of using raw numbers or multiple booleans, an enum gives your options meaningful names, making code easier to read and safer to use. An **enum is its own type**, similar to a class or struct. It defines a new data type with a fixed set of named values that it can have.

**Define the enum** for testing different actions in our `MoveTransform`

```csharp
#if UNITY_EDITOR
    [Header("FOR TESTING ONLY")]
    [SerializeField] 
    private bool _enableEditorTesting = false;

   //Enum for list of possible actions to test 
    private enum TestAction
    {
        None,   // No action selected
        Move,   // Run Move() test
        Stop    // Run Stop() test
    }

    #endif

```

<br>

> [!NOTE]
> Because an **enum** act as a new **data type**, the naming convnetion should always be in **ParsalCase**. 

<br>

### Storing Enum Selection 

With the enum defined, the next step is to make **a variable of that type** and choose one of its named values to control our test. When assigning a value in code, we use **dot syntax**: `EnumType.Value`. This starts with the **enum type** (the name) followed by the **named value**.

```csharp
#if UNITY_EDITOR
    [Header("FOR TESTING ONLY")]
    [SerializeField] 
    private bool _enableEditorTesting = false;

   //Enum for list of possible actions to test 
    private enum TestAction
    {
        None,   // No action selected
        Move,   // Run Move() test
        Stop    // Run Stop() test
    }

    [SerializeField]
    [Tooltip("Select which action to test in the Editor.")
    private TestAction _testAction = TestAction.None;

#endif

```

<br>

> **✔️ CHECK POINT**
> 
> Save your script, switch back to the Unity editor, and press **Play** to test the changes in action.
> 
<br>

## Execute Movement Tests in Editor

Now that our testing flags are set up, we need to modify `Update()` to check if `_enableEditorTesting` is `true`. If it is, we then check **which action** we want to test. Since there are multiple conditions to handle, it’s best to put this functionality in its **own method**, which we will call `RunMovementTest()`.

**Modify the `Update()` with the following:
```csharp
    // Update is called once per frame
    private void Update()
    {
        
#if UNITY_EDITOR
        if (_enableEditorTesting)
        {
            RunMovementTest();
        }//end if(_enableEditorTesting)
#endif
        
        if(_isMoving)
        {
            Move();

        }//end if(_isMoving)

    }//end Update()
```

---
## Create the `RunMovementTest()` 

The `RunMovementTest()` method checks the value of `_testAction` and executes the corresponding movement method:
- If `_testAction` is `Move`, it calls `Move()`.

- If `_testAction` is `Stop`, it calls `Stop()`.

- After executing an action, it resets `_testAction` to `None` so the action only runs once per selection.

- If `_testAction` is `None`, nothing happens.

This method keeps our **`Update()` loop clean** by separating the decision-making logic from frame updates.

### Implementing a Switch Statement 
A switch statement is a control structure that evaluates a single value and executes code based on which case matches that value. It is especially useful when you have multiple discrete options to check, since it avoids long chains of `if/else if` statements and makes the logic easier to read.

In our example, we want to handle different actions (`Move`, `Stop`, `None`). Using a switch here makes the intent very clear; each possible action has its own dedicated case block. This also makes the code easier to extend: if we later add new actions such as Reverse or Reset, we simply add new case blocks without touching the existing logic.

**Best Practices with Switch Statements:**
- **Handle every possible case:** Even if an action doesn’t need to do anything, include an empty case. This makes the code explicit and avoids ambiguity for future readers.
- **Always include a default case:** This acts as a safety net. If the condition value (such as an enum) is updated but no new case is added to the switch, the default ensures that unexpected values are caught—typically by logging an error or throwing an exception.

```cshar

#if UNITY_EDITOR    
    /// <summary>
    /// Runs the selected movement test in the Editor based on the _testAction enum.
    /// </summary>
    private void RunMovementTest()
    {
        switch (_testAction)
        {
            case TestAction.Move:
                Debug.Log("Testing Move");
                Move();
                break;

            case TestAction.Stop:
                Debug.Log("Testing Stop");  
                Stop();
                break;

            case TestAction.None:
                Debug.Log("Testing None");
                //Do nothing
                break;
                
            default:
                Debug.Log("Unhandled TestAction: " + _testAction); 
                break;

        }//end switch(_testAction)

    }//end RunMovementTest()
#endif


```

<br>

> **✔️ CHECK POINT**
> 
> Save your script, switch back to the Unity editor, and press **Play** to test the changes in action.
> 
<br>


---

# 🎉 New Achievement: Editor Testing

Congratulations! You’ve now added **Editor-only testing** to the `MoveTransform` component. With these improvements, you can:

- Quickly test movement and stopping directly in the Inspector using `_enableEditorTesting`.

- Select the desired test action (`Move()` or `Stop()`) via the `_testAction` dropdown.

- Keep your testing logic completely **separated from gameplay code**, thanks to the `#if UNITY_EDITOR` wrapper.

- Easily extend tests in the future by adding new actions without modifying core movement logic.

These changes make your component **safer to experiment with**, **easier to debug**, and **provide designers with tools to verify behavior** while building scenes, all without affecting the final game build.

```csharp
using UnityEngine;

public class MoveTransform : MonoBehaviour
{

    // Maximum speed allowed
    private const float MAX_SPEED = 10f;

    // Serialized fields for initial values

    [Header("MOVEMENT SETTINGS")]
    
    [SerializeField]
    [Tooltip("Should the object be moving on initialization?")]
    private bool _moveOnAwake = true;
    
    [SerializeField]
    [Tooltip("Direction of movement. Will be normalized automatically to ensure consistent movement.")]
    private Vector3 _direction = Vector3.right;

    [SerializeField]
    [Range(0f, MAX_SPEED)]
    [Tooltip("Speed of the object’s movement. Cannot exceed maximum speed.")]
    private float _speed = 5f;

   

    // Runtime movement flag
    private bool _isMoving;
    

#if UNITY_EDITOR
    [Header("FOR TESTING ONLY")]
    [SerializeField] 
    private bool _enableEditorTesting = false;

   //Enum for list of possible actions to test 
    private enum TestAction
    {
        None,   // No action selected
        Move,   // Run Move() test
        Stop    // Run Stop() test
    }

    [SerializeField]
    [Tooltip("Select which action to test in the Editor.")]
    private TestAction _testAction = TestAction.None;

#endif    
    
    // Public properties with encapsulation
    
    public float Speed
    {
        get => _speed; 
        set => _speed = Mathf.Clamp(value, 0f, MAX_SPEED);
    }
    
    public Vector3 Direction
    {
        get => _direction;
        set => _direction = value.normalized;
    }
    
    
    // Awake is called once on initialization         
    private void Awake()
    {
        // Validate initial speed and direction via properties
        Speed = _speed;
        Direction = _direction;
        
        //Set GameObject's initial position
        //transform.position = Vector3.zero;
        
        // Determine if the object should start moving
        _isMoving = _moveOnAwake;
    }//end Awake()

    // Update is called once per frame
    private void Update()
    {
        
#if UNITY_EDITOR
        if (_enableEditorTesting)
        {
            RunMovementTest();

        } //end if(_enableEditorTesting)
#endif
        
        if(_isMoving)
        {
            Move();

        }//end if(_isMoving)
        
    }//end Update()
    
     
    /// <summary>
    /// Moves the object in a specified direction at a specified speed.
    /// </summary>
    /// <param name="direction">The direction to move the object (optional).</param>
    /// <param name="speed">The speed at which the object should move (optional).</param>
    public void Move(Vector3? direction = null, float? speed = null)
    {
        // Resolve the effective values for this frame
        Vector3 moveDirection = direction ?? Direction;
        float moveSpeed = speed ?? Speed;

        // Update properties to ensure validation and internal consistency
        Direction = moveDirection;
        Speed = moveSpeed;

        //Debug.Log("Direction: " + Direction);
        //Debug.Log("Speed " + Speed);
        
        // Flags the object as moving
        _isMoving = true;

        // Move the GameObject using the resolved frame values
        transform.position += Speed * Time.deltaTime * Direction;

    }//end Move()
    
    /// <summary>
    /// Stops the object's movement by updating the movement flag.
    /// </summary>
    public void Stop()
    {
        // Flags the object as stopped
        _isMoving = false;

    }//end Stop()
    
#if UNITY_EDITOR    
    /// <summary>
    /// Runs the selected movement test in the Editor based on the _testAction enum.
    /// </summary>
    private void RunMovementTest()
    {
        switch (_testAction)
        {
            case TestAction.Move:
                Debug.Log("Testing Move");
                Move();
                break;

            case TestAction.Stop:
                Debug.Log("Testing Stop");  
                Stop();
                break;

            case TestAction.None:
                Debug.Log("Testing None");
                //Do nothing
                break;
                
            default:
                Debug.Log("Unhandled TestAction: " + _testAction); 
                break;

        }//end switch(_testAction)

    }//end RunMovementTest()
#endif
    
}//end MoveTransform
```

--- 

**[<< Return to Refactor Movement tutorial](refactor-movement.md)**








