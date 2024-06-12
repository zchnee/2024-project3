### nerf部分

#### trained model

链接：https://pan.baidu.com/s/1eA_CNqmxcWShfntp8vQlrQ?pwd=6erk 
提取码：6erk

#### Input Video
传入自己拍摄的视频，生成transforms.json：
```
ns-process-data video --data data/VID_20240524_165130.mp4 --output-dir data/output/VID_20240524_165130
```
#### Train Data
```
ns-train nerfacto --data data/output/VID_20240524_165130 --vis viewer --max-num-iterations 30000
```
可以在 http://localhost:7007/ 页面查看可视化训练过程
#### Export Video
训练完毕后，手动添加keyframe，使用网页端的generate command功能获取指令生成视频

```
ns-render camera-path --load-config outputs\VID_20240524_165130\nerfacto\2024-05-24_182248\config.yml --camera-path-filename E:\miniconda3\envs\nerfstudio\data\output\VID_20240524_165130\camera_paths\2024-05-24-18-23-00.json --output-path renders/VID_20240524_165130/2024-05-24-18-23-00.mp4
```

### DreamGaussian部分


#### [Colab](https://github.com/camenduru/dreamgaussian-colab)

- Image-to-3D: [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1sLpYmmLS209-e5eHgcuqdryFRRO6ZhFS?usp=sharing)
- Text-to-3D: [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/camenduru/dreamgaussian-colab/blob/main/dreamgaussian_colab.ipynb)


#### Usage

Image-to-3D:

```bash
### preprocess
# background removal and recentering, save rgba at 256x256
python process.py data/name.jpg

# save at a larger resolution
python process.py data/name.jpg --size 512

# process all jpg images under a dir
python process.py data

### training gaussian stage
# train 500 iters (~1min) and export ckpt & coarse_mesh to logs
python main.py --config configs/image.yaml input=data/name_rgba.png save_path=name

# gui mode (supports visualizing training)
python main.py --config configs/image.yaml input=data/name_rgba.png save_path=name gui=True

# load and visualize a saved ckpt
python main.py --config configs/image.yaml load=logs/name_model.ply gui=True

# use an estimated elevation angle if image is not front-view (e.g., common looking-down image can use -30)
python main.py --config configs/image.yaml input=data/name_rgba.png save_path=name elevation=-30

### training mesh stage
# auto load coarse_mesh and refine 50 iters (~1min), export fine_mesh to logs
python main2.py --config configs/image.yaml input=data/name_rgba.png save_path=name

# specify coarse mesh path explicity
python main2.py --config configs/image.yaml input=data/name_rgba.png save_path=name mesh=logs/name_mesh.obj

# gui mode
python main2.py --config configs/image.yaml input=data/name_rgba.png save_path=name gui=True

# export glb instead of obj
python main2.py --config configs/image.yaml input=data/name_rgba.png save_path=name mesh_format=glb

### visualization
# gui for visualizing mesh
# `kire` is short for `python -m kiui.render`
kire logs/name.obj

# save 360 degree video of mesh (can run without gui)
kire logs/name.obj --save_video name.mp4 --wogui

# save 8 view images of mesh (can run without gui)
kire logs/name.obj --save images/name/ --wogui

### evaluation of CLIP-similarity
python -m kiui.cli.clip_sim data/name_rgba.png logs/name.obj
```

Please check `./configs/image.yaml` for more options.

Image-to-3D (stable-zero123):

```bash
### training gaussian stage
python main.py --config configs/image_sai.yaml input=data/name_rgba.png save_path=name

### training mesh stage
python main2.py --config configs/image_sai.yaml input=data/name_rgba.png save_path=name
```

Text-to-3D:

```bash
### training gaussian stage
python main.py --config configs/text.yaml prompt="a photo of an icecream" save_path=icecream

### training mesh stage
python main2.py --config configs/text.yaml prompt="a photo of an icecream" save_path=icecream
```

Please check `./configs/text.yaml` for more options.

Text-to-3D (MVDream):

```bash
### training gaussian stage
python main.py --config configs/text_mv.yaml prompt="a plush toy of a corgi nurse" save_path=corgi_nurse

### training mesh stage
python main2.py --config configs/text_mv.yaml prompt="a plush toy of a corgi nurse" save_path=corgi_nurse
```

Please check `./configs/text_mv.yaml` for more options.

Image+Text-to-3D (ImageDream):

```bash
### training gaussian stage
python main.py --config configs/imagedream.yaml input=data/ghost_rgba.png prompt="a ghost eating hamburger" save_path=ghost

### training mesh stage
python main2.py --config configs/imagedream.yaml input=data/ghost_rgba.png prompt="a ghost eating hamburger" save_path=ghost
```

Helper scripts:

```bash
# run all image samples (*_rgba.png) in ./data
python scripts/runall.py --dir ./data --gpu 0

# run all text samples (hardcoded in runall_sd.py)
python scripts/runall_sd.py --gpu 0

# export all ./logs/*.obj to mp4 in ./videos
python scripts/convert_obj_to_video.py --dir ./logs
```

Gradio Demo:

```bash
python gradio_app.py
```

#### Citation

```
@article{tang2023dreamgaussian,
  title={DreamGaussian: Generative Gaussian Splatting for Efficient 3D Content Creation},
  author={Tang, Jiaxiang and Ren, Jiawei and Zhou, Hang and Liu, Ziwei and Zeng, Gang},
  journal={arXiv preprint arXiv:2309.16653},
  year={2023}
}
```
