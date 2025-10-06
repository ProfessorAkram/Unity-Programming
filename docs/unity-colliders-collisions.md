# Basics of Colliders, Collisions, and Triggers

In Unity, collisions are the backbone of interaction in most games. They allow objects to detect each other, respond physically, and trigger events like collecting items, taking damage, or bouncing off walls.

## Coliders and Rigidbody Components 
For collisions to be detected, all objects will need a **Collider**. For collisions or triggers to register, at least one object of the interacting objects needs a **Rigidbody**.

- **Colliders** define the shape of a game object for physics interactions. Every game object that should participate in collision detection must have a collider. By default, primitive shapes like cubes or spheres automatically have colliders attached when created.

- **Rigidbody** allows an object to participate in Unity’s physics simulation. A collision will only be detected if at least one of the colliding objects has a Rigidbody. This is an important rule to keep in mind when designing interactive objects.
   - _Example:_ If a player runs into a coin, either the player or the coin must have a Rigidbody attached for Unity to register the collision.- 

- **Triggers** are a special type of collider that do not physically block other objects _(no physic calculations)_. Instead, they detect overlaps with other colliders. Triggers are useful for areas, pickups, or zones where you want something to happen without stopping movement. 
   - _Example:_  If the player walks through an invisible area that triggers a checkpoint, the area’s collider can be a trigger. The player passes through it without stopping, but Unity still detects the overlap, so you can run logic like saving progress.
 
### Simple Physics vs. Scripted Reactions
As long as a GameObject has a **Rigidbody**, Unity's physics engine will affect it automatically. That means:

- A Rigidbody will **fall due to gravity**.
- If it **collides with another Rigidbody**, both objects will respond physically (bounce, slide, topple, etc.).
- A player with a Rigidbody **can push other Rigidbody objects** just by moving into them — _no code required_.

**But physics alone only handles movement and force.**

If you want _additional gameplay logic_, like playing a sound, subtracting health, collecting items, or triggering animations, then physics isn’t enough. You must define this behavior with either `OnCollision` or `OnTrigger` methods.

### Invisible Colliders and Boundaries

Sometimes, you may want to detect collisions or triggers with objects that don’t have a visible mesh. For example:

- **Invisible walls or boundaries:** Prevent the player from leaving a level or entering restricted areas.
- **Trigger zones:** Activate cutscenes, spawn enemies, or save progress when the player enters a specific area.

In these cases, you can attach a **Collider** to an **empty GameObject**. Even though the object has no visible mesh, its collider will still participate in collision and trigger detection. This allows you to control gameplay flow and interactions without cluttering the scene with visible objects.

**Example:** A player reaches the edge of a level. An invisible boundary collider detects the overlap and triggers a “Level Complete” event, without requiring any visible wall or barrier in the scene.

> [!TIP]
>  Mark the collider as a **Trigger** if you don’t want it to physically block movement, or leave it as a solid collider if you want the player to collide with it physically.

## Collision and Trigger Methods 
Unity provides built-in life-cycle methods that automatically fire when objects interact using **Colliders** and **Rigidbodies**. These methods belong inside scripts that define behavior, such as `Player`, `Enemy`, `Coin`, etc.

Unity provides two types of physics event methods: `OnCollision` and `OnTrigger`. Which one is called depends on whether the collider **has `IsTrigger**` enabled.

- `IsTrigger` OFF =  Physical collision calculated  = `OnCollision`
- `IsTrigger` ON =  Overlap detection only, no physics =  `OnTrigger`

### Event Timing 

Both collision and trigger events come in three forms:

| Event Type | Collision Method Name | Trigger Method Name | When It Fires                 | Typical Use Case                        |
| ---------- | --------------------- | ------------------- | ----------------------------- | --------------------------------------- |
| **Enter**  | `OnCollisionEnter`    | `OnTriggerEnter`    | When contact begins           | Pickups, hits, opening doors            |
| **Stay**   | `OnCollisionStay`     | `OnTriggerStay`     | Every frame while overlapping | Damage zones, conveyor belts            |
| **Exit**   | `OnCollisionExit`     | `OnTriggerExit`     | When objects separate         | Turning off effects, leaving safe zones |


