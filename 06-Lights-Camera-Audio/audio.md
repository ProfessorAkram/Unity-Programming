# Audio Behaviors in Unity
Audio in Unity is **essential for creating atmosphere, providing feedback, and guiding player attention**. Using scripts, you can dynamically control audio, play or stop clips, adjust volume, pitch, or spatial settings, based on gameplay events.

## Unity Audio Components

Unity has several audio components, each with specific purposes:

| Component          | Description                                                                            | 
| ------------------ | -------------------------------------------------------------------------------------- |
| **Audio Source**   | Plays an audio clip in the scene. Can be 2D or 3D (spatialized).                       | 
| **Audio Listener** | Captures audio in the scene. Usually attached to the main camera.                      | 
| **Audio Mixer**    | Allows grouping of audio sources for shared control over volume, effects, and routing. | 

> [!WARNING]
> Always have **one Audio Listener** in the scene. Multiple listeners can cause strange audio behavior.
> By default the **Main Camera** has an **Audio Listener** component.

## Controlling Audio

To modify audio in code, you need a reference to the AudioSource component.

```csharp

    private AudioSource _audioSource;

    private void Awake()
    {
        // Get the AudioSource component on this GameObject
        _audioSource = GetComponent<AudioSource>();

    }//end Awake()

```

### Audio Properties You Can Control via Script
The Audio Source component has several properties that can be accessed via script. 

| Property        | Description                                   |
| --------------- | --------------------------------------------- |
| `clip`          | Assigns the audio clip to play.               |
| `volume`        | Controls loudness (0–1).                      |
| `pitch`         | Changes playback speed and tone.              |
| `loop`          | Determines if the clip repeats automatically. |
| `spatialBlend`  | Controls how the volume and direction of the sound are perceived <br>• **2D** = 0 (volume the same everywhere) <br>• **3D** = 1 (volume changes with distance and direction). |
| `Play()`        | Starts playback on AudioSource.                              |
| `PlayOneShot()` | Plays a clip once without stopping others on the same source. |
| `Stop()`        | Stops all sounds on this AudioSource, immediately.                   |

> [!NOTE]
> 3D sounds depend on both direction and distance to the **Audio Listener**. By default, the Audio Listener is on the **main camera**. If the Audio Listener is stationary while the player moves, 3D sounds will not update correctly, breaking the sense of spatial audio. To fix this, either move the camera with the player or place the Audio Listener on the player GameObject.


## Audio Clips
Unity supports several types of audio clips that serve different purposes in your game. Understanding these helps you choose the right type for each sound effect or music track.
| Audio Type      | Description                                                             | Best Use Case                                                |
| --------------- | ----------------------------------------------------------------------- | ------------------------------------------------------------ |
| **.WAV**        | Uncompressed audio with high quality and larger file size.              | Short, high-quality sound effects like footsteps or impacts. |
| **.MP3**        | Compressed format with smaller file size and minor quality loss.        | Background music or long ambient tracks.                     |
| **.OGG**        | Compressed open-source format with good balance of quality and size.    | Music or ambient loops (recommended over MP3 for looping).   |
| **.AIFF**       | Uncompressed format, similar to WAV but less common on Windows systems. | Cross-platform projects or sound libraries from macOS.       |

### Audio Clip Properties
Audio Clips have several properties that can be adjusted in the **Inspector**. Once the clip has been imported to the **Assets** folder, select it in the **Project window** to view its import settings. These control how Unity loads, processes, and plays the audio in your game.

| Setting                | Description                                                                            | Best Use Case                                    |
| ---------------------- | -------------------------------------------------------------------------------------- | ------------------------------------------------ |
| **Force To Mono**      | Combines stereo or multi-channel audio into a single mono track.                       | Use when stereo isn’t needed to save memory.     |
| **Normalize**          | Balances the volume level when converting to mono so it doesn’t get too quiet or loud. | Keep audio volume consistent after forcing mono. |
| **Load In Background** | Loads audio in the background instead of blocking the game while it loads.             | Large audio files or games with many sounds.     |
| **Ambisonic**          | Enables 3D “surround” audio that reacts to the listener’s rotation.                    | 360° videos, VR, or AR experiences.              |

#### Audio Loading and Compression Settings

