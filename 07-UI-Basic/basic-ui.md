# Unity UI Basics

The **User Interface (UI)** in Unity is a special layer that displays information and allows players to interact with your game. Common examples include health bars, score counters, buttons, and menus. Unity provides three UI systems: **UI Toolkit**, **uGUI**, and **IMGUI**. This lesson focuses on uGUI, a legacy, GameObject-based UI system that uses GameObjects and components to create, arrange, and style UI elements directly in the Scene and Game views.

When you create a new UI element, Unity automatically creates two things if they don’t already exist:
- **Canvas** – the root object that holds your UI.
- **EventSystem** – the component that listens for and handles user input like mouse clicks, touch events, and key presses.

> [!NOTE]
> A scene can have **multiple Canvases** to organize different UI groups (like HUD, menus, or popups).
> However, **only one Event System per scene is needed**, as it can handle input for all UI elements across all canvases.

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

---
## UI Panel
A **Panel** is a UI element that lives under a Canvas and is used to organize and group other UI elements. For example, you might use a Panel to contain all the buttons for a main menu, or to group a health bar and score display in the HUD.

Besides grouping elements, panels can also control layout and alignment of their child elements using layout components. These components make it easy to automatically position, size, and space UI elements consistently.

### Common Alignment and Layout Components

- **Horizontal Layout Group**
Aligns child elements in a horizontal row. Options allow you to control spacing, padding, and whether children expand to fill the available width.

- **Vertical Layout Group**
Aligns child elements in a vertical column. You can adjust spacing, padding, and child alignment similarly to the horizontal layout group.

- **Grid Layout Group**
Arranges child elements in a uniform grid with defined cell sizes. Useful for inventories, skill trees, or button menus.

- **Content Size Fitter**
Automatically resizes the panel based on its children’s size. You can control horizontal and vertical fitting modes, such as Preferred Size or Min Size.

- **Layout Element**
Applied to individual child elements to override layout group behavior, such as setting fixed width, height, or flexible sizing.

Using these components, a panel can not only group elements visually, but also maintain consistent alignment and spacing, making your UI scalable and easier to manage.

---

## UI Visual Components

Unity’s UI system includes several visual components that allow you to display information and create interactive interfaces. These components are all designed to work inside a Canvas, and each has its own set of properties and uses. By combining them, you can build menus, HUDs, overlays, and other graphical interfaces.

Below is a quick reference of the main UI visual components:
| Component               | Purpose / Description                                                                                                                                                                   |
| ----------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Text**                | Displays text in the UI. Allows setting font, size, style, alignment, overflow, and auto-sizing (Best Fit).             |
| **Image**               | Displays a sprite. Supports Simple, Sliced (with 9-slicing), Tiled, or Filled render modes. Can apply color, material, and “Set Native Size” to match the sprite’s original dimensions. |
| **Raw Image**           | Displays a texture directly (without borders or slicing). Use when you need a texture not compatible with the Image component.                                                          |
| **Mask**                | Restricts child elements to the shape of the parent. Children outside the mask are hidden. The mask itself is not visible.                                                              |
| **Effects**             | Adds visual modifications like Drop Shadow or Outline to Text or Image components. Useful for improving readability and visual style.                                                   |

These components can be combined and customized to create rich, interactive user interfaces. Later, you’ll learn how to layer these elements, apply anchors and pivots, and handle user input through the Event System.

> [!NOTE]
> By default, all text components in Unity use **TextMesh Pro (TMP)** for high-quality text rendering and advanced styling options. TMP must be imported into your project. When you add your first text element to the Canvas, a TMP Import dialog will appear, prompting you to **Import TMP Essentials**.

---
## Interaction Components

Unity’s UI system includes **interaction components** that allow players to interact with your game through mouse, touch, keyboard, or controller input. Unlike visual components, these elements are **not visible** on their own and must be paired with one or more visual components (such as Button or Image) to function correctly.

Most interaction components share some common functionality:
- They are **Selectables**, meaning they include built-in support for visual transitions between states like **Normal**, **Highlighted**, **Pressed**, and **Disabled**.
- They support **navigation** between other selectable components using keyboard or controller input.
- Each component has one or more **UnityEvents** that are triggered when the user interacts with it. Unity automatically handles any exceptions thrown by code attached to these events.

| Component                     | Purpose / Description                                                                                                                                                                             |
| ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Button**                    | A clickable component that triggers an **OnClick** UnityEvent when pressed. Often paired with an Image for visuals.                                                                               |
| **Toggle**                    | A switch that can be turned **on** or **off**. Includes an **OnValueChanged** UnityEvent. Can display a checkmark to indicate state.                                                              |
| **Toggle Group**              | Groups multiple Toggles so only one can be active at a time. Selecting one automatically deselects the others.                                                                                    |
| **Slider**                    | Allows the user to select a **numeric value** within a range by dragging a handle. Can be horizontal or vertical. Fires an **OnValueChanged** UnityEvent when adjusted.                           |
| **Scrollbar**                 | Similar to a Slider but typically used to navigate content in a **Scroll Rect**. The **Size** property controls the handle’s proportion relative to content. Fires **OnValueChanged** when moved. |
| **Dropdown**                  | Displays a list of selectable options, each with text and optionally an image. Fires **OnValueChanged** when the selected option changes.                                                         |
| **Input Field**               | Makes a Text element editable by the user. Fires UnityEvents when the text changes and when editing ends.                                                                                         |
| **Scroll Rect (Scroll View)** | Provides a scrollable area for large content. Often used with a **Mask** to limit visible content and optional **Scrollbars** for navigation.                                                     |

