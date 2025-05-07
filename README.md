<p align="center">
  <a href="https://ai4life.eurobioimaging.eu/open-calls/">
    <img src="https://github.com/ai4life-opencalls/.github/blob/main/AI4Life_banner_giraffe_nodes_OC.png?raw=true" width="70%">
  </a>
</p>


# Project #21: Automated segmentation of actin filaments in intact cells (EM)

<!-- [![DOI](https://zenodo.org/badge/DOI/)](https://doi.org/) -->

## Overview

The goal of the project was to segment actin filaments in cryo-ET tomograms, building a model that can generalise to future tomograms to produce quality segmentations. This project presents a significant challenge both in terms of the segmentation task and data acquisition. The filamentous structure of actin, along with the technical challenges of working with cryo-ET tomograms, such as, low SNR and distortions caused by the missing wedge effect and tomogram reconstruction errors, make the segmentation task particulary challenging. Data acquisition is also technically difficult and time consuming, however, in order to generalise across tomograms many examples are required. Another difficulty arises from ground truth generation which is also difficult and time consuming. Taking into consideration all the aforementioned challenges and their time scales this project took shape as a series of consultations. Below is a description of the insights gained and the steps taken to progress towards a cryo-ET actin segmentation model.

## Data

In large part the ability of a model to generalise to new unseen data is dependent on the variability and distribution of data observed during training. Over the course of the project a large time investment has been put into obtaining and annotating (semi-automated, more detail below) more tomograms bringing the total to 83 tomograms binned by 4 – pixel size of 13.48 Å and filtered by [spectrum matching](https://github.com/ZauggGroup/DeePiCt/tree/main/spectrum_filter).

### Example Tomogram

The below example shows a cropped tomogram with actin filaments annotated in brown.

<div style="display: flex; justify-content: center;">

<img src="./ims/Tomogram_example.png" style="width: 50%; display: block; margin: auto;" alt="Tomogram Example"/>

</div>


### Simulated Tomogram

We also discussed utilising simulated data and obtained 400 simulated tomograms. Further investigation is required to determine the practical advantages of using such simulations as additional training data, but we beilive they at least provide a useful resource to investigate ground truth generation as discussed in the next section. The below example shows one such simulated tomogram with annotated actin filaments.

<div style="display: flex; justify-content: center;">

<img src="./ims/simulated_tomogram.png" style="width: 50%; display: block; margin: auto;" alt="simulated tomogram"/>

</div>

## Ground truth generation

The first stage of the project was ground truth label generation, this presented a challenge as the manual annotation of a large number of tomograms is prohibitively time consuming. We therefore opted for a semi-automated label generation process using the cylinder and trace correlation functionalities in the [Amira software](https://www.thermofisher.com/de/de/home/electron-microscopy/products/software-em-3d-vis/amira-software.html) to generate initial labels, which could then be manually cleaned to remove spuriously labelled objects such as tomogram edges, membranes or microtubules. While much faster than purely manual annotation, the process is still time consuming hence the need for a fully automated Actin segmentation model. Another drawback to the Amira annotation pipeline is that it relies upon fitting cylinder objects to the filamentous structures in a tomogram, however due to the low SNR, reconstruction errors and the orientation of the actin in the tomogram this can lead to a mismatch between the signal observed in the tomogram and what is being annotated as actin. In order to try and reduce the number of false positive labels we investigated further processing the cleaned Amira annotation, by finding the intersection between an otsu thresholded image and the Amira annotation. For comparison we also manually annotated a reduced set of cropped tomograms using the [Dragonfly software](https://dragonfly.comet.tech/).


### Comparison of Label Generation Methods

The images below compare the labels generated using the semi-automated Amira annotation pipeline and the manual annotation pipeline Dragonfly pipeline. 

<div style="display: flex; justify-content: center;">

<img src="./ims/actin_labels.svg" style="width: 100%; display: block; margin: auto;" alt="Tomogram Example"/>

</div>

The Amira labels and the manual Dragonfly labels produced similar groundtruth as highlighted in the example cropped tomograms above, however, in general the Amira pipeline has a tendency to over annotate relative to the manual annotations. The quality of the ground truth was also discussed during the project, for while the semi automated label generation was more time efficient, it potentially led to some misleading and incorrect pixel annotations. We therefore investigated if any substantial differences in performance were observed when using fully manually annotated ground truth. This was done by manually annotating a small subset of sub tomograms and then comparing the predictions of a model trained using the manual annotations with a model trained using the semi automated annotations. We did not observe any major difference between the performance of the models, however due to the prohibitively slow process of manual annotation we could not train the models on many tomograms making comparison difficult. Another approach being investigated is to rely on simulated data, where accurate groundtruth is inherently available, this would allow a larger set of data to be considered for the comparison as only the faster semi-automated amira labelling is required to be produced.


## Models

For all the Actin segmentation predictions so far the project has been relying upon training a supervised 3D U-net, using the [DeePiCt](https://github.com/ZauggGroup/DeePiCt) repository to provide the architecture and training framework. DeePict provides a convenient and simple to use implementation of the U-net architecture that is controlled via config files, allowing our project collaborator to define and train their own U-net models. An example of one such config file is shown in this repo.

<div style="display: flex; justify-content: center;">

<img src="./ims/Deepict.svg" style="width: 100%; display: block; margin: auto;" alt="DeePict Unet"/>

</div>

## Model Hyperparameters

We advised upon the tuning of hyperparameters for the U-net varying the depth, number of initial features, learning rate and augmentations applied during training. All these experiments were carried out by training on a set of 28 tomograms and testing on a held out test tomogram. 


### Evaluation metric
Evaluation of the segmentation performance was also a key aspect of the challenge and we discussed using more appropriate segmentation performance metrics for filamentous structures, such as the [centerline dice score](https://openaccess.thecvf.com/content/CVPR2021/html/Shit_clDice_-_A_Novel_Topology-Preserving_Loss_Function_for_Tubular_Structure_CVPR_2021_paper.html). A good resource to discover and evaluate suitable performance metrics is [metrics reloaded](https://metrics-reloaded.dkfz.de/).


## Conclusion

This project mostly focused on consulting and providing image analysis support. We advised on the gathering of additional data and suitable generation of ground truth annotations, additionally we helped to set up a pipeline based on DeePict to train and test a supervised U-net based model to perform segmentation and advised on further hyperparmeter tuning. As it currently stands a large and diverse training set of real tomograms has been obtained along with simulated data, a pipeline for semi-automated groundtruth annotation set up and a pipleline for supervised segmentation setup and tested. Actin filaments in cryo-ET tomograms remain a challenging segmentation task, but a strong basis for future work has been established.


## Bibliography

- de Teresa-Trueba, I., Goetz, S.K., Mattausch, A. et al. Convolutional networks for supervised mining of molecular patterns within cellular context. Nat Methods 20, 284–294 (2023). https://doi.org/10.1038/s41592-022-01746-2

- S. Shit et al., "clDice - a Novel Topology-Preserving Loss Function for Tubular Structure Segmentation," 2021 IEEE/CVF Conference on Computer Vision and Pattern Recognition (CVPR), Nashville, TN, USA, 2021, pp. 16555-16564, doi: 10.1109/CVPR46437.2021.01629.

- Reinke, A., Tizabi, M.D., Baumgartner, M. et al. Understanding metric-related pitfalls in image analysis validation. Nat Methods 21, 182–194 (2024). https://doi.org/10.1038/s41592-023-02150-0




