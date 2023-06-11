# rime2flower generator

This is a project using cycleGAN to realize the image to image transformation. Referenced sources are at the end of this Readme file.

## Project Idea

Since the rime ice has been a metaphor for an ice flower. Based on my observations in real life, I perceive that some people around me try to close off their emotions, personalities, sharp edges, etc. with ice. From my perspective, everyone can bloom a unique flower, and sometimes you don't have to suppress yourself in order to conform to your surroundings, just let yourself bloom. I will test the outcome by loading new rime photos and see if it can bloom into a flower shape.

## Network Architecture
The cycleGAN model has 2 generators and 2 discriminators. 

### Discriminator

•	Instance normalization is used, instead of batch normalization. 

•	For the discriminator networks, 70 x 70 PatchGANs (to classify whether 70 x 70 overlapping image patches are real or fake) is used for best performance. 

•	The discriminator is trained directly on generated (fake) and real images.

<img src="https://git.arts.ac.uk/storage/user/709/files/6d498e9a-02f6-4fac-b426-3847a4ff7e41" width="90%"></img> 

### Generator

•	The generator uses resnet block, which concatenate the output with the input.

•	The model has 2 generator models:

1.Generator-A: Generates images for the first domain (flower paintings).

2.Generator-B: Generates images for the second domain (rime photos).

•	Inputs to generators come from the other domain:

Rime –-- Generator-A –-- Flower

Flower –-- Generator-B --- Rime

•	Each generator has a corresponding discriminator model (Discriminator A and B)

Flower --- Discriminator-A ---Real/Fake

Rime --- Generator-A (Generates Flower) --- Discriminator-A --- Real/Fake

Rime --- Discriminator-B ---Real/Fake

Flower --- Generator-B (Generates Rime) --- Discriminator-B --- Real/Fake

•	Cycle consistency: Generator models are trained to reproduce the original image. Use generates image as input to the corresponding generator model and compare the output image to the original image.

•	Identity mapping: When an image from the other domain is provided to the Generator, it is expected to generate the same image. The identity mapping loss helps preserve the color of the input photos.

<img src="https://git.arts.ac.uk/storage/user/709/files/cc5cdbb8-207d-41dd-8dfa-e0152a430f54" width="90%"></img>
<img src="https://git.arts.ac.uk/storage/user/709/files/3ae39fe4-15a7-4e3a-887e-a74f91b774c7" width="90%"></img> 

### Combine the models

*When loading the real samples, it is important to use ‘tanh’ which goes from -1 to 1 to scale the images.

#### Generator training
Generator is trained via the combined model to minimize 4 different losses.

1.Adversarial loss (L2 - MSE): Minimize the loss predicted by the discriminator for generated images marked as ‘real’

2.Identity loss (L1 - MAE): Output the source image just as it is without translation.

3.Cycle loss forward (L1 - MAE): Regeneration of a source image when used with the model (Rime to Flower).

4.Cycle loss backward (L1 - MAE): Regeneration of a source image when used with the model (Flower to Rime).

*MSE = Main Squared Error, MAE = Main Absolute Error

#### Discriminator training
•	Setting the label to 1: tell the discriminator the samples are real.

•	Setting the label to 0: tell the discriminator the samples are fake.

*Save the trained model to avoid repeating the training step each time running the code.

<img src="https://git.arts.ac.uk/storage/user/709/files/1cb1621d-0e2d-4fbd-a13a-2267a648e35a" width="90%"></img>

### Training
Load 50 images to each poolA and poolB as an input to discriminator.

For each iteration, generate X_realA (real flower), y_realA, X_realB (real rime), y_realB; X_fakeA (fake flower), y_fakeA, X_fakeB (fake rime), y_fakeB.

Update the images to the pool and train the model.

The generated results and models are saved every 500 epochs to record the progress. 


## Load Dataset & Train Model

Load images from dataset. Since I ran the code on GoogleColab, I connected the path to GoogleDrive to get the uploaded dataset which cotains the images.

testA: 168 flower paintings, testB: 168 rime photos, trainA: 1632 flower paintings, trainB: 992 rime photos

Once the images are loaded, the model could be trained. The trained models are saved as ‘g_model_AtoB_002500.h5’ and ‘g_model_BtoA_002500.h5’.

<img src="https://git.arts.ac.uk/storage/user/709/files/95b51998-18ee-43ee-b630-b9f52f93b810" width="90%"></img> 

Here are some test outputs.

<img src="https://git.arts.ac.uk/storage/user/709/files/9f4323b7-b49d-4df2-bb10-16129fade4e4" width="90%"></img>
<img src="https://git.arts.ac.uk/storage/user/709/files/fbb9a8fc-6690-4d91-b3fa-ed44514bbc72" width="90%"></img>

