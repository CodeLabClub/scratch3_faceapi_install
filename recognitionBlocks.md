
在Scratch3中加入人脸识别功能
=================================

README.md中涉及了fece-api.js在Scratch3中基本的安装过程，基本实现了实时人脸检测的功能。
此.md文件的主要内容为如何进一步地使用face-api.js所提供的API在Scratch3的index.js中进行修改，使其实现人脸识别的功能。
如果对face-api.js的API有进一步的兴趣，或者希望利用它自主开发更多有趣的功能，建议查看[face-api.js](https://github.com/justadudewhohacks/face-api.js)。

### 好的，我们进入代码

如果只是希望实现人脸识别功能，可以跳过以下段落，直接进入“操作方法”部分。

人脸识别的第一步是人脸的检测，这项功能我们已经通过调用face-api.js中的detectSingleFace()或detectAllFaces()方法实现。这两个函数均以this.video作为输入，即直接从Scratch3的摄像头(webcam)中读取数据进行分析，提取出单个(detectSingleFace())或者多个(detectAllFaces())人脸。为了实时性，在index.js中的detectSingleFace()函数和detectAllFaces()函数中，我们均使用了faceapi.TinyFaceDetectorOptions()作为人脸检测方式(Face Detector)
