# Generative AI

## Autoencoders & Variational Autoencoders (VAE)

### 1. Explicit Density (Tractable)

These models **explicitly calculate the probability** \(p(x)\) of any data sample. Since the probability is known, they can answer questions like:

> **"How likely is this image or sentence?"**

#### Types

##### Autoregressive Models

Factorize the joint probability as:

`p(x) = ∏ p(x_i | x_<i)`

Generate one token (or pixel) at a time.

Every new prediction depends on previous outputs.

Examples: GPT

##### Advantages
- Exact likelihood
- Very stable training
- Excellent for text generation

##### Disadvantages

- Slow inference because generation is sequential

---

#### Normalizing Flows

These transform simple Gaussian noise into complex data using **invertible neural networks**.

Examples:  Glow, RealNVP

##### Advantages

- Exact likelihood
- Fast sampling

##### Disadvantages

- Architecture must remain invertible, limiting flexibility

---

### 2. Explicit Density (Approximate)

Sometimes computing the exact probability is impossible. These models instead optimize an approximation called the **Evidence Lower Bound (ELBO)**. Rather than optimizing the real probability directly, the model optimizes this lower bound, which is much easier to compute.

#### Variational Autoencoder (VAE)

A VAE has two parts:

- **Encoder:** Compresses input into a latent vector.
- **Decoder:** Reconstructs the original data.

The latent space is smooth, allowing new samples to be generated.

##### Advantages

- Stable training
- Continuous latent space

##### Disadvantages

- Generated images are often blurry

---

#### Diffusion Models

##### Training Process

1. Add noise to images step by step.
2. Train a neural network to remove the noise.
3. During inference, start from random noise and repeatedly denoise until an image appears.

##### Advantages

- Highest image quality
- Very stable

##### Disadvantages

- Slow because hundreds of denoising steps may be required

---

### 3. Implicit Density (GANs)

Implicit density models never calculate the probability of data. Instead, they only learn how to generate realistic samples. The most famous example is the Generative Adversarial Network (GAN). Since these models do not know the explicit probability distribution, they cannot answer questions about the likelihood of a sample. Their only goal is to generate outputs that look indistinguishable from real data.


##### Advantages

- Extremely sharp images
- One forward pass during inference
- Very fast generation

##### Disadvantages

- Difficult to train
- Mode collapse (producing only a few kinds of images)

---

### 4. Score-Based Models / Flow Matching

Score-based models do not learn probabilities directly. Instead, they learn the **score function**, which is the gradient of the logarithm of the probability distribution. Intuitively, the score tells the model **which direction to move in order to reach more realistic data**.

##### Advantages

- 4–10× faster than traditional diffusion
- Better sampling efficiency
- Excellent image and video quality

##### Examples

- Stable Diffusion 3
- Flux
- AudioCraft 2

---

### 5. Token-Based Autoregressive Models

High-resolution images, audio, and videos contain huge amounts of information. Instead of modeling raw pixels directly, these models first compress the data into a sequence of discrete tokens using techniques like VQ-VAE. These tokens behave like words in a sentence. A Transformer then predicts one token after another, exactly as language models generate text. Finally, a decoder reconstructs the original image, speech, or video from the generated tokens. This approach allows powerful Transformer architectures to be used for many types of data beyond text.



##### Advantages

- Efficient for speech and music
- Leverages Transformer architectures

##### Disadvantages

- Depends on high-quality tokenization

<br/>
<hr style="border: none; border-top: 2px dashed #999; height: 0;">


## Autoencoders & Variational Autoencoders (VAE) 

### What is the Problem with a Normal Autoencoder?
Although an autoencoder learns to reconstruct images very well, its **latent space is completely unstructured**. Means that when it compresses data into small numbers, those numbers are not arranged in a smooth or organized way. They are spread randomly, with empty gaps in between. Because of this, if you pick a random set of numbers from that space, the decoder usually cannot understand it and produces random noise instead of a real image. So, it works well for reconstruction, but not for generating new data.


