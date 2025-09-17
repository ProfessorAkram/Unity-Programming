# Unity Physcis Movment

Previously, we created a [`MoveTransform Class`](unity-basic-movement.md) class that allowed for basic movement of a GameObject using transform.position. While functional, there’s one drawback: it doesn’t make use of **Unity’s physics system**. This means:

- The object does not naturally respond to **collisions**
- It won’t be affected by **gravity**, **friction**, or other **forces**
- It can pass straight through walls and other objects

In other words, while `MoveTransform` gives us direct control, it doesn’t make the object part of the physical world.

If we want our GameObjects to interact realistically, with walls, the ground, or other moving bodies, we need to move them using **`Rigidbody` physics**, not just by setting their  `Transform`.

---
## Introducing Rigidbody

A `Rigidbody` component allows a GameObject to be affected by Unity’s physics properties, such as **gravity**, **mass**, **drag**, and **collisions**. Once a `Rigidbody` is added, the object becomes part of the physics simulation and behaves **more realistically**.

The key difference is that movement should now **happen through the `Rigidbody`**, not by changing `transform.position` directly. If you continue to move an object by setting its `Transform`, you’ll bypass physics again, resulting in unrealistic behavior and broken collision detection.

> **🗡️ Quest: Adding a Rigidbody**
>
> Test how game objects react when using physcis.
> 
> 1. Create a simple **Cube** in your scene.
> 2. Add a `Rigidbody` component from the Inspector.
> 3. Press **Play**; notice how gravity immediately pulls the Cube downward.
> 4. Place the **Cube** on top of another object (like a **Plane** or **Quad**) and watch it rest naturally on the surface instead of falling through.

### Types of Physics Objects

Now that we've added a `Rigidbody` and seen how it interacts with **gravity** and **collisions**, it’s important to understand the different **types of physics** objects in Unity. Each type behaves differently in the physics simulation:

**1. Static Objects**

Objects that never move should be set as `Static` in the **Inspector**. These objects **do not have a `Rigidbody`**. While they don’t move on their own, they can **still collide with dynamic objects**.
- **Use case:** Floors, walls, platforms—anything that stays in place but should block or support other objects.

**2. Kinematic Objects**
Objects whoes movement is controlled entirely through scripts, usually via `trasnform.position`, and **ignore physic forces** like gravity, but still need to collide with other objects. 
  - **Use case:** Moving platforms, doors, or any object that must be animated or moved by code while still interacting with dynamic objects.

> [!NOTE]
>
> If you only moved it with `transform.position` and no `Rigidbody`, fast-moving objects might clip through these objects. For examplea moving platform in a platformer where the player can land on it, and the platform moves via `trasnform.position`. Since the platform still needs interact with the player ( and possibly other objects), they still require a `Rigidbody` with `Is Kinematic` checked. This ensures **collisions** work correctly **without applying physics forces** to the object itself.

**3. Dynamic Objects**
 Objects that are fully affected by physcis (e.g. gravity, forces, drag, and collisions). 
  - **Use case:** Characters, balls, projectiles—anything that moves naturally in response to forces or collisions.

Choosing the right type is important. Use `Static` for immovable objects, `Kinematic` when you need scripted motion without full physics control and `Dynamic` for physics-driven objects.

By understanding these three types, you’ll know when to use `Rigidbody`, how your objects will interact, and how to design movement and collisions in your scene.

### Accessing Rigidbody
To move a GameObject with physics, you need to work with its `Rigidbody` component. Unlike the `Transform` (which every GameObject has by default), not all GameObjects automatically include a `Rigidbody`, you have to **add it yourself** in the Inspector.

That means you can't just call it directly like we do with:

`transform.position` 

Instead we have to make a **reference** to store the `Rigidbody` attached to this object:
```csharp
    // Reference to the object's Rigidbody component
    private Rigidbody _rigidBody;
```
Then we _set_ this variable to the actual `Rigidbody` on the GameObject:

