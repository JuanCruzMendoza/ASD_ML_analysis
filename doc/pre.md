# Pre-analysis
--------------

## Introduction
Autism spectrum disorder (ASD) is a lifelong neurological and developmental disorder that often affects communication and interaction with other people and is characterized by restricted interests and repetitive behaviours. Due to a lack of biological evidence for its causes, the diagnosis is only made by evaluating the patient’s behaviours.

Lately, there have been a number of studies intending to extract a biomarker for ASD from resting state fMRI, using functional connectivity and machine learning (ML) models, with the ultimate goal of helping to diagnose this disorder. It is worth noting that many of these studies were possible due to the creation of an open-source and multi-site neuroimaging database called ABIDE (Autism Brain Imaging Data Exchange) $^{(1)}$, which has more than 1000 subjects. 

In this work, we will apply a method similar to these recent studies in order to build a machine learning classifier to distinguish fMRI from individuals with ASD and typical development (TD), and also try to interpret the resulting model, attempting to find under- and over-connectivity in different brain regions compared to TD controls.



## Methodology
Firstly, we will only use the labelled resting state fMRI data (specifically the BOLD signals) from ABIDE, without any demographic information. There are 871 rs-fMRI samples, after a quality screening. All this data is preprocessed with the Configurable Pipeline for the Analysis of Connectomes (CPAC) pipeline $^{(2)}$.

In order to train our machine learning models, we will use static functional connectivity, which is the temporal correlation from different areas in the brain across the entire BOLD signal. To estimate it, we calculate the correlation matrix of each subject’s time-series signal, using Pearson’s correlation, which is the most popular measure for this purpose.

On the other hand, we opted for the Bootstrap Analysis of Stable Clusters (BASC) atlas, particularly the version with 197 regions of interest, since the combination of this atlas and Pearson’s correlation measure has produced optimal results in other studies $^{(3)}$.

Considering that the computation of these features have already been made multiple times, we decided to use the public data from another study (Xin Yang et al., 2022) $^{(4)}$.

Due to the symmetry of Pearson’s correlation, we only need the upper or lower triangle of the correlation matrix, without the diagonal. However, there is still a large number of features (197 x 196 / 2 = 19306 features) compared to the few samples we have, which would probably result in over-fitting. For this reason, we perform feature selection with two stages. Firstly, we compute the ANOVA F-value score for each feature to select the most relevant ones and make the next stage more efficient. Afterwards, a recursive feature elimination method (RFE) with 5-folds cross-validation and a linear SVM classifier is used to discard less significant features $^{(5)}$. This method also allows us not to lose the features’ semantic, which is not possible with PCA or Linear Discrimination. It should be noted that this process is only done in the training data to avoid leakage, and then we remove the features from the test data to be evaluated with the main ML model.

We will assess the performance of the ASD-DiagNet model, which is currently a state-of-the-art technique for ASD classification that achieved a 82% accuracy $^{(6)}$, and consists of training an autoencoder and a single-layer-perceptron simultaneously. In this model, following the original paper, instead of the feature reduction method described above, we make an average between all connectivity arrays, select the 25% biggest and 25% lowest features values and create a mask, keeping half of the features. 

As well as that, we will compare the ASD-DiagNet model with two traditional ML models: kernel SVM and Logistic Regression. Finally, we will also try a XGBoost model, which is an ensemble learning algorithm based on decision trees.

The training process will be made using a 5 fold nested cross-validation method: for each split, we perform feature selection on the training data, we use Grid Search with 5 fold cross-validation in order to optimize the hyperparameters of the model, we remove features from test data based on previous feature selection and evaluate it with the best model from Grid Search. Finally, having the error for each of the five folds, we estimate the overall performance of the model making an average.

Apart from evaluating the accuracy of the resulting models, we will compare specificity (classifier’s ability to capture true positives), sensitivity (ability to identify true negatives), ROC curve and AUC.

Lastly, to interpret the models, we will adopt the Sapley Additive explanations (SHAP) method, which quantifies the importance of each feature, regardless of the model. This will help us identify which brain connections have a bigger impact on the classifier and which features differ from ASD and TD.

### References

(1) https://fcon_1000.projects.nitrc.org/indi/abide/

(2) http://preprocessed-connectomes-project.org/abide/Pipelines.html

(3) Xin Yang, Ning Zhang, Paul Schrader, “A study of brain networks for autism spectrum disorder classification using resting-state functional connectivity”, Machine Learning with Applications, Volume 8, 2022.

(4) https://drive.google.com/drive/folders/1hrGzfM2QH6ECqUWp23CRi0cfEtqMVGNm

(5) Method extracted from: ElNakieb, Y.; Ali, M.T.; Elnakib, A.; Shalaby, A.; Mahmoud, A.; Soliman, A.; Barnes, G.N.; El-Baz, A. Understanding the Role of Connectivity Dynamics of Resting-State Functional MRI in the Diagnosis of Autism Spectrum Disorder: A Comprehensive Study. Bioengineering 2023.

(6) Eslami Taban, Mirjalili Vahid, Fong Alvis, Laird Angela R., Saeed Fahad, “ASD-DiagNet: A Hybrid Learning Approach for Detection of Autism Spectrum Disorder Using fMRI Data” , Frontiers in Neuroinformatics, Volume 13, 2019.  

