# panda3d-complexpbr
Functional node level scene shader application for Panda3D. complexpbr supports realtime environment reflections for BSDF materials. These reflections are implemented with IBL (Image-based lighting) and PBR (Physically Based Rendering) forward shading constructs. Your machine must support GLSL version 430 or higher. Sample screenshots below.

Featuring support for vertex displacement mapping, SSAO (Screen Space Ambient Occlusion), SSR (Screen Space Reflections), HSV color correction, Bloom, and Sobel based antialiasing in a screenspace kernel shader, which approximates temporal antialiasing. complexpbr.screenspace_init() automatically enables the AA, SSAO, SSR, and HSV color correction. To use the vertex displacement mapping, provide your displacement map as a shader input to your respective model node -- example below in the Usage section.

By default, the environment reflections dynamically track the camera view. You may set a custom position with the 'env_cam_pos' apply_shader() input variable to IE fix the view to a skybox somewhere on the scene graph. This env_cam_pos variable can be updated live afterwards by setting base.env_cam_pos = Vec3(some_pos). The option to disable or re-enable dynamic reflections is available. 

As of version 0.5.2, complexpbr will default to a dummy BRDF LUT which it creates on the fly. complexpbr will remind you that you may create a custom BRDF LUT with the provided 'brdf_lut_calculator.py' script or copy the sample one provided. This feature is automatic, so if you provide the output_brdf_lut.png file in your program directory, it will default to that .png image ignoring the lut_fill input. The sample 'output_brdf_lut.png' and the creation script can be found in the panda3d-complexpbr git repo. For advanced users there is an option to set the LUT image RGB fill values via apply_shader(lut_fill=[r,g,b]) . See Usage section for an example of lut_fill.

As of version 0.5.3, hardware skinning support is provided via complexpbr.skin(your_actor) for models with skeletal animations. See Usage section for an example of hardware skinning.

The goal of this project is to provide extremely easy to use scene shaders to expose the full functionality of Panda3D rendering, including interoperation with CommonFilters and setting shaders on a per-node basis.