A **Variational Autoencoder (VAE)** works by first compressing an input (like an image) into a small hidden representation called a latent space, but instead of storing one `fixed code (single exact set of numbers)`, it stores a range of values using `mean and variance`. During training, it randomly picks a point from this range and sends it to a decoder, which tries to rebuild the original image. At the same time, it is forced to keep all these latent codes organized in a smooth pattern (like a standard Gaussian shape). Because of this, after training, you can pick any random point from this smooth space and the decoder will turn it into a realistic new image.

Instead of producing only one latent vector, the encoder predicts two outputs:

- **Mean (μ)**: the center of the Gaussian distribution.
- **Log Variance (log σ²)**: describes how spread out the distribution is.

  If network network predicted σ (Variance) directly, might accidentally produce negative values. the network predicts log(σ²) because logarithms can take any real value. During computation, we convert it back using the exponential function.
    

One major challenge is that **random sampling is not differentiable**,

Instead of sampling directly:

z∼N(μ,σ²)

we rewrite it as:

z=μ+σ⋅ϵ

start from a fixed value (μ), “then slightly shift it using randomness (ε × σ)

This simple mathematical trick is what makes VAEs trainable.

#### VAE Loss Function (ELBO)

**Reconstruction Loss**:This measures how similar the reconstructed image is to the original image.

**KL Divergence Loss** : This measures how different the learned latent distribution is from the desired Gaussian distribution.

The total loss is:

Loss=Reconstruction + β×KL

#### Common Problems in VAEs

One major issue is **Posterior Collapse**. This happens when the KL loss becomes too strong. The encoder stops encoding useful information and simply outputs the standard Gaussian distribution for every input. As a result, the decoder ignores the latent vector and tries to reconstruct images on its own. Researchers solve this by gradually increasing β during training, a technique called β Annealing.

Another issue is **blurry images**. Since VAEs often minimize Mean Squared Error, the decoder predicts the average of many possible outputs. The average of many plausible images appears blurry. Modern systems often solve this by using the VAE only for compression while allowing a diffusion model to generate fine details.

<br/>
<hr style="border: none; border-top: 2px dashed #999; height: 0;">


## GANs — Generator vs Discriminator


Training is tricky. One common problem is mode collapse, where the Generator finds a few outputs that consistently fool the Discriminator and keeps producing only those, ignoring the diversity of the real dataset. Another problem is instability: if the Discriminator becomes too strong too quickly, the Generator gets almost no useful learning signal and stops improving. If the Generator becomes too strong, the Discriminator fails to learn meaningful distinctions.

To fix these issues, many improvements were developed over time. For example, DCGAN introduced convolutional architectures that made training more stable for images.

<br/>
<hr style="border: none; border-top: 2px dashed #999; height: 0;">


## Conditional GANs & Pix2Pix


A **Conditional GAN (cGAN)** is just a GAN where you give the model extra information about what you want it to generate. Instead of the Generator blindly producing random outputs from noise, you also provide a condition—like a label. This condition acts like a “hint” that tells the model what kind of output is expected.

In a normal GAN, the Discriminator only checks whether something looks real or fake. In a Conditional GAN, the Discriminator also checks whether the output matches the condition.

Pix2Pix is a very important and practical version of a Conditional GAN.

<br/>
<hr style="border: none; border-top: 2px dashed #999; height: 0;">


## StyleGAN
StyleGAN is a type of GAN designed to fix a core weakness. In older GANs like DCGAN, a single vector z is fed into the generator, and it ends up controlling all aspects of the image simultaneously—identity, pose, lighting, background, and tiny details all get tangled together. This makes the representation “entangled,” meaning you cannot independently control features. For example, you cannot easily say “keep the same person but change only the lighting,” because changing one part of z affects everything at once.

StyleGAN solves this by splitting the generation process into two stages. First, instead of using z directly, it passes z through a mapping network (an MLP) that produces an intermediate latent vector called w. This w lives in a more structured space where different directions tend to control more separated features. In other words, w is easier to interpret and manipulate than z.

