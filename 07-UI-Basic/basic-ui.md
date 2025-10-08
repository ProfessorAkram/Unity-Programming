# Unity UI Basics

The **User Interface (UI)** in Unity is a special layer that displays information and provides ways for the player to interact with your game. Common examples include health bars, score counters, buttons, and menus.

When you create a new UI element (for example, by going to GameObject → UI → Button), Unity automatically creates two things if they don’t already exist:
- **Canvas** – the root object that holds your UI.
- **EventSystem** – the component that listens for and handles user input like mouse clicks, touch events, and key presses.

## Unity Event System
The **EventSystem** uses something called a **Graphic Raycaster**. This Raycaster sends an invisible ray from the pointer (like a mouse or touch position) into the Canvas.
When that ray **“hits”** a UI element, it triggers an **event**, such as `OnClick()` on a Button.

## Canvas
The **Canvas** is the foundation of every Unity UI. It's a special **GameObject** that defines the area where all UI elements (such as text, images, buttons, and sliders) are drawn.
Every UI element must exist as a **child** of a Canvas, if a UI element isn’t inside one, it won’t be visible.

In the Scene view, the Canvas area appears as a rectangular outline, allowing you to position and arrange UI elements visually, even without switching to the Game view. This makes it easier to align and layout your interface directly within the editor.

> [!TIP]
> If you ever can’t see your UI in the Scene view, try selecting the Canvas and pressing **F** to frame it, this centers your view on the Canvas rectangle.

### Draw Order of Elements
UI elements within a Canvas are drawn in the order they appear in the Hierarchy:
- The first child in the list is drawn first (appearing behind others).
- The last child is drawn last (appearing on top).

If two UI elements overlap, the one lower in the Hierarchy will appear above the other.
You can change their order simply by **dragging elements up or down** in the Hierarchy.

From a script, you can also control the order using:

- `SetAsFirstSibling()` – moves the element to the back.
- `SetAsLastSibling()` – brings the element to the front.
- `SetSiblingIndex(int index)` – places it at a specific position.

### Canvas Render Modes
The **Render Mode** setting on the Canvas determines how the UI is displayed in relation to the scene. There are three main options:

**Screen Space – Overlay**
UI elements are rendered directly on top of the screen, independent of any camera.
If the screen is resized or its resolution changes, the Canvas automatically adjusts its size to match.
_✅ This is the default mode and works best for most traditional UI layouts._

**Screen Space – Camera**
The Canvas is positioned a set distance in front of a chosen Camera.
Because it’s tied to a Camera, the UI can be affected by camera settings like depth or post-processing.
_✅ Useful for UIs that should appear as part of the camera view._

**World Space**
The Canvas behaves like any other 3D GameObject in the scene.
UI elements exist in world coordinates and can appear in front of or behind other 3D objects.
_✅ Commonly used for in-world displays like control panels or holograms (also called diegetic interfaces)._

> [!TIP]
> While editing UI elements, you may also want to **switch the Scene view to 2D mode** (toggle the 2D button in the Scene view toolbar). This flattens the view, making it much easier to align and position your UI elements precisely on the Canvas.
