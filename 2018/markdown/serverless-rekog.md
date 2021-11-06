Serverless 打造智能微信小程序：图像识别分析文本
===

结束了五一国庆的八天假期后，便开始着手为微信小程序《代码协作》制作一个图片识别代码的功能。这个需求的主要来源是，在有的公司、组织、团队上，代码是不能直接拷贝出来的。但是，拍照是允许的。为了协作方便，一般会拍照在微信群里讨论。对于我而言，因此这样一个真实需求的存在，我便想试试能不能做这样的一个功能。

对于图片识别功能来说，只需要两步：

1. 上传图片到 AWS S3 上
2. 找到一个合适的图片识别服务，来识别这张图片

然而，第一步是最难的一步。

Serverless 上传图片
---

按理来说，我们只需要在 Serverless 应用里，接受这个数据，然后再存储到 AWS S3 就可以了。

微信小程序上传图片时，采用的 ContentType 是 ``multipart/form-data``，这就意味着上传的图片不是 base64 形式的。在 AWS Gateway 里，接受到的数据变成了：

```
{ fieldname: 'file',
  originalname: 'tmp_41852593e221427e029327f987f3d588.png',
  encoding: '7bit',
  mimetype: 'image/png',
  buffer: <Buffer 89 50 4e 47 0d 0a 1a 0a 00 00 00 0d 49 48 44 52 00 00 03 e8 00 00 02 80 08 06 00 00 00 0f 73 52 67 00 00 00 01 73 52 47 42 00 ae ce 1c e9 00 00 40 00 ... >,
  size: 870857 }
```

最开始的时候，我以为是微信小程序上传了。于是，我开始用 curl 来测试上传图片：

```
curl -v -F file=@test.jpg https://code.wdsm.io/upload
```

然后对比了本地 readFileSync 的 Buffer 数据，发现图片的编码变了。

然后，我测试了文本上传：

```
curl -v -F file=@hello.txt https://code.wdsm.io/upload
```

发现文本的编码结果是不变的。在 Google 了一番之后，发现是因为 AWS Gateway 是能识别文本编码的。

在我尝试了 N 次之后，并在本地构建了个类似的环境。发现了 AWS Gateway 做了一个神奇的转换：``Buffer.from(imageData, 'utf8').toString('utf8')``。理论上，经过这样的转换，数据应该是变回原来的格式。然而，这个图片并不是 utf8 编码，所以这个数据无法转换回去。

在继续 Google 了一番之后，发现了一个 Serverless Framework 的插件：serverless-apigw-binary。它可以将图片转换为二进制形式的，而 AWS Gateway 不会对二进制的内容进行特殊编码。其配置如下所示：

```
custom:
  apigwBinary:
    types:
      - 'image/jpeg'
      - 'image/png'
      - 'text/html'
      - 'multipart/form-data'
```

上面配置中，最重要的其实是 ``'multipart/form-data'``，它会将我们的请求转换为二进制形式的。

正在我以为要大功告成的时候，我发现直接用 AWS Lambda 解析出来还是有问题。在尝试了多次之后，结合了一下 express 里的 multer 来解析这个数据，然后它工作了：

```
const express = require("express");
const multer = require('multer');

const multerupload = multer();
const app = express();

app.post('/upload', multerupload.any(), function (req, res) {
  let file = req.files[0];
  ...
});
```

果然，编码是要靠猜的。

Serverless 图片识别
---

在开始这部分之前，需要先了解一下 AWS Rekognition。

> Amazon Rekognition 让您能够轻松地为应用程序添加图像和视频分析功能。您只需向 Rekognition API 提供图像或视频，然后此服务就能识别对象、人员、文字、场景和活动，以及检测任何不适宜的内容。

由于，我们的《代码协作》应用整个是基于 AWS 的，因此我们也就继续使用 AWS 的服务来识别文本。先将图片放到 AWS S3 上，然后才是识别：

```
s3.putObject({
  Body: file.buffer,
  Bucket: bucketName,
  Key: file.originalname,
  ContentType: file.mimetype
}, function (err, data) {
  const s3Config = {
    bucket: bucketName,
    imageName: file.originalname,
  };

  return ImageAnalyser
    .getImageText(s3Config)
    .then((text) => {
      console.log(JSON.stringify(text, null, '\t'));
      res.status(200).send(text)
    })
    ...
})
```

识别部分代码如下：

```
const rek = new AWS.Rekognition();

class ImageAnalyser {
  static getImageText(s3Config) {
    ...

    return new Promise((resolve, reject) => {
      rek.detectText(params, (err, data) => {
        if (err) {
          console.log(params, err);
          return reject(new Error(err));
        }
        return resolve(data);
      });
    });
  }
}
```

对应的返回结果：

```
{"TextDetections":[{"DetectedText":"test","Type":"LINE","Id":0,"Confidence":94.28083038330078,"Geometry":{"BoundingBox":{"Width":1.0391244888305664,"Height":1.0596693754196167,"Left":-0.023785971105098724,"Top":0.0029582083225250244},"Polygon":[{"X":-0.023785971105098724,"Y":0.0029582083225250244},{"X":1.0153385400772095,"Y":-0.012681752443313599},{"X":1.018007755279541,"Y":1.046987533569336},{"X":-0.021116789430379868,"Y":1.0626275539398193}]}},{"DetectedText":"test","Type":"WORD","Id":1,"ParentId":0,"Confidence":94.28083038330078,"Geometry":{"BoundingBox":{"Width":1.0354670286178589,"Height":1.058911919593811,"Left":-0.02123582363128662,"Top":-0.0029199719429016113},"Polygon":[{"X":-0.023785971105098724,"Y":0.0029582083225250244},{"X":1.0153385400772095,"Y":-0.012681752443313599},{"X":1.018007755279541,"Y":1.046987533569336},{"X":-0.021116789430379868,"Y":1.0626275539398193}]}}]}
```

微信小程序展示
---

于是在小程序端，我们只需要处理对应的结果即可：

```
wx.uploadFile({
  url: 'https://code.wdsm.io/upload',
  filePath: path,
  name: 'file',
  success: function (res) {
    let textDetections = JSON.parse(res.data)['TextDetections'];
    let code = '';

    for (let i = 0; i < textDetections.length; i++) {
      let textDetection = textDetections[i];
      if (textDetection.Type === 'LINE') {
        code = code + textDetection.DetectedText + '\n';
      }
    }

    that.setData({
      code: code,
      isUploading: false
    });
  },
  ...
})
```

然后，在页面上显示即可：

```
<view class="ai">
  <button type="default" class="btn" bindtap="chooseImage">上传图片</button>
  <view wx:if="{{code}}" class="weui-cells__title" size="mini">（点击复制文本）</view>
  <view wx:if="{{code}}" bindtap="copyCode">
     {{code}}
  </view>
</view>

<loading hidden="{{!isUploading}}">
  上传中...
</loading>
```

不过，这里有一个问题，这个 loading 好像有问题。

代码：[https://github.com/phodal/code](https://github.com/phodal/code)