In addition to the properties above, you can control how Unity loads and plays your audio clips. You can set one overall default configuration or adjust these settings for different platform builds. Most of the time, the defaults work fine, but it’s helpful to understand what they do.

| Property                | What It Does                                                                                                                                                                                                                                                                                  | When to Use                                            |
| ----------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------ |
| **Load Type**           | Controls how Unity loads the sound at runtime. <br>• **Decompress On Load** – Best for short sounds; uses more memory. <br>• **Compressed In Memory** – Keeps the file small but slightly increases CPU use. <br>• **Streaming** – Reads from disk while playing; best for long music tracks. | Short SFX → *Decompress* <br> Long music → *Streaming* |
| **Compression Format**  | Sets how audio is stored: <br>• **PCM** – Uncompressed, highest quality, large size. <br>• **ADPCM** – Smaller, good for repeated effects like footsteps. <br>• **Vorbis/MP3** – Smallest size, good for music.                                                                               | PCM for short, ADPCM for SFX, Vorbis/MP3 for music.    |
| **Sample Rate Setting** | Controls sound quality and file size. Usually, leave it on **Preserve** or **Optimize**.                                                                                                                                                                                                       | Leave default unless you need smaller files.           |
| **Preload Audio Data**  | Loads audio as soon as the scene starts.                                                                                                                                                                                                                                                      | Leave on for quick playback.                           |
| **Load In Background**  | Loads audio clips asynchronously while the game runs to prevent stuttering during load.                                                                                                                                                                                                       | Useful for large audio files.                          |
| **Quality**             | Adjusts compression level for smaller files vs. better sound.                                                                                                                                                                                                                                 | Higher = better sound, larger file.                    |



---
# Playing Audio
Audio is one of the most powerful ways to create a game’s atmosphere. Continuous sounds, such as background music or environmental effects (wind, birds, etc.), help establish the tone and mood of the game world. 

Many tutorials attach these continuous sounds to the main camera; this can blur responsibilities and make the camera manage things it shouldn’t. Because these sounds are not tied to a specific game object, it’s best to place them on a dedicated GameObject. Using a single object keeps audio separate from gameplay and camera logic, making your system easier to manage.

#### 1: Create the GameObject
1. In the Hierarchy, create an **Empty GameObject**.
2. Rename it **EnvironmentAudio** or **Ambience**.

#### 2: Add an AudioSource
1. Select the GameObject.
2. In the **Inspector window**, Choose `Add Component` > `Audio` > `Audio Source`.
3. Assign the desired audio clip (background music, wind, or other environmental sound).

#### 3: Configure the AudioSource
1. Apply the following settings to the audio source
    - **Play On Awake:** Enabled, so audio starts automatically.
    - **Loop:** Enabled for continuous playback.
    - **Volume:** Adjust to a low level to avoid overpowering other sounds.
    - **Spatial Blend:** Usually set to 2D for background sounds, but 3D if you want the audio to appear directional (e.g., a river or waterfall).

---

# Controlling Audio from Script
Playing looping audio, such as background music, is easy to set up directly in the editor and typically does not require any additional scripts. However, if you want to control audio dynamically, such as pausing, stopping, or changing clips, you need to access the **AudioSource via a script**.

#### Who owns the sound?
AudioSources and their clips should generally be attached to the object that “owns” the sound:
- **Player footsteps** → attached to the Player GameObject.
- **Coin collection sound** → attached to the coin.

If multiple objects can trigger the same sound, think carefully about ownership. Only move the AudioSource to a different object if it logically makes sense for your game (i.e., the sound should continue or exist independently of the original object). For example, if the same clip is played when the Player collects coins and power-ups, the behavior for playing the clip should be in the Player class.

#### Destroying Game Objects
**When a GameObject is destroyed, its AudioSource is also destroyed**. This means if a sound is played in the object's `OnDestroy()` method, the clip may be cut off before finishing. The best way to handle this is to **call a custom method to play the clip**, then destroy the object after the sound finishes.

For example: A barrel explodes when hit by an enemy. The explosion sound should be attached to the barrel because it describes the barrel’s destruction. The enemy should not directly play the sound; instead, it calls `barrel.Explode()`, which plays the clip and then destroys the barrel.

---