### Method Parameters

When Unity calls one of these event methods, it automatically passes **information about the other object involved**.

| Method Signature Example                     | Parameter Type | What It Gives You                                                       |
| -------------------------------------------- | -------------- | ----------------------------------------------------------------------- |
| `void OnCollisionEnter(Collision collision)` | `Collision`    | Detailed collision info (contact points, impulse, and the other object) |
| `void OnTriggerEnter(Collider other)`        | `Collider`     | Only the other object’s collider (no physics force data)                |

Think of a collision or other as Unity handing you a report about what you just hit or entered.

> [!CAUTION]
> Do not confuse these parameters with the `Collider` components on your GameObjects.
> The parameters (`collision` or `other`) are **NOT references to visual components** that you add in the Inspector, like a BoxCollider or SphereCollider.
> Instead, they are instances of special `Collision` and `Collider` classes **automatically created by Unity**. They represent information about the other object involved in the interaction (e.g., its GameObject, contact points, etc.) and are populated by the physics engine when the event occurs.

### Example Implementation

Collision (Physical Contact)
```csharp
      void OnCollisionEnter(Collision collision)
      {
          // Do something on collision
      
      }//end OnCollisionEnter
```

Trigger (No Physical Pushback)
```csharp
      void OnTriggerEnter(Collider other)
      {
         // Do something when triggered

      }//end OnTriggerEnter

```

## Accessing the Other Object 
The parameter (`Collision collision` or `Collider other`) contains information about the other object involved in the interaction. From there, we get a reference to the object's `gameObject` via `collision.gameObject` (for collisions) or `other.gameObject` (for triggers).

This reference is extremely useful because it allows your script to inspect and interact with the other object. From here, you can:
- Check **basic properties** like its **name**, **tag**, or **layer**
- Access **components** attached to it, such as Rigidbody, Collider, or custom scripts
- Control its behavior, such as **enabling/disabling** it, applying forces, or triggering animations

In other words, the GameObject reference is your **entry point to understand and manipulate the object your player (or any interactive object) has collided or overlapped with**.

### Common Game Object Properties 

| Property            | Description                                                      | Example Usage                                                        |
| ------------------- | ---------------------------------------------------------------- | -------------------------------------------------------------------- |
| `name`              | The name of the GameObject in the hierarchy                      | `Debug.Log(otherObject.name);`                                       |
| `tag`               | The tag assigned to the GameObject (for grouping/identification) | `if (other.CompareTag("Coin")) { ... }`                              |
| `layer`             | The physics layer of the object (used in collision layers)       | `if (otherObject.layer == LayerMask.NameToLayer("Enemy")) { ... }`   |
| `activeSelf`        | Whether the GameObject is active in the scene                    | `if (!otherObject.activeSelf) return;`                               |
| `transform`         | The Transform component for position, rotation, scale            | `Vector3 dir = otherObject.transform.position - transform.position;` |
| `SetActive(bool)`   | Enable or disable the GameObject                                 | `otherObject.SetActive(false);`                                      |
| `GetComponent<T>()` | Access any component attached to the GameObject                  | `Rigidbody rb = otherObject.GetComponent<Rigidbody>();`              |

**Example: Get GameObject's Name**
```csharp
      void OnCollisionEnter(Collision collision)
      {
          // Get the GameObject we collided with
          GameObject otherObject = collision.gameObject;

          Debug.Log("Collided with: " + otherObject.name);
          Debug.Log("Tag: " + otherObject.tag);
          Debug.Log("Layer: " + otherObject.layer);

          // Check for a Rigidbody component
          Rigidbody otherRB = otherObject.GetComponent<Rigidbody>();

          //Move other's Rigidbody with force
          if (otherRB != null)
          {
              rb.AddForce(Vector3.up * 5f);

          }//end if (otherRB != null)

      }//end OnCollisionEnter

```

### GameObject Tags
A **tag** is a built-in Unity feature used for **grouping and identifying GameObjects**. Tags allow you to determine the type or role of an object (like `"Player"`, `"Enemy"`, or `"Coin"`) **without relying on its exact name or a specific component**. This makes tags more flexible and robust, especially when multiple objects share the same purpose in your game.

