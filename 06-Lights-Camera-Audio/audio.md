# Audio Behaviors in Unity
Audio in Unity is **essential for creating atmosphere, providing feedback, and guiding player attention**. Using scripts, you can dynamically control audio, play or stop clips, adjust volume, pitch, or spatial settings, based on gameplay events.

## Unity Audio Components

Unity has several audio components, each with specific purposes:

| Component          | Description                                                                            | Best Use Case                                                 |
| ------------------ | -------------------------------------------------------------------------------------- | ------------------------------------------------------------- |
| **Audio Source**   | Plays an audio clip in the scene. Can be 2D or 3D (spatialized).                       | Footsteps, background music, sound effects.                   |
| **Audio Listener** | Captures audio in the scene. Usually attached to the main camera.                      | Required for any sound to be heard.                           |
| **Audio Mixer**    | Allows grouping of audio sources for shared control over volume, effects, and routing. | Managing music vs. SFX, applying effects to multiple sources. |

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
The Audio Source component has several properites that can be accessed via script. 

| Property        | Description                                   |
| --------------- | --------------------------------------------- |
| `clip`          | Assigns the audio clip to play.               |
| `volume`        | Controls loudness (0–1).                      |
| `pitch`         | Changes playback speed and tone.              |
| `loop`          | Determines if the clip repeats automatically. |
| `spatialBlend`  | Adjusts 2D vs. 3D sound.                      |
| `Play()`        | Starts playback.                              |
| `PlayOneShot()` | Plays a clip once without stopping others.    |
| `Stop()`        | Stops playback immediately.                   |

---
# Adding an Audio Source 
Audio is usually placed on the GameObject that produces the sound. For example, a **Player** object might have an AudioSource for footsteps, and the player’s behavior script sends the appropriate clip to the AudioSource as the player moves. A **bridge** might play a warning sound when the player gets close. In this case, the AudioSource is attached to the bridge, and the bridge’s script plays the clip when the player enters the bridge’s trigger area.

Continuous sounds, like background music or environmental effects, should be placed on an empty GameObject. While many tutorials attach music to the main camera, this can blur responsibilities and make the camera manage things it shouldn’t. Using a dedicated **Audio Manager** or **Background Music** GameObject keeps audio logic separate, makes your system easier to manage, and avoids unintended side effects from mixing audio with camera behavior.



