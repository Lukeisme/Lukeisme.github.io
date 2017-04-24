---
layout: post
title:  "tensorflow实现图像风格迁移网络"
date:   2017-03-10 14:24:18
categories: tech
---

风格迁移网络，就是将一幅图（通常是绘画作品）的风格应用到另一张图上。比如将一张水墨画的风格应用到阿里的照片上。

![风格迁移](https://img.alicdn.com/tps/TB1pYJgPFXXXXbsXXXXXXXXXXXX-1200-847.png)

## 基本原理

而在物体检测的方向上，基于卷积神经网络的实现已经可以达到和人类差不多的识别水平。通常认为经过训练的卷积神经网络是“理解”了图像中的一些信息，如语义和场景等，而卷积神经网络是由多层的小型运算单元也就是神经元组成的，每个神经元都会在训练物体识别的过程中提取到一些特征，成为一个filter，各层的filter组成不同的feature maps，在低层次的网络单元会提取到一些诸如线条之类的特征，而在高层次的网络单元就会提取到一些较为抽象的特征，而风格迁移网络就是基于卷积神经网络这样的特点来实现的。

通过将输入的图片经过feature maps计算，再进行重建和可视化之后可以提取到图片中的一些信息。

![重建](https://img.alicdn.com/tps/TB18f.nPpXXXXbnaXXXXXXXXXXX-1804-1272.png_600x600.jpg)

由于网络网络的底层提取到的是一些低层的信息如content reconstructions的a、b、c，而高层如d、e则认为理解了图像中的一些内容，所以作为内容的参照物的时候，选取高层的输出作为参照物。

对于风格的重建则用了特征空间来获得图像的纹理信息，特征空间通过各层的输出和各层之间的关系来构建。通过构建特征空间，就可以将风格静态化，并且能延伸成不同的尺寸。

## 实现方法

有了基本原理之后，就是模型的实现了。主要2部分组成：内容和风格图片的feature提取，定义loss。
一开始初始化一张图片，通常是一张白噪图。然后和提取的内容和风格对比，目标就是找到一张和内容与风格尽量接近的图片，通过训练调整图片的像素来接近这个目标。下面是实现的主要代码：


1.提取content内容。

{% highlight python %}
image = tf.placeholder('float', shape=shape)
        net, mean_pixel = vgg.net(network, image)
        content_pre = np.array([vgg.preprocess(content, mean_pixel)])
        content_features[CONTENT_LAYER] = net[CONTENT_LAYER].eval(
                feed_dict={image: content_pre})
{% endhighlight %}

2.提取style内容（注意style的部分需要提取多层的信息）。

{% highlight python %}
image = tf.placeholder('float', shape=style_shapes[i])
            net, _ = vgg.net(network, image)
            style_pre = np.array([vgg.preprocess(styles[i], mean_pixel)])
            for layer in STYLE_LAYERS:
                features = net[layer].eval(feed_dict={image: style_pre})
                features = np.reshape(features, (-1, features.shape[3]))
                gram = np.matmul(features.T, features) / features.size
                style_features[i][layer] = gram
{% endhighlight %}

3.计算loss:在计算style的loss的时候，会进行各层的loss计算，最后进行相加。同时，content和style的比重也可以进行调整，也就是生成的结果是content的影响更大还是style的影响更大。除了content和style的loss的计算，还计算了total variation loss用来对图像进行降噪处理。

{% highlight python %}

# content loss
content_loss = content_weight * (2 * tf.nn.l2_loss(
        net[CONTENT_LAYER] - content_features[CONTENT_LAYER]) /
        content_features[CONTENT_LAYER].size)
# style loss
style_loss = 0
for i in range(len(styles)):
    style_losses = []
    for style_layer in STYLE_LAYERS:
        layer = net[style_layer]
        _, height, width, number = map(lambda i: i.value, layer.get_shape())
        size = height * width * number
        feats = tf.reshape(layer, (-1, number))
        gram = tf.matmul(tf.transpose(feats), feats) / size
        style_gram = style_features[i][style_layer]
        style_losses.append(2 * tf.nn.l2_loss(gram - style_gram) / style_gram.size)
    style_loss += style_weight * style_blend_weights[i] * reduce(tf.add, style_losses)
# total variation denoising
tv_y_size = _tensor_size(image[:,1:,:,:])
tv_x_size = _tensor_size(image[:,:,1:,:])
tv_loss = tv_weight * 2 * (
        (tf.nn.l2_loss(image[:,1:,:,:] - image[:,:shape[1]-1,:,:]) /
            tv_y_size) +
        (tf.nn.l2_loss(image[:,:,1:,:] - image[:,:,:shape[2]-1,:]) /
            tv_x_size))
# overall loss
loss = content_loss + style_loss + tv_loss

{% endhighlight %}

接下来就是训练了。关于loss计算的相关数学细节，可以通过原[paper](https://arxiv.org/pdf/1508.06576v2.pdf)进行了解，完整的tensorflow实现代码可以[参考](https://github.com/anishathalye/neural-style),值得一提的是，这个实现里的pool层是用的max pollling，而paper中的作者认为average pooling会有更好的效果。感兴趣的同学可以试一下两种pool层的效果。

