# System Generation Specification

A System is defined as a collection of Bodies - a class prototype. Bodies may be the children of other bodies.
In the present model of the Database, no restriction on which kind of bodies may be the parent of other bodies - as that is an application rule.
Functionally, there is **planned** to be four or five types of bodies (which all uniquely) subclass, defining additional features and behaviours.
The kinds of Bodies are shown below:
- **VOID**
  - Default; not intended for production; instead acting as a failsafe in the case of an integrity violation; does not define any additional parameters.
- **Star**
  - Defines the key properties of a star, including:
    - Temperature (RGB and Kelvin),
    - Mass (in ratio with the Solar Mass of the Sun; 1=Mass of the Sun),
    - Radius (in ratio with the Solar Radius of the Sun; 1=Radius of the Sun),
    - Star Class (one of \[M,K,G,F,A,B,O...\], which defines the archetype and limits of the above temperature, mass and radius; as well as the proportion in the milky-way for auto-generation purposes.)
  - (*) Note that Star Class can be extended to reflect more specialized types of stellar pheneomena- as it is another table.
- **Planet**
  - Define the key properties of a planet (of any size, including dwarf)
    - d
- **Asteroid Belt**
  - Defines the key properties of an Asteroid belt.
  - Due to its unique state as a circular belt of asteroids (simplified from eccentric-) its anchor is always at the midpoint of the circle.
  - This means that angle does not matter to an asteroid belt, instead only distance (the radius of the circle) is material.
  - The properties of an asteroid belt can be summarised as:
    - Composition; and
    - 
- **Trojan Belt**



A System Hierarchy may look like (ignoring realistic requirements):
```
System XYZ
|
| -> Star 1 (systemid=System XYZ, ...)
|     | -> Mercury (*,parent=Star 1, ...)
|     | -> Venus (*,parent=Star 1, ...)
|     | -> Earth (*,parent=Star 1, ...)
|           | -> Luna (*,parent=Earth, ...)
|     | -> Mars (*,parent=Star 1, ...)
|           | -> Pheibos (*,parent=Mars, ...)
|              ...
|  
|
| -> Star 2 (systemid=System XYZ, ...)
|     | -> Saturn (*,parent=Star 2, ...)
|           | -> Europa (*,parent=Saturn, ...)
|     | -> Jupiter (*, parent=Star 2, ...)
| -> Nemesis (systemid=System XYZ) [PLANET]
```
All Objects in a system have a `Distance` and `Angular_Offset_Deg`, 
which define the distance (in simulation units) 
and angular rotation around the major body that it is parented to. 
- In the case of **Asteroid Belts**, `Distance` shall be the distance offset from that of the 
Belt (positive or negative). Bodies which are the direct children of an Asteroid Belt **do not generate orbit lines**. 
- In the case of a body (whose **parent is an Asteroid Belt**, i.e. '**Major Body**') which has child bodies ('**Minor Bodies**'), orbit lines will be generated for those Minor Bodies around the Major body.
- In the case of **Planets** (or '**Major Bodies**'), `Distance` shall be the **distance from the body or star it is parented to**.
- In the case of **Stars**, `Distance` shall be the distance from the **Barycenter** of the system (virtual midpoint in simulation terms). **Stars do not have orbit lines around the barycenter (for obvious reasons)**

**Orbit Lines** are circular, no eccentricity, procession or other real-life orbital properties. This is for the sake of ease of simulation.

# Encoding
The Information Above is encoded over three tables: `SystemIndexTB`, `StarsTB` and `BodiesTB`
Encoding is based on the API scopes.
- `/api/open/sysmap/get/system/<systemid>`
```json
{
   "status": 200,
   "message": "OK",
   "system_id": "System_XYZ",
   "datetime": "{datetime osi object repr}",
   "System_XYZ": {
      "Gal_X": 0,
      "Gal_Y": 0,
      "Gal_Z": 0,
      "Preferred_Name": "Epsilon Eridani",
      "Arity": 1,
      "Temperature": [0,0,0]
   }
}
```
Note that Arity and Temperature are determined (calculated) fields by the number of stars in the system and composite spectrum; it's not present in the `SystemTB` datamodel.


- `/api/open/sysmap/get/barycenter/<systemid>`
```json
{
   "status": 200,
   "message": "OK",
   "systemid": {
      "Star 1": {
         "Name": "Sol",
         "Type": "Star",
         "Star_Class": "G",
         "Temperature": [200,100,100],
         "Distance": 0,
         "Angular_Offset_Deg": 0,
         "Child_Bodies": ["Murcury","Venus","Earth"]
      },
      "Star 2": {
         "Name": "Nemesis",
         "Type": "Star",
         "Star_Class": "Dwarf",
         "Temperature": [255,25,25],
         "Distance": 100,
         "Angular_Offset_Deg": 0,
         "Child_Bodies": ["Mars", "Jupiter", "Saturn"]
      },
      "Nemesis": {
         "Name": "Nemesis",
         "Type": "Planet",
         
      }
   }
   
}
```

# Decoding (from Json from Flask REST API)
This will be decoded in series on the front end in the order shown below (to ensure referential integrity):
1. Identify System
2. Get Stars in System
3. Position Stars around the System 'Barycenter' (distance from barycenter and angle from 'north')
4. Get Bodies in System (prefer direct children of a Star; a "*Major Body*")
5. Position Major Bodies around Stars us
6. For each body `B` (whose parent is a star);
   1. Get bodies whose parent is body `B`
   2. For each body in 