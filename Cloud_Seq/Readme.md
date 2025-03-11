# Cloud Sequencer Component (.tox) Documentation

## Overview

This component streamlines loading, arranging, and sequencing pointclouds in TouchDesigner. It imports normalized .ply files, arranges them in a layout texture, and performs LOD-style selection from two textures. It supports dynamic blending between pointclouds, enabling continuous pointcloud paths.

[Motivation and examples](https://auratyk.com/community/cloud-seq/)

**Features**

- **Pointcloud Loading & Normalization:**
    
    Imports .ply files and normalizes the data.
    
- **Layout Texture Arrangement:**
    
    Arranges multiple pointclouds into a layout texture for spatial alignment.
    
- **LOD-Style Point Selection:**
    
    Uses the camera direction to select and blend points efficiently from two textures.

- **Dynamic Sequencing Parameters:**
    
    Custom parameters control sequencing, blending thresholds, and source swapping.
    

## Technical Details
> This core logic is run on a shader inside the comp.
1. **Determine Particle Roles:**
    - A boolean (e.g., via a uniform `uSwap`) determines which scan is “disappearing” (structure preserved) and which is “incoming” (blended in).
    - Particle data is read from two corresponding textures.
2. **Camera Orientation:**
    - The camera’s forward direction is computed using its position (`uCamPos`) and a look-at target (`uLookAtPos`):
        
        ```glsl
        vec3 computeForwardFromLookAt(vec3 camPos, vec3 lookAtPos) {
            return normalize(lookAtPos - camPos);
        }
        
        ```
        
3. **Depth-Based Blending:**
    - For each particle (from the disappearing scan), compute depth along the camera’s forward vector.
    - If depth < 0 (particle is behind the camera), its attributes remain unchanged.
    - For depth ≥ 0, blend attributes between the disappearing and incoming scans using a smoothstep function between `uBlendNear` and `uBlendFar` thresholds:
        
        ```glsl
        float ComputeBlendWeight(float depth) {
            return smoothstep(uBlendNear, uBlendFar, depth);
        }
        
        ```
        
4. **Output:**
    - The final blended particle data (position, age, color, alpha) is written to output textures for rendering.

## Typical Workflow

1. **Load Pointclouds:**
    - Add .ply files in the provided PLY_Cloud components. (Inside the Cloud_Seq component)
    
        This is a manual process. Copy Paste the “PLY_Cloud…” comp, add your ply file and connect the first wire to the switch_pos TOP. The second wire will be added automatically.
2. **Arrange Pointclouds:**
    - Use Layout Mode to view a downscaled layout of up to 4 clouds.
    - Adjust transform parameters in PLY_Cloud comps to spatially align pointclouds.
3. **Sequencing Setup:**
    - Position the camera where you would like it to be at the start of the sequence.
    - Initialize the sequencer:
        - Set desired pointclouds and reset Cloud* parameters.
        - Define the LOD distance with Blend Near.
4. **Transitioning Pointclouds:**
    - As the camera moves along your chosen path, you will need to update the sequencer:
        - The cloud you want to "disappear" (Structure preserved - set by "Preserve A/ B")
        - The cloud you want to "appear" (Gradually Revealed)
    - Continue alternating between scans (e.g., A → B → A) as needed.
    Typically you would go from A0 → B0 → A1 → B1 → A0 etc
5. **Camera Path Integration:**
    - Optionally bind the camera path to the sequencer’s Progress parameter to drive transitions dynamically.
