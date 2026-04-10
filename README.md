Quick Website <br> <br>
Website: https://wild-rod.github.io/WildRod/ <br> <br>
Visit for lots of fun! (Not yet but hopefully soon!)


import json
import sys
import os

# ====== CONFIG ======
k = 4.0  # mirror across y = k
# ====================

def mirror_rotation(deg):
    return (-deg) % 360

def mirror_point(point):
    # Position
    if "anchor" in point:
        point["anchor"]["y"] = 2*k - point["anchor"]["y"]
    if "prevControl" in point and point["prevControl"] is not None:
        point["prevControl"]["y"] = 2*k - point["prevControl"]["y"]
    if "nextControl" in point and point["nextControl"] is not None:
        point["nextControl"]["y"] = 2*k - point["nextControl"]["y"]

    # Rotation
    if "holonomicRotation" in point:
        point["holonomicRotation"] = mirror_rotation(point["holonomicRotation"])

def mirror_file(input_path):
    with open(input_path, "r") as f:
        data = json.load(f)

    # Waypoints
    if "waypoints" in data:
        for point in data["waypoints"]:
            mirror_point(point)

    # Rotation targets
    if "rotationTargets" in data:
        for rot in data["rotationTargets"]:
            rot["rotation"] = mirror_rotation(rot["rotation"])

    # Output filename
    base, ext = os.path.splitext(input_path)
    output_path = base + "_mirrored" + ext

    with open(output_path, "w") as f:
        json.dump(data, f, indent=2)

    print(f"Saved: {output_path}")

# Handle drag & drop
if len(sys.argv) < 2:
    print("Drag a .path file onto this script!")
else:
    for path in sys.argv[1:]:
        mirror_file(path)