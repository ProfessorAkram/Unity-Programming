# Basics of Colliders, Collisions, and Triggers

In Unity, collisions are the backbone of interaction in most games. They allow objects to detect each other, respond physically, and trigger events like collecting items, taking damage, or bouncing off walls.

## Coliders and Rigidbody Components 
In order for collisions to be detected, all objects will need a **Collider**. For collisions or triggers to register, at least one object of the interacting objects needs a **Rigidbody**.

- **Colliders** define the shape of a game object for physics interactions. Every game object that should participate in collision detection must have a collider. By default, primitive shapes like cubes or spheres automatically have colliders attached when created.
   - _Example:_ If a player runs into a coin, either the player or the coin must have a Rigidbody attached for Unity to register the collision.- 

- **Rigidbody** allows an object to participate in Unity’s physics simulation. A collision will only be detected if at least one of the colliding objects has a Rigidbody. This is an important rule to keep in mind when designing interactive objects.

- **Triggers** are a special type of collider that do not physically block other objects _(no physic calculations)_. Instead, they detect overlaps with other colliders. Triggers are useful for areas, pickups, or zones where you want something to happen without stopping movement. 
   - _Example:_  If the player walks through an invisible area that triggers a checkpoint, the area’s collider can be a trigger. The player passes through it without stopping, but Unity still detects the overlap so you can run logic like saving progress.


## Collision and Trigger Event Methods 
Unity provides built-in life-cycle methods that automatically fire when objects interact using **Colliders** and **Rigidbodies**. These methods belong inside scripts that define behavior, such as `Player`, `Enemy`, `Coin`,etc.

> [!NOTE]
> The object responsible for the behavior that occurs as a result of the collision should handle the check.

### Collisions vs Triggers
Unity provides two types of physics event methods: `OnCollision` and `OnTrigger` Which one is called depends on whether the collider **has `IsTrigger**` enabled.

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

Think of collision or other as Unity handing you a report about what you just hit or entered.

> [!CAUTION]
> Do not confuse these parameters with the `Collider` components on your GameObjects.
> The parameters (`collision` or `other`) are **NOT references to visual components** that you add in the Inspector, like a BoxCollider or SphereCollider.
> Instead, they are instances of special `Collision` and `Collider` classes **automatically created by Unity**. They represent information about the other object involved in the interaction (e.g., its GameObject, contact points, etc.) and are populated by the physics engine when the event occurs.

### Example Implmentation

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
The parameter (`Collision collision` or `Collider other`) contains information about the other object involved in the interaction. From there we get a reference to the object's `gameObject` via `collision.gameObject` (for collisions) or `other.gameObject` (for triggers).

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
      void OnTriggerEnter(Collider other)
      {
          // Get the GameObject we triggered
          GameObject otherObject = other.gameObject;

            if (otherObject.tag == "Coin")
            {
                AddScore(1);
                Destroy(otherObject);
      
            }//end if("Coin")
      
      }//end OnTriggerEnter()
```

Unity provides the `CompareTag()` method, which is **more efficient and safer** than directly comparing the `.tag` string. This method exists on the `Collider` (or `Collision`) parameter and **returns a bool** indicating whether the GameObject’s tag matches the value you provide.

Because `CompareTag()` automatically checks the GameObject associated with the Collider/Collision, there is **no need to explicitly reference** `other.gameObject`.

```csharp
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

>[!WARNING]
>**Always use `CompareTag()`** instead of comparing the **tag** string directly. It is faster and prevents potential errors if the tag is changed in the Inspector.
### 


## Who Handles Collision Checks

Which object should run the `OnCollision` and `OnTrigger` methods depends on on **which object owns the majority of the behavior triggered by the event** or which object is **primarily responsible for initiating the interaction**.

Here are some common scenarios:

---

**Example: Falling Spikes**
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
> [!NOTE]
> While the **tag check** typically occurs inside the `OnCollision` and `OnTrigger` methods, the actual tasks or game logic are usually handled by other methods within the same class or component.
> 
> In this example:
> - The `UpdateHealth(-10)` call is a method in the **Player class** that handles changing the player’s health.
>    - The `-10` value is passed as a parameter to indicate damage.
> - The `_audioSource` variable is a reference to the **AudioSource component** attached to the GameObject (_i.e. Player_). The `PlayOneShot(_hitClip, 2)` method on the AudioSource.
>    - The method then plays the specified audio clip (`_hitClip`) at a volume level of `2`.
>      
>  This pattern keeps the `OnCollision` and `OnTrigger` methods **clean and focused on detecting the collision**, while delegating the actual behavior to **dedicated methods** that manage health, animations, sound, or other gameplay effects.

---
 
**Example: Coin Collection**
Player collects coins from around an area. 
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

      //Destory the Coin
      Destroy(other.gameObject);

    } //end if("Coin")

}//end OnTriggerEnter

```
Even though the coin is the object being removed, the **player owns most of the game logic** for collecting coins: updating score, playing sounds, etc. Because of this, it’s simpler and more efficient for the player to handle the destruction of the coin.

---

**Example: Opening Door**
Door opens when player touches it
- Actions that happen:
  - Door checks for key
  - Door opens or closes
  - Door animation plays
  - Door plays sound 
  
→ The door is what changes, so the **door should handle the collision check**.

---
**Example: Explosive Barrel vs. Enemy**
When an enemy walks into an explosive barrel
- Actions that happen:
  - Enemy takes damage 
  - Enemy gets launched back
  - Enemy plays “hit” animation
  - Barrel spawns explosion VFX/SFX _(oarticle system)_
  - Barrel destroys itself
 
→ Both objects have meaningful self-contained effects, which one owns the check? 

While each object could **check for collisions and control its own behavior**, performing multiple collision checks can become **expensive**, especially if there are many interactive objects in the scene.

We can **optimize** this by having the object that is **most likely to initate the interaction** handel the collision check and then call the behaviors of the other object. 

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

    }//end if("Explosive")

} // end OnCollisionEnter

```
**How it Works**

- `collision.gameObject` gives a reference to the **other object involved in the collision** (the barrel).

- Check if the object has the "Barrel" tag.
- Local Enemy behaviors run:
   - Calls the `TakeDamage()` method.
   - Accesses the Rigidbody component via a variable (e.g., `_rigidBody`) and applies `AddForce` for knockback.
- Checks if the other object (the barrel) has a `Barrel` component:
   - If it does, calls the `Explode()` method.
 
> [!IMPORTANT]
> In order for the Enemy to call `barrel.Explode()`, it needs to **know that this method exists** on the Barrel class. This highlights why **clear documentation and consistent method naming are so important** in game development. For example, having a naming convention for explosion methods (like always calling them `Explode`) makes your code easier to read, maintain, and integrate.

Another approach is to use an **interface** (e.g., `IExplodable`) that guarantees any object implementing it has an `Explode()` method. This makes interactions more flexible, but using interfaces is **beyond the scope of this lesson**.

---