Tags are created and set in the inspector. These are just a **string data type**, and the typical use case is to **check the tag of a GameObject during a collision or trigger event** and perform an action based on it.

While you could write a comparison like:

```csharp
//❌ NOT RECOMMENDED

      void OnTriggerEnter(Collider other)
      {
            //Check the other gameObject tag
            if (other.gameObect.tag == "Coin")
            {
                AddScore(1);
                Destroy(otherObject);
      
            }//end if("Coin")
      
      }//end OnTriggerEnter()
```
However, if you try to directly compare `.tag` (e.g., `other.tag == "Coin"`) and the GameObject has **no tag** or `other` is unexpectedly `null`, Unity can throw a runtime error.  

Unity provides the `CompareTag()` method, which is **safer and more efficient** than directly comparing the `.tag` string. This method exists on the `Collider` parameter and **returns a `bool`** indicating whether the GameObject’s tag matches the value you provide.

> [!TIP]
> In recent versions of Unity, `CompareTag()` will warn you in the editor if the tag doesn’t exist in the Tag Manager.

Because `CompareTag()` automatically checks the GameObject associated with the `Collider`, and the collider is the `other` reference in an `OnTriggerEnter` method, you can call it directly on `other`.

```csharp
//✅Optimize Trigger Implmentation

      void OnTriggerEnter(Collider other)
      {
          // Check if it has the "Coin" tag
          if (other.CompareTag("Coin"))
          {
              AddScore(1);
              Destroy(otherObject);

          }//end if("Coin")

      }//end OnTriggerEnter()
```

When using `OnCollisionEnter`, the parameter `other` is a **Collision** object, not a GameObject or Collider. `Collision` represents the contact points and other collision information. **It does not directly have a `CompareTag()` method**, so you must check the tag through the `gameObject` involved:

```csharp
//✅Optimize Collision Implmentation

      void OnCollisionEnter(Collision other)
      {
          // Check if it has the "Coin" tag
          if (other.gameObject.CompareTag("Coin"))
          {
              AddScore(1);
              Destroy(otherObject);

          }//end if("Coin")

      }//end OnTriggerEnter()
```


>[!WARNING]
>**Always use `CompareTag()`** instead of comparing the **tag** string directly. It is faster and prevents potential errors if the tag is changed in the Inspector.

## Who Handles Collision Checks

The `OnCollision` and `OnTrigger` methods should reside on the **object that owns the majority of the behavior triggered** or which object is **primarily responsible for initiating the interaction**.

Here are some common scenarios:

---

### Falling Spikes - Example
Spike is a falling hazard that damages the player.
- Actions that happen:
  - Player loses health
  - Player plays sound
    
→ All of the effects are on the player, so the **player handles the collision**.

**Example code for Player Class**

```csharp
// Player.cs
void OnTriggerEnter(Collider other)
{
    if (other.CompareTag("Spikes"))
    {

      //Call the update health method
      UpdateHealth(-10);
      
      //Play Soundfx
      _audioSource.PlayOneShot(_hitClip, 2);
         
    }//end if("Spikes)

}//end OnTriggerEnter

```
**How it Works** 
- `CompareTag()` method checks if the `other` object is tagged with _"Spikes"_
- The `UpdateHealth(-10)` call is a method in the **Player class** that handles changing the player’s health.
   - The `-10` value is passed as a parameter to indicate damage.
- The `_audioSource` variable is a reference to the **AudioSource component** attached to the GameObject (_i.e. Player_). The `PlayOneShot(_hitClip, 2)` method on the AudioSource.
   - The method then plays the specified audio clip (`_hitClip`) at a volume level of `2`.

> [!NOTE]
> While the **tag check** typically occurs inside the `OnCollision` and `OnTrigger` methods, the actual tasks or game logic are usually handled by other methods within the same class or component.  This pattern keeps the `OnCollision` and `OnTrigger` methods **clean and focused on detecting the collision**, while delegating the actual behavior to **dedicated methods** that manage health, animations, sound, or other gameplay effects.