--- 

## Unity Event System

The **Event System** in Unity is responsible for handling input events from the mouse, keyboard, touch, or controllers and sending them to the appropriate objects in your scene.

Think of it as a **manager** that connects user actions to the UI or other interactive objects.

Key Roles of the Event System:
- Keeps track of which **GameObject is currently selected**
- Determines which **Input Module** is active
- Handles **raycasting** to figure out what the pointer is over
- Updates all input modules as needed

### Input Modules
Input Modules define **how input is handled**. They:
- Detect user input (mouse, keyboard, touch)
- Track the current state of interactions
- Send events to objects in the scene

Only one Input Module can be active at a time. They are always attached to the same GameObject as the Event System.

### Raycasters
Raycasters help determine **what the pointer is pointing at**. Different types exist for different kinds of objects:
- Graphic Raycaster – for UI elements
- Physics 2D Raycaster – for 2D physics objects
- Physics Raycaster – for 3D physics objects

Raycasters work with Input Modules to detect which objects the user is interacting with. Non-UI objects can also receive events if they have a script that implements an event interface, such as `IPointerClickHandler` or `IPointerEnterHandler`.

--- 

## Attaching Events to Interaction Components

Once the **Event System** is in place, you can make UI components respond to player input by connecting their **UnityEvents** to your scripts. This is how Buttons, Toggles, Sliders, and other interactive components perform actions in your game.

Each interaction component has one or more **event fields** in the Inspector (for example, **OnClick** for a Button). To use these events:
1. **Assign a GameObject** that has the script containing the method you want to call.
2. **Select the method** from the dropdown list of public functions in that script.

Additional event types include: 

| Component  | Main UnityEvent            |
| ---------- | -------------------------- |
| Button     | OnClick                    |
| Toggle     | OnValueChanged             |
| Slider     | OnValueChanged             |
| InputField | OnValueChanged / OnEndEdit |
| Dropdown   | OnValueChanged             |


### UI Interaction Logic
The game object that has the script with the method for the UI element should be placed on a class specficially for UI Interaction. 
For example, suppose we are desiging a main menu. The **Canvas** itself can be renamed as **Main Menu** and a `MainMenu` class can be attached to it. The `MainMenu` class might also inlcude a method `PlayGame()`. 

In order for the **Play button** start the game when clicked:
- Drag the GameObject with the MainMenu script (i.e., the **Canvas** named **MainMenu**) into the Button’s **OnClick field**.
- Select `MainMenu` > `PlayGame()` from the method dropdown.

Now, whenever the player clicks the Button, the `PlayGame()` method is executed.

This approach allows you to link UI interactions to your game logic without writing extra input-handling code.

---

# UI Asset Naming in Unity
Organized and consistent naming of UI elements in Unity is essential for maintaining a clean hierarchy, making it easier to navigate, and improving communication with other team members. Following a clear naming convention also simplifies scripting and reduces confusion between objects in your scene.

## Naming the Canvas

The Canvas is the root for all UI elements, so its name should describe what the canvas contains, not the fact that it is a Canvas. This avoids confusion with game objects in the scene. 
Examples:
- A Canvas holding the main menu:
  ```
  Main Menu
  ```

- A Canvas for the inventory layout:
  ```
  Inventory UI
  
  ```

- A Canvas for the HUD:
  ```
  HUD
  ```

> [!NOTE]
> The `Inventory UI` is so named to differenate an Inventory game object from the **UI** component.

---

## Prefix Rules for UI Elements

UI elements under a Canvas should use consistent prefixes to indicate their type. This makes it easier to identify elements at a glance. 
Common prefixes include:

| UI Element | Prefix | Example                       |
| ---------- | ------ | ----------------------------- |
| Panel      | `pnl`  | `pnlMainMenu`, `pnlInventory` |
| Button     | `btn`  | `btnStart`, `btnExit`         |
| Text       | `txt`  | `txtScore`, `txtHealth`       |
| Image      | `img`  | `imgBackground`, `imgIcon`    |
| Slider     | `sld`  | `sldVolume`, `sldHealth`      |
| Toggle     | `tgl`  | `tglMute`, `tglFullscreen`    |
| InputField | `inp`  | `inpUsername`, `inpChat`      |

### Rules for Prefix Usage:
1. Always start the object name with the prefix.
2. Follow the prefix with a descriptive name using PascalCase.
3. Keep names clear and concise, reflecting the object’s purpose in the UI.
4. Avoid using spaces or special characters in the object name (underscores _ are optional if necessary).

---
## Example Hierarchy

A clean, well-named hierarchy for a main menu Canvas might look like this:
```
Main Menu (Canvas)
 └─ pnlBackground
     ├─ txtTitle
     ├─ imgLogo
     └─ pnlButtons
         ├─ btnStart
         ├─ btnOptions
         └─ btnExit
```
Using this convention makes it easy to understand the structure and purpose of each element, even at a glance.