```csharp
    //Set reference to the Rigidbody component
    _rigidBody = GetComponent<Rigidbody>(); 
```

#### Using `GetComponet<T>()`
Unity uses a **component-based system**, where each GameObject is made up of components (Transform, Rigidbody, Collider, etc.).

To access any of these components via script, you use the `GetComponent<T>()` method, which essentially tells Unity to **find the component of type _T_ attached to this GameObject**.

The type isn't limited to built-in Unity components, you can also use it for any custom component (i.e., class) you create and attach to a GameObject.

Once you store the reference in a variable, you can use it to **access public properties** (like velocity, mass, or drag) or **call methods** on that component.

#### What if No Component is Found?

If `GetComponent<T>()` doesn’t find the requested component on the GameObject, it returns **null**.

This means any logic that tries to access the component (e.g., `_rigidBody.velocity = ...`) will throw a `NullReferenceException`.

To prevent this, it’s good practice to **check for null** before using the reference:

```csharp
if (_rigidBody == null)
{
    return; // Exit the method if the Rigidbody is missing
}
else
{
    _rigidBody.velocity = Speed * Direction;
} 
```

If the **component is essential for the class to work**, you can enforce its presence using Unity’s `[RequireComponent]` attribute. This ensures that the **component is always added** to the GameObject, so the reference will never be `null`:
```csharp
    [RequireComponent(typeof(Rigidbody))]
    public class MoveRigidbody : MonoBehaviour
    {
        // Your class logic here
    }
```

Using `[RequireComponent]` helps avoid null reference errors and ensures your class always has the components it needs to function properly.


## Physics Movement 

By default, if two objects with physics are placed above each other in the scene, gravity will immediately take effect, and they will fall naturally. But Unity’s physics system does more than just apply **gravity**, it also handles **collisions, momentum, and other forces**:

- **Collisions:** When two Rigidbody objects collide, Unity calculates how they should bounce, slide, or stop based on their mass, velocity, and drag. For example, a heavy ball hitting a lighter cube will push the cube more than the other way around.

- **Momentum:** Moving objects carry momentum, which affects how they interact during collisions. A fast-moving object will have a stronger impact than a slow one.

- **Forces and Drag:** Objects can be affected by forces like wind, explosions, or friction. Drag (air resistance) slows them down naturally over time.

- **Continuous Interaction:** Physics objects automatically respond to each other and the environment. For instance, stacking boxes will cause them to settle realistically, topple over, or slide if one is pushed.

Once a Rigidbody is added, Unity simulates all of these physics behaviors automatically, based on the setting applied in the Rigidbody. This allows for realistic interactions between objects without you having to manually calculate collisions or momentum.

However, if you want **more control over how your GameObjects move**, Unity provides two main approaches using physics: **velocity-based movement** and **force-based movement**.

### Velocity Movement 

**Velocity** describes how fast and in which direction an object moves; essentially displacement over time, measured in meters per second **(m/s)**.
Its basic equation is:

$$
v = \frac{\Delta x}{\Delta t}
$$

Where:  

- **\( v \)** = velocity (vector, includes direction)  
- **\( \Delta x \)** = change in position (displacement)  
- **\( \Delta t \)** = change in time  

**For Example**

If a cube moves **5 meters to the right in 2 seconds**, its velocity is:

$$
v = \frac{5 \, \text{m}}{2 \, \text{s}} = 2.5 \, \text{m/s (to the right)}
$$

> [!NOTE]
> 
> In physics, **speed** is just **how fast an object moves**, while **velocity** includes both **speed and direction**.
> Unity’s `Rigidbody.velocity` follows this definition, as it is a `Vector3`. Storing both **how fast and which way the object is moving**.







Force-based movement

rb.AddForce(direction * forceAmount);


Feels more “realistic.”

Acceleration, momentum, drag apply naturally.

Good for cars, projectiles, floating objects.