---
 
### Coin Collection - Example
The player collects coins from around an area. 
- Actions that happen:
  - Coin is destroyed
  - Player gains score
  - Player plays sound

→ Most of the effects are on the player, so the **player handles the collision**.

**Example code for Player Class**

```csharp
// Player.cs
void OnTriggerEnter(Collider other)
{

    if (other.CompareTag("Coin"))
   {
      //Call the AddScore method
      AddScore(1);

      //Play Soundfx
      _audioSource.PlayOneShot(_scoreClip, 2);

      //Destroy the Coin
      Destroy(other.gameObject);

    } //end if("Coin")

}//end OnTriggerEnter

```
Even though the coin is the object being removed, the **player owns most of the game logic** for collecting coins, such as updating the score and playing sounds. The coin itself only needs to handle its own removal. Since Unity automatically calls the `OnDestroy` method as part of its object lifecycle, the coin doesn’t require any special setup. This means the **Player class can simply call `Destroy(coin)`**, making the interaction straightforward and efficient.

---

**Example: Opening Door**
The door opens when the player touches it
- Actions that happen:
  - Door checks for key
  - Door animation plays
  - Door plays a sound 
  
→ The door is what changes, so the **door should handle the collision check**.

While all the main actions are on the door, though, we still need to query information from the other object. 

**Example code for Door Class**

```csharp
// Door.cs
void OnTriggerEnter(Collider other)
{
    // Check if the object is the player
    if (other.CompareTag("Player"))
    {
        // Get the Player component from the other object
        if (other.TryGetComponent<Player>(out Player player))
        {
            // Only open the door if the player has the key
            if (player.HasKey)
            {
                OpenDoor();

            }//end if (player.HasKey)

        }//end if (other.TryGetComponent<Player>(out Player player))

    }//end if("Player")

}//end OnTriggerEnter()


// Example Open Door method
void OpenDoor()
{
   //Play Soundfx                         
   _audioSource.PlayOneShot(_openClip); 

   //Play Animation
      //Animation stuff happens here....

}//end Open Door()

```
This structure emphasizes **delegation and data querying**. The door checks for collision and delegates actions to internal methods. It also queries other objects for necessary information.

---

### Explosive Barrel vs. Enemy - Example
When an enemy walks into an explosive barrel
- Actions that happen:
  - Enemy takes damage 
  - Enemy gets launched back
  - Enemy plays “hit” animation
  - Barrel spawns explosion VFX/SFX _(particle system)_
  - Barrel destroys itself
 
→ Both objects have meaningful self-contained effects. Which one owns the check? 

While each object could **check for collisions and control its own behavior**, performing multiple collision checks can become **expensive**, especially if there are many interactive objects in the scene.

We can **optimize** this by having the object that is **most likely to initiate the interaction** handle the collision check and then call the behaviors of the other object. 

**Rule of thumb:**
- Moving objects usually detect collisions against static objects.
- Example: Enemy walks into a stationary Barrel → the Enemy “causes” the interaction.

**Example of Enemy class:**
```csharp
// Enemy.cs
void OnCollisionEnter(Collision collision) 
{
    // Get the GameObject we collided with
    GameObject otherObject = collision.gameObject;

    if (otherObject.CompareTag("Barrel")) 
    {
        // Handle self-contained effects
        TakeDamage(50);

        //Get Knocked back
        _rigdBody.AddForce(-transform.forward * 10f, ForceMode.Impulse); 

        // Trigger behavior on the other object safely
        if (otherObject.TryGetComponent<Barrel>(out Barrel barrel))
        {
            barrel.Explode();
        }

    }//end if("Barrel")

} // end OnCollisionEnter

```
**How it Works**

- `collision.gameObject` gives a reference to the **other object involved in the collision** (the barrel).
- Check if the object has the "Barrel" tag.
- Local Enemy behaviors run:
   - Calls the `TakeDamage()` method.
   - Accesses the Rigidbody component via a variable (e.g., `_rigidBody`) and applies `AddForce` for knockback.
- Checks if the other object (the barrel) has a `Barrel` component:
   - If it does, call the `Explode()` method.
 