# Triggering Audio Playback 
A common dynamic sound in games is one that plays in response to a player interaction, such as picking up an item or entering a specific area. These sounds are usually triggered using the `OnTriggerEnter()` method in a script.

#### 1: Create an Audio Source
1. Add an `AudioSource` to a game object.
2. Assign an **audio clip**.
3. Make sure **Play On Awake** is **unchecked**, so the sound only plays when triggered.
4. Adjust settings (volume, loop, 3D spatial blend).

#### 2. Create the Trigger Area
1. Add a Box Collider to the same game object.
2. Check **Is Trigger** and resize the collider to the target area.

#### 3. Create the Script
1. Create a new C# script for the game object.
    - For example: Coin, Enemy, Player 
2. Reference the AudioSource

```csharp

    private AudioSource _audioSource;

    private void Awake()
    {
        // Get the AudioSource component on this GameObject
        _audioSource = GetComponent<AudioSource>();

    }//end Awake()

```
#### 4. Play/Stop Audio on Trigger
1. Create an `OnTriggerEnter()` method to `Play()` the audio
2. Create an `OnTriggerExit()` method to `Stop()` the audio

```csharp
private void OnTriggerEnter(Collider other)
{
    if (other.CompareTag("Player"))
    {
        _audioSource.Play();

    }//end if(Player)

}//end OnTriggerEnter()

private void OnTriggerExit(Collider other)
{
    if (other.CompareTag("Player"))
    {
        _audioSource.Stop();

    }//end if(Player)

}//end OnTriggerExit
```
#### 5. Test the Trigger
Have a player controller GameObject in the scene that is tagged as "Player".
Press Play and test moving the player into and out of the trigger area.
Ensure the light turns on when entering the trigger and turns off when exiting.

> [!WARNING]
> If audio doesn’t play, check that **Is Trigger** is enabled and the AudioSource is assigned.
> Also, make sure that the **audio toggle** in **Game window** is turned on in the Unity editor.

---

## Playing a Custom Clip
In the previous example, we assigned an audio clip directly to the AudioSource in the Inspector. But what if a GameObject needs multiple sounds? For example, a Player might have one sound for powering up and another for losing a life.

Instead of changing the AudioSource in the Inspector each time, we can store multiple clips in the script and play the appropriate one dynamically.

#### 1: Create an Audio Source
1. Add an `AudioSource` to a game object.
2. **Do Not Assign an Audio Clip**
3. Make sure **Play On Awake** is **unchecked**, so the sound only plays when triggered.
4. Adjust settings (volume, loop, 3D spatial blend).

#### 2. Add Clip Fields in the Script
- Open the class for the game object (e.g., Player, Enemy, Collectable)
- Create public fields for each clip you want to play:

```csharp
    [Header("Audio Settings")]
    [SerializeField]
    private AudioClip _powerUpClip;
    [SerializeField]
    private AudioClip _loseLifeClip;

    private AudioSource _audioSource;


    private void Awake()
    {
        // Get the AudioSource component on this GameObject
        _audioSource = GetComponent<AudioSource>();

    }//end Awake()

    // Example method for powering up
    public void PowerUp()
    {
        _audioSource.clip = _powerUpClip;
        _audioSource.Play();

    }//end PowerUp()

    // Example method for losing a life
    public void LoseLife()
    {
        _audioSource.clip = _loseLifeClip;
        _audioSource.Play();

    }//end LoseLife()

```
> [!NOTE]
> The code above is just an example for defining and playing audio clips. An actual Player class might have many other functions and fields.
> The `[Header("...")]` attribute helps to group and organize fields in the Inspector.
> The example methods `PowerUp()` and `LoseLife()` could also perform additional tasks, such as adding to the score or subtracting from a lives variable. Additionally, these methods would need to be called from somewhere in your game, such as a trigger or collision event in the Player class.

#### 3: Assign Clips in the Inspector
1. From the **Project window** drag your audio clips into the `_powerUpClip` and `_loseLifeClip` fields in the **Inspector Window**

#### 4. Test
1. Call `the PowerUp()` or `LoseLife()` methods (via code or UI) during gameplay.
2. Verify that the correct clip plays each time.

> [!TIP]
> Use `AudioSource.PlayOneShot(AudioClip clip)` to play a clip without changing the AudioSource's main clip. This is useful if you want multiple sounds to overlap.

