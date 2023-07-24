
# FairCVtest: Testbed for Fair Automatic Recruitment and Multimodal Bias Analysis


![](https://github.com/BiDAlab/FairCVtest/blob/master/data/Figures/CV.PNG)
**Figure 1. Information blocks in a resume and personal attributes that can be derived from each one. The number of crosses represent the level of sensitive information (+++ =  high, ++ = medium, + = low).**


We present a new experimental framework aimed to study how multimodal machine learning is influenced by biases present in the training  datasets. The framework is designed as a fictitious automated recruitment system, which takes a feature vector with data obtained from a resume as input to predict a score  within the interval [0, 1]. We have generated 24,000 synthetic resume profiles including 2 demographic attributes (gender and ethnicity), an occupation, a face image, a name, 7 attributes obtained from 5 information blocks, and a short biography. The 5 information blocks are: education attainment (generated from US Census Bureau 2018 Education Attainment data, without gender or ethnicity distinction), availability, previous experience, the existence of a recommendation letter, and language proficiency on a set of 3 different and common languages. We refer to the information from these 5 blocks as candidates competencies. Each profile has been associated according to the gender  and  ethnicity  attributes  with  an  identity  of  the  DiveFace  database [2], from which we get the face photographs. Short biographies, occupation and names were obtained from [3], and associated according to gender to our CVs. Furthermore, we grouped the occupations in 4 different labor sectors, hence defining a suitabilty attribute representing the affinity degree of each ector with the potential job to which the resume applies (the association has purely academic purposes, without seeking to state the usefulness of any of the occupations)

Each resume is scored using a linear combination of the candidates competencies and the suitability attribute, adding a slight Gaussian noise to introduce a small degree of variability. Since we're not taking into account gender or ethnicity information during the score generation, these become agnostic to this information and should be equally distributed among different demographic groups. Thus, we refer to this target function as blind scores, from which we define two target functions that include two types of bias, Gender bias and Ethnicity bias. Biased scores are generated by applying a penalty factor to certain individuals belonging to a particular demographic group.

We use the pretrained model ResNet-50 to extract feature embeddings from the face photograps. ResNet-50’s last convolutional layer outputs embeddings with 2048 features, so we added a fully connected layer to perform a bottleneck that compresses these embeddings to just 20 features (maintaining competitive face  recognition  performances). Despite being trained exclusively for the task of face recognition, the embeddings extracted by ResNet-50 contain enough information to infer gender and ethinicity, as this information is part of the face attributes. For this reason, we also extract feature embeddings applying to the pretrained model the method proposed in [2] to remove sensitive information, and so obtaining gender/ethnicity agnostic feature embeddings. Regarding the short biographies, a gender-blinded version where explicit gender indicators were removed is included in the bios.

![](https://github.com/BiDAlab/FairCVtest/blob/master/data/Figures/figure_learning_network.png)
**Figure 2. Multimodal learning architecture composed by a Convolutional Neural Network (ResNet-50), an LSTM-based network, and a fully connected network used to fuse the features from different domains (image, text and structured data).  Note that some features are included or removed from the learning architecture depending of the scenario under evaluation.**

The code to replicate our experiments is available [here](https://github.com/BiDAlab/FairCVtest/blob/master/FairCV.py). The code uses common data science libraries (numpy, sklearn, tensorflow, matplotlib and seaborn). Note that we used the [fast-text word embedding](https://fasttext.cc/docs/en/english-vectors.html) in our experiments.

# FairCVdb

This framework present the gender and ethnicity cases as two separate but analogous experiments, maintaining a similar structure in both cases. Of the 24,000 synthetic profiles generated for each experiment, we retain the 80% (i. e. 19,200 CVs) as training set, and leave the remaining 20% (i. e. 4,800 CVs) as validation set. Both splits are equally distributed among the demographic attribute of the experiment. You can donwload the **FairCVdb** [here](https://github.com/BiDAlab/FairCVtest/blob/master/data/FairCVdb.npy). The following example illustrates how to load the information in python:
```python

import numpy as np

fairCV = np.load(os.path.join(data_path, database_file), allow_pickle = True).item()

# Load the information from the train set
profiles_train = fairCV['Profiles Train']
bios_train = fairCV['Bios Train']
names_train = fairCV['Names Train']
blind_labels_train = fairCV['Blind Labels Train']
biased_labels_gender_train = fairCV['Biased Labels Train (Gender)']
biased_labels_ethnicity_train = fairCV['Biased Labels Train (Ethnicity)']
image_list_train = fairCV['Image List Train']

# Load the information from the test set
profiles_test = fairCV['Profiles Test']
bios_test = fairCV['Bios Test']
names_test = fairCV['Names Test']
blind_labels_test = fairCV['Blind Labels Test']
biased_labels_gender_test = fairCV['Biased Labels Test (Gender)']
biased_labels_ethnicity_test = fairCV['Biased Labels Test (Ethnicity)']
image_list_test = fairCV['Image List Test']

```

The **profiles** variable is a numpy array, where each row stores a different resume. You can access each of the i-th profile's attributes as follows:

```python

ethnicity = profiles_train[i,0] # 0 = G1, 1 = G2, 3 = G3
gender = profiles_train[i,1] # 0 = Male, 1 = Female
occupation = profiles_train[i,2] # Discrete variable [0-9]
suitability = profiles_train[i,3] # Discrete variable [0.25, 0.5, 0.75, 1]
educ_attainment = profiles_train[i,4] # Discrete variable [0.2, 0.4, 0.6, 0.8, 1]
prev_experience = profiles_train[i,5] # Discrete variable [0, 0.2, 0.4, 0.6, 0.8, 1]
recommendation = profiles_train[i,6] # Binary variable [0, 1]
availability = profiles_train[i,7] # Discrete variable [0.2, 0.4, 0.6, 0.8, 1]
language_prof = profiles_train[i,8:11] # Discrete variables [0, 0.2, 0.4, 0.6, 0.8, 1]
face_embedding = profiles_train[i,11:31] # 20-dimensional embedding, norm 1
blind_face_embedding = profiles_train[i,31:] # 20-dimensional embedding, norm 1

```
The occupation attribute has the following correspondence: **nurse** (0); **surgeon** (1); **physician** (2); **journalist** (3); **photographer** (4); **filmmaker** (5); **teacher** (6); **professor** (7); **attorney** (8); and **accountant** (9).


# License

Any entity using this dataset agrees to the following conditions:

THIS DATASET IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.


# References

For further information on the benchmark and on different applications where it has been used, we refer the reader to (all these articles are publicly available in the [publications](http://atvs.ii.uam.es/atvs/listpublications.do) section of the BiDA group webpage).

[1] A. Peña, I. Serna, A. Morales, J. Fierrez, A. Ortega, A. Herrarte and M. Alcantara, “Human-Centric Multimodal Machine Learning: Recent Advances and Testbed on AI-based Recruitment”, in Springer Nature Computer Science, Vol. 4, n. 434, 2023.

[2] A. Morales,   J. Fierrez,   R. Vera-Rodriguez, and R. Tolosana, “Sensitivenets: Learning Agnostic Representations with Application to Face Images,” in IEEE Transactions on Pattern Analysis and Machine Intelligence, 2020.

[3] M. De-Arteaga, R. Romanov, H. Wallach, J. Chayes, et al., “Bias in Bios: A Case Study of Semantic Representation Bias in a High-Stakes Setting”, in Proceedings of the ACM Conference on Fairness, Accountability, and Transparency (FAccT), page 120-128, New York, NY, USA, 2019. 

Please remember to reference these articles on any work made public, whatever the form, based directly or indirectly on any part of the FairCVtest benchmark.


# Contact:

For more information contact Aythami Morales, associate professor UAM at aythami.morales@uam.es, or Alejandro Peña, research associate at alejandro.penna@uam.es