> [!IMPORTANT]
> In order for the Enemy to call `barrel.Explode()`, it needs to **know that this method exists** on the Barrel class. This highlights why **clear documentation and consistent method naming are so important** in game development. For example, having a naming convention for explosion methods (like always calling them `Explode`) makes your code easier to read, maintain, and integrate.

Another approach is to use an **interface** (e.g., `IExplodable`) that guarantees any object implementing it has an `Explode()` method. This makes interactions more flexible, but using interfaces is **beyond the scope of this lesson**.

---

### When Both Objects Have Collision Logic

While collision and trigger checks are usually handled by the object that owns the majority of the game logic or initiates the interaction, there are cases where the object being checked may also need its own collision logic for interactions with other objects.

Take our **Explosive Barrel** scenario above:
**Enemy** walks into an explosive **barrel**
- Actions that happen:
  - Enemy takes damage 
  - Enemy gets launched back
  - Enemy plays “hit” animation
  - Barrel spawns explosion VFX/SFX _(particle system)_
  - Barrel destroys itself

 Player tosses **explosive matter** onto **barrel**
 - Actions that happen:
   - Barrel increases in power
   - Barrel increases in scale
   - Explosive matter gets destroyed 
    
Because this interaction between the barrel and explosive matter mainly affects the barrel itself, the barrel now needs its own collision check logic to handle these new interactions, even though the enemy is already checking collision with the barrel. 

**Example of Barrel class:**
```csharp
      // Barrel.cs
      void OnTriggerEnter(Collider other) 
      {
      
          if (other.CompareTag("Explosive")) 
          {
              // Add to barrel's power
              IncreasePower(50);
      
              // Grow the barrel in size (doubling scale as an example)
              transform.localScale *= 2f;
      
              // Destroy the explosive object
              Destroy(otherObject);
      
          } // end if("Explosive")
      
      } // end OnCollisionEnter

```

Notice that the **barrel does not check for collisions with the enemy**; this is handled by the enemy itself. Also, because we don't need physics interactions with the explosive matter, we can set its collider as a **trigger** and use `OnTriggerEnter()` instead of a collision event.

## Checking for Multiple Collisions 
In all of the examples above, we have been checking for interaction with game objects with one specific tag. But what do we need to check for multiple interactions? Like our Player character may need to check for both _Spikes_ and _Coins_.  

A simple condition will work in this scenario: 

```csharp
      void OnTriggerEnter(Collider other)
      {
          if (other.CompareTag("Spikes"))
          {
              TakeDamage(10);  
              
          }
          else if (other.CompareTag("Coin"))
          {
              CollectCoin(other); 
      
          }//end Tag Check
      
      }//end OnTriggerEnter()
      
      // Handle taking damage
      private void TakeDamage(float damageAmount)
      {
          // Update player's health
          UpdateHealth(-damageAmount);
      
          // Play hit sound
          _audioSource.PlayOneShot(_hitClip, 2f);
      
      }//end TakeDamage()
      
      // Handle collecting a coin
      private void CollectCoin(Collider coinCollider)
      {
          // Update score
          AddScore(1);
      
          // Play coin sound
          _audioSource.PlayOneShot(_scoreClip, 2f);
      
          // Destroy the coin
          Destroy(coinCollider.gameObject);
      
      }//end CollectCoin

```
**How it Works** 

- `OnTriggerEnter()` checks the tag of the object the player collides with:
   - `other.CompareTag("Spikes")` → calls `TakeDamage(10)`
   - `other.CompareTag("Coin")` → calls `CollectCoin(other)`

- `TakeDamage(float damageAmount)`
   - Updates the player's health `(UpdateHealth(-damageAmount))`
   - Plays the hit sound (`_audioSource.PlayOneShot(_hitClip, 2f)`)

- `CollectCoin(Collider coinCollider)`
   - Updates the score `(AddScore(1))`
   - Plays the coin collection sound (`_audioSource.PlayOneShot(_scoreClip, 2f)`)
   - Destroys the coin (`Destroy(coinCollider.gameObject)`)

