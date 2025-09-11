## Basic Movement in Unity 
When designing games, one of the most common tasks is moving an object from one place to another. For example, imagine you want a projectile, like a fireball, to travel across the screen from point A to point B. How can we make this happen in Unity?

The answer lies in one of the most important components every GameObject has: the **Transform** component.

## The Transform Component

Every GameObject in Unity automatically has a **Transform**. You can think of it as the object's "address" in the game world, it tells Unity where the object is, how it'’'s rotated, and how large it is.

The Transform has three main properties:

- **Position** - where the object is located.  
- **Rotation** - the object's orientation (how it's turned).  
- **Scale** - the size of the object.  

In a 3D Unity project, the position is represented by an **(X, Y, Z)** vector:

- **X** - Left and right  
- **Y** - Up and down  
- **Z** - Forward and backward  

For example:
```csharp
transform.position += Vector3.right;
```