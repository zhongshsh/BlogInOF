# [添加 nn init orthogonal](https://github.com/Oneflow-Inc/oneflow/pull/8009) 

参考：

- [PyTorch](https://pytorch.org/docs/stable/_modules/torch/nn/init.html#orthogonal)
- [mxnet](https://github.com/apache/incubator-mxnet/blob/5f0efbbe33a1ef2af140e91a8fd367cd3bf92373/python/mxnet/initializer.py#L547)

## 工作流程

通过 VSCode 文件夹查找进行文件溯源，溯源流程如下：

- ./python/oneflow/nn/init.py
- ./python/oneflow/framework/tensor.py
- ./python/oneflow/\__init__.py
- ./python/oneflow/ops/initializer_util.py



## 问题

### 算子添加位置

溯源到 initializer_util.py，发现如下文件：

- ./python/oneflow/core/job/initializer_conf_pb2.py

- ./oneflow/core/job/initializer_conf.proto

initializer 返回的是 initializer_conf_util.InitializerConf，该类来源于 initializer_conf_pb2，initializer_conf_pb2 引用了 initializer_conf.proto，initializer_conf.proto 已经到了 C++ 层，通过检索已经不能准确在 C++ 中进行定位。

开始分析 C++ 算子作用。这个 initializer_conf 通过 _init_by_initializer_conf( ./python/oneflow/framework/tensor.py) 发挥作用，内部逻辑不是很清晰

```python
def _init_by_initializer_conf(tensor, initializer_conf, random_seed=None):
    if random_seed is None:
        random_seed = flow.default_generator.seed()
    shape = tuple(tensor.shape)
    initializer = initializer_util.GetInitializer(initializer_conf, random_seed, shape)

    np_arr = initializer_util.generate_values_by_initializer(
        initializer, shape, tensor.dtype
    )
    if tensor.is_global:
        src_tensor = flow.tensor(np_arr)
        src_tensor = src_tensor.to_global(
            placement=tensor.placement,
            sbp=tuple(flow.sbp.broadcast for _ in range(len(tensor.sbp))),
        )
        tensor.copy_(src_tensor)
    else:
        _copy_from_numpy_to_eager_local_tensor(
            tensor, np_arr,
        )
    return tensor
```

通过检索相关算子 PR，总结 initializer 算子的实现路径：

- [Dev kaiming uniform and normal initializer](https://github.com/Oneflow-Inc/oneflow/pull/1844)，是在 C++ 进行修改
- 也可以在 . ./python/oneflow/framework/tensor.py 修改



最终通过询问 晓雨 确定在 tensor 添加，但是仍然对 C++ 内部逻辑很茫然。

### 文档统一问题

- 需不需要写导包呢？
- 有些文档写示例代码的时候 加了 `.. code-block:: python` ，有些没有，它们的区别也许是在是否能识别空白分行，是否需要统一？

![image (3)](https://user-images.githubusercontent.com/62104945/163172877-d2fc9a5f-040b-4ff6-b195-cbd6e78e3205.png)

![image (6)](https://user-images.githubusercontent.com/62104945/163172966-b55fd976-9c95-4a9e-a33d-27673a9845a5.png)

![image (4)](https://user-images.githubusercontent.com/62104945/163172927-7fa69902-cea1-497a-98d5-84d8dd0a60b9.png)
![image (5)](https://user-images.githubusercontent.com/62104945/163172945-a40a7683-e365-4ed5-bccf-b9723a050042.png)

当前对齐了同一网页，但是并没有做到整个文档对齐。

### 算子对齐问题

测试用不了 AutoTest，因为 init 模块的算子内嵌了通过分布函数生成权值的模块，本身就具有随机性。因此对算子是否对齐进行了分析。

测试 [PyTorch 版本的代码](https://pytorch.org/docs/stable/_modules/torch/nn/init.html#orthogonal_)，发现填充矩阵并不是严格的（半）正定矩阵，矩阵与矩阵转置的乘积的对角线值为 1，但乘积其他元素不一定为 0，而且乘积是对称矩阵。说明矩阵向量是单位向量，但是不一定正交。测试代码如下：

```python
import torch
import numpy as np

tensor = torch.Tensor(3, 5)
gain = 1
if tensor.ndimension() < 2:
    raise ValueError("Only tensors with 2 or more dimensions are supported")

rows = tensor.size(0)
cols = tensor.numel() // rows
flattened = tensor.new(rows, cols).normal_(0, 1)

if rows < cols:
    flattened.t_()
q, r = torch.linalg.qr(flattened)
d = torch.diag(r, 0)
ph = d.sign()
q *= ph

if rows < cols:
    q.t_()

print(torch.matmul(q, q.t()))

'''
tensor([[ 1.0000e+00,  2.9802e-08, -1.4901e-08],
        [ 2.9802e-08,  1.0000e+00, -4.4703e-08],
        [-1.4901e-08, -4.4703e-08,  1.0000e+00]])
'''
```
测试 [mxnet 版本的代码](https://github.com/apache/incubator-mxnet/blob/5f0efbbe33a1ef2af140e91a8fd367cd3bf92373/python/mxnet/initializer.py#L547)，结果也不是（半）正定矩阵。测试代码如下：
```python
import torch
import numpy as np

tensor = torch.Tensor(3, 5)
rows = tensor.shape[0]
cols = np.prod(tensor.shape[1:])
flattened = np.random.normal(0.0, 1.0, size=(rows, cols))
if rows < cols:
    flattened = flattened.T
u, _, v = np.linalg.svd(flattened, full_matrices=False) 
if u.shape == flattened.shape:
    res = u
else:
    res = v

print(np.matmul(res, res.T))

'''
[[ 0.62967606  0.14242194  0.02739397 -0.45696218  0.05775161]
 [ 0.14242194  0.93914954 -0.0648611   0.1776109   0.03333284]
 [ 0.02739397 -0.0648611   0.51230715  0.05051142  0.49228014]
 [-0.45696218  0.1776109   0.05051142  0.43555551  0.05417968]
 [ 0.05775161  0.03333284  0.49228014  0.05417968  0.48331174]]
'''
```
最终使用与 PyTorch 对齐的版本：
```python
import torch
import numpy as np

tensor = torch.Tensor(3, 5)
rows = tensor.shape[0]
cols = np.prod(tensor.shape[1:])
flattened = np.random.normal(0.0, 1.0, size=(rows, cols))
if rows < cols:
    flattened = flattened.T
q, r = np.linalg.qr(flattened) 
d = np.diag(r, 0)
d = np.where(d < 0, -1, d)
d = np.where(d > 0, 1, d)
q *= d
if rows < cols:
    q = q.T

print(np.matmul(q, q.T))
```

