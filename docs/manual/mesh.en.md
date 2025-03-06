# Mesh

The **Mesh** and its corresponding **Mesh** define the 3D environment and other objects, which means that they are one of the most important details of a project.

---
## Markup

Since the meshes will be used exclusively for collisions, we don't need a lot of modeling. The question is, how do we size the model based on 2D sprites? The solution is “markup”.

The basic 3D block is 1x1x1 meter and this corresponds to two 2D tiles in orthographic projection (for lack of perspective):

![Pasted image 20250301111246.png](../assets/images/pasted-images/Pasted%20image%2020250301111246.png)

!!!note
	The size of the tiles is NOT IMPORTANT. In this case it is 16x16 one tile (top is light). You can use 32x32, 48x48, 64x64, etc, which will also correspond to a 1x1x1 meter block.

Needless to say, your game will not have orthographic projection, instead you will probably use normal perspective. Nevertheless, this is important from a technical point of view, and the tiles themselves can be drawn however you like.

---
### Constructor

Let's consider a simple case where you can build meshes for objects as a constructor:

![Pasted image 20250301114628.png](../assets/images/pasted-images/Pasted%20image%2020250301114628.png)

!!!note
	You will do something similar in the editor, when you will build a three-dimensional environment from individual tiles and attached to them elements of the mesha library.

---
### Proportion

For particularly complex objects, the constructor method doesn't always work.

So you have a base tile of your chosen size (16x16 for example), which refers to a 1x1x1 meter block. This is how the proportion is formed:

```
16 -> 1 meter

16 -> 1 meter
```

Now we need to calculate pixel by pixel the size of some element, taking into account the orthogonal projection. In other words, the width and height of the front and top:

![Pasted image 20250301203823.png](../assets/images/pasted-images/Pasted%20image%2020250301203823.png)

Following the proportion, we calculate the required size in meters for each axis:

For the X axis:

```
16 -> 1 meter

27 -> X meter

X = (27*1)/16 = 1.6875
```

For the Y axis:

```
16 -> 1 meter

15 -> Y meter

Y = (15*1)/16 = 0.9375
```

For the Z axis:

```
16 -> 1 meter

15 -> Z meter

Z = (15*1)/16 = 0.9375
```

Now create a model based on the data you have obtained:

![Pasted image 20250302112143.png](../assets/images/pasted-images/Pasted%20image%2020250302112143.png)

---
### Image

There is also a simpler alternative to proportions - using the sprites themselves.
The idea is to create a model based on an already drawn sprite, and then scale it to the desired size.

For example, this is how I made Mitsubishi Zero:

![Pasted image 20250302153040.png](../assets/images/pasted-images/Pasted%20image%2020250302153040.png)

![Zero.png](../assets/images/Zero.png)

---
### Export
### MeshLibrary

For the meshes library, you have room for implementation - use the formats you are most comfortable with.

For this example, I'll start with blender itself, where we'll create the meshes. To do this, you just need to add new geometry and edit its vertices - it's pretty simple. You should end up with a structure like this:

![Pasted image 20250301124539.png](../assets/images/pasted-images/Pasted%20image%2020250301124539.png)

---

Now let's move on to the export. Choose any format you like, and for this example I will use **.blend** or **.glb** format:

1. Right-click on the imported file and click on “New Inherited Scene” item
2. Highlight all the **MeshInstance3D** blocks and at the top of the screen click on “Mesh” and create a collision shape.
3. Now for each **MeshInstance3D** block, create a **StaticBody3D** and drop the collision mesh there.

It's done! Now just export the mesh library by clicking on the “Scene” section, and under “Export As” select “Mesh Library”.

!!!tip
	The center of the blocks must match the center of the space in the editor

---
### OBJ

We will use the OBJ format for the objects, as it is the most suitable for our purposes.

1. Create a mesh.
2. Export it to **.obj** format.
3. In the Godot editor add **MeshInstance3D** to the scene.
4. By clamping the imported OBJ mesh, drop it into **MeshInstance3D**.

After that, generate the mesh as shown in the last header and place the two objects in the core.


!!!tip
	The bottom of the blocks should coincide with the horizontal plane of the space in the editor. You can of course change the position of the mesh later on, but this is a matter of convenience and time saving