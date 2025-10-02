# Basics of Colliders, Collisions, and Triggers

In Unity, collisions are the backbone of interaction in most games. They allow objects to detect each other, respond physically, and trigger events like collecting items, taking damage, or bouncing off walls.

## Coliders and Rigidbody Components 
In order for collisions to be detected, all objects will need a **Collider**. For collisions or triggers to register, at least one object of the interacting objects needs a **Rigidbody**.

- **Colliders** define the shape of a game object for physics interactions. Every game object that should participate in collision detection must have a collider. By default, primitive shapes like cubes or spheres automatically have colliders attached when created.
   - _Example:_ If a player runs into a coin, either the player or the coin must have a Rigidbody attached for Unity to register the collision.- 

- **Rigidbody** allows an object to participate in Unity’s physics simulation. A collision will only be detected if at least one of the colliding objects has a Rigidbody. This is an important rule to keep in mind when designing interactive objects.

- **Triggers** are a special type of collider that do not physically block other objects _(no physic calculations)_. Instead, they detect overlaps with other colliders. Triggers are useful for areas, pickups, or zones where you want something to happen without stopping movement. 
   - _Example:_  If the player walks through an invisible area that triggers a checkpoint, the area’s collider can be a trigger. The player passes through it without stopping, but Unity still detects the overlap so you can run logic like saving progress.


## Checking for Collisions and Triggers 
Unity provides built-in life-cycle methods that automatically fire when objects interact using **Colliders** and **Rigidbodies**. These methods belong inside scripts that define behavior, such as `Player`, `Enemy`, `Coin`,etc.

> [!NOTE]
> The object responsible for the behavior that occurs as a result of the collision should handle the check.

### Events and Paramters

Unity gives us two sets of event methods `OnCollision` and `OnTrigger`. If the object's collider **has `IsTrigger` enabled** then the `OnTrigger` method is used. 

Both `OnCollision` and `OnTrigger` have three different event checks

| Event Type | When It Fires                 | Use Case                                |
| ---------- | ----------------------------- | --------------------------------------- |
| `Enter`    | First moment contact happens  | Pickups, hit reactions                  |
| `Stay`     | Every frame while overlapping | Damaging zones, pressure plates         |
| `Exit`     | When objects separate         | Leaving an area or shutting off effects |

Both `OnCollision` and `OnTrigger` pass a paramter for the colider of the object that triggered the event. 


### Who Should Handle Collision Checks

Unity provides life-cycle events like **OnCollisionEnter** or **OnTriggerEnter** that are automatically triggered by the engine. Not all objects in a game should be responsible for checking these event, only the object that owns the bulk of the behaviours or game logic, should run the check. Below are a few examples: 

**Example: Falling Spikes**
Spike is a falling hazard that damages the player.
- Actions that happen:
  - Player loses health
  - Player Injured aniimation plays
  - Player plays sound
    
→ All of the effects are on the player, so the **player handles the collision**.

---
 
**Example: Coin Collection**
Player collects coins from around an area. 
- Actions that happen:
  - Coin is destroyed
  - Player gains score
  - Player plays sound

→ Most of the effects are on the player, so the **player handles the collision**.

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
 
→ Both objects have meaningful self-contained effects, so **both should have collision logic**.

**Example of Enemy class:**
```csharp
    // Enemy.cs
    void OnCollisionEnter(Collision collision) {
        if (collision.gameObject.CompareTag("Explosive")) {
            TakeDamage(50);
        }
    }//end OnCollisionEnter()
```

**Example of Barrel class:**
```csharp
    
    // ExplosiveBarrel.cs
    void OnCollisionEnter(Collision collision) {
        if (collision.gameObject.CompareTag("Enemy")) {
            Explode();
        }
    }//end OnCollisionEnter()
```

---

## Unity Life-Cycle Events
