20200805 Yingtaoさんご講演
# Dealing with different data modalities with machine learning: in case of culture specific image datasets and network
<img src="./image_yingtao/20200805125715.png">

## Outline
- KaoKore
    - A machine learning friendly dataset of facial expression in pre-modern Japanese art
- FastRP
    - A network embedding method almost as perfomant as DeepWalk but runs 4,000 times faster

### KaoKore
A pre-modern Japanese Art Facial Expression Dataset

## Agenda
- Pre-modern Japanese Literature
- Kaokore Dataset
- Qquantitative Results
- Qualitative / Creative Results

### Pre-modern Japanese Literature
Cursive texts telling the story + illustrations explaining the story

### Make humanities research benefit more from machine learning
「機械学習は人文学研究にどう使えるのか（または使えないのか）について、考えを巡らせてみるのもよいかもしれません」

### Machine learning accelerates research
- Benefit from ML: Cursive Writing Recognition

- Missing elephant in the room:
    - Illustrations
    - Especially faces

- Also to benefit from ML?

### 顔貌コレクション（KAO KATACHI KOREKUSHON / Facial Expression Collection）
dataset providing cut out and collected parts of faces appearing in art works for researches into art history, Especially in the study of artisitic style.

This Project currently focues mostly on illustrations from Late Muromachi Period (~13th century) to Early Edo Period (~17th century)

### Kaokore Dataset
make a machine learning friendly dataset!

#### Kaokore dataset
- Totally 8573 256x256 RGB images
- Full metadata labeled by experts
- For supervised learning, classes labels
    - gender (male / female)
    - status (noble 貴族/ warrior 武士/ incarnation 化身/ commoner庶民)
- For supervised learning, official training/ testing dataset.

### Quantitative Results => Supervised machine learning
vgg, alexnet, ... inception v3 などで実験

### Qualitative / Creative Results
GAN/ Style Transfer/ Neural Painting

#### GAN
A Style-Based Generator Architecture for Generative Adversarial Networks

#### (Intrinsic) Style Transfer
<img src="./image_yingtao/20200805132312.png">
Nakano: Neutral Painters: A learnined differentiable constraint for generating brushstroke paintings

#### Creativity: Neural Painting

<img src="./image_yingtao/20200805132651.png">
Huang et al: Learning to Paint with Model-based Deep Reinforcement Learning

### Paper & open sourced data:
<img src="./image_yingtao/20200805225124.png">
https://github.com/rois-codh/Kaokore

<img src="./image_yingtao/20200805225139.png">

----

## FastRP
<img src="./image_yingtao/20200805225150.png">
Fast and Accurate Network Embeddings via Very Sparse Random Projection

FastRP: a network embedding method almost as perfomant as DeepWalk but runs 4,000 times faster

<img src="./image_yingtao/20200805225159.png">
### Agenda
- Background and Motivation
- FastRP - Fast NE via Very sparce random Projection
- evaluation

### Background 
Q. What is network embedding?
A. Low-dimensional latent representation of nodes in a network

<img src="./image_yingtao/20200805225208.png">

<img src="./image_yingtao/20200805225217.png">

<img src="./image_yingtao/20200805225226.png">

<img src="./image_yingtao/20200805225300.png">

### DeepWalk's Answer to Two Questions

<img src="./image_yingtao/20200805225310.png">

### DeepWalk is Good but Slow. Why?

<img src="./image_yingtao/20200805225317.png">

Drawback #1: huge number of samples needed

Drawback #2: Skip-gram is not that fast

### Motivation: Scalability in Existing Methods

<img src="./image_yingtao/20200805225326.png">

## FastRP - Fast NE via Very sparce random Projection

<img src="./image_yingtao/20200805225335.png">

### Very Sparse Random Projection: Scalable Dimension Reduction

<img src="./image_yingtao/20200805225344.png">

### Random Projection: Input Matrix

<img src="./image_yingtao/20200805225352.png">

### Element-wise transformation is important

<img src="./image_yingtao/20200805225401.png">

<img src="./image_yingtao/20200805225408.png">

<img src="./image_yingtao/20200805225415.png">

<img src="./image_yingtao/20200805225421.png">

<img src="./image_yingtao/20200805225428.png">

### evaluation
#### Multi-lavel Node Classification

<img src="./image_yingtao/20200805225435.png">

- FastRP has best or almost the best perfomance
- FastRP is more than 4,000 times faster than DeepWalk

<img src="./image_yingtao/20200805225442.png">

<img src="./image_yingtao/20200805225452.png">

<img src="./image_yingtao/20200805225500.png">

---
## QA

- Approach to animes?
    - For the animes, I thinks one of the recent work is pretty interesting: https://lllyasviel.github.io/AppearanceEraser/ -- just an example, there are many more.

- Do you think graph nns can enable abstract reasoning?
    - This is a cool direction that is being studied intensively recently. For example, see  https://logicalreasoninggnn.github.io for the very recent advances.

- Example of manipulating the generation for Anime Characters: https://waifulabs.com/

- https://pkhungurn.github.io/talking-head-anime/ => (For anime) looks like 3D but not really 3D

- https://magenta.tensorflow.org/
