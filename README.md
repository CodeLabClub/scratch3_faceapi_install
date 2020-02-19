在Scratch3网页版中加入face-api模块过程记录
=========================================
注：此文档所述步骤基于“在 Scratch3 网页版中加入 knn 模块过程记录”，如果尚未搭建Scratch3网页版的开发架构，可参见该文“Scratch3 开发环境的下载和安装“部分，或 https://blog.just4fun.site/post/少儿编程/create-first-scratch3-extension/进行搭建。<br>

##    1.	基本架构的搭建
这一部分叙述内容亦可参见“在 Scratch3 网页版中加入 knn 模块过程记录” 中的2.1部分，因为同处于Scratch3构架，新建一个extension的方式大同小异。<br>

1.	首先新建index.js文件（空白文件，以下部分会重点叙述index.js文件的编写）放置到\Scratch3\scratch-vm\src\extensions\scratch3_faceapi目录下(在\Scratch3\scratch-vm\src\extensions\ 下新建scratch_faceapi )，完成最基本的构架。<br>
2.	设置extension-manager.js 文件，该文件位于\Scratch3\scratch-vm\src\extension-support路径下。修改方式如下，+所指部分为添加的代码：<br>
```text
const dispatch = require('../dispatch/central-dispatch');
const log = require('../util/log');
const maybeFormatMessage = require('../util/maybe-format-message');

const BlockType = require('./block-type');

const Scratch3KnnBlocks = require('../extensions/scratch3_knn');

+const Scratch3FaceapiBlocks = require('../extensions/scratch3_faceapi');
// These extensions are currently built into the VM repository but should not be loaded at startup.
// TODO: move these out into a separate repository?
// TODO: change extension spec so that library info, including extension ID, can be collected through static methods

const builtinExtensions = {
    // This is an example that isn't loaded with the other core blocks,
    // but serves as a reference for loading core blocks as extensions.
    coreExample: () => require('../blocks/scratch3_core_example'),
    // These are the non-core built-in extensions.
    pen: () => require('../extensions/scratch3_pen'),
    wedo2: () => require('../extensions/scratch3_wedo2'),
    music: () => require('../extensions/scratch3_music'),
    microbit: () => require('../extensions/scratch3_microbit'),
    text2speech: () => require('../extensions/scratch3_text2speech'),
    translate: () => require('../extensions/scratch3_translate'),
    videoSensing: () => require('../extensions/scratch3_video_sensing'),
    ev3: () => require('../extensions/scratch3_ev3'),
    makeymakey: () => require('../extensions/scratch3_makeymakey'),
    boost: () => require('../extensions/scratch3_boost'),
    gdxfor: () => require('../extensions/scratch3_gdx_for'),
    knnAlgorithm:() =>require('../extensions/scratch3_knn'),
 +  faceapi:()=>require('../extensions/scratch3_faceapi')
};
......
```
3.	在\Scratch3\scratch-gui\src\lib\libraries\extensions路径下新建文件夹faceapi。在其中放入faceapi.svg和faceapi-small.png。
4.	设置index.jsx 文件，该文件位于\Scratch3\scratch-gui\src\lib\libraries\extensions路径下。修改方式如下，+所指部部分为添加的代码：
首先调用刚才放置好的.svg和.png图片作为模块的封面：<br>
```text
......
import makeymakeyIconURL from './makeymakey/makeymakey.png';
import makeymakeyInsetIconURL from './makeymakey/makeymakey-small.svg';

import knnalgorithmImage from './knnAlgorithm/knnAlgorithm.png';
import knnalgorithmInsetImage from './knnAlgorithm/knnAlgorithm-small.svg';

import faceapiImage from './faceapi/faceapi.png';
import faceapiInsetImage from './faceapi/faceapi-small.svg';

import microbitIconURL from './microbit/microbit.png';
import microbitInsetIconURL from './microbit/microbit-small.svg';
import microbitConnectionIconURL from './microbit/microbit-illustration.svg';
import microbitConnectionSmallIconURL from './microbit/microbit-small.svg';
......
```
&emsp;之后设置faceapi extension的封面：
```text
export default [
......
},
+{
+       name: (
+           <FormattedMessage
+               defaultMessage="faceapi"
+               description="Name for the 'faceapi' extension"
+               id="gui.extension.faceapi.name"
+           />
+       ),
+       extensionId: 'faceapi',
+       iconURL: faceapiImage,
+       insetIconURL: faceapiInsetImage,
+       description: (
+           <FormattedMessage
+               defaultMessage="faceapi."
+               description="Description for the 'faceapi' extension"
+               id="gui.extension.faceapi.description"
+           />
+       ),
+       featured: true
+   },
......
 ```
