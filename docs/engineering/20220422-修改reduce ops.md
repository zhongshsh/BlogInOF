# [修改 reduce ops](https://github.com/Oneflow-Inc/oneflow/pull/8085)

reduce_ops.py 里的max、min、sum等操作改成Functor直接导出。



## 问题1

ninja 突然出现编译错误问题，重新clone之后切换到当前分支也会编译报错，最终删除分支重建。

```
/home/zhongshanshan/workspace/f/oneflow/user/kernels/eager_nccl_kernels.cu(275): warning: overloaded virtual function "oneflow::user_op::OpKernel::Compute" is only partially overridden in class "oneflow::EagerNcclS2SKernel<oneflow::float16>"
          detected during:
            instantiation of class "oneflow::EagerNcclS2SKernel<T> [with T=oneflow::float16]" 
/home/zhongshanshan/workspace/f/oneflow/core/framework/op_kernel.h(332): here
            instantiation of "oneflow::user_op::OpKernel *oneflow::user_op::NewOpKernel<T,Args...>(Args &&...) [with T=oneflow::EagerNcclS2SKernel<oneflow::float16>, Args=<>]" 
/home/zhongshanshan/workspace/f/oneflow/core/framework/user_op_kernel_registry.h(84): here
            instantiation of "oneflow::user_op::OpKernelRegistry &oneflow::user_op::OpKernelRegistry::SetCreateFn<T>() [with T=oneflow::EagerNcclS2SKernel<oneflow::float16>]" 
(400): here

/home/zhongshanshan/workspace/f/oneflow/user/kernels/eager_nccl_kernels.cu(75): warning: overloaded virtual function "oneflow::user_op::OpKernel::Compute" is only partially overridden in class "oneflow::EagerNcclAllReduceKernel"

/home/zhongshanshan/workspace/f/oneflow/user/kernels/eager_nccl_kernels.cu(108): warning: overloaded virtual function "oneflow::user_op::OpKernel::Compute" is only partially overridden in class "oneflow::EagerNcclBroadcastKernel"

/home/zhongshanshan/workspace/f/oneflow/user/kernels/eager_nccl_kernels.cu(147): warning: overloaded virtual function "oneflow::user_op::OpKernel::Compute" is only partially overridden in class "oneflow::EagerNcclTouchKernel"

/home/zhongshanshan/workspace/f/oneflow/user/kernels/eager_nccl_kernels.cu(164): warning: overloaded virtual function "oneflow::user_op::OpKernel::Compute" is only partially overridden in class "oneflow::EagerNcclReduceKernel"

/home/zhongshanshan/workspace/f/oneflow/user/kernels/eager_nccl_kernels.cu(202): warning: overloaded virtual function "oneflow::user_op::OpKernel::Compute" is only partially overridden in class "oneflow::EagerNcclReduceScatterKernel"

/home/zhongshanshan/workspace/f/oneflow/user/kernels/eager_nccl_kernels.cu(244): warning: overloaded virtual function "oneflow::user_op::OpKernel::Compute" is only partially overridden in class "oneflow::EagerNcclAllGatherKernel"

[188/407] Building CUDA object CMakeFiles/oneflow.dir...core/ndarray/ndarray_apply_broadcast_binary_core.cu.o
ninja: build stopped: subcommand failed.
```



## 问题2

如何让同一个接口传入int 和 list？我看到有三种实现版本

- Int32List[1] axis=None
- Int32List[1] axis
- 写两个接口，例如

```
signature: [
"TensorTuple (Tensor input, Int32 indices_or_sections) => HsplitInt",
"TensorTuple (Tensor input, Int32List indices_or_sections) => HsplitVec",
]
```

选择 Int32List[1] axis=None `/oneflow/core/functional/impl/nn_functor.cpp` 会报错，显示参数不匹配，这个文件调用和初始化了 ReduceSum，即使将 axis 从开始的 {} 写成 {NULL}，仍然会出现奇怪的其他文件报错。

使用 Int32List[1] axis，则无法在缺失 axis 的情况下直接计算结果。

最终选择写两个接口。写两个接口同样存在瑕疵，即不可以使用 axis=None，应当使用 axis=[]