Then, instead of feeding w only at the input, StyleGAN injects it into every layer of the generator. This is done using a technique called AdaIN (Adaptive Instance Normalization). At each layer, the feature maps are first normalized (so they lose their original scale and bias), and then w is used to re-scale and shift them. This means w acts like a “style controller” at every resolution level of the image. Early layers control large-scale structure like pose and face shape, while later layers control fine details like skin texture, hair strands, and lighting.

Another important idea is that StyleGAN starts from a learned constant tensor (not noise). Instead of building the image from random input at the start, it begins with a fixed 4×4 feature map and progressively builds the image upward through convolution layers. This helps stabilize structure and makes the role of w clearer as a style modifier rather than a structural generator.

StyleGAN also adds small random noise at each layer. This noise does not affect the overall structure of the image, but it controls stochastic details like freckles, pores, or hair randomness. So structure comes from w, while fine randomness comes from noise. This separation is important because it prevents the model from mixing global identity with tiny texture details.

A key improvement introduced later is the truncation trick. During generation, you can move w closer to the average latent vector. This reduces diversity but increases image quality. So you trade variety for realism by controlling how far you sample from the center of the learned distribution.

The major breakthrough of StyleGAN is that it “disentangles” the latent space. This means different aspects of the image become more independently controllable. You can change pose without changing identity, or lighting without changing face shape. This is what makes StyleGAN extremely useful for face editing and image inversion tasks.

Later versions improved stability and realism even further. StyleGAN2 fixed artifacts caused by AdaIN by replacing it with weight demodulation, which made images cleaner. StyleGAN3 removed subtle “texture sticking” problems by making the generator translation-equivariant, so textures don’t shift incorrectly when the image is moved. Each version improved the quality and consistency of generated images.

<br/>
<hr style="border: none; border-top: 2px dashed #999; height: 0;">


## Diffusion Models — DDPM from Scratch


The process consists of two parts: the **forward process** and the **reverse process**.
Imagine you have a beautiful photograph. Instead of asking the computer to create a brand-new image from nothing, you first slowly destroy the photograph by adding a tiny amount of random static (noise) over many steps. Eventually, after enough steps, the picture becomes nothing but random noise. Then you train the computer to learn the reverse process: given a slightly noisy image, remove just a little bit of the noise. If it learns this tiny denoising task at every level of noise, it can later start from pure random noise and gradually turn it back into a realistic image. This simple idea is the foundation of modern AI image generators like Stable Diffusion.

**GANs (Generative Adversarial Networks)** trained two neural networks that competed against each other, and training was often unstable because one network could overpower the other. **VAEs (Variational Autoencoders)** were easier to train but usually produced blurry images because they tried to compress images into a smooth latent space. Diffusion models solved many of these issues by using only one neural network and a simple prediction task. Instead of learning to generate an entire image at once, the model only learns how to remove a small amount of noise. This makes training much more stable.

mathematics gives us a shortcut. There is a formula that allows us to jump directly from the clean image to any noise level. If we want the image at timestep 500 out of 1000, we do not have to simulate all 500 steps. We can compute it .

**x1 = sqrt(ᾱₜ)x₀ + sqrt(1 - ᾱₜ*ε)**

**sqrt(ᾱₜ)x₀** -> keeps part of the original image.   
**sqrt(1 - ᾱₜ*ε)** -> adds the appropriate amount of noise.



Here, **x₀** is the original clean image, **ε** is random Gaussian noise, and **ᾱₜ (alpha bar)** tells us how much of the original image should remain at timestep **t**. Early in the process, alpha bar is close to 1, meaning most of the original image is preserved. **beta (β)** control how much noise is added at each step. You can think of beta as the strength of corruption. 

The training process is surprisingly simple. First, pick a real image from the dataset. Next, randomly choose a timestep. Then generate random Gaussian noise. Mix the clean image and the noise using the diffusion equation to create a noisy image. Feed the noisy image and the timestep into the neural network. The network predicts the noise. Finally, compare the predicted noise with the actual noise using **Mean Squared Error (MSE)**. If the prediction is wrong, update the network weights using backpropagation. This is the entire training loop.

