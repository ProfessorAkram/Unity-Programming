# Rotating Objects in Unity

When designing games, movement is only half the story. Many objects also need to turn, spin, or face a direction—whether it's a character looking around, a turret aiming at a target, or a door opening.

So how do we control rotation in Unity?

Once again, the answer lies in the same essential component: the **Transform**.

## Understanding Quaternions and Euler Angles

While we usually think of rotation using **Euler angles (X, Y, Z degrees)**, Unity actually stores rotations as Quaternions. A **Quaternion uses four values (x, y, z, w)** instead of three. Unity uses quaternion because they prevent rotation issues like **gimbal lock** and **make smooth rotation easier**.

Even though the Transform stores rotation as a Quaternion, you can still work with rotations using Euler angles, which are much easier to understand.

**Euler angles** break rotation down into three axes:

- **X** – Pitch (tilting up and down)
- **Y** – Yaw (turning left and right)
- **Z** – Roll (spinning sideways)

## Rotating with Transform.Rotate
Unity provides a method for rotating an objec with **Euler angales**, called `Transform.Rotate`. 

```csharp
transform.Rotate(0f, 50f * Time.deltaTime, 0f);
```
In the example above we are the method will rotate the game object along the Y-axis. 
This axis is also multipled by **Time.deltaTime** to ensure a consistent, gradual and smooth rotation. 

