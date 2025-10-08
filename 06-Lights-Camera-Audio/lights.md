# Light Behaviors in Unity

Lights in Unity are powerful tools to shape the mood, guide players, and highlight important areas in your scene. Using scripts, you can dynamically control lights, turn them on and off, adjust intensity, color, or range, based on gameplay events.

## Unity Light Types

Unity has several types of lights, each with specific purposes:
| Light Type                    | Description                                                 | Best Use Case                                 |
| ----------------------------- | ----------------------------------------------------------- | --------------------------------------------- |
| **Directional Light**         | Simulates a sun or moon. Affects the whole scene uniformly. | Outdoor scenes or sunlight.                   |
| **Point Light**               | Emits light in all directions from a single point.          | Lamps, torches, or small light sources.       |
| **Spot Light**                | Emits light in a cone shape.                                | Flashlights, streetlights, or stage lighting. |
| **Area Light** *(Baked only)* | Emits light from a rectangle or surface.                    | Soft, realistic lighting in interiors.        |

> [!TIP]
Lights can wash out each other. For example, a directional light acting as the sun will overpower smaller spot or point lights. Make sure the lighting hierarchy matches your scene goals.

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

# Creating a Light Behaviors
Sometimes in a game, you may want a light to turn on or off only when a player enters a specific area. For example, a torch in a dark hallway, a streetlamp that activates when the player approaches, or a warning light that flashes when a player enters a zone. We can achieve this easily using a trigger.

## 👉 Light Trigger

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

#### 5: Trigger Lights
1. Add methods to turn the light on when the player enters.
2. Add a method to turn off when the player leaves the trigger area.


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
        //Turn light off
        _light.enabled = false;

    }}//end if(Player)

}//end OnTriggerExit

```
#### 6: Test the Trigger
1. Have a player controller GameObject in the scene that is tagged as "Player".
2. Press Play and test moving the player into and out of the trigger area.
3. Ensure the light turns on when entering the trigger and turns off when exiting.

> [!WARNING]
> If the light doesn’t turn on, make sure the Box Collider is set to **Is Trigger**.

## 🎚️ Toggle Switch
Instead of using an invisible trigger that automatically turns a light on or off, you can create a **switch** that toggles a light manually. This could be a **physical object in the scene that the player interacts with**, which would require a **trigger collider** to detect the interaction, or it could be controlled by a **key/button press action**. The idea is simple: each time the method is called, the light changes to the opposite of its current state.

```csharp
  private void ToggleLight()
  {

   // Invert the current state of the light
   _light.enabled = !_light.enabled;

  }//end ToggleLight

```
---

# Other Light Behaviors

Once you know how to control lights with code, you can start using them creatively to bring your scenes to life. Lights can do far more than just turn on or off, they can flicker like a candle, pulse like a warning beacon, or even change color to set the mood of a scene.

Many of these behaviors rely on **math functions** to create smooth, repeating, or random variations over time. By using functions like `Mathf.Sin()` for rhythmic motion or `Mathf.PerlinNoise()` for organic randomness, we can animate lights in ways that feel natural, dynamic, and interactive.

In this section, we’ll explore several fun examples that show how simple scripts and math can add atmosphere, interactivity, and personality to your Unity projects. Each example demonstrates a different way to animate or respond to the world using the Light component.

## 💡 Ease In-Out Light
In the previous example, our Trigger simply turned the light on or off. But what if we wanted the light to slowly come on as the player stayed within the trigger? This can be done dynamically by chaining the **intensity** of the light. 

```csharp

public class TriggerEaseLight : MonoBehaviour
{
    [SerializeField]
    [Tooltip("Light to Toggle on/off")]
    private Light _light;

    [SerializeField]
    [Tooltip("Max intensity (brightness) of light")]
    private float _maxIntensity = 50f;

    [SerializeField]
    [Tooltip("Speed of the ease-in")]
    private float _easeInSpeed = 20f;

    [SerializeField]
    [Tooltip("Speed of the ease-out")]
    private float _easeOutSpeed = 40f;

    private void Awake()
    {
        // Start with the light turned off and intensity 0
        _light.enabled = false;
        _light.intensity = 0f;

    }//end Awake()

    private void OnTriggerEnter(Collider other)
    {
        if (other.CompareTag("Player"))
        {
            // Turn light on
            _light.enabled = true; 

        }//end if(Player)

    }//end OnTriggerEnter()


    private void OnTriggerStay(Collider other)
    {
        if (other.CompareTag("Player"))
        {
            // Gradually increase intensity while player stays
            _light.intensity = Mathf.MoveTowards(_light.intensity, _maxIntensity, _easeInSpeed * Time.deltaTime);

        }//end if(Player)

    }//end OnTriggerStay()

