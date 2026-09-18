# A1 – Forensic BIM

## Group

Group 4

## Focus Area

Architecture

## Identified Issue

The existing B308 building lacks large, flexible open spaces that support contemporary working and learning environments. The current layout is dominated by enclosed classrooms, internal walls, and small corridors, limiting the flexibility of the building.

## Cause

### Design

The existing architectural layout prioritises small enclosed rooms connected by internal corridors and passageways. This reduces the availability of open-plan spaces and makes the building less adaptable to contemporary environments.

### Modelling

The IFC model represents the existing building, but it does not explicitly identify or classify spaces according to their suitability for contemporary work. This makes it difficult to automatically distinguish enclosed rooms from open collaborative spaces.

### Tool

Bonsai and IFC can display rooms and walls, but they do not automatically analyse spatial openness or workspace flexibility. A custom Python/IfcOpenShell script is therefore needed to extract and analyse the relevant spaces and areas.

## Potential Solutions

### Design

Remove selected internal non-load-bearing walls and modify the basement deck where structurally feasible to create larger, more flexible open-plan spaces for collaborative work and teaching.

### Modelling

Improve the IFC model by adding consistent space classifications and properties, such as room function, occupancy, and space type, so that open and enclosed spaces can be analysed automatically.

### Tool

Develop a Python/IfcOpenShell rule checker that identifies `IfcSpace` objects, calculates their floor areas, and reports the proportion of open versus enclosed spaces in the building.

## Source

**Report:** 26-09-A-ClientReport-Anon

**Pages:** 3–7

**Sections:** 3.1, 3.2.1, 3.3
import bonsai.tool as tool
import ifcopenshell.util.element

model = tool.Ifc.get()

print("================================")
print("B308 - Architecture Check")
print("================================")

# Spaces
spaces = model.by_type("IfcSpace")

print("\nSPACES")
print("Number of spaces:", len(spaces))

for space in spaces:
    psets = ifcopenshell.util.element.get_psets(space)
    quantities = psets.get("Qto_SpaceBaseQuantities", {})
    area = quantities.get("NetFloorArea", 0)

    print(space.Name, "-", round(area, 2), "m2")


# Walls
walls = model.by_type("IfcWall")

internal = 0
external = 0

for wall in walls:

    psets = ifcopenshell.util.element.get_psets(wall)
    wall_common = psets.get("Pset_WallCommon", {})

    if wall_common.get("IsExternal") == True:
        external += 1

    elif wall_common.get("IsExternal") == False:
        internal += 1


print("\nWALLS")
print("Total walls:", len(walls))
print("Internal walls:", internal)
print("External walls:", external)


# Other elements
print("\nOTHER ELEMENTS")
print("Doors:", len(model.by_type("IfcDoor")))
print("Windows:", len(model.by_type("IfcWindow")))
print("Slabs:", len(model.by_type("IfcSlab")))

print("\n================================")
print("END")
print("================================")