The loss function is (measures the difference between the real noise and the predicted noise)

**L = ∥ ϵ − ϵθ​(xt​,t) ∥²** 

After training, image generation works in reverse. Instead of starting with a real image, we begin with pure random Gaussian noise. The trained model looks at this noisy image and predicts what part of it is noise. We subtract that predicted noise, making the image slightly cleaner. We repeat this process again and again. Each step removes a little more noise, gradually revealing meaningful shapes, textures, and objects.



**why the model also receives the timestep t as input?**. The reason is that denoising depends on how noisy the image is. If the image is almost clean, only tiny corrections are needed. If the image is nearly pure noise, the network must perform much larger changes. Without knowing the timestep, the network would not know how aggressively it should denoise.



Modern AI image generators build upon this same DDPM foundation. Instead of generating images directly in pixel space, systems like Stable Diffusion first compress images into a much smaller **latent space** using a VAE. Diffusion is then performed inside this compact representation, making generation much faster and requiring far less memory.


**Classifier-Free Guidance (CFG)** 

<br/>
<hr style="border: none; border-top: 2px dashed #999; height: 0;">

## Latent Diffusion & Stable Diffusion
Imagine you want an AI to generate a **512 × 512** image from a text prompt  A normal diffusion model (DDPM) works directly on every pixel of the image. Since a 512 × 512 RGB image has **512 × 512 × 3 = 786,432 pixel values**, the neural network has to process all these numbers during every denoising step.


**not every pixel is equally important**. Small texture details can be reconstructed later. This observation led to **Latent Diffusion**, the idea behind **Stable Diffusion**.

Instead of running diffusion directly on the original image, Stable Diffusion first **compresses the image into a much smaller representation called a latent space** by **Variational Autoencoder (VAE)**. This latent contains the important semantic information about the image while removing unnecessary details. Because the latent is much smaller, diffusion becomes dramatically faster.


Stable Diffusion works in **two stages**. In the **first stage**, the VAE is trained. It learns how to compress an image into a latent representation and reconstruct it back with minimal quality loss. After the VAE is trained, its weights are frozen. In the **second stage**, the diffusion model is trained **only on these latent vectors**, not on the original images. This means the diffusion model never directly sees pixels—it only learns to denoise compressed representations.


At generation time, the process is reversed. The model starts with pure random noise **inside the latent space**. Over many denoising steps, it gradually transforms this random latent into a meaningful latent representation. Once denoising is complete, the VAE decoder converts the final latent into a full-resolution image. In other words, Stable Diffusion **creates a compressed image first and then expands it into the final picture**.


The biggest advantage of latent diffusion is efficiency. A **64 × 64 × 4** latent contains far fewer numbers than a **512 × 512 × 3** image.

Stable Diffusion also allows image generation from **text prompts**. To achieve this, it uses a **text encoder**. The text encoder converts a sentence into a set of numerical embeddings. These embeddings capture the meaning of the prompt. 

The diffusion model needs a way to combine the text information with the image generation process. This is done using **cross-attention**. At every stage of denoising, the latent image features "look at" the text embeddings and decide which words are important. For example, if the prompt is *"A red sports car in the snow,"* the model pays attention to words like **red**, **sports**, **car**, and **snow**, ensuring the generated image reflects those concepts

Another important technique used during generation is **Classifier-Free Guidance (CFG)**. During training, the model is sometimes given the text prompt and sometimes not. This teaches it how to generate both conditioned and unconditioned images. During inference, it makes two predictions: one using the prompt and one without it. These predictions are combined so the model follows the prompt more strongly. 


**Stable Diffusion** is a practical text-to-image model built using the Latent Diffusion technique. It combines a pretrained VAE for image compression, a diffusion model for denoising the latent representations, and a text encoder to understand text prompts.              
It also supports **negative prompts**. Instead of only telling the model what should appear, you can also specify what should **not** appear. 

<br/>
<hr style="border: none; border-top: 2px dashed #999; height: 0;">


## ControlNet, LoRA & Conditioning

