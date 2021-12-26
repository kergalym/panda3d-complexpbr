# panda3d-complexpbr
Functional node level shader application for Panda3D.

This is an early prototype for applying prebuilt scene shaders with an OpenGL 330 PBR workflow using Panda3D. The module assumes you have .gltf files prepared with fully complete 4-slot metal-rough texturing on a BSDF Principled Node. Currently, the shaders are modified and prearranged versions based on https://github.com/Moguri/panda3d-simplepbr

The goal of this project is to provide extremely easy to use scene shaders to expose the full functionality of Panda3D rendering, including interoperation with CommonFilters and setting shaders on a per-node basis. 

## Usage:
```python
from direct.showbase.ShowBase import ShowBase
from direct.actor.Actor import Actor
import complexpbr
import gltf

class main(ShowBase):
     def __init__(self):
         super().__init__()
         gltf.patch_loader(self.loader)
         
         # apply a scene shader with no hardware skinning
         complexpbr.apply_shader(node=render, scene=True)
         
         # apply an "Actor shader" for hardware skinning
         your_character = Actor(loader.load_model('character.gltf'))
         your_character.reparent_to(render)
         complexpbr.apply_shader(node=your_character, skin=True)
```
## Building:

The module may be built using setuptools. 
```bash
python3 setup.py bdist_wheel
```
```bash
pip3 install 'path/to/panda3d-complexpbr.whl'
```
## Installing with PyPI:

To-do.

## Future Project Goals:

- Implementing a GLSL raytracer or pathtracer scene shader
- Installation over pip

## Requirements:

- panda3d-gltf