![complexpbr_screen_2](https://github.com/rayanalysis/panda3d-complexpbr/assets/3117958/a8a7d360-6b52-4fa8-91f8-31f052421043)
 
![complexpbr_reflections_2](https://github.com/rayanalysis/panda3d-complexpbr/assets/3117958/d6d3867a-6dfb-4512-8a79-de80bf35bc26)

7/6/23 Lumberyard Bistro ([Amazon Lumberyard Bistro | NVIDIA Developer](https://developer.nvidia.com/orca/amazon-lumberyard-bistro))

![bistro_exterior_11](https://github.com/rayanalysis/panda3d-complexpbr/assets/3117958/0cd476bb-d313-41f4-b5ea-d793589711e4)

![bistro_interior_5](https://github.com/rayanalysis/panda3d-complexpbr/assets/3117958/ad75afa7-e1ef-41ea-aae9-4bb1cea54135)

![bistro_exterior_10](https://github.com/rayanalysis/panda3d-complexpbr/assets/3117958/79df6bd6-14d8-4d19-ae5f-45c3418a7607)

6/1/23 Sponza ([Intel GPU Research Samples](https://www.intel.com/content/www/us/en/developer/topic-technology/graphics-research/samples.html))

![sponza_screen_1-Thu-Jun-01-08-16-18-2023-26](https://github.com/rayanalysis/panda3d-complexpbr/assets/3117958/5d6a603f-9da1-49a1-affb-042658f343ed)

![sponza_screen_1-Thu-Jun-01-08-17-47-2023-15](https://github.com/rayanalysis/panda3d-complexpbr/assets/3117958/7fffc0f4-75b3-476b-a328-127d231b9171)

![sponza_screen_1-Thu-Jun-01-06-02-59-2023-22](https://github.com/rayanalysis/panda3d-complexpbr/assets/3117958/913a5263-7750-47c1-b4c4-9f7dace84d6e)

![sponza_screen_2-Thu-Jun-01-05-56-06-2023-591](https://github.com/rayanalysis/panda3d-complexpbr/assets/3117958/b5055164-3235-48fa-86a7-0f6e3222b903)

![sponza_screen_1-Fri-Jun-02-08-54-07-2023-428](https://github.com/rayanalysis/panda3d-complexpbr/assets/3117958/7a5c3f1f-1bb9-4dec-9e92-92dc52f77f29)

## Usage:
```python
from direct.showbase.ShowBase import ShowBase
import complexpbr

class main(ShowBase):
    def __init__(self):
        super().__init__()
         
        # apply a scene shader with PBR IBL
        # node can be base.render or any model node, intensity is the desired AO
        # (ambient occlusion reflection) intensity (float, 0.0 to 1.0)
        # you may wish to define a specific position in your scene where the 
        # cube map is rendered from, to IE have multiple skyboxes preloaded
        # somewhere on the scene graph and have their reflections map to your
        # models -- to achieve this, set env_cam_pos=Vec3(your_pos)
        # you may set base.env_cam_pos after this, and it will update in realtime
        # env_res is the cube map resolution, can only be set once upon first call
        
        complexpbr.apply_shader(self.render)
        # complexpbr.screenspace_init()  # optional, starts the screenspace effects
        
        # apply_shader() with optional inputs
        # complexpbr.apply_shader(self.render, intensity=0.9, env_cam_pos=None, env_res=256, lut_fill=[1.0,0.0,0.0])

        # initialize complexpbr's screenspace effects (SSAO, SSR, AA, HSV color correction)
        # this replaces CommonFilters functionality
        complexpbr.screenspace_init()
        
        # make the cubemap rendering static (performance boost)
        complexpbr.set_cubebuff_inactive()
        
        # make the cubemap rendering dynamic (this is the default state)
        complexpbr.set_cubebuff_active()

        # example of how to apply hardware skinning
        fp_character = actor_data.player_character  # this is an Actor() model
        fp_character.reparent_to(self.render)
        fp_character.set_scale(1)
        # set hardware skinning for the Actor()
        complexpbr.skin(fp_character)

        # example of how to use the vertex displacement mapping
        wood_sphere_3 = loader.load_model('assets/models/wood_sphere_3.gltf')
        wood_sphere_3.reparent_to(base.render)
        wood_sphere_3.set_pos(0,0,1)
        dis_tex = Texture()
        dis_tex.read('assets/textures/WoodFloor057_2K-PNG/WoodFloor057_2K_Displacement.png')
        wood_sphere_3.set_shader_input('displacement_map', dis_tex)
        wood_sphere_3.set_shader_input('displacement_scale', 0.1)
        
        # example of how to set up bloom -- complexpbr.screenspace_init() must have been called first
        screen_quad = base.screen_quad
        
        bloom_intensity = 5.0  # bloom defaults to 0.0 / off
        bloom_blur_width = 10
        bloom_samples = 6
        bloom_threshold = 0.7

        screen_quad.set_shader_input("bloom_intensity", bloom_intensity)
        screen_quad.set_shader_input("bloom_threshold", bloom_threshold)
        screen_quad.set_shader_input("bloom_blur_width", bloom_blur_width)
        screen_quad.set_shader_input("bloom_samples", bloom_samples)
        
        # example of how to customize SSR
        ssr_intensity = 0.5  
        ssr_step = 4.0
        ssr_fresnel_pow = 3.0
        ssr_samples = 128  # ssr_samples defaults to 0 / off
        
        screen_quad.set_shader_input("ssr_intensity", ssr_intensity)
        screen_quad.set_shader_input("ssr_step", ssr_step)
        screen_quad.set_shader_input("ssr_fresnel_pow", ssr_fresnel_pow)
        screen_quad.set_shader_input("ssr_samples", ssr_samples)
        
        # example of how to customize SSAO
        ssao_samples = 32  # ssao_samples defaults to 8
        
        screen_quad.set_shader_input("ssao_samples", ssao_samples)
        
        # example of how to HSV adjust the final image
        screen_quad.set_shader_input("hsv_g", 1.3)  # hsv_g (saturation factor) defaults to 1.0
        
        # example of how to modify the specular contribution
        self.render.set_shader_input("specular_factor", 10.0)  # the specular_factor defaults to 1.0
        
        # example of how to directly fill your BRDF LUT texture instead of providing one in your game folder
        complexpbr.apply_shader(base.render, 1.0, env_res=1024, lut_fill=[1.0,0.0,0.0])  # lut_fill=[red, green, blue]
        
        # if complexpbr.screenspace_init() has not been called, you may use CommonFilters
        # scene_filters = CommonFilters(base.win, base.cam)
        # scene_filters.set_bloom(size='medium')
        # scene_filters.set_exposure_adjust(1.1)
        # scene_filters.set_gamma_adjust(1.1)
        # scene_filters.set_blur_sharpen(0.9)
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
- Function triggers for building new BRDF LUT samplers in realtime
- Installation over pip

## Requirements:

- panda3d

![sponza_screen_1-Thu-Jun-01-05-39-29-2023-111](https://github.com/rayanalysis/panda3d-complexpbr/assets/3117958/f366077b-b6d6-4c4a-896d-f456a06a53d1)

![sponza_screen_3-Thu-Jun-01-05-56-32-2023-657](https://github.com/rayanalysis/panda3d-complexpbr/assets/3117958/23014163-4c7d-4a4d-9f6a-4b874ea364f2)

![sponza_screen_1-Thu-Jun-01-05-55-48-2023-540](https://github.com/rayanalysis/panda3d-complexpbr/assets/3117958/ef2a71c3-169b-428c-a1a9-378c8906c644)

![sponza_screen_1-Thu-Jun-01-08-28-36-2023-104](https://github.com/rayanalysis/panda3d-complexpbr/assets/3117958/4e40e642-f363-4328-bf99-4056f449e28a)

![sponza_screen_2-Thu-Jun-01-08-23-21-2023-1500](https://github.com/rayanalysis/panda3d-complexpbr/assets/3117958/9fbe97e8-d350-480e-bbca-9ef2d5a92b24)

![complexpbr_daytime_screen_1](https://user-images.githubusercontent.com/3117958/235431990-d8ea4364-2526-4739-963c-dce122815f2a.png)

![complexpbr_daytime_screen_2](https://user-images.githubusercontent.com/3117958/235431991-d1f40263-f442-46ed-98a7-056e6186c148.png)

![complexpbr_daytime_screen_3](https://user-images.githubusercontent.com/3117958/235432001-07091c4c-9bc1-4385-81d2-9d50c6fd61b9.png)

![complexpbr_screen_2](https://user-images.githubusercontent.com/3117958/234434099-c6add6ce-578c-4c03-a142-adcf955c14fc.png)

![complexpbr_screen_3](https://user-images.githubusercontent.com/3117958/234434136-9418663d-2304-451b-a318-d3cb4d945a8b.png)

Vertex Displacement Mapping:

![complexpbr_screen_4](https://user-images.githubusercontent.com/3117958/234434178-1e14fa32-2be4-4072-ae15-9ee235d8c036.png)