    private void OnTriggerExit(Collider other)
    {
        if (other.CompareTag("Player"))
        {
            // Gradually decrease intensity after leaving
            StartCoroutine(EaseOut());

        }//end if(Player)

    }//end OnTriggerExity()

    private IEnumerator EaseOut()
    {
        while (_light.intensity > 0f)
        {
            _light.intensity = Mathf.MoveTowards(_light.intensity, 0f, _easeOutSpeed * Time.deltaTime);
            yield return null;

        }//end while

        _light.enabled = false;

    }//end EaseOut()

}//end TriggerEaseLight

```

### Breakdown of `TriggerEaseLight`
The `_maxIntensity` value represents the brightest the light can reach. While the player remains inside the trigger, the light’s intensity gradually rises toward this maximum value.

The `_easeInSpeed` and `_easeOutSpeed` values control **how quickly the light brightens or dims**. Larger values produce faster transitions, while smaller values create slower, smoother changes.

The `OnTriggerStay()` method is **called continuously every frame as long as the player stays within the trigger**. This allows the light to gradually **ease in** using `Mathf.MoveTowards()`.

The `Mathf.MoveTowards()` method **moves a value at a constant speed toward a target**, ensuring it reaches the target without overshooting. It takes three parameters:  
- The **current value** of the light
- The **target value** (i.e, light maximum intensity)
- The **maximum amount the value can change per frame** (represented by `_easeInSpeed` and `_easeOutSpeed`)  

Multiplying this by `Time.deltaTime` makes the transition **frame-rate independent**.

Since `OnTriggerExit()` is **called only once when the player leaves the trigger**, a **coroutine** is used to create a gradual **ease-out**. This coroutine also uses `Mathf.MoveTowards()`, but sets the **target value to zero**. Once the intensity reaches zero, the light is turned off by setting `_light.enabled = false`.


---

## 🔥 Flickering Light

Simulates a torch, candle, or flickering bulb. The flicker speed and range can be adjusted in the Inspector to achieve different effects.

```csharp
public class FlickeringLight : MonoBehaviour
{
    private Light _light;
    private float _baseIntensity;
    
    [Header("Flicker Settings")]
    [SerializeField] 
    [Tooltip("How much intensity changes")]
    private float _flickerRange = 0.2f; 
   
    [SerializeField] 
    [Tooltip("How fast it flickers")]
    private float _flickerSpeed = 10f; 
 
    // Awake is called once on initialization
    private void Awake()
    {
        //Get the light component
        _light = GetComponent<Light>();
        
        _baseIntensity = _light.intensity;
        
    }//end Awake()
 
    // Update is called once per frame
    private void Update()
    {
        Flicker();
        
    }//end Update()


  //Flicker the light Intensity
  private void Flicker()
  {
        // Randomly change intensity over time
        float noise = Mathf.PerlinNoise(Time.time * _flickerSpeed, 0f) * 2 - 1;

        //Set light intensity
        _light.intensity = _baseIntensity + noise * _flickerRange;

  }//end Flicker() 

 
}//end FlickeringLight
```

### Breakdown of `Flicker()`
To create a flickering effect, the light’s intensity needs to increase and decrease over time. Using completely random values would make the effect look harsh and erratic, like the light is flashing instead of flickering. To achieve a smoother, more natural variation, we can use `Mathf.PerlinNoise()`, a Unity function that generates smooth pseudo-random values. Unlike pure randomness, Perlin noise produces gentle waves where values gradually rise and fall, resulting in a more organic motion often used for effects like fire, clouds, or terrain.

In our case, our first value is `Time.time`, the number of seconds since the game started, multiplied by a flickerSpeed value controls how quickly we move through the noise pattern. The second parameter (0f) simply represents the Y-coordinate of the noise function, which we keep constant since we only need one dimension. The result is a smoothly oscillating number between 0 and 1, perfect for adjusting the light’s intensity in a natural, flickering way.

The `Mathf.PerlinNoise()` method **returns a value between 0 and 1**. That’s fine for some uses, but it only gives us positive values,  meaning the flicker would only ever increase from a base intensity, not dip below it. By multiplying the result by 2 and then subtracting 1, we remap the range from **[0, 1]** to **[-1, 1]**. This symmetrical range lets the intensity fluctuate above and below a midpoint, creating a more balanced and natural variation.

The light’s intensity is then set to the `_baseIntensity`, the value of intensity defined on the Light component, plus the product of `noise * flickerAmount`, which adds or subtracts subtle variation from it.

### 🚨 Creating Plusing Light
The flicker light can easily be changed into a smooth **pulsing light**, like for damage, energy, or magical effects. Instead of randomly chaining intensity with `Mathf.PerlinNoise()`, we can set a perfectly smooth and periodic motion with `Mathf.Sin()`.

```csharp
  //Pulse the light Intensity
  private void Pluse()
  {
        // Smoothly oscillate the light’s intensity in a rhythmic pattern
        float wave = Mathf.Sin(Time.time * pulseSpeed);

        //Set light intensity
        _light.intensity = _baseIntensity + wave * _pluseRange;

  }//end Pluse() 

