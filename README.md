# faceswap-GAN
Adding Adversarial loss and perceptual loss (VGGface) to deepfakes' auto-encoder architecture.

## Descriptions
### GAN-v1
* [FaceSwap_GAN_github.ipynb](https://github.com/shaoanlu/faceswap-GAN/blob/master/FaceSwap_GAN_github.ipynb)

  1. Build a GAN model. 
  2. Train the GAN from scratch. 
  3. Use GAN to swap a single face image to target face.
  4. Detect faces in an image using dlib's cnn model. 
  5. Use GAN to transform detected face into target face. 
  6. Use moviepy module to output a video clip with swapped face.  
  
### GAN-v2
* [FaceSwap_GAN_v2_train.ipynb](https://github.com/shaoanlu/faceswap-GAN/blob/master/FaceSwap_GAN_v2_train.ipynb): Detailed training procedures can be found in this notebook.
  1. Build a GAN model. 
  2. Train the GAN from scratch. 
  3. (Optoinal) Detect faces in an image using dlib's cnn model. 
  4. (Optoinal) Use GAN to transform detected face into target face. 
  5. (Optoinal) Use moviepy module to output a video clip with swapped face.
  
* [FaceSwap_GAN_v2_test_img.ipynb](https://github.com/shaoanlu/faceswap-GAN/blob/master/FaceSwap_GAN_v2_test_img.ipynb): Provides `swap_face()` function that require less VRAM.
  1. Load trained model.
  2. Swap a single face image to target face.
  
* [FaceSwap_GAN_v2_test_video.ipynb](https://github.com/shaoanlu/faceswap-GAN/blob/master/FaceSwap_GAN_v2_test_video.ipynb)
  1. Load trained model.
  2. Detect faces in an image using dlib's cnn model. 
  3. Use GAN to transform detected face into target face. 
  4. Use moviepy module to output a video clip with swapped face. 
  
* [faceswap_WGAN-GP_keras_github.ipynb](https://github.com/shaoanlu/faceswap-GAN/blob/master/temp/faceswap_WGAN-GP_keras_github.ipynb)
  - This notebook contains a class of GAN mdoel using [WGAN-GP](https://arxiv.org/abs/1704.00028). 
  - Perceptual loss is discarded for simplicity. 
  - The WGAN-GP model gave me similar result with LSGAN model after tantamount (~18k) generator updates.
  ```python
  gan = FaceSwapGAN() # instantiate the class
  gan.train(max_iters=10e4, save_interval=500) # start training
  ```
* [FaceSwap_GAN_v2_sz128_train.ipynb](https://github.com/shaoanlu/faceswap-GAN/blob/master/FaceSwap_GAN_v2_sz128_train.ipynb)
  - Input and output images have shape `(128, 128, 3)`.
  - Minor updates on the architectures: 
    1. Add instance normalization to generators and discriminators.
    2. Add additional regressoin loss (mae loss) on 64x64 branch output.
  - Not compatible with `_test_video` and `_test_img` notebooks above.
  
### Others
* [dlib_video_face_detection.ipynb](https://github.com/shaoanlu/faceswap-GAN/blob/master/dlib_video_face_detection.ipynb)
  1. Detect/Crop faces in a video using dlib's cnn model. 
  2. Pack cropped face images into a zip file.
 
* Training data: Face images are supposed to be in `./faceA/` and `./faceB/` folder for each target respectively. Face images can be of any size. (Updated 3, Jan., 2018)

## Results

In below are results that show trained models transforming Hinako Sano ([佐野ひなこ](https://ja.wikipedia.org/wiki/%E4%BD%90%E9%87%8E%E3%81%B2%E3%81%AA%E3%81%93), left) to Emi Takei ([武井咲](https://ja.wikipedia.org/wiki/%E6%AD%A6%E4%BA%95%E5%92%B2), right).  
###### Source video: [佐野ひなことすごくどうでもいい話？(遊戯王)](https://www.youtube.com/watch?v=tzlD1CQvkwU)
### 1. Autorecoder baseline

Autoencoder based on deepfakes' script. It should be mentoined that the result of autoencoder (AE) can be much better if we trained it for longer.

![AE_results](https://github.com/shaoanlu/faceswap-GAN/raw/master/readme_imgs/AE_results.png)

### 2. Generative Adversarial Network, GAN (version 1)

**Improved output quality:** Adversarial loss improves reconstruction quality of generated images. In addition, when perceptual loss is apllied, the direction of eyeballs becomes more realistic and consistent with input face.

![GAN_PL_results](https://github.com/shaoanlu/faceswap-GAN/raw/master/readme_imgs/wPL_results_resized.png)

**VGGFace[(GitHub repo)](https://github.com/rcmalli/keras-vggface) perceptual loss (PL):** The following figure shows nuanced eyeballs direction in model output trained with/without PL. 

![Comp PL](https://github.com/shaoanlu/faceswap-GAN/raw/master/readme_imgs/comparison_PL_rev.png)

**Smoothed bounding box (Smoothed bbox):** Exponential moving average of bounding box position over frames is introduced to eliminate jittering on the swapped face. See the below gif for comparison.

![bbox](https://github.com/shaoanlu/faceswap-GAN/raw/master/gifs/bbox_comp_annotated.gif)
  - A. Source face.
  - B. Swapped face, using smoothing mask (smoothes edges of output image when pasting it back to input image).
  - C. Swapped face, using smoothing mask and face alignment.
  - D. Swapped face, using smoothing mask and smoothed bounding box.

### 3. Generative Adversarial Network, GAN (version 2)

**Version 1 features:** Most of features in version 1 are inherited, including perceptual loss and smoothed bbox.

**Segmentation mask prediction:** Model learns a proper mask that helps on handling occlusion, eliminating artifacts on bbox edges, and producing natrual skin tone.

![mask0](https://github.com/shaoanlu/faceswap-GAN/raw/master/readme_imgs/comp_mask_rev.png)

![mask1](https://github.com/shaoanlu/faceswap-GAN/raw/master/gifs/mask_comp1.gif)  ![mask2](https://github.com/shaoanlu/faceswap-GAN/raw/master/gifs/mask_comp2.gif)
  - Left: Source face.
  - Middle: Swapped face, before masking.
  - Right: Swapped face, after masking.

**Mask visualization**: The following gif shows output mask & face bounding box.

![mask_vis](https://github.com/shaoanlu/faceswap-GAN/raw/master/gifs/mask_vis_rev.gif)
  - Left: Source face.
  - Middle: Swapped face, after masking.
  - Right: Mask heatmap & face bounding box.
  
**Optional 128x128 input/outptu resolution**: Increase input and output size to 128x128.

## Requirements

* keras 2
* Tensorflow 1.3 
* Python 3
* OpenCV
* dlib
* [face_recognition](https://github.com/ageitgey/face_recognition)
* [moviepy](http://zulko.github.io/moviepy/)

## Acknowledgments
Code borrows from [tjwei](https://github.com/tjwei/GANotebooks), [eriklindernoren](https://github.com/eriklindernoren/Keras-GAN/blob/master/aae/adversarial_autoencoder.py), [fchollet](https://github.com/fchollet/deep-learning-with-python-notebooks/blob/master/8.5-introduction-to-gans.ipynb), [keras-contrib](https://github.com/keras-team/keras-contrib/blob/master/examples/improved_wgan.py) and [deepfakes](https://pastebin.com/hYaLNg1T). The generative network is adopted from [CycleGAN](https://github.com/junyanz/pytorch-CycleGAN-and-pix2pix).