&emsp;注意extensionId部分的内容和步骤2中第二个黄色框指示代码冒号前内容相同，这是faceapi extension的id 属性，因此必须相同，之后index.js的编写中也会有相应部分的提示。
 5.	测试。此时运行（/Scratch3/scratch-gui/ 中运行webpack-dev-server –https，之后新建console dialog 在/Scratch3/scratch-vm/ 中运行yarn run watch，在新建console dialog 在/Scratch3/scratch-gui/ 中运行yarn link scratch-vm）所得的网页，在https://127.0.0.1:8601/ 端口可以看到faceapi的封面已经完成，但此时点击不会有内容，我们此时需要对index.js的内容进行编辑。<br>
 
 
 &emsp;![image](https://github.com/TyutWzz-beep/scratch_faceapi_install/blob/master/images/faceapi_1.png)<br>
 
 ##     2.	index文件的编辑
注：这一部分index.js的编写主要是对之前knn extension的index.js的文件进行了删减，去掉了对于faceapi extension不必要的部分，并添加了构建faceapi所必要的部分。因此并没有对index.js比较一般的编写方式本身做深入说明。<br>

###     2.1	faceDetection block 的架构
1.	开始导入部分（1-13行）代码如下：<br>
```text
+require('babel-polyfill');
+//require ('@tensorflow/tfjs-node');
+const Runtime = require('../../engine/runtime');

+const ArgumentType = require('../../extension-support/argument-type');
+const BlockType = require('../../extension-support/block-type');
+const Clone = require('../../util/clone');
+const Cast = require('../../util/cast');
+const Video = require('../../io/video');
+const formatMessage = require('format-message');
+const canvas = require('canvas'); 
+const faceapi = require('face-api.js');
+const { Canvas, Image, ImageData } = canvas
......
```
6.	此时运行所得网页，会发现webpack-dev-server和yarn run watch两个窗口会报错（Can’t resolve……），解决方法可参见“Scratch3 网页版中加入 knn 模块过程记录”中“2.2 运行调试及附加依赖的安装”部分。根据yarn run watch 中的报错，在对应目录下执行yarn add xxxx 即可，其中xxxx指Can’t resolve 对应的依赖名称，此处为 face-api.js 和canvas两个依赖（dependency），需要停止批处理，即webpack-dev-server –https和yarn run watch（Ctrl + C）后执行，建议分步执行，即安装一个依赖运行一次网页确认依赖已经安装且有新依赖需要安装。执行后再运行网页看到不再报错即可。<br>
2.	之后复制knn extension对应的index.js文件的代码。
从* Sensor attribute video sensor block should report.* @readonly* @enum {string} 这一部分开始，直到代码结束。<br>
3.	修改复制得到代码中的Scratch3Knn为faceApi<br>

&emsp;![image](https://github.com/TyutWzz-beep/scratch_faceapi_install/blob/master/images/faceapi_2.png)<br>
&emsp;注：上图为使用vs code的全部修改方式，此外notepad++或其他编辑软件中一般有对应方式。此段需要修改的地方共有16处。<br>

4.	移动代码至getInfo()处（indx.js 344行左右）修改extensionId为‘faceapi‘，此处应和“1.基本架构的搭建”中第2、4步的id相同，否则无法在点击时自动显示extension对应的块。此外，name 为该extension在列表中的名字，仅标识用，安装需要修改即可。
5.	之后删除getInfo()中block[ ]中除 videoToggle和setVideoTransparency之外的其他模块，并将这些模块的opcode所对应的function一并删除（位置在class faceApi{ }中,getInfo(){ }的block[ ]之下）。
6.	在getInfo(){ }中加入如下代码：
```text
 blocks: [
                {
                    opcode: 'videoToggle',
                    text: 'turn video [VIDEO_STATE]',
                    arguments: {
                        VIDEO_STATE: {
                            type: ArgumentType.NUMBER,
                            menu: 'VIDEO_STATE',
                            defaultValue: VideoState.ON
                        }
                    }
                },
                {
                    opcode: 'setVideoTransparency',
                    text: 'set video transparency to [TRANSPARENCY]',
                    arguments: {
                        TRANSPARENCY: {
                            type: ArgumentType.NUMBER,
                            defaultValue: 50
                        }
                    }
                },
+               {
+                   opcode: 'faceDetection',
+                   blockType: BlockType.COMMAND,
+                   text: formatMessage({
+                       id: 'faceapi.faceDetection',
+                       default: 'faceDetection',
+                       description: 'faceDetection is loaded'
+                   })
+               }
            ],
```
&emsp;此时再次运行所得网页，点击faceapi对应封面，则有对应菜单弹出：<br>

&emsp;![image](https://github.com/TyutWzz-beep/scratch_faceapi_install/blob/master/images/faceapi_3.png)<br>

&emsp;且此时摄像头自动打开。<br>

&emsp;至此，block的架构完成，我们准备开始模型的装载（Loading the Models）<br>

###     2.2	faceapi 的模型装载
以下模型的装载过程基于https://github.com/justadudewhohacks/face-api.js#getting-started-loading-models 中README.md文件的 Loading the Models 模块编写，除了loadFromUri外的其他方式建议参见该网页。

1.	从github上下载faceapi.js的repo（git clone https://github.com/justadudewhohacks/face-api.js.git ）将其中的weights文件夹复制粘贴到\Scratch3\scratch-gui\static\ 路径下，重命名为faceapi。<br>
2.	运行所得到的网页，打开其控制台（console，使用Ctrl+Shift+I），点击控制台中Sourses选项，之后在Filesystem下点击Add folder to workspace 选项，转到\Scratch3\scratch-gui\static\ 路径下，选择faceapi。之后停止批处理（webpack-dev-server –https和yarn run watch，使用Ctrl+C），之后重新运行所得网页。示意图如下：<br>

&emsp;![image](https://github.com/TyutWzz-beep/scratch_faceapi_install/blob/master/images/faceapi_4.png)<br>
&emsp;注：此时Add folder to workspace目录下出现faceapi文件夹，则说明操作完成。

3.	此时打开scratch_faceapi下的index.js文件，将文件末尾的async knnInit() { } 文件内部的内容替换为（参见 https://www.youtube.com/watch?v=CVClHLwv-4I ）：
//载入模型（网络）
````text
+await faceapi.nets.tinyFaceDetector.loadFromUri('./static/faceapi'),
+await faceapi.nets.faceLandmark68Net.loadFromUri('./static/faceapi'),
+await faceapi.nets.faceRecognitionNet.loadFromUri('./static/faceapi'),
+await faceapi.nets.faceExpressionNet.loadFromUri('./static/faceapi')
+console.log(faceapi.nets) //控制台显示
````
主要载入了四个模型，也可以根据自己的需要载入更多的模型，可以参见 https://github.com/justadudewhohacks/face-api.js#getting-started-loading-models  此时重新运行（或者直接保存修改后的index.js文件，webpack会自动reload）所得网页，可以看到控制台显示如下：<br>

&emsp;![image](https://github.com/TyutWzz-beep/scratch_faceapi_install/blob/master/images/faceapi_5.png)<br>
&emsp;注：也可以在Network中查看结果。控制台中或Network中出现上述结果则说明模型载入成功。

&emsp;至此，模型的装载完成。我们下一步进行对face-api.js中部分API的调用。

###     2.3	API的调用
这一部分内容基于https://github.com/CodeLabClub/scratch3_knn/issues/7 所述内容编写。除了本部分设计API之外的其他属于face-api.js repo 的API 可以参考 https://github.com/justadudewhohacks/face-api.js 中的High Level API 部分。也可以参考 https://www.youtube.com/watch?v=CVClHLwv-4I 所示过程。

1.	首先将VideoToggle(args){ }部分做如下修改：
```text
videoToggle(args) {
        		const state = args.VIDEO_STATE;
        		this.globalVideoState = state;
       		 if (state === VideoState.OFF) {
            		this.runtime.ioDevices.video.disableVideo();
        		} else {
+            		this.runtime.ioDevices.video.enableVideo().then(() => {
                		// 获得video数据
+                		this.video = this.runtime.ioDevices.video.provider.video
+                		console.log('this.video got')
+            		});

            		// Mirror if state is ON. Do not mirror if state is ON_FLIPPED.
            		this.runtime.ioDevices.video.mirror = state === VideoState.ON;
        		}   
    		}

```
即，在this.runtime.ioDevices.video.enableVideo()添加then(() => {
                		// 获得video数据
                		this.video = this.runtime.ioDevices.video.provider.video
                		console.log('this.video got')
            		});
		部分，获得video数据。或者我觉得可以说是获得video的一个tag，方便下一步的调用和处理。
2.	在setVideoTransparency(args) { } 函数后添加faceDetection()函数：
```js
faceDetection()
    {
        return new Promise((resolve, reject) => {

            const originCanvas = this.runtime.renderer._gl.canvas  // 右上侧canvas
            const canvas = faceapi.createCanvasFromMedia(this.video)  // 创建用于绘制canvas

            canvas.width = 480
            canvas.height = 360

           // 将绘制的canvas覆盖于原canvas之上
            originCanvas.parentElement.style.position = 'relative'
            canvas.style.position = 'absolute'
            canvas.style.top = '0'
            canvas.style.left = '0'
            originCanvas.parentElement.append(canvas)

            // 循环检测并绘制检测结果
            this.timer = setInterval(async () => {
                const results = await faceapi
                    .detectSingleFace(this.video, new faceapi.TinyFaceDetectorOptions({ inputSize: 224, scoreThreshold: 0.5 })).withFaceLandmarks().withFaceExpressions()

                // 确认仅得到数据后进行绘制
                if (results) {
                    const displaySize = {width: 480 , height: 360}
                    const resizedDetections = faceapi.resizeResults(results, displaySize)
                    canvas.getContext('2d').clearRect(0, 0, canvas.width, canvas.height)
                    faceapi.draw.drawDetections(canvas, resizedDetections)
                    faceapi.draw.drawFaceLandmarks(canvas, resizedDetections)
                    faceapi.draw.drawFaceExpressions(canvas, resizedDetections) 
                }
                resolve('success')
            }, 1000);
        })
        }
```
3.	此时，运行所得网页，打开faceapi extension，搭建如下程序块组合，当按下“space”时即可实现人脸的提取和表情识别。<br>

&emsp;![image](https://github.com/TyutWzz-beep/scratch_faceapi_install/blob/master/images/faceapi_6.png)&emsp;![image](https://github.com/TyutWzz-beep/scratch_faceapi_install/blob/master/images/faceapi_7.png)<br>

```text
操作系统：windows10 家庭版
电脑型号：lenovo ThinkPadE460
浏览器：Google Chrome
```







 
 


