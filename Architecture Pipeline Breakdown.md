# Architecture Pipeline Breakdown

The system is broken down into a series of specialized AI agents and procedural scripts, each mimicking a specific role in an architectural firm. Here is what each agent is doing as the progress bar moves from 0% to 100%.

## Phase 1: Ground Floor Architecture (0% to ~60%)

*   **Planner Agent:** Acts as the Lead Architect. It reads your text prompt (e.g., "3 bedroom house, big kitchen") and creates a mathematical blueprint. It decides exactly how large each room should be, what type of rooms are needed, and which rooms should ideally be next to each other.
*   **Feasibility & Zoning Agents:** Act as the City Planner. They check your property boundary and setbacks to ensure the Planner's massive house can actually legally fit on the lot without breaking local zoning rules. 
*   **Site Plan Agent:** Calculates the exact "buildable footprint" on your land (the boundary minus the setbacks). 
*   **Floor Plan Agent (The Heavy Lifter):** This is the core LLM AI. It takes the room list and the buildable footprint, and plays a massive game of Tetris. It generates the exact 2D polygon coordinates for every single room, ensuring no rooms overlap, there are no weird gaps, and the entire house is fully enclosed. 

## Phase 2: Multifloor Stacking (~60% to ~65%)

*   **Multifloor Agent:** If you requested a 2-story house, this agent takes over. It looks at the exact shape of the ground floor roof, creates a new boundary perfectly matching it, and procedurally generates the upstairs bedrooms, bathrooms, and hallways to fit exactly inside that new footprint.

## Phase 3: 3D Modeling & CAD (65% to 100%)

*   **Elevation & Side Views Agents:** These procedural agents extrude the 2D floor plans into the 3rd dimension. They assign 10-foot ceilings, build exterior walls, and automatically calculate where to punch holes in the exterior walls to place windows.
*   **Roof Plan Agent:** A geometric algorithm that analyzes the final top-floor footprint and mathematically calculates a physically accurate roof (like a Gable or Hip roof) with proper slopes and overhangs to cap the house.
*   **3D Viewer / DXF Agent:** Acts as the 3D Modeler. It feeds all the coordinates to a headless instance of **Blender** (a 3D graphics software) running invisibly in the background. Blender builds a full 3D `.glb` model of the house, adds lighting, and snaps professional "photos" of the front, back, and side elevations. It also generates `.dxf` CAD files for real-world engineers.
*   **Final PDF Agent:** Acts as the Drafter. It collects all the 2D plans, the 3D renders, the roof plan, and the location data, and compiles them into a highly professional, multi-page Architectural Permit Set PDF.
