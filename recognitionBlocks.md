
在Scratch3中加入人脸识别功能
=================================

README.md中涉及了fece-api.js在Scratch3中基本的安装过程，基本实现了实时人脸检测的功能。
此.md文件的主要内容为如何进一步地使用face-api.js所提供的API在Scratch3的index.js中进行修改，使其实现人脸识别的功能。
如果对face-api.js的API有进一步的兴趣，或者希望利用它自主开发更多有趣的功能，建议查看[face-api.js](https://github.com/justadudewhohacks/face-api.js#high-level-api)。

### 好的，我们进入代码

如果只是希望实现人脸识别功能，可以跳过以下段落，直接进入“操作方法”部分。

首先我们从本地读取图片用于人脸识别：
``` javascript
facialFeatureDiskObtain(args)
        {
            var num = args.IMAGENUM //the number of base images that you want to load
            var faceRef = []

            this.timer = setTimeout(async () => {
                referenceData = [] //clear previous data
                const labels = ['sheldon','raj', 'leonard', 'howard']
                for(var i = 0; i< num ; i++){
                    if(referenceData.length == num) break;
                        // fetch image data from urls and convert blob to HTMLImage element
                    imgUrl = './static/imageBase/base_'+i+'.png'
                    const img = await faceapi.fetchImage(imgUrl)
                    if(img) {
                        // detect the face with the highest score in the image and compute it's landmarks and face descriptor
                        faceRef = await faceapi
                        .detectSingleFace(img)
                        .withFaceLandmarks()
                        .withFaceDescriptor()
    
                        const faceDescriptor = [faceRef.descriptor]
                        referenceData[i] = new faceapi.LabeledFaceDescriptors(labels[i], faceDescriptor)
                        console.log(referenceData[i])
                        if (!referenceData[i]) {
                            return
                        }
                    }
                }
             },1000)   
        }      
```
首先，我们读取输入的形参args.IMAGENUM，这一参数由用户设置，用于指出读取几张图片。faceRef作为数组，存放人脸检测得到的人脸数据。referenceData[]为全局变量，用于存放读取得到的人脸特征数据，设置为全局变量的原因是考虑到之后通过摄像头(webcam)读取的人脸特征数据也可以存储到同一数组下，便于人脸识别和统一管理（此处指统一清空）



