Quick Website <br> <br>
Website: https://wild-rod.github.io/WildRod/ <br> <br>
Visit for lots of fun! (Not yet but hopefully 



import json
import os
import tkinter as tk
from tkinter import filedialog, messagebox

# ====== CONFIG ======
k = 4.0  # mirror across y = k
# ====================

def mirror_y(y):
    return 2 * k - y

def safe_mirror_rotation(obj, key):
    if key in obj and obj[key] is not None:
        try:
            obj[key] = (-obj[key]) % 360
        except:
            pass

def mirror_point(point):
    # Anchor position
    if "anchor" in point and point["anchor"] is not None:
        if "y" in point["anchor"]:
            point["anchor"]["y"] = mirror_y(point["anchor"]["y"])

    # Control points
    if "prevControl" in point and point["prevControl"] is not None:
        if "y" in point["prevControl"]:
            point["prevControl"]["y"] = mirror_y(point["prevControl"]["y"])

    if "nextControl" in point and point["nextControl"] is not None:
        if "y" in point["nextControl"]:
            point["nextControl"]["y"] = mirror_y(point["nextControl"]["y"])

    # Rotation fields (handle all known variants)
    safe_mirror_rotation(point, "holonomicRotation")
    safe_mirror_rotation(point, "rotation")

def mirror_file(input_path):
    with open(input_path, "r") as f:
        data = json.load(f)

    # Waypoints
    if "waypoints" in data and isinstance(data["waypoints"], list):
        for point in data["waypoints"]:
            mirror_point(point)

    # Rotation targets
    if "rotationTargets" in data and isinstance(data["rotationTargets"], list):
        for rot in data["rotationTargets"]:
            safe_mirror_rotation(rot, "rotation")

    # (Optional) future-proof: handle other objects with positions
    # You can expand this if needed

    # Output file
    base, ext = os.path.splitext(input_path)
    output_path = base + "_mirrored" + ext

    with open(output_path, "w") as f:
        json.dump(data, f, indent=2)

    return output_path

# ===== GUI =====
root = tk.Tk()
root.withdraw()

file_path = filedialog.askopenfilename(
    title="Select a PathPlanner .path file",
    filetypes=[("PathPlanner files", "*.path")]
)

if file_path:
    try:
        output = mirror_file(file_path)
        messagebox.showinfo("Success", f"Mirrored file saved as:\n{output}")
    except Exception as e:
        messagebox.showerror("Error", str(e))
else:
    messagebox.showinfo("Cancelled", "No file selected.")