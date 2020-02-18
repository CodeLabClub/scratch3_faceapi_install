在Scratch3网页版中加入face-api模块过程记录
=========================================
注：此文档所述步骤基于“在 Scratch3 网页版中加入 knn 模块过程记录”，如果尚未搭建Scratch3网页版的开发架构，可参见该文“Scratch3 开发环境的下载和安装“部分，或 https://blog.just4fun.site/post/少儿编程/create-first-scratch3-extension/进行搭建
##    1.	基本架构的搭建
这一部分叙述内容亦可参见“在 Scratch3 网页版中加入 knn 模块过程记录” 中的2.1部分，因为同处于Scratch3构架，兴建一个extension的方式大同小异。

1.	首先新建index.js文件（空白文件，以下部分会重点叙述index.js文件的编写）放置到\Scratch3\scratch-vm\src\extensions\scratch3_faceapi目录下(在\Scratch3\scratch-vm\src\extensions\ 下新建scratch_faceapi )，完成最基本的构架。
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
之后设置faceapi extension的封面：
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
 注意extensionId部分的内容和步骤2中第二个黄色框指示代码冒号前内容相同，这是faceapi extension的id 属性，因此必须相同，之后index.js的编写中也会有相应部分的提示。
 5.	测试。此时运行（/Scratch3/scratch-gui/ 中运行webpack-dev-server –https，之后新建console dialog 在/Scratch3/scratch-vm/ 中运行yarn run watch，在新建console dialog 在/Scratch3/scratch-gui/ 中运行yarn link scratch-vm）所得的网页，在https://127.0.0.1:8601/ 端口可以看到faceapi的封面已经完成，但此时点击不会有内容，我们此时需要对index.js的内容进行编辑。
 


