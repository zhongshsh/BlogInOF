
# 23 ninja: build stopped: subcommand failed

debug log：

- 更改 miniconda 为 anaconda，同样报错
- 按照 https://github.com/Oneflow-Inc/conda-env 配置环境，同样报错
- 重新 git clone git@github.com:Oneflow-Inc/oneflow.git，同样报错
- 考虑到 cuda 依赖包等问题，安装 PyTorch，同样报错
- 考虑到不管咋搞 black 版本报错都存在，因此新建一个干净的环境，更换python版本，同样报错
- 将 click 从 8.1 版本更换成 8.0 版本，可以解决问题，参考  https://github.com/psf/black/issues/2964 



报错如下

```
[226/1835] Performing build step for 'lz4'
make[1]: Entering directory '/home/zhongshanshan/workspace/tmp/oneflow/build/lz4/src/lz4/lib'
compiling static library
make[1]: Leaving directory '/home/zhongshanshan/workspace/tmp/oneflow/build/lz4/src/lz4/lib'
make[1]: Entering directory '/home/zhongshanshan/workspace/tmp/oneflow/build/lz4/src/lz4/lib'
compiling dynamic library 1.9.2
creating versioned links
make[1]: Leaving directory '/home/zhongshanshan/workspace/tmp/oneflow/build/lz4/src/lz4/lib'
[234/1835] cd /home/zhongshanshan/workspace/tmp/oneflow/...workspace/tmp/oneflow/tools/oneflow-tblgen --fix --quiet
FAILED: CMakeFiles/of_format /home/zhongshanshan/workspace/tmp/oneflow/build/CMakeFiles/of_format 
cd /home/zhongshanshan/workspace/tmp/oneflow/build && /home/zhongshanshan/anaconda3/envs/py38/bin/python3 /home/zhongshanshan/workspace/tmp/oneflow/ci/check/run_license_format.py -i /home/zhongshanshan/workspace/tmp/oneflow/oneflow --fix && /home/zhongshanshan/anaconda3/envs/py38/bin/python3 /home/zhongshanshan/workspace/tmp/oneflow/ci/check/run_license_format.py -i /home/zhongshanshan/workspace/tmp/oneflow/python --fix --exclude="oneflow/include" --exclude="oneflow/core" && /home/zhongshanshan/anaconda3/envs/py38/bin/python3 /home/zhongshanshan/workspace/tmp/oneflow/ci/check/run_clang_format.py --source_dir /home/zhongshanshan/workspace/tmp/oneflow/oneflow --fix --quiet && /home/zhongshanshan/anaconda3/envs/py38/bin/python3 /home/zhongshanshan/workspace/tmp/oneflow/ci/check/run_py_format.py --source_dir /home/zhongshanshan/workspace/tmp/oneflow/python --fix && /home/zhongshanshan/anaconda3/envs/py38/bin/python3 /home/zhongshanshan/workspace/tmp/oneflow/ci/check/run_clang_format.py --source_dir /home/zhongshanshan/workspace/tmp/oneflow/tools/oneflow-tblgen --fix --quiet
[files] 2473
[files] 730
clang-format version 10.0.0-4ubuntu1 

clang-format version 11.0.0

2458 files checked
Traceback (most recent call last):
  File "/home/zhongshanshan/anaconda3/envs/py38/lib/python3.8/runpy.py", line 192, in _run_module_as_main
    return _run_code(code, main_globals, None,
  File "/home/zhongshanshan/anaconda3/envs/py38/lib/python3.8/runpy.py", line 85, in _run_code
    exec(code, run_globals)
  File "/home/zhongshanshan/.local/lib/python3.8/site-packages/black.py", line 4139, in <module>
    patched_main()
  File "/home/zhongshanshan/.local/lib/python3.8/site-packages/black.py", line 4134, in patched_main
    patch_click()
  File "/home/zhongshanshan/.local/lib/python3.8/site-packages/black.py", line 4123, in patch_click
    from click import _unicodefun  # type: ignore
ImportError: cannot import name '_unicodefun' from 'click' (/home/zhongshanshan/.local/lib/python3.8/site-packages/click/__init__.py)
Please install black 19.10b0. For instance, run 'python3 -m pip install black==19.10b0 --user'
[295/1835] Performing configure step for 'opencv'
loading initial cache file /home/zhongshanshan/workspace/tmp/oneflow/build/opencv/tmp/opencv-cache-Release.cmake
-- The CXX compiler identification is GNU 9.4.0
```



