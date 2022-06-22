### keras配置gpu

>注意：keras,tensorflow-gpu,cuda，cudnn都要对应好版本。还要查看电脑支持的最高cuda版本号，小于等于此版本号即可。
>
>tensorflow-gpu 2.1.0
>
>cuda 10.1
>
>cudnn 7.6.1 for cuda 10.1
>
>python 3.6.5
>
>keras 2.2.4

```
参考此知乎教程即可：https://zhuanlan.zhihu.com/p/114649213
#tensorflow和cuda，cuDNN对应版本关系
https://tensorflow.google.cn/install/source_windows
#cudnn下载地址
https://developer.nvidia.com/cuda-10.1-download-archive-base?target_os=Windows&target_arch=x86_64&target_version=10&target_type=exelocal
#需要下载此visual c++程序包,避免坑。
os.environ["CUDA_VISIBLE_DEVICES"] = "0"
#python版本python3.6.5
```

##### tensorflow测试

```
#程序检查是否安装成功，来个代码看看。查看本机的GPU设备。
from tensorflow.python.client import device_lib
import tensorflow as tf
print(device_lib.list_local_devices())
print(tf.test.is_built_with_cuda())
```

##### keras测试

```
from keras import backend as K
print(K.tensorflow_backend._get_available_gpus())
```

##### 使用gpu

```
os.environ["CUDA_VISIBLE_DEVICES"] = "0"
```
