# UniversalStyleTransfer
Torch implementation of our [paper](https://arxiv.org/pdf/1705.08086.pdf) on universal style transfer. For academic use only.

<img src='figs/p3.jpg' width=800>

## Prerequisites

- Linux
- NVIDIA GPU + CUDA CuDNN
- Torch 
- Pretrained encoder and decoder [models](https://drive.google.com/open?id=0B8_MZ8a8aoSeWm9HSTdXNE9Eejg) for image reconstruction only (download and put them under models/)

## Style transfer

- For a single pair test:

```
th test_wct.lua -content YourContentImagePath -style YourStyleImagePath -alpha YourStyleWeight
```

- For large numbers of pair test:

```
th test_wct.lua -contentDir YourContentImageDir -styleDir YourStyleImageDir -alpha YourStyleWeight
```

By default, we perform WCT (whitening and coloring transform) on conv1-5 features. Some transfer results and comparisons with existing methods are shown [here](https://drive.google.com/file/d/0B8_MZ8a8aoSed3RrcTBfM1hES3c/view).


## Texture synthesis

```
th test_wct.lua -style YourStyleImagePath -synthesis 1 
```


## Spatial control

Often times, the one-click global transfer is not what professinal artists want. Users prefer to transfer different styles to different regions in the content image. We provide an example of transferring two styles for foreground and background respectively, i.e., Style I for foreground (mask=1), Style II for background (mask=0), provided a binary mask.

<img src='figs/p2.jpg' width=800>

```
th test_wct_mask.lua -content YourConentPath -style YourStylePath1,YourStylePath2 -mask YourBinaryMaskPath
```

## Swap on conv5

We also include the [Style-swap](https://github.com/rtqichen/style-swap) function in our algorithm but we perform the swap operation based on whitened features. For each whitened feature patch, we swap it with nearest whitened style patch. Please refer to their [paper](https://arxiv.org/pdf/1612.04337.pdf) for more details.

We provide a parameter "-swap5" to perform swap operation on conv5 features. The swap operation is computationally expensive as it is based on searching nearest patches. So we do not provide the swap on layers with large feature maps (e.g., conv1-4).

```
th test_wct.lua -content YourContentImagePath -style YourStyleImagePath -swap5 1
```
Below is an exemplary comparison between w/o and w/ swap operation on conv5. It is obsearved that with the swapping, the eyeball in the content is replaced with the ball in the style (bottom) as they are cloeset neighbours in whitened feature space.

<img src='figs/p1.jpg' width=840>


## Note

- In theory, the covariance matrix of whitened features should be Identity. In practise, it is not because we need to eliminate some extremely small eigen values (e.g., <1e-10) or add a small constant (e.g., 1e-7) to all eigen values in order to perform the inverse operation (D^-1/2) in the whitening.

- To save memory for testing image of large size, we need to often load and delete model. So in our code, for the transferring on each content/style pair, we need to reload the model.

- For a GPU of memory ~12G, it is suggested that the contentSize and styleSize are all less than 900 (800 recommended for the largest size).

- We found that using "CUDA_VISIBLE_DEVICES=X" is better than using "-gpu X" as the former choice will guarantee that all weights/gradients/input will be located on the same GPU.

## Citation

```
@inproceedings{WCT-XXX-2017,
    author = {Li, Yijun and Fang, Chen and Yang, Jimei and Wang, Zhaowen and Lu, Xin and Yang, Ming-Hsuan},
    title = {Universal Style Transfer via Feature Transforms},
    booktitle = {XXX},
    year = {2017}
}
```

## Acknowledgement

We express gratitudes to the great work [AdaIN](https://github.com/xunhuang1995/AdaIN-style) and [Style-swap](https://github.com/rtqichen/style-swap) as we benefit a lot from both their paper and codes.
