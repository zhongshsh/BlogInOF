# 魔术方法

魔术方法，python中所有以”__”(双下划线)作为名字开头和结尾的方法。具体介绍见  [Python 魔术方法指南 — PyCoder's Weelky CN](https://pycoders-weekly-chinese.readthedocs.io/en/latest/issue6/a-guide-to-pythons-magic-methods.html) 。

## oneflow 中的魔术方法——以 \__setitem__为例

### c++

```c++
class TensorSetItemFunctor {
 public:
  TensorSetItemFunctor() {}
  Maybe<void> operator()(const std::shared_ptr<one::Tensor>& x, const TensorIndex& index,
                         const std::shared_ptr<one::Tensor>& value) const {
    std::vector<detail::Slice> slice_indices;
    TensorTuple tensor_indices;
    std::vector<int64_t> expand_dims;
    std::vector<int64_t> target_dims;
    JUST(PrepareSliceIndices(index, *(x->shape()), &slice_indices, &tensor_indices, &expand_dims,
                             &target_dims));
    if (expand_dims.size()) {
      slice_indices = *JUST(RemoveExpandDimSlice(slice_indices, expand_dims));
    }
    int64_t ndims = x->shape()->NumAxes();
    CHECK_EQ_OR_RETURN(slice_indices.size(), ndims)
        << Error::RuntimeError() << "Failed to prepare slice indices.";
    // Not support combined indexing now
    if (!tensor_indices.empty()) {
      CHECK_OR_RETURN(tensor_indices.size() == ndims
                      && std::all_of(tensor_indices.begin(), tensor_indices.end(),
                                     [](const std::shared_ptr<Tensor>& index) { return index; }))
          << Error::RuntimeError()
          << "Combining indexing is not support for tensor setitem currently";
    }

    Shape target_shape(DimVector(target_dims.begin(), target_dims.end()));
    if (target_shape.Count(0) == 0) { return Maybe<void>::Ok(); }

    const auto& value_shape = value->shape();
    bool matched = [&]() {
      for (int i = 0; i < value_shape->NumAxes() - target_shape.NumAxes(); ++i) {
        if (value_shape->At(i) != 1) { return false; }
      }
      return true;
    }();
    CHECK_OR_RETURN(matched) << Error::RuntimeError() << "The tensor size mismatch. Target sizes: "
                             << target_shape.ToString()
                             << ", value sizes: " << value_shape->ToString();
    std::shared_ptr<one::Tensor> value_tensor(value);
    // TODO: replace reshape by unsqueeze with view mechanism.
    // after here, each scalar tensor will be one with one dimension.
    for (auto& tensor : tensor_indices) {
      if (tensor->ndim() == 0) { tensor = JUST(functional::Reshape(tensor, Shape({1}))); }
    }
    if (tensor_indices.size() == ndims) {  // advance indexing
      std::shared_ptr<Tensor> indices = JUST(functional::Stack(tensor_indices, 0));
      if (indices->shape()->elem_cnt() == 0) { return Maybe<void>::Ok(); }
      indices = JUST(functional::Transpose(indices, {1, 0}));
      value_tensor = JUST(functional::Expand(value_tensor, {indices->shape()->At(0)}));
      JUST(functional::TensorScatterNdUpdate(x, indices, value_tensor, /*inplace=*/true));
    } else {                              // slice update
      if (target_shape.NumAxes() != 0 &&  // NOLINT
          /*need_expand=*/value_shape->Count(0) != target_shape.Count(0)) {
        // Remove the beginning redundant 1-dimensions.
        if (value_shape->NumAxes() > target_shape.NumAxes()) {
          int64_t start_axis = value_shape->NumAxes() - target_shape.NumAxes();
          const auto& shape = JUST(value_shape->Slice(start_axis, value_shape->NumAxes()));
          value_tensor = JUST(Reshape(value, *shape));
        }
        value_tensor = JUST(Expand(value_tensor, target_shape));
      }
      std::vector<int64_t> start(ndims), end(ndims), step(ndims);
      DimVector slice_dims(ndims);
      for (int i = 0; i < ndims; ++i) {
        const auto& slice = slice_indices.at(i);
        start[i] = slice.start();
        end[i] = slice.end();
        step[i] = slice.step();
        slice_dims[i] = (end[i] - start[i] + step[i] - 1) / step[i];
      }
      Shape slice_shape(slice_dims);
      if (slice_shape != *(value_tensor->shape())) {
        value_tensor = JUST(Reshape(value_tensor, slice_shape));
      }
      if (x->is_local()) {
        JUST(SliceUpdate(x, value_tensor, start, end, step, /*inplace=*/true));
      } else {
        if (x->requires_grad() && autograd::GradMode::is_enabled()) {
          return Error::RuntimeError() << "Backward is not support for consistent tensor setitem,"
                                          "please use oneflow.no_grad() to disable autograd "
                                          "currently. We will fix this problem soon.";
        }
        JUST(LogicalSliceAssign(x, value_tensor, start, end, step));
      }
    }
    return Maybe<void>::Ok();
  }
};

ONEFLOW_FUNCTION_LIBRARY(m) {
  m.add_functor<impl::TensorSetItemFunctor>("TensorSetItem");
}
```



### yaml

```yaml
- name: "tensor_setitem"
  signature: "Void (Tensor x, TensorIndex index, Tensor value) => TensorSetItem"
  bind_python: True
```



### python

```python
def _setitem(self, key, value):
    if self.is_global:
        if isinstance(value, (int, float)):
            value = flow._C.global_constant(
                [1],
                value,
                dtype=self.dtype,
                placement=self.placement,
                sbp=flow.sbp.broadcast,
            )
        else:
            if value.is_global:
                value = value.to_global(sbp=flow.sbp.broadcast)
                # TODO: remove these lines after asymmetric boxing is ready
                local_tensor = value.to_local()
                if local_tensor.nelement() == 0:
                    local_tensor = flow.zeros(*value.shape)
                value = local_tensor.to_global(self.placement, sbp=flow.sbp.broadcast)
            else:
                value = value.to_global(self.placement, sbp=flow.sbp.broadcast)
    else:
        if isinstance(value, (int, float)):
            value = flow._C.constant([1], value, dtype=self.dtype, device=self.device)
        else:
            value = value.to(device=self.device)

    flow._C.tensor_setitem(self, key, value)
    return self

def RegisterMethods():
    Tensor.__setitem__ = _setitem
```

