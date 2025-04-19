This document provides a reverse-engineered breakdown of the YOBJ binary file format, commonly used in certain games to store 3D models, meshes, bones, textures, and related data. It is meant to assist modders, developers, or enthusiasts in understanding the internal structure of `.yobj` files for analysis, editing, or tool development purposes.

# YOBJ Hex File Structure
## 1. Header

- `4 Bytes`: String `"YOBJ"`  
- `4 Bytes`: POF0 Offset (see section 6)  
- `4 Bytes`: Empty  
- `4 Bytes`: POF0 Offset (same as above)  
- `8 Bytes`: Empty  
- `4 Bytes`: Mesh Count  
- `4 Bytes`: Bones Count  
- `4 Bytes`: Textures Count  
- `4 Bytes`: Mesh Header Offset (see section 2)  
- `4 Bytes`: Bones List Offset (see section 3)  
- `4 Bytes`: Textures List Offset (see section 4)  
- `4 Bytes`: Model Name Offset (see section 5)  
- `4 Bytes`: Model Count (Default: 1)  
- `16 Bytes`: Empty

---

## 2. Mesh

### 2.1 Mesh Header

Header length per mesh: 64 Bytes

- `4 Bytes`: Always 1  
- `4 Bytes`: Material Count  
- `4 Bytes`: Bones Mesh Header Offset (see 2.2)  
- `4 Bytes`: Material Offset (see 2.5)  
- `4 Bytes`: Empty  
- `4 Bytes`: Always 1  
- `4 Bytes`: Vertice Header Offset (see 2.3)  
- `4 Bytes`: Unknown  
- `8 Bytes`: Unknown  
- `4 Bytes`: Vertice Count  
- `20 Bytes`: Unknown

### 2.2 Bones Mesh Header

Header length depends on Mesh Bones Count:

- `4 Bytes`: Vertice Count (same as in mesh header)  
- `4 Bytes`: Mesh Bones Count  
- `4 Bytes`: Vertice Header Offset  
- `4 Bytes`: Empty  
- **Bones Assignments**: 4 Bytes per bone according to Mesh Bones Count (usually 4–16 bytes), using bone index.

### 2.3 Vertice Header Offset

- `4 Bytes`: Vertice Offset (see section 2.4)

### 2.4 Vertices

- `16 Bytes`: Always the same  
- Then: `67 * Vertice Count` bytes  
  - (Some information like vertex positions used by YOBJ File Tools is located here.)  
- Most of the rest is unknown.

### 2.5 Material

Each material is `144 Bytes` long (based on Material Count):

- `22 Bytes`: Unknown  
- `2 Bytes`: Assigned Texture by texture index  
- `108 Bytes`: Unknown  
- `4 Bytes`: Faces Count  
- `4 Bytes`: Faces Header Offset (see 2.6)  
- `4 Bytes`: Faces Start Offset (see 2.7)

### 2.6 Faces Header

Each face header: `16 Bytes`

- `8 Bytes`: Unknown  
- `4 Bytes`: Faces Count  
- `4 Bytes`: Faces Offset

### 2.7 Faces

Each face: `2 Bytes`  
- Total length = `2 * Faces Count`  
- (Used for face rendering in Blender.)

---

## 3. Bones

Mostly difficult to understand due to positional data, but the following is known:  
Typical size: `80 Bytes * Bones Count`

- `16 Bytes`: Bone Name  
- `32 Bytes`: Unknown (possibly positional data)  
- `4 Bytes`: Parent Bone (based on bone order)  
- `28 Bytes`: Unknown

---

## 4. Textures

Nothing special:  
- `16 Bytes * Texture Count`: Texture Name

---

## 5. Model Name

Length: `32 Bytes`  

- `16 Bytes`: Model Name  
- `16 Bytes`: Unknown

---

## 6. POF0

Contains all offsets in the YOBJ file, encrypted with a specific logic.  
Length varies, but always starts with:

- `4 Bytes`: String `"POF0"`  
- `4 Bytes`: Length of the POF0 section  
- Followed by: Encrypted POF0 content

---
