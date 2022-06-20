---
title: 在tensorflow中实现hashmap（dict）的操作
typora-root-url: tensorflow_dict
typora-copy-images-to: tensorflow_dict
date: 2022-01-02 10:47:50
tags: TensorFlow
categories:
---



# 背景

希望实现float key -> index的行为，类似lookup table，然后基于index获得one hot key，做个矩阵乘法实现多个op中输出一个的操作



例子：

给定

```json
{
"3":"time_1",
"5":"time_2",
"89":"time_3"
}
```

判断某个feature（`(batch_size, 1)`）的取值，为3的时候输出`time_1`，为5的输出`time_2`，为89的时候输出`time_3`



~~只要获得了对应的one-hot形式的矩阵，做一个矩阵乘法即可获得~~

并不是矩阵乘法，应该直接用`tf.boolean_mask`来实现，传入两个shape完全相同的params和indicies，就够实现挑选出每一行对应的target，比如这样：

```python
mask_mat = tf.equal(flag_feat, flag_values)  # [batch_size, num_flag_values]

input_targets = tf.concat(
  [tf.reshape(original_preds[target], [-1, 1]) for target in sub_conf["input_target2flat_value"].keys()],
  axis=1)

original_preds[output_target] = tf.reshape(tf.boolean_mask(input_targets, mask_mat), [-1, 1])
```





# 法1：基于`tf.lookup`相关接口实现

```python
keys = [3, 5, 89]
input_tensor = tf.strings.as_string(tf.constant([3.0, 3.0, 5.0]), precision=1)
table = tf.lookup.StaticHashTable(tf.lookup.KeyValueTensorInitializer(list(map(lambda x: "{:.1f}".format(x), keys)),list(range(0, 3)), tf.string, tf.int32), default_value=-1)
tf.one_hot(table.lookup(input_tensor), depth=3).numpy()
```

![image-20220102110749394](/image-20220102110749394.png)



这里有个缺点是：要求输入为string，因此需要用`tf.strings.as_string`对输入的tensor做转换。可能会有精度的损失



# 法2：基于`tf.map_fn`或者`tf.vectorized_map`实现

`tf.vectorized_map`理论上比`tf.map_fn`更快，但是消耗更多内存



```python
keys = [3, 5, 89]
import copy


def convert_ind(input_x):
    all_cases = []
    for ind, key in enumerate(keys):
        all_cases.append(
            (
                tf.equal(input_x, float(key)),
                lambda: tf.constant(ind),
            )
        )
    return tf.case(all_cases)


input_tensor = tf.constant([3.0, 3.0, 5.0])
res = tf.map_fn(convert_ind, input_tensor, dtype=tf.int32)
print(res)
```



![image-20220103125331435](/image-20220103125331435.png)



# 法3：最佳实践，直接用`tf.equal`的broadcast实践

```python
keys = [3, 5, 89]
input_tensor = tf.constant([3.0, 3.0, 5.0])


res = tf.cast(
    tf.equal(
        tf.constant(keys, shape=(1, len(keys)), dtype=tf.float32),
        tf.reshape(input_tensor, shape=(-1, 1)),
    ),
    tf.int32,
)
print(res)
```

![image-20220103130309921](/image-20220103130309921.png)

这种方法是最好的，直接一步到位，获得最后的one-hot矩阵