```
The `Mathf.Sin()` method produces a smooth, repeating wave between **-1 and 1**. Multiplying the wave by `_pulseRange` controls how much the light fluctuates around the base intensity. Adjusting `_pulseSpeed` changes how fast the pulsing occurs, making it faster or slower.

> [!NOTE]
> #### Understanding Unity’s Time Properties
> Unity provides several ways to track time, each useful in different situations:
> - `Time.timeScale` : a Unity property that controls the speed at which time progresses in the game. It essentially scales how fast everything that depends on `Time.time` or `Time.deltaTime` updates.
> - `Time.time` : The total number of seconds since the game started. It includes any time scaling (like slowing down the game with `Time.timeScale`). This is commonly used for continuous animations, like cycling light colors or oscillating intensity.
> - `Time.unscaledTime` : Similar to `Time.time`, but ignores `Time.timeScale`. Use this when you want something to animate or progress, even if the game is paused or slowed, such as UI effects or background lights.
> - `Time.deltaTime` : The time (in seconds) since the last frame. This is typically used to make motion or changes frame-rate independent, e.g., moving objects smoothly or interpolating values per frame.
> - `Time.unscaledDeltaTime` : Like `Time.deltaTime`, but ignores time scaling, useful for smooth motion in UI or effects during slow motion or pause.
>   
> _For most light animations, `Time.time` or `Time.unscaledTime` is preferred because you want a continuous, smooth progression rather than small per-frame changes._


---

## 🎨 Color Cycle Light

This effect smoothly rotates a light through a spectrum of colors, creating a dynamic or “party” atmosphere. By cycling the hue over time, you can make lights feel alive and vibrant, perfect for sci-fi scenes, dance floors, or magical effects.

```csharp
public class ColorCycleLight : MonoBehaviour
{
    private Light _light;
    
    [Header("Cycle Settings")]
    [SerializeField] 
    [Tooltip("Speed of the cycle")]
    private float _cycleSpeed = 1f; 
   

    // Awake is called once on initialization
    private void Awake()
    {
        //Get the light component
        _light = GetComponent<Light>();
        
    }//end Awake()
 
    // Update is called once per frame
    private void Update()
    {
        CycleColors();
        
    }//end Update()


    // Cycle the light color over time
    private void CycleColors()
    {
        // Generate a hue value that moves smoothly between 0 and 1
        float hue = Mathf.PingPong(Time.time * _cycleSpeed, 1f);

        // Convert HSV to RGB and apply it to the light color
        _light.color = Color.HSVToRGB(hue, 1f, 1f);
    }//end CycleColors() 

 
}//end CycleColors()
```

### Breakdown of `CycleColors` 
The `Mathf.PingPong()` method **“bounces” a value back and forth between 0 and the specified length**. Its output follows a triangle wave pattern, rising linearly to the peak and then falling back down. The first parameter should be a continuously increasing value, such as `Time.time` or `Time.unscaledTime`. Multiplying by `_cycleSpeed` controls how quickly the hue cycles.

The light’s color is then set by converting the hue value to an **RGB** color using `Color.HSVToRGB()`, which takes three parameters: **hue, saturation, and value**. In this example, saturation and value are both set to `1f` (100%), producing fully vivid and bright colors.

> [!CAUTION]
> Keep `_cycleSpeed` relatively small for a slow, smooth transition. Avoid values over 2, which can make the light change too abruptly and may cause discomfort or visual strain. You can even use a **range cap** in the Inspector to ensure the value stays within a suitable range:
> ```csharp
> [SerlizedField]
> [Range(0f,2f)]
> [ToolTip("Speed of cycle")]
> private float _cycleSpeed = 0.25f;
> ```

---
# 🎉 New Achievement: Lighting

These are just a few possibilities of how we can control lights in our scripts. These beahviors can be on the indvidual lights or on objects that control the lights.  
Lights are highly versatile, and with a little creativity, you can design entirely new behaviors, such as : 
- A warning beacon that responds to enemy proximity
- A magical glow that reacts to player actions
- A disco light that syncs with music
- A Day/Night cycle on the Directional Light

We’ve also explored several math functions that allow us to **create smooth, rhythmic, or procedural changes in intensity and color**, giving your lights personality and interactivity beyond simple on/off behavior.

--- 
<< Return to Lesson Contents | Continue to Camera tutorial >>

