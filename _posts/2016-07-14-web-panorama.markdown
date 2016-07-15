---
layout: post
title:  "WEB 全景图"
date:   2016-07-14 16:40:18
categories: tech
---

## 全景图的基本原理

全景图是一种广角图。通过全景播放器可以让观看者身临其境地进入到全景图所记录的场景中去。比如像是[这个](https://ju.taobao.com/m/jusp/alone/panoramaaud/mtp.htm)。这种看起来很高大上的效果其实背后的原理并不复杂。

通常标准的全景图是一张2：1的图像，其背后的实质就是等距圆柱投影。等距圆柱投影是一种将球体上的各个点投影到圆柱体的侧面上的一种投影方式，投影完之后再将它展开就是一张2：1的长方形的图像。比较常见的就是应用在地图上的投影。

![等距投影](https://img.alicdn.com/tps/TB1v2brKFXXXXahXVXXXXXXXXXX-773-266.jpg)

而在对全景图进行展示之前就需要得到一张这样的图像，这种图像可以自己用普通相机拍摄再自己合成，也可以直接使用专门的全景相机进行拍摄。全景照片的拍摄在网上有比较多的教程，由于这不是摄影分享就不详细的去说了:P。

在得到了全景图之后，就是要怎么去展示的问题了。接下来就要说说全景展示的原理。全景展示其实是等距圆柱投影的逆过程，我们要做的就是将我们得到的全景图，贴图到一个球体上，熟悉webgl的，可以用它画一个球体，然后将全景图作为材质贴到这个球体上进行渲染。由于使用webgl来进行编程的话，需要自己进行比较多的3d运算，所以也可以选择使用api更加友好的3D库，如THREE.JS来编程。比如下面的这张全景图，在球面上进行贴图。

![全景图](https://img.alicdn.com/tps/TB1RJTJKFXXXXaiXXXXXXXXXXXX-1024-512.jpg)
![球贴图](https://img.alicdn.com/tps/TB1pPboKFXXXXbZXVXXXXXXXXXX-600-509.png)

这时我们看到的还跟预想的全景不一样，那是因为我们在球的外面，当我们在球的里面时，看到的就是跟一开始的示例一样的效果了。像是下面的这个示意图这样。

![原理](https://img.alicdn.com/tps/TB1govgKFXXXXXkapXXXXXXXXXX-840-780.png_400x400.jpg)

用threejs进行编程的话，关键的代码如下：

{% highlight javascript %}
//新建一个球体
var geometry = new THREE.SphereGeometry( 500, 60, 40 );
//沿x轴进行-1的scale，让球体的面朝内（因为我们将从球内进行观看）。
geometry.scale( - 1, 1, 1 );
//载入一张全景图生成threejs中可以使用的材质
var material = new THREE.MeshBasicMaterial( {
	map: new THREE.TextureLoader().load( 'panoPic.jpg' )
} );
//将几何体和材质进行结合。
mesh = new THREE.Mesh( geometry, material );
{% endhighlight %}

## 兼容性
虽然使用webgl可以很容易的就生成一个全景的场景，但是在web里，兼容性似乎是个挥之不去的话题。主要是由于webgl不支持Android5.0以下的机器，所以，用webgl来实现将会将很多用户排除在外。所以只能寻求更好的解决方案。首先想到的就是css 3D transform 和 2D canvas画布。在threejs里支持在2D的canvas里进行绘制，本是一个比较好的方案，但是经过测试之后，发现2d画布来绘制3d的场景，性能上太吃力。所以，也被排除。剩下的就是css变换了。但是，要怎么用css来画一个一个球体呢？答案显然是不行的。虽然css不能绘制一个球体，但是css通过3D变换来绘制立方体还是简单一些的。那么用立方体可不可以实现一样的效果呢？

## 球体到立方体

根据全景图的原理,我们是把视角放在球的中心，通过从球心观看球面上正式场景在球面上的映像从而产生一种空间中全方位的视觉体验，同理，对于立方体，应该也可以使用相同的方式来实现。而我们要做，就是把球面上的像素点映射到立方体上。

![s2c](https://img.alicdn.com/tps/TB17HZxKFXXXXcRXVXXXXXXXXXX-918-924.png_400x400.jpg)

说了基本的原理，接下来就是进行数学建模了。首先我们建立一个球坐标系，坐标系描述的变量分别为半径r,竖直方向上的夹角θ,水平方向上的夹角ø，对于球体，我们可以假定

```
r=1
0 < θ < π
 -π/4 < ø < 7π/4
```
这样我们就可以得到球面上的各个点在直角坐标系中的x,y,z

```
x= r sin θ cos ø
y= r sin θ sin ø
z= r cos θ
```

对于球面到立方体上的投影，我们需要的是角度θ和ø相同时，延长球的半径r直到和立方体的面相交，假设这个长度是R，由于我们设了半径r是1所以球面上的点为 (sin θ cos ø, sin θ sin ø, cos θ) 对应的立方体上的点是(Rsin θ cos ø, Rsin θ sin ø, Rcos θ)

如果我们要求x=1这个平面上的点，则

```
1=Rsin θ cos ø
```
则可以求出来

```
R= 1/(sin θ cos ø)
```

所以在x平面上映射的点就是

```
(1, tan ø, cot θ / cos ø)
```

在立方体另外的五个平面上的投影也可以类似地得出。通过上述方法转换一张全景图，可以得到以下结果。

![结果1](https://img.alicdn.com/tps/TB1KCMSKFXXXXXfXpXXXXXXXXXX-2048-1536.jpg)

离我们想要的效果还有一些差距，图中似乎多了一些黑色的线。导致这种现象的原因是，由于我们的处理是以像素为单位来进行处理的，通过遍历球面图上的每个像素然后投影到立方体上的面来实现。经过这种方式进行投射之后，立方体的面上就会有一些像素被重复设置，而一些地方的像素就会缺失，比如图中的黑色部分（由于底色的黑色的）。为了解决这个问题，我们可以通过逆向的方法来解决，也就是遍历立方体上面上的每个点，求得映射到球面上的位置，然后获取球面上最接近的位置的像素。

![结果2](https://img.alicdn.com/tps/TB1ligtKFXXXXXfapXXXXXXXXXX-2048-1536.jpg)

得到了立方体上需要的图之后，就可以用css 3D变换来实现全景图了。

## 展示更多信息
单单地进行全景观看可能还不能满足我们的需求，也许我们还需要展示更多的信息，而这些信息可能是跟全景中的内容相关的，比如给全景中的物品打标，给全景中的内容添加评论，像是下图这样

![标签](https://img.alicdn.com/tps/TB1XcE4KFXXXXXcXpXXXXXXXXXX-734-1272.png_400x400.jpg)

对于这个实现的关键在于，屏幕2D坐标和空间3D坐标之间的相互转换。第一步需要实现的是记录，当用户点击屏幕，要根据点击的位置来计算出和空间中的立方体相交的点并记下这个点的位置信息。一些3d库当中会有一些api来帮助完成这项工作。而在THREEJS中使用的是Raycaster，Raycaster可以生成一条直线，然后可以很方便地得到三维空间中，和这条直线相交的物体和点。由于THREEJS中是以绘制的中心作为原点，而鼠标的点击位置是以左上角为原点，所以需要进行一下转换。


{% highlight javascript %}
//将鼠标点击事件中的位置信息，转换到位置中心
  var mouse = new THREE.Vector2(
    ( ev.clientX / _this.wrapper.width() ) * 2 - 1,
    -( ev.clientY / _this.wrapper.height() ) * 2 + 1
  )
{% endhighlight %}
然后就可以初始化一个Raycaster获得需要的内容了

{% highlight javascript %}
//创建一个Raycaster实例
  var raycaster = new THREE.Raycaster()
//根据点击的位置，从镜头开始初始化一和镜头的屏幕垂直的直线
  raycaster.setFromCamera(mouse, _this.camera)
//获得和直线相交的物体
  var intersects = raycaster.intersectObjects(_this.scene.children)
{% endhighlight %}

这样就在空间中记录下了一个目标位置。

有了三维中的位置之后，如果我们想要展示这个位置相关信息，比如打一个标签，就是需要将空间中的位置还原到屏幕的二维坐标上，然后用传统的css方法来进行展示就可以了。

{% highlight javascript %}
//根据上一步中记录的位置生成一个向量
var vector  = new THREE.Vector3(pos.x, pos.y, pos.z)
//将这个向量映射到镜头的平面上
vector.project(camera)
//将位置信息还原成以左上角为原点的位置信息
var screenPos = {
  left: Math.round((   vector.x + 1 ) * wrapper.width() / 2),
  top: Math.round(( -vector.y + 1 ) * wrapper.height() / 2)
}
{% endhighlight %}





