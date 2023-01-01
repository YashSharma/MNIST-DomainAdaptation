# MNIST-DomainAdaptation

## Introduction

In this project, we implement the mutual information maximization loss proposed in MaNi: Maximizing Mutual Information for Nuclei Cross-Domain Unsupervised Segmentation for MNIST data domain adaptation task. The domain shift issue is a common problem in Deep Learning when the source and target images come from different distributions. Multiple ongoing works are trying to address this problem using different approaches: feature-level alignment, input-level alignment, clustering assumption, auxiliary unsupervised learning, and alternative training. This problem is defined as unsupervised domain adaptation (UDA) when we have access to labeled source images and unlabeled target images. In this work, we used our recently published Mutual Information maximization loss to tackle the domain-shift issue for UDA. For setting up the environment for domain shift, we used MNIST data as the source domain and MNIST-M data as the target domain. MNIST-M data is created by combining MNIST digits with randomly extracted patches of color photos of BSDS500 as their background. This shift from grayscale images to color images with different backgrounds leads to a domain shift in original images and results in a drop in the performance of the model trained on the MNIST dataset. To counter this domain shift drop, we experimented with two approaches, Gradient Reversal and Mutual Information maximization. 

## Methods

All our experiments were conducted in a PyTorch environment. We used 2-layer CNN model with ReLU activation, Max Pooling, and Dropout layer followed by linear layers as our base model. We trained this base model with Adam optimizer and Cross Entropy loss and reported our performances.

For adopting the base model with a gradient reversal layer approach proposed by Ganin et al., we added a branch of linear layers with a gradient reversal layer after the convolutional feature extractor for predicting the domain of the image. The aim of the gradient reversal approach is to force the model to learn discriminative features for the classification task while being invariant with respect to the shift between domains. As a result, the class classifier is trained to predict the class of the source image accurately, and the domain classifier is trained to predict the domain of the image - source or target. An interesting point to note here is that we don't need access to the class of target images to train the model to predict the domain of images. Instead, we need to know if the image comes from the source or target domains. By including a gradient reversal layer in the domain classifier, the model is trained in the opposite direction of gradients required for accurately discriminating the domain of images. Hence in this way, we can negate any learning happening specifically for predicting domains. To summarize the training steps, we train a model with a class-classifier branch with cross-entropy loss and a domain-classifier branch with binary cross-entropy loss for 40 epochs. 

As the final extension, we further fine-tune the previous model by including another branch of linear layers for retrieving the feature projection. The projected representation is used for mutual information maximization between the same class number representation across different domains. Since we don't have access to class labels in the target domain, we use the model trained with a gradient reversal layer for pseudo-labeling the target domain and use only high confidence prediction (probability $\ge$ 0.9) for pseudo-labeling our target data points. The pseudo-labels and feature projection are then used with the Jensen-Shanon divergence bound critic function proposed by Sharma et al. for maximizing the feature similarity between the same-class source and target images. To summarize the training paradigm, first, a model with a gradient reversal layer is trained for 40 epochs with cross-entropy loss for the classifier and binary cross-entropy loss for the domain classifier. This model is further fine-tuned for another 40 epochs with mutual information maximization critics function, class classifier loss, and domain classifier loss. 

## Results

Our base model trained using only class-classifier loss resulted in an accuracy of 53\%. The inclusion of a color background in these MNIST images leads to a substantial drop in performance. By including a domain classifier branch with a gradient reversal layer, we are able to bring up the performance to 71\%. This gain is observed due to the domain classifier forcing the model to learn domain-invariant and task-relevant features. Further, by including MI maximization loss, we are able to improve the performance to 87\%. This gain can be attributed to the MI maximizer successfully increasing the feature similarity between the same class source and target numbers.  