---

## Fading Audio In and Out
There are times when you want audio to gradually fade in or out, for example, when a player enters or exits a trigger area. This demonstrates how to dynamically control audio volume over time. The functionality should be placed on the GameObject with the trigger, such as a boss room. For instance, when the player enters the room, the music can fade in to build tension, and fade out when they leave.

```csharp
public class BossRoom: MonoBehaviour
{
    [Header("Audio Settings")]

    [SerializeField, Tooltip("Speed at which volume fades in/out (per second)")]
    private float _fadeSpeed = 0.5f;

    private AudioSource _audioSource;

    // Tracks whether the player is currently inside the trigger
    private bool _playerInRoom = false;

    private void Update()
    {
        //If the audio source is playing fade music 
        if(_audioSource.isPlaying)
        {
            FadeAudio();
        }

    }//end Update()

    private void OnTriggerEnter(Collider other)
    {
        if (other.CompareTag("Player"))
        {
            _isPlayerInside = true;

            // Start playing the music if it isn’t already
            if (!_audioSource.isPlaying)
            {
                _audioSource.Play();
            }
        }
    }

    private void OnTriggerExit(Collider other)
    {
        if (other.CompareTag("Player"))
        {
            _isPlayerInside = false;
        }
    }

    /// <summary>
    /// Gradually fades audio in and out when the player enters or exits a trigger area.
    /// </summary>
    
    private void FadeAudio()
    {
        // Determine the target volume based on player presence
        float targetVolume = _playerInRoom ? 1f : 0f;

        // Smoothly move the current volume toward the target volume
        _musicSource.volume = Mathf.MoveTowards(_audioSource.volume, targetVolume, _fadeSpeed * Time.deltaTime);

        // Stop the audio if the player leaves and the volume reaches zero
        if (!_playerInRoom && _audioSource.volume == 0f && _audioSource.isPlaying)
        {
            _audioSource.Stop();

        }//end if
    }///end FadeAudio()

}//end FadeMusic()


```

### Breakdown of `BossRoom`
The `_fadeSpeed` values control **how quickly the audio volume rises and falls**. Larger values produce faster transitions, while smaller values create slower, smoother changes.

The `_playerInRoom` flag is a simple boolean that keeps track of whether the player is currently inside the trigger area.
- `OnTriggerEnter()` sets `_playerInRoom = true`
- `OnTriggerExit()` sets `_playerInRoom = false`

`OnTriggerEnter()` also calls `Play()` on the audio source. The `Update()` method checks if the audio source `isPlaying`, and if so, calls `FadeAudio()`. This method gradually increases or decreases the audio volume based on the `_playerInRoom` flag. Once the volume reaches zero, the audio source is stopped using `Stop()`.

The `FadeAudio()` method calls the **`Mathf.MoveTowards()`** method to create the smooth transition. This method moves a **value at a constant speed toward a target**, ensuring it reaches the target without overshooting. It takes three parameters:
- The **current value** of the audio volume
- The **target value** (e.g., maximum 1f or minimum 0f)
- The **maximum amount the value can change per frame** (controlled by `_fadeSpeed`)  
    - Multiplying this by `Time.deltaTime` makes the transition **frame-rate independent**.

The `FadeAudio()` method also ensures the **audio stops only after it has fully faded out**, rather than cutting off immediately **when the player leaves the room**. By waiting until `_audioSource.volume` reaches `0f`, the fade-out feels smooth and natural. The `isPlaying` check prevents `Stop()` from being called on an audio source that is already stopped.

----

# 🎉 New Achievement: Audio

These are just a few ways you can control and enhance audio through scripting. Audio behaviors can be attached to the objects that produce sound or to managers that control sound globally.
Sound adds depth, emotion, and atmosphere to your game, and with a bit of creativity, you can design dynamic audio systems, such as:

- Ambient soundscapes that fade as players move between areas
- Music that intensifies during battles or boss encounters
- Environmental sounds that react to weather or time of day
- Interactive sound effects that respond to player actions or physics events

Mastering audio scripting helps bring your worlds to life, turning simple gameplay into immersive experiences.

---

**[<< Return to Camera Lesson ](camera.md)**