**Delegating** actions to separate methods keeps the `OnTriggerEnter()` method clean and modular, preventing it from becoming cluttered with multiple steps for different interactions. Each method can focus on a single responsibility: 
- `DamageTaken()` handles all aspects of taking damage
- `CoinCollected()` handles all aspects of collecting a coin.

Notice that we are also passing a positive damage value to `TakeDamage()`. It's best practice to **use positive values when possible**, making the code easier to read and understand. The method itself will then subtract the value from the player's health.

For coin collection, we need a reference to the **other object** (_i.e. coin's Collider_) because the method must destroy the specific GameObject that was collected.

This structure also makes the code more **maintainable and extensible**. For example, if you later want to add additional effects, such as a particle effect when taking damage, or a combo sound for coins, you can update the respective method without modifying the core collision detection logic. It also makes it easier to reuse these methods in other parts of the game, improving consistency and reducing bugs.

---

### Many more conditions 
While `if`/`else` statements work well for two or three checks, when you have more conditions, it’s better to use a `switch` statement. The switch can even be delegated to its own method, keeping the `OnTrigger` or `OnCollision` method clean.
```csharp
      void OnTriggerEnter(Collider other)
      {
          HandleTriggerByTag(other);

      }//end OnTriggerEnter

      
      private void HandleTriggerByTag(Collider other)
      {
          switch (other.tag)
          {
              case "Enemy":
                  Attack();
                  break;
      
              case "Spikes":
                  TakeDamage(10); // pass a value if needed
                  break;
      
              case "Coin":
                  CollectCoin(other); // pass the collider to destroy
                  break;
      
              case "PowerUp":
                  IncreaseSpeed();
                  break;
      
              case "Ground":
                  isGrounded = true;
                  break;
      
              default:
                  break;
          }
      }//end HandleTriggerByTag
```

While a `switch` works well for a handful of tag checks, if your game involves many interactions, this approach can become cumbersome and harder to maintain. In those cases, it’s better to optimize the logic for scalability and flexibility. Some common approaches include:
- **Dictionary of Actions:** Map each tag or object type to a corresponding action. This avoids large switch blocks while still centralizing behavior.
- **Event System:** Use events or messaging so the player can trigger reactions without hardcoding every tag check. This reduces coupling and makes it easier to extend functionality.
- **Scriptable Objects:** Create Scriptable Objects that define “interaction rules,” mapping object types to player actions. The player reads these rules at runtime and applies the appropriate effects.

### Checking Beyond Tag

So far, we’ve mostly used tags to identify objects during collisions or triggers. However, you can also check for specific components on the other object. For example, you could use `TryGetComponent<Rigidbody>(out var otherRB)` to see if the object has a Rigidbody.

You can also check for **custom classes** by adding them as components. For instance, if you have an `Enemy` class, you can test whether the object has an `Enemy` component. This approach works well with inheritance: if other objects, like `Spikes` or `Troll`, inherit from `Enemy`, the check will still succeed.

This method allows for more flexible and robust collision handling, especially when dealing with complex object hierarchies or multiple object types that share behaviors.

---

# Challenge: Build Your Own Collision and Trigger Handler

Using what we've learned about Colliders, Rigidbody, and Triggers, create a simple class that detects interactions with other objects.

## Instructions

1. Create a new class called `Player` and attach it to your player GameObject.
4. Create an empty GameObject in the scene with a Collider to act as a trigger (e.g., a coin, spike, or special zone).
5. Check the **Is Trigger** box on the Collider for trigger detection.
6. In the `Player` script:
   - Implement both `OnCollisionEnter()` and `OnTriggerEnter()` methods.
   - Use `Debug.Log()` to output a message when a collision or trigger is detected. Include the **name** or **tag** of the other object.
   - Optionally, experiment with `CompareTag()` to identify different objects.

## Example Debug Output

- `"Collided with Wall"`
- `"Triggered Coin pickup"`

## Bonus Challenge

- How could you differentiate between multiple types of objects using **tags** or **components**?
- Extend your debug messages to include the type of interaction (e.g., `"Player hit enemy"`, `"Player collected coin"`).
- Try combining collision and trigger detection in a single player class while keeping the code **modular**.