## Now it’s time to do some experiments with new images

These are some new rime photos I found online. By loading the images into the model, scaling them to 1 to -1, some beautiful outputs are generated.

<img src="https://git.arts.ac.uk/storage/user/709/files/e9d82f0b-f5b7-44a9-a59c-da1bbe49daae" width="90%"></img> 
<img src="https://git.arts.ac.uk/storage/user/709/files/b5dbae0d-b2c9-4bbd-8d92-9192cc8b96eb" width="90%"></img>

<img src="https://git.arts.ac.uk/storage/user/709/files/85ff2e7c-9e51-4f16-be6e-0df88ea08c27" width="30%"></img> <img src="https://git.arts.ac.uk/storage/user/709/files/70af4ae0-2047-45b7-9a42-195d5792f15b" width="30%"></img> <img src="https://git.arts.ac.uk/storage/user/709/files/b1b9264f-6dcd-4a96-ba26-f6bb1290b916" width="30%"></img> <img src="https://git.arts.ac.uk/storage/user/709/files/292e2862-2f28-4d33-b743-461d49e99744" width="30%"></img> <img src="https://git.arts.ac.uk/storage/user/709/files/1a66afc8-1772-4560-b52d-349e7fc1a38f" width="30%"></img> <img src="https://git.arts.ac.uk/storage/user/709/files/e785a610-487a-4abe-8850-9c484bbd3b0e" width="30%"></img> <img src="https://git.arts.ac.uk/storage/user/709/files/2f9659f5-320d-4683-8a8b-572b0422f921" width="30%"></img> <img src="https://git.arts.ac.uk/storage/user/709/files/770d2c43-f9ce-4acd-8851-f369a9f2493e" width="30%"></img> <img src="https://git.arts.ac.uk/storage/user/709/files/12c11946-9eed-4f74-9f24-17f01720870f" width="30%"></img> <img src="https://git.arts.ac.uk/storage/user/709/files/9d82b76f-3281-4246-a690-3c8d366a8795" width="30%"></img> <img src="https://git.arts.ac.uk/storage/user/709/files/867f10b5-e70e-43a4-a101-e6dc02898b8f" width="30%"></img> <img src="https://git.arts.ac.uk/storage/user/709/files/1f3690c0-8e38-4394-81e3-1ad3f7dd4a9b" width="30%"></img> 

## Evaluation Method

The performance of the project is evaluated based on both the generated image and the reconstructed image.

In terms of the AtoB model, the performance of the model can be assessed by comparing how well the generated images and the input flower paintings agree in terms of pattern colour and style. In this project, the generated images generally met this criterion and showed some surprising outputs.
Another evaluation criterion was the degree of reproduction by comparing the reconstructed images with the input rime photos. The closer the two are, the better the model performs. In this project again, the reconstructed images largely reproduced the colours of the original image and did not allow any stray colours to blend in, although the contrast was slightly higher than in the original image.

Overall, I think this project is excellent in terms of achieving the core idea and meeting the objectives.

## References

### Datasets

Wiki-Art : Visual Art Encyclopedia (folder: flower paintings)
https://www.kaggle.com/datasets/ipythonx/wikiart-gangogh-creating-art-gan

Weather Image Recognition (folder: rime)
https://www.kaggle.com/datasets/jehanbhathena/weather-dataset 


### Referenced code


instancenormalization.py 
      https://github.com/keras-team/keras-contrib/blob/master/keras_contrib/layers/normalization/instancenormalization.py

cycleGAN_model.py 
      https://github.com/bnsreenu/python_for_microscopists/blob/master/253_254_cycleGAN_monet2photo/254-cycleGAN_model.py

cycleGAN_monet2photo.py
      https://github.com/bnsreenu/python_for_microscopists/blob/master/253_254_cycleGAN_monet2photo/254-cycleGAN_monet2photo.py


### Tutorial
'A Gentle Introduction to CycleGAN for Image Translation'
      https://machinelearningmastery.com/what-is-cyclegan/

'Unpaired image to image translation​ using cycleGAN in keras'
      https://www.youtube.com/watch?v=2MSGnkir9ew


### Papers

'Image-to-Image Translation with Conditional Adversarial Networks'
      https://openaccess.thecvf.com/content_cvpr_2017/papers/Isola_Image-To-Image_Translation_With_CVPR_2017_paper.pdf

'Toward Realistic Image Compositing with Adversarial Learning'
      https://openaccess.thecvf.com/content_CVPR_2019/papers/Chen_Toward_Realistic_Image_Compositing_With_Adversarial_Learning_CVPR_2019_paper.pdf
