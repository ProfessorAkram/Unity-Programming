# Unity UI Basics

The **User Interface (UI)** in Unity is a special layer that displays information and provides ways for the player to interact with your game. Common examples include health bars, score counters, buttons, and menus.

When you create a new UI element (for example, by going to GameObject → UI → Button), Unity automatically creates two things if they don’t already exist:
- **Canvas** – the root object that holds your UI.
- **EventSystem** – the component that listens for and handles user input like mouse clicks, touch events, and key presses.

---

## Unity Event System
The **EventSystem** uses something called a **Graphic Raycaster**. This Raycaster sends an invisible ray from the pointer (like a mouse or touch position) into the Canvas.
When that ray **“hits”** a UI element, it triggers an **event**, such as `OnClick()` on a Button.

---

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

--- 
## Basic UI Layout
The layout of your UI determines how elements are positioned, sized, and scaled within the **Canvas**. Every UI element’s position is defined relative to its parent, meaning that changes to a parent object (like resizing a panel) can affect how its children are displayed.

In this section, we’ll look at several tools and concepts that help you control the placement of UI elements: the Rect Tool, Rect Transform, pivots, and anchors.

### The Rect Tool

Every UI element in Unity is represented as a **rectangle** that defines its position and size on the Canvas. You can manipulate this rectangle using the **Rect Tool**, found in the toolbar.

The Rect Tool lets you:
- **Move** an element by dragging anywhere inside the rectangle
- **Resize** it by dragging the edges or corners
- **Rotate** it by hovering near a corner until the rotation icon appears

Although designed for UI, the Rect Tool can also be used for 2D and 3D objects. When working with UI, it’s best to keep the toolbar set to **Pivot** and **Local** modes to maintain predictable transformations.

### Rect Transform
All UI elements use a **Rect Transform** component instead of the standard Transform.
A Rect Transform includes the usual **Position**, **Rotation**, and **Scale**, but also adds:
- Width and Height values that define the size of the rectangle.

This allows Unity to treat UI elements as flexible rectangles rather than static objects in 3D space.

#### Resizing vs. Scaling
When using the Rect Tool:
- **3D objects** change their **scale** when resized.
- **UI elements** (with Rect Transform) change their **width and height**, keeping the scale values untouched.

This means resizing a UI element won’t affect text size, borders, or image slicing, it only changes the element’s physical dimensions on the Canvas.

#### Pivot
The **pivot point** determines the center of rotation, scaling, and resizing.
Changing the pivot’s position affects how an element transforms, for example, rotating around the top-left corner instead of the center.

When the **Pivot** button in the toolbar is active, you can move the pivot directly in the Scene view. Adjusting it helps control how elements animate, resize, or align with other UI components.

### Anchors

**Anchors** define how a UI element’s position and size adapt when its parent object changes.
They’re displayed as **four small triangles** around the UI rectangle in the Scene view.

Each corner of a UI element can be anchored to a specific point on its parent’s rectangle:
- For instance, you can anchor an element to the **center**, **corner**, or **edge** of its parent.
- Anchors can also be set to **stretch horizontally or vertically** with the parent.

Anchor positions are defined as fractions of the parent’s size:
- `0.0` → left or bottom
- `0.5` → center
- `1.0` → right or top

You can drag anchors in the Scene view. **Holding Shift while dragging** moves both the anchor and its associated corner, maintaining their relative offset. Anchors also **snap automatically** to sibling anchors for precise alignment.

#### Anchor Presets
In the Inspector, you’ll find the Anchor Presets button (upper-left corner of the Rect Transform). Clicking it opens a dropdown of common anchoring options. For example:
- Anchor to the center
- Stick to a corner
- Stretch with the parent’s width or height

Horizontal and vertical anchors can be set independently. If your anchors don’t match any preset, Unity shows a custom icon instead.

#### Anchor Min & Max

Expanding the Anchors section in the Inspector reveals numerical fields for fine-tuning.
- **Anchor Min** corresponds to the lower-left handle.
- **Anchor Max** corresponds to the upper-right handle.

The position fields displayed depend on the anchor configuration:
- When **anchors are together**, you’ll see **Pos X**, **Pos Y**, **Width**, and **Height** fields, defining fixed size and position.
- When anchors are separated, the fields change to **Left**, **Right**, **Top**, and **Bottom**, which define padding between the element and its parent’s edges.

Normally, changing anchor or pivot values automatically adjusts position to keep the element in place. If you want to move anchors without Unity compensating, enable **Raw Edit Mode**. This allows you to freely adjust anchors and pivots, though it may visually move or resize the element.


