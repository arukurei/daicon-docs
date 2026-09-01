# 3D Meshes & MeshLibrary

In Daicon, the 3D **mesh** and its corresponding **collision shape** define the world geometry and physical bounds of objects.

Since meshes are primarily used for physics and collision detection, there is no need for complex, high-poly modeling. The main goal is calculating the correct dimensions of the 3D model based on your 2D sprites.

---

## 1. Markup & Base Proportions

The standard 3D block measures **1×1×1 meter**. In an oblique 2.5D projection, this single cube corresponds to **two 2D tiles** (the top face and the front wall):

![Pasted image 20250301111246.png](../assets/images/pasted-images/Pasted%20image%2020250301111246.png)

> [!NOTE] Tile Size
> The actual pixel dimensions of your tiles can be anything: 16×16, 32×32, 48×48, 64×64, etc. In mathematical formulas, it is determined by the entity's `tile_size` property (by default, 16 pixels = 1 meter).

---

### Method 1: The Constructor (Block Assembly)

For simple structures (walls, crates, steps), the mesh can be assembled like building blocks out of standard 1-meter cubes:

![Pasted image 20250301114628.png](../assets/images/pasted-images/Pasted%20image%2020250301114628.png)

> [!INFO]
> You will use the exact same modular approach when designing levels in the Godot editor using `DaiconMap`.

---

### Method 2: Pixel-by-Pixel Math Proportions

For complex objects (furniture, angled rooftops), dimensions are calculated pixel-by-pixel using a simple ratio.

Given a base tile size of 16×16 pixels (= 1 meter), measure the pixel dimensions of the front and top faces of the sprite:

![Pasted image 20250301203823.png](../assets/images/pasted-images/Pasted%20image%2020250301203823.png)

**Calculating dimensions in meters for each axis:**

* **X Axis (Width):**
  $$\text{X} = \frac{27 \text{ px} \times 1 \text{ m}}{16 \text{ px}} = 1.6875 \text{ m}$$
* **Y Axis (Height):**
  $$\text{Y} = \frac{15 \text{ px} \times 1 \text{ m}}{16 \text{ px}} = 0.9375 \text{ m}$$
* **Z Axis (Depth):**
  $$\text{Z} = \frac{15 \text{ px} \times 1 \text{ m}}{16 \text{ px}} = 0.9375 \text{ m}$$

Create the 3D model in your modeling suite (e.g. Blender) using these calculated dimensions:

![Pasted image 20250302112143.png](../assets/images/pasted-images/Pasted%20image%2020250302112143.png)

---

### Method 3: Modeling Over Sprites

For organic props or vehicles, you can use the drawn sprite directly as a reference image in Blender: model the rough silhouette directly over the texture, then scale the model to match the desired meter proportions.

Example workflow for a Mitsubishi Zero fighter:

![Pasted image 20250302153040.png](../assets/images/pasted-images/Pasted%20image%2020250302153040.png)

![Zero.png](../assets/images/Zero.png)

---

## 2. Exporting MeshLibrary (For TileSets)

To construct level maps in `DaiconMap`, you will need a **`MeshLibrary`** resource.

### Blender Preparation:

1. Build your collection of modular blocks (cubes, slopes, corner pieces).
2. Ensure each block mesh is centered at its local coordinate origin.

![Pasted image 20250301124539.png](../assets/images/pasted-images/Pasted%20image%2020250301124539.png)

3. **Collision Suffix:** Add `-col` to the end of each mesh object's name in Blender (e.g. `Wall-col`, `Slope-col`). Godot will automatically generate static collision shapes for them upon import.

![Pasted image 20250819190644.png](../assets/images/pasted-images/Pasted%20image%2020250819190644.png)

### Exporting into Godot:

1. Export the scene from Blender as a **`.blend`** or **`.glb`** file.
2. In Godot, right-click the imported asset → **«New Inherited Scene»**.
3. In the opened scene, go to the top menu: **Scene** → **Export As...** → **MeshLibrary** (`.tres` or `.res`).
4. Assign the resulting library resource to the `mesh_library` property on your `DaiconMap`.

> [!TIP] Origin Alignment
> The base center of blocks in Blender must align precisely with the local origin `(0, 0, 0)`.

---

## 3. Exporting Discrete Objects (OBJ)

For standalone interactive objects (crates, barrels, vehicles), the **OBJ** format is ideal:

1. Model the mesh in your 3D software.
2. Export it as an `.obj` file.
3. In Godot, add a `MeshInstance3D` node to your scene and assign the `.obj` file to it.
4. Assign the `MeshInstance3D` into the **Mesh Node** slot of the corresponding Daicon entity.