---
nav_cat: blog
layout: blog_post
title: "Exploring how to train ML models on Apple Silicon efficiently"
---

While trying to train an image classification model locally on an Apple Silicon system with TensorFlow and Keras, I noticed a few peculiarities between trying to run the training on the CPU vs. on the GPU that I hadn’t found documented elsewhere online. This post is a documentation of my investigations into determining how floating point precision behaves and manifests while training TensorFlow/Keras models on M1 and M2 generations of Apple Silicon.

### Technical Details

A few technical details about the tests I describe here: These tests were conducted on TensorFlow v2.18. I used v1.2.0 of the [tensorflow-metal plugin](https://developer.apple.com/metal/tensorflow-plugin/) developed by Apple, which accelerates the training of TensorFlow machine learning models on the Apple Silicon architecture, particularly using Metal on Mac GPUs. The data plotted in this post was generated while training the described ML model on an M1 Pro system. I was also able to reproduce similar behavior on a system equipped with the M2 Max chip, but have not yet tested it on chips with newer GPU architecture generations. Although this seems to be rooted in a hardware limitation or limitation in the Metal integration with TensorFlow, for thoroughness I was able to reproduce this behavior in both macOS v15.x and v26.x.

### The issue: a cap on accuracy and exploding loss when training on a GPU?

I noticed this puzzling behavior initially when trying to run an image classification model (with over 30 categories) based on a 2D convolutional neural network architecture based on the [Xception architecture](). When trying to train this model on the CPU, the model achieved high accuracy but was (understandably) slow to run. Switching to run model training on the GPU, the model training ran about twice as fast. However, my model training on a GPU appeared to result in a behavior where the training run would start to approach high accuracy, but then the loss would start blowing up and accuracy would tumble.

### A simpler CNN model for diagnosing this issue

So, in order to try and figure out what was going on, I tried reproducing and testing fixes for this issue on a much simpler CNN model. My test model was based on several tutorial CNN models used for classifying cats vs. dogs (such as [this example tutorial in the Keras docs](https://keras.io/examples/vision/image_classification_from_scratch/)), albeit with a simpler CNN architecture. The architecture I deployed was a simple one, with three sequential 2D Convolution layers increasing in size, another 2D convolution layer, followed by a final dense, fully-connected layer before the output, as shown in this sequential model construction code snippet ([model diagram here](/assets/blog/2025-12/dog_cat_model.png)):

```python
model = tf.keras.models.Sequential([
    tf.keras.layers.InputLayer(
        shape=(150, 150, 3),
    ),
    tf.keras.layers.Conv2D(
        32, (3,3),
        activation="relu",
    ),
    tf.keras.layers.MaxPooling2D(2, 2),
    
    tf.keras.layers.Conv2D(
        64, (3,3),
        activation="relu",
    ),
    tf.keras.layers.MaxPooling2D(2,2),
    
    tf.keras.layers.Conv2D(
        128, (3,3),
        activation="relu",
    ),
    tf.keras.layers.MaxPooling2D(2,2),
    
    tf.keras.layers.Conv2D(
        128, (3,3), activation="relu",
    ),
    tf.keras.layers.MaxPooling2D(2,2),
    
    tf.keras.layers.Dropout(0.5),
    tf.keras.layers.Flatten(),
    tf.keras.layers.Dense(
        512, activation="relu",
    ),
    tf.keras.layers.Dense(
        2,
    )
])
```

In order to run the training on the Apple Silicon CPU, I used a `tf_device` variable which could be switched to either CPU or the GPU:
```python
# When training with CPU
tf_device = "CPU: 0"

# When training with GPU
tf_device = "GPU: 0"
```
and then when running the model compilation and fitting steps, it can be run in the following way:
```python
with tf.device(tf_device):
    model.compile(…)
    
    history = model.fit(…)
```

Trying out this model with the cat and dog training and validation images, I saw the same discrepant GPU vs CPU training behavior: the model would achieve ~80% accuracy on validation data when training on the CPU. However, when switching training over to the GPU, the model first approached ~75% accuracy on validation data, but then the loss would inevitably start blowing up and accuracy would tumble in subsequent training steps.

<figure>
	<img src="/assets/blog/2025-12/cpu_gpu_acc_loss_comparisons.png" alt="CPU vs. GPU accuracy and loss comparison" title="CPU vs. GPU accuracy and loss comparison" />
    <figcaption>Comparison of accuracy (left column) and loss (right column) on the simple cat vs. dog model when training on a CPU vs. GPU on Apple Silicon. The top row is showing a training run on the CPU, while the bottom row is showing a training run on the GPU. On the GPU, once accuracy approaches ~75%, loss starts to explode. These training runs were run with the TensorFlow default of using float32 for floating point precision. </figcaption>
</figure>

### Issue with floating point precision?
This behavior suggested to me to be an issue with floating point precision: once a model achieves higher accuracy levels, an optimizer / adaptive gradient descent algorithm used during training should decrease the learning rate in response to the higher accuracy level. However, using floating points and not working correctly with the precision, I might expect an issue like this to show up when changing between large and small learning rates with an adaptive optimizer. Small changes to weights or biases with low learning rates may incorrectly become very large with improper handling of floating point precision.

The issue and behavior was surprising to me, since TensorFlow by default works with 32-bit floating point variables, and my (admittedly not super deep) understanding of Apple Silicon GPUs indicated that they should be perfectly capable of handling 32-bit precision [^1]. However, the observed behavior showed otherwise.

[^1]: For example, see [this analysis](https://github.com/philipturner/metal-benchmarks) of Apple Silicon GPU and Metal performance.

All of this motivated me to explore using different floating point precisions with tensorflow-metal on Apple Silicon, and compare their performance on my simple model architecture. Is this issue fixed when using lower precision floating point values for training? Is training time or accuracy affected if I use lower precision floating point values?

### Testing different floating point precision with TensorFlow on Apple Silicon
TensorFlow and Keras support a feature called [mixed precision](https://keras.io/api/mixed_precision/), where most parts of the model can use smaller 16-bit variables while critical parts of the model are kept in 32-bit floating point variables. As the Keras documentation states, the 16-bit training is well supported and encouraged when training on Nvidia GPUs and Google TPUs, but there is very little to no documentation about training performance on Apple Silicon. The only bit of *documentation* about support for mixed precision and 16-bit floating point variable types with TensorFlow on Apple Silicon that I was able to track down was in [the incredibly brief version release notes for tensorflow-metal](https://pypi.org/project/tensorflow-metal/)[^2]. On there, it describes the addition of support for `BF16` and `FP16` in v1.0.0 of tensorflow-metal.

[^2]: Cue the [seemingly perennial](https://www.caseyliss.com/2020/11/10/on-apples-pisspoor-documentation) [complaints](https://atp.fm/618) about Apple’s lack of documentation… 🙃

When running the training, mixed precision with either the [standard IEEE half-precision 16-bit floating point](https://en.wikipedia.org/wiki/Half-precision_floating-point_format) (`FP16`) variables or the [*brain* 16-bit floating point](https://en.wikipedia.org/wiki/Bfloat16_floating-point_format) (`BF16`) can be set in the following way:
```python
# If using FP16
tf.keras.mixed_precision.set_global_policy("mixed_float16")

# If using BF16
tf.keras.mixed_precision.set_global_policy("mixed_bfloat16")
```

Particularly, as the Wikipedia page on bfloat16 describes, these floating point variables allow a larger exponent component (8 bits in bfloat16 vs. 5 bits on half-precision 16-bit floats), while sacrificing precision in the fraction component (7 bits in bfloat16 vs. 10 bits on half-precision 16-bit floats). This tradeoff allows still using 16 bits total for the floating point number representations, but allows the same dynamic range that single-precision 32-bit floats offer. The additional dynamic range is very useful for training ML models, while ML training does not suffer much from the decreased precision resulting from the smaller fraction component.

Trying these different types of floating point representations, I got the following characteristics while running training of the my simple ML model on the Apple Silicon CPU or GPU.

<figure>
	<img src="/assets/blog/2025-12/training_comparisons.png" alt="CPU vs. GPU accuracy and time comparisons with different floats" title="CPU vs. GPU accuracy and time comparisons with different floats" />
    <figcaption>
        Training on an Apple Silicon CPU (top row) and GPU (bottom row) on different types of floating point representations. The left column compares accuracy, the second column compares maximum accuracy achieved on the validation data subset, and the right column compares the time taken to run 200 training epochs on an M1 Pro system.
        The different types of floating point variable representations I tested are shown in different colors.
    </figcaption>
</figure>

The above plot compares the accuracy achieved when running training on a CPU vs. a GPU using different floating point representations. All floating point representations follow a very similar accuracy trend over the course of training on the CPU. However, on the GPU both `fp32` and `fp16` fail once accuracy achieves about 75%, the odd behavior that initially stuck out to me. Only `bf16` performs as expected when training on the GPU.

I also compared the maximum accuracy achieved during training runs on the validation data subset. On the CPU, all floating point representations achieve similar maximum accuracy. However, on the GPU, `bf16` achieves a higher accuracy, comparable to the CPU runs. Using either `fp32` or `fp16` on the Apple Silicon GPU resulted in the loss exploding once training starts to achieve high accuracy.

Finally, I compared the time taken for running 200 epochs of training, running these tests on an M1 Pro system. On the CPU, `bf16` is much slower, as expected, likely due to the optimizations the standard floating-point `fp32` and `fp16` representations have for matrix math on the CPU architecture which `bf16` lacks. All training on the GPU took similar amounts of time, and on my M1 Pro setup, GPU training was approximately 2x faster than the fastest CPU performance.

Overall, the takeaway from this series of tests is to use the very fast GPU architecture offered on Apple Silicon, with the caveat of using the `bf16` floating-point representation. This setup offers both the speed advantages of the GPU, and doesn’t suffer from the loss and accuracy issues that the other floating point representations appear to suffer from on Apple Silicon GPUs.

### Issue with floating point precision handling on Apple Silicon, Metal frameworks, TensorFlow / Keras implementations?

So the open questions I still have at the end of my investigation are what the origin of these issues might be. I’m still a bit surprised that using 32-bit precision on Apple Silicon GPUs leads to not just a performance hit in training, but also seems to introduces errors while training. Is the ultimate source of this error due to how floating point precision handled on Apple Silicon GPUs? Or is the source of the problem in the software stack, with Metal or TensorFlow / Keras? This will require more software spelunking and researching to detemine (especially with the lack of thorough documentation on Apple’s side here…)!
