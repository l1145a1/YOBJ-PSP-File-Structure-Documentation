# YOBJ Hex File Structure

This document provides a reverse-engineered breakdown of the **YOBJ binary file format**, commonly used in certain games to store 3D models, meshes, bones, textures, and related data.  
It is intended to assist modders, developers, and enthusiasts in understanding the internal structure of `.yobj` files for analysis, editing, or tool development.

---

## Table of Contents

- [1. Header](#1-header)
- [2. Mesh](#2-mesh)
  - [2.1 Mesh Header](#21-mesh-header)
  - [2.2 Bones Mesh Header](#22-bones-mesh-header)
  - [2.3 Mesh Data Header Offset](#23-mesh-data-header-offset)
  - [2.4 Mesh Data](#24-mesh-data)
  - [2.5 Material](#25-material)
  - [2.6 Faces Header](#26-faces-header)
  - [2.7 Faces](#27-faces)
- [3. Bones](#3-bones)
- [4. Textures](#4-textures)
- [5. Model Name](#5-model-name)
- [6. POF0](#6-pof0)

---

## 1. Header

**Length:** 72 Bytes

- `4 Bytes`: String `"YOBJ"`  
- `4 Bytes`: YOBJ End Offset (see section 6)  
- `4 Bytes`: Empty  
- `4 Bytes`: POF0 Offset (usually same as YOBJ End Offset)  
- `4 Bytes`: Empty  
- `4 Bytes`: Unknown  
- `4 Bytes`: Mesh Count  
- `4 Bytes`: Bones Count  
- `4 Bytes`: Textures Count  
- `4 Bytes`: Mesh Header Offset (see section 2)  
- `4 Bytes`: Bones List Offset (see section 3)  
- `4 Bytes`: Textures List Offset (see section 4)  
- `4 Bytes`: Model Name Offset (see section 5)  
- `4 Bytes`: Model Count (default: 1)  
- `16 Bytes`: Empty  

---

## 2. Mesh

### 2.1 Mesh Header

**Length per mesh:** 64 Bytes

- `4 Bytes`: Always 1  
- `4 Bytes`: Material Count  
- `4 Bytes`: Bones Mesh Header Offset (see 2.2)  
- `4 Bytes`: Material Offset (see 2.5)  
- `4 Bytes`: Empty  
- `4 Bytes`: Always 1  
- `4 Bytes`: Mesh Data Header Offset (see 2.3)  
- `4 Bytes`: Flag  
- `8 Bytes`: Empty  
- `4 Bytes`: Mesh Data Count  
- `20 Bytes`: Unknown  

---

### 2.2 Bones Mesh Header

**Length:** depends on Mesh Bones Count

- `4 Bytes`: Mesh Data Count (same as in mesh header)  
- `4 Bytes`: Mesh Bones Count  
- `4 Bytes`: Mesh Data Header Offset  
- `4 Bytes`: Empty  
- **Bones Assignments:** 4 Bytes per bone according to Mesh Bones Count (usually 4–16 bytes), using bone index  

---

### 2.3 Mesh Data Header Offset

- `4 Bytes`: Mesh Data Offset (see section 2.4)

---

### 2.4 Mesh Data

Mesh Data length and structure depend on the **Flag**:

1. Flag is stored in the mesh header as an integer.  
2. Convert flag to boolean: `flag & 1536 != 0`  
3. Convert flag to 32-bit binary string and reverse: `format(flag, '032b')[::-1]`  
4. Decode bits 16–14 into integer: `int(flag_binary[16:13:-1], 2)`  

- **If Flag Boolean = True** → Mesh Data Length = `(flag_decode + 10) * 4`  
- **If Flag Boolean = False** → Mesh Data Length = `Mesh Data Count * 36`  

**UV Coordinates:**
- If Flag Boolean = True → `Mesh Data Offset + ((flag_decode + 1) * 4)`  
- If Flag Boolean = False → UV Coordinate at Mesh Data Offset  
- `4 Bytes`: U Coordinate  
- `4 Bytes`: V Coordinate  

**Vertex Data:**
- If Flag Boolean = True → `(Mesh Data Offset + Mesh Data Length) - 12`  
- If Flag Boolean = False → `(Mesh Data Offset + 36)`  
- `4 Bytes`: Vertex 1  
- `4 Bytes`: Vertex 2  
- `4 Bytes`: Vertex 3  

**Vertex ordering alternates:**
```python
for i in range(MeshDataCount):
    if i % 2 == 0:  # even
        Vertex1 = x
        Vertex2 = y
        Vertex3 = z
    else:           # odd
        Vertex1 = x
        Vertex2 = z
        Vertex3 = y
```
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
- `8 Bytes`: Unknown
- `4 Bytes`: Mesh Count
- `4 Bytes`: Empty

---

## 6. POF0

Contains all offsets in the YOBJ file, encrypted with a specific logic.  
Length varies, but always starts with:

- `4 Bytes`: String `"POF0"`  
- `4 Bytes`: Length of the POF0 section  
- Followed by: Encrypted POF0 content

---
