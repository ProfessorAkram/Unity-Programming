# Light Behaviors in Unity

Lights in Unity are powerful tools to shape the mood, guide players, and highlight important areas in your scene. Using scripts, you can dynamically control lights—turn them on and off, adjust intensity, color, or range—based on gameplay events.

## Unity Light Types

Unity has several types of lights, each with specific purposes:
| Light Type                    | Description                                                 | Best Use Case                                 |
| ----------------------------- | ----------------------------------------------------------- | --------------------------------------------- |
| **Directional Light**         | Simulates a sun or moon. Affects the whole scene uniformly. | Outdoor scenes or sunlight.                   |
| **Point Light**               | Emits light in all directions from a single point.          | Lamps, torches, or small light sources.       |
| **Spot Light**                | Emits light in a cone shape.                                | Flashlights, streetlights, or stage lighting. |
| **Area Light** *(Baked only)* | Emits light from a rectangle or surface.                    | Soft, realistic lighting in interiors.        |

> [!TIP]
> Lights can wash each other out. For example, a directional light acting as the sun will overpower smaller spot or point lights. Make sure the lighting hierarchy matches your scene goals.

## Controlling lights 
To modify a light in code, you need a reference to the Light component on a GameObject. How you access a light’s properties depends on where your behavior script is located. If the script is attached directly to the GameObject that has the Light component, you can simply call `GetComponent<Light>()` to get a reference. For example, if we wanted to create a day-night cycle behavior, the script would be attached to the **directional light** itself.

**Behavior on Light**
```csharp
    //Reference to Light component
    private Light _sunLight;

    private void Awake()
    {
        // Get the Light component on this GameObject
        _sunLight = GetComponent<Light>();
    
    }//end Awake()
```

On the other hand, if the behavior lives on a separate object—like a trigger or manager—you can expose a `[SerializeField]` in the inspector and assign the specific Light you want to control.

**Behavior on Trigger**
```csharp
    [SerializeField] 
    [Tooltip("Light to Toggle on/off")]
    private Light _light;

```

### Light Properties You Can Control via Script 

Unity’s Light component exposes many properties you can modify in code:

| Property          | Description                                 |
| ----------------- | ------------------------------------------- |
| `enabled`         | Turns the light on or off.                  |
| `intensity`       | Controls brightness.                        |
| `color`           | Changes the color of the light.             |
| `range`           | Affects how far point or spot lights reach. |
| `spotAngle`       | Adjusts the angle of a spot light cone.     |
| `shadows`         | Enable or disable shadows.                  |
| `bounceIntensity` | Controls indirect lighting intensity.       |

---

# Creating a Light Trigger
Sometimes in a game, you may want a light to turn on or off only when a player enters a specific area. For example, a torch in a dark hallway, a streetlamp that activates when the player approaches, or a warning light that flashes when a player enters a zone. We can achieve this easily using a trigger.

#### 1: Add a Point Light
1. In your scene, create a **Point Light**.
2. Position it where you want the light effect to occur.


#### 2: Create the Trigger Area
1. Create an **Empty GameObject** in the scene and name it something like **LightTrigger**.
2. Add a `Box Collider` component to this object.
3. Check **Is Trigger** on the collider so it doesn’t physically block the player.
4. Resize the Box Collider to cover the area where you want the light to toggle.

#### 3: Create the LightTrigger Script
1. In your project, create a new C# script called `LightTrigger`.
2. Attach the `LightTrigger` script to the **LightTrigger** GameObject.
3. Open the script and start by creating a **reference to the light** you want to control:

**Behavior on Trigger**
```csharp
using UnityEngine;

public class LightTrigger : MonoBehaviour
{
    [SerializeField] 
    [Tooltip("Light to Toggle on/off")]
    private Light _light;

}//end LightTrigger    

```

#### 4. Disable the Light on Awake
1. Set the light to be off when the scene begins: 
```csharp

private void Awake()
{
    // Start with the light turned off
    _light.enabled = false; 

}//end Awaked()


```

#### 5: Toggle the Light on Trigger Events
1. Add methods to turn the light on when the player enters.
2. Add a method to turn off when the player leaves the trigger are.


```csharp

private void OnTriggerEnter(Collider other)
{
    if (other.CompareTag("Player"))
    {
        //Trun light on
        _light.enabled = true;

    }//end if(Player)

}//end OnTriggerEnter


private void OnTriggerExit(Collider other)
{
    if (other.CompareTag("Player"))
    {
        //Trun light off
        _light.enabled = false;

    }}//end if(Player)

}//end OnTriggerExit

```
#### 6: Test the Trigger
1. Have a player controller GameObject in the scene and is tagged as **"Player"**.
2. Press **Play** an test moving the player into and out of the trigger area.
3. Ensure the light turn on when entering the trigger and turn off when exiting.

> [!WARNING]
> If the light doesn’t turn on, make sure the Box Collider is set to **Is Trigger**.