最终的报错

```
a future release (Use -Wno-deprecated-gpu-targets to suppress warning).
Compiling  broadcast.cu                        > /home/zhongshanshan/workspace/tmp/oneflow/build/nccl/src/nccl/build/obj/collectives/device/broadcast_sumpostdiv_u64.o
nvcc warning : The 'compute_35', 'compute_37', 'compute_50', 'sm_35', 'sm_37' and 'sm_50' architectures are deprecated, and may be removed in a future release (Use -Wno-deprecated-gpu-targets to suppress warning).
Compiling  broadcast.cu                        > /home/zhongshanshan/workspace/tmp/oneflow/build/nccl/src/nccl/build/obj/collectives/device/broadcast_sumpostdiv_f16.o
nvcc warning : The 'compute_35', 'compute_37', 'compute_50', 'sm_35', 'sm_37' and 'sm_50' architectures are deprecated, and may be removed in a future release (Use -Wno-deprecated-gpu-targets to suppress warning).
Compiling  broadcast.cu                        > /home/zhongshanshan/workspace/tmp/oneflow/build/nccl/src/nccl/build/obj/collectives/device/broadcast_sumpostdiv_f32.o
nvcc warning : The 'compute_35', 'compute_37', 'compute_50', 'sm_35', 'sm_37' and 'sm_50' architectures are deprecated, and may be removed in a future release (Use -Wno-deprecated-gpu-targets to suppress warning).
Compiling  broadcast.cu                        > /home/zhongshanshan/workspace/tmp/oneflow/build/nccl/src/nccl/build/obj/collectives/device/broadcast_sumpostdiv_f64.o
nvcc warning : The 'compute_35', 'compute_37', 'compute_50', 'sm_35', 'sm_37' and 'sm_50' architectures are deprecated, and may be removed in a future release (Use -Wno-deprecated-gpu-targets to suppress warning).
Compiling  broadcast.cu                        > /home/zhongshanshan/workspace/tmp/oneflow/build/nccl/src/nccl/build/obj/collectives/device/broadcast_sumpostdiv_bf16.o
nvcc warning : The 'compute_35', 'compute_37', 'compute_50', 'sm_35', 'sm_37' and 'sm_50' architectures are deprecated, and may be removed in a future release (Use -Wno-deprecated-gpu-targets to suppress warning).
nvcc warning : The 'compute_35', 'compute_37', 'compute_50', 'sm_35', 'sm_37' and 'sm_50' architectures are deprecated, and may be removed in a future release (Use -Wno-deprecated-gpu-targets to suppress warning).
Archiving  objects                             > /home/zhongshanshan/workspace/tmp/oneflow/build/nccl/src/nccl/build/obj/collectives/device/colldevice.a
make[2]: Leaving directory '/home/zhongshanshan/workspace/tmp/oneflow/build/nccl/src/nccl/src/collectives/device'
Linking    libnccl.so.2.11.4                   > /home/zhongshanshan/workspace/tmp/oneflow/build/nccl/src/nccl/build/lib/libnccl.so.2.11.4
Archiving  libnccl_static.a                    > /home/zhongshanshan/workspace/tmp/oneflow/build/nccl/src/nccl/build/lib/libnccl_static.a
/home/zhongshanshan/workspace/tmp/oneflow/build/nccl/src/nccl/src
make[1]: Leaving directory '/home/zhongshanshan/workspace/tmp/oneflow/build/nccl/src/nccl/src'
ninja: build stopped: subcommand failed.
```