### The Problem
A prompt like "a woman in a red dress walking a dog on a busy street" gives the model no information about where the dog is, what pose the woman is in, or the perspective of the street. Text pins down about 10% of what you need to specify an image. The rest is visual and cannot be described efficiently in words.

Training a new conditional model from scratch for every signal  is prohibitive.  

### ControlNet
Instead of modifying the original Stable Diffusion model, ControlNet creates a copy of the encoder portion of the U-Net while keeping the original network frozen.This copied network learns to understand an additional conditioning image, such as a pose skeleton, depth map, or edge image. The information extracted from this conditioning input is gradually added back into the original model through zero-convolution layers, which are initialized with zero weights. Because these layers initially contribute nothing, the pretrained model behaves exactly as before at the start of training, preventing sudden changes or loss of previously learned knowledge. As training progresses, the zero-convolution layers slowly learn how much conditioning information should influence the final image. This design allows ControlNet to guide the placement, pose, perspective, and structure of objects while preserving the powerful image-generation capabilities of the original diffusion model.

### LoRA
LoRA (Low-Rank Adaptation) is a lightweight fine-tuning technique that allows the model to learn new concepts, styles, or subjects without updating billions of parameters in the original model. Normally, fine-tuning a large diffusion model requires modifying every weight, which demands significant computational resources and storage. LoRA solves this by freezing the original model and learning only a small low-rank weight update represented by two much smaller matrices. 

### IP-Adapter
it which enables the model to use a reference image in addition to the text prompt. Instead of learning a new style through training like LoRA, the IP-Adapter extracts visual features from an input image using a CLIP image encoder and injects these features into the diffusion model during image generation. This allows users to generate images that preserve the appearance, style, or identity of a reference image without requiring any additional training.

<br/>
<hr style="border: none; border-top: 2px dashed #999; height: 0;">


## Inpainting, Outpainting & Editing

Inpainting, outpainting, and image editing are techniques that allow AI to modify an existing image instead of generating a completely new one.

**Inpainting** is the process of filling or regenerating only a selected part of an image while keeping the remaining portion unchanged. The area to be edited is marked using a mask, where the masked region indicates what should be regenerated.

**Outpainting** is similar to inpainting, but instead of filling a missing region inside the image, it extends the image beyond its original boundaries. Imagine having a landscape photo that is too narrow. Using outpainting, the AI can generate new scenery outside the current edges, creating the impression that the image was originally much larger. The newly generated regions match the colors, lighting, and overall style of the original image, making the extension look seamless. This technique is widely used for expanding backgrounds, creating wallpapers, or converting portrait images into wider landscape formats.

However, this method often produces unrealistic transitions because the model does not fully understand the relationship between the masked region and the surrounding image.To solve this problem, modern diffusion models use a specialized **inpainting model**. Instead of taking only the noisy image as input, the model also receives the original encoded image and the mask. This allows it to understand what exists around the masked region and generate content that blends smoothly with the surrounding pixels.

**SDEdit (Stochastic Differential Editing)**, which allows users to modify an image without retraining the model. The idea is simple: first, the original image is partially corrupted by adding noise, and then the diffusion model removes the noise while following a new text prompt. The amount of noise determines how much the image changes. If only a small amount of noise is added, the final image remains very similar to the original with only minor changes.


### Another advanced methods are:

**InstructPix2Pix**,. During inference, users simply provide commands like "make it sunset," "add snow," or "change the shirt to blue," and the model performs the requested edits while preserving the rest of the image.


**RePaint**, During the reverse diffusion process, the model occasionally adds noise back into the image before denoising it again. This repeated refinement helps reduce visible boundaries between the edited and unedited regions, resulting in smoother transitions. Although slower than dedicated inpainting

## Video Generation
## Audio Generation
## 3D Generation
## Flow Matching & Rectified Flows
## Evaluation: FID, CLIP Score
## Visual Autoregressive Modeling (VAR): Next-Scale Prediction

<br/>
<hr style="border: none; border-top: 2px dashed #999; height: 0;">
