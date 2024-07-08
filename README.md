

# STREAM : A software tool to auto-generate fluidic routing


This project is a Blender addon tool designed to help create Fluidic Networks.

The software is intended to be part of a workflow for fabricating macro-fluidic circuits. Instead of manually wiring logic components, designers can input their designs into the software to generate a 3D printable fluidic network model. Logic components can be placed onto the network to form a functional fluidic circuit, as shown below.

A video tutorial for using the software can be found
[HERE](https://youtu.be/gbxtxueENJk).

![](pictures/process.png)

## Requirements

* [Blender version 3.2](https://download.blender.org/release/Blender3.2/)


## Installation

* Download this repository.
  * The required file is `fluid_circuit_generator.zip` inside the repository folder. The rest can be discarded.
  * DO NOT unzip the file. Blender will automatically extract the contents when it installs addons.
* In Blender, go to Edit -> Preferences -> Add-ons.
  * This window manages the addons installed in Blender.
* Click the install button at the top of this popup window, navigate to the `fluid_circuit_generator.zip` file, and click Install Add-on.
* The installed addon should be shown. Enable the addon by checking the checkbox. You should now be able to start using the Fluid Circuit Generator.


![](pictures/enable_addon.png)
* At this point, there should be a new tab appearing on the sidebar. If you don't see the sidebar, press "N".

![](pictures/side_bar.png)


## User Manual

### Basic Use Case

1. In the Main Panel, press the "Reset Addon" button after each finalized assembly is generated and before starting a new one. This deletes everything.
2. In the Add Component Panel, choose a component for your circuit. Select the circuit component you want to add and press the "Add Object" button.
    * The addon provides a component library that is fully 3D printable with desktop FDM printers (we use Prusa MK3S).
    * The library consists of:
      * Monostable valves, configurable as AND, OR, NOT, INHIBIT logic gates (refer to our paper).
      * Bistable valves, usable as non-volatile memory.
      * Free ends, which are placeholders for open ends to atmospheric pressure.
    * Designers can also create their own circuit components following the "Creating Custom Circuit Components" section of the manual.
    * You can move, rotate, and scale the selected object with this panel. You can also use default Blender operations to maneuver the objects. A simple cheat sheet of basic Blender operations is provided below.
3. In the Add Connection Panel, add connections between components.
    * Choose the component object with the eyedropper or drop-down menu. Choose the port if applicable. Click again to deselect the port.
    * If a connection point is selected correctly, a hollow sphere will appear at the point of selection.
    * Each row represents a connection. To delete a connection, cross out both selections in that row.
4. In the Tubing Properties Panel, customize your tubing.
    * The default values in the software match our actual circuit elements and experiments, but feel free to change them if you wish.
    * The first column contains basic properties. Unit length refers to the resolution/density of tubing routing.
      * Usually, you don't need to adjust this. A larger unit length gives faster performance and permits larger tubing diameters. However, it could cause the system to fail in routing dense tubing.
    * The second column, if enabled, adds a staging block under each logic gate component.
    * The third column adds a custom tip to each tubing tip.
      * The custom tip is appended to the very top of the tubing, and the offset moves the custom tip down.
      * To create your own custom tip, refer to the sample custom tip in the Gate_Library folder in the zip file.
    * After adjusting the properties, press the "Make Preview Tubing" button to create a preview.
5. In the Generate Assembly Panel, check your connections and create the final module.
    * Press the "Preview Connection" button to visualize the connections you made. You can still edit your connections at this step. The preview is refreshed after you hide it.
    * If everything looks correct, click "Confirm Changes."
    * WARNING: The Generate Assembly operation is NOT reversible! Ensure everything looks correct before generating the final module.
    * NOTE: Ctrl+Z won't undo the changes correctly and will cause weird errors.
    * If the final module is not what you want, you NEED to start over by pressing the "Reset Addon" button at the top.
    * If you press the "Generate Assembly" button and an error message pops up, read the error message and make the necessary changes.
6. In the Calculate Propagation Delay Panel, calculate the theoretical propagation delay in the system.
    * This panel is only functional once you generate the final module.
    * Select the port in the same way you choose the connections.
7. In the Set Group Visibility Panel, hide parts of the final module.
    * This is mostly for convenience when exporting the final module.
    * To export the final module, use this panel to hide things you don't want to export.
      * Press "A" to select all visible objects.
      * Go to File -> Export -> STL
      * Remember to check "Selection Only" on the right of the popup window.
      * Choose your export location and export.

### Scripting Support

The software has a hidden layer of API that supports scripting. Users can provide a script to automate the design process described above. The software can also generate a script based on your design/changes.

The "Save Current Progress" button inside the Main Panel generates a script that captures all the tool's statuses. However, only models imported with the "Add Object" button will be saved. The script generated is a `.json` file with the attributes of objects created by the software stored inside.

The "Load Saved Progress" button loads the selected file into the software, which will parse the file and automatically execute a set of commands saved inside the script file.

This functionality enables users to share designs, reduce repetitive work, and even perform version control on their designs.

However, the feature is most useful for designers iterating their designs. Since the software will erase all changes after generating the final tubing, saving a script before generating the tubing results in a smoother workflow.

### Creating Custom Circuit Components

Logic gates need an `.stl` file and a `.json` file with the same name. Use the provided Circuit library as a template and change values in each field. Remember to follow the `.json` file format.

### Fabricating Our Logic Components

The details of the 3D model, printing parameters, and assembly guide can be found in the "keep-everything" branch of this repository.

### Debugging Guide

There are several common types of bugs:

1. **Stuff not selected properly:**
    * Go over the UI and ensure everything is properly selected.

2. **Port position not valid:**
    * Error message: "... is not in the first coordinate ..." or "... is too close to the ground ..."
    * This occurs when a port location is less than a unit away from the ground, resulting in errors later. The software throws an error when it detects this.
    * To fix it, move the violating circuit component up.

3. **Path for some connection not found:**
    * Debugging can be done by bringing up the Blender console window.
    * Look for "ERROR" in the print. Most likely, it will say "Error: Can't retrieve the whole path ..." or "ERROR: Path [...] is too short to be joined by ..."
    * This means there are too many connections to one port, and there is no place to add more connections.
    * The solution is to assign some connections to other ports. For example, A->B & A->C could be changed to A->B->C.
      * This error occurs because when the software joins a tube to an existing tube, the existing one will be divided into two. If you add too many connections to one port, it will run out of length to be joined, resulting in the error "Path [...] is too short to be joined by ..."

### Design Principles for Better Quality Fluidic Networks

1. Avoid having more than two connections to one output.
    * Discouraged: A->B, A->C, A->D, A->E
    * Encouraged: A->B, B->C, C->D, D->E
2. The minimum height for ports is recommended to be 2 * unit + 1.
    * For example, with software presets, unit=7, a good port height would be 15.
3. It's generally a good idea to have all components at around the same height.
4. Keep some spacing between components to ensure they fit physically during assembly.
5. For design examples, refer to the "keep-everything" branch of the repository. A few well-structured examples of circuit design are available (file paths need to be changed before use).

## Blender Operation Cheat Sheet

| Key | Description |
|---|---|
| A | Select all visible objects |
| X | Delete selected object |
| N | Show/Hide sidebar |
| G | Grab/Move selected object |
| R | Rotate selected object |
| S | Scale selected object |
| Tab | Object/Edit mode |

For more detailed information on Blender Hotkeys, visit this
[website](https://www.dummies.com/article/technology/software/animation-software/blender/blender-for-dummies-cheat-sheet-208646/).




