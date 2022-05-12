# [Fix upsample shape infer bug](https://github.com/Oneflow-Inc/oneflow/pull/8159) 

通过这个PR理解 [如何在OneFlow中新增User Op](https://github.com/Oneflow-Inc/OneTeam/blob/0a3f8b2d4beb75ed902cc6ce210e9af4e19882da/docs/如何在OneFlow中新增User Op.md)。



## 流程

定义op：oneflow/ir/include/OneFlow/OneFlowUserOps.td

实现op：oneflow/user/ops/upsample_op.cpp

实现支持op计算的CPU和GPU逻辑：

- CPU: oneflow/user/kernels/\*.cpp
- GPU: oneflow/user/kernels/\*.cu

注册Functional接口：

- oneflow/core/functional/impl/array_functor.cpp
- oneflow/core/functional/functional_api.yaml

实现求导逻辑：oneflow/core/autograd/gradient_funcs/upsample.cpp

实现python接口：python/oneflow/nn/modules/interpolate.py

完成测试：python/oneflow/test/modules/test_upsample.py



## 问题总结

- 没有梳理逻辑导致有些文件没有改全，例如td文件没有增加参数会导致core dump问题
- scale原始类型为float，由于部分upsample对参数精度敏感，需要将scale改为double