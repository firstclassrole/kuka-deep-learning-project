# 作者网络结构与 ReLU：160 个神经元还是 160 层？

已核对用户提供的 `C:\Users\dyf\Downloads\tiago_dnn_ik-1.0.0.zip`。相关原文件完整复制在 `references/author_v1/tiago_dnn_mlp`，压缩包及文件校验值见 `references/author_v1/source.json`。没有执行作者的 TensorFlow 训练程序。

## ReLU 对什么起作用

每个隐藏神经元先接收上一层的数值，计算加权和：

```text
z = w1*x1 + w2*x2 + ... + wn*xn + b
a = ReLU(z) = max(0, z)
```

例如 z=-2 时输出 0，z=3 时输出 3。ReLU 是对神经元算出的数值进行激活处理，连接负责传递带权重的数值。一个隐藏层有 160 个神经元，就会对这 160 个加权和逐个执行 ReLU，再把 160 个输出传给下一层。

示意图常把 ReLU 画在 Dense 层之后，因为这是该层完成加权和之后的处理。在作者的 `Dense(..., activation='relu')` 写法中，激活已经包含在这一层内。它不表示额外增加了 160 层。

## SimpleMlp：一个隐藏层，160 个神经元

`simple_mlp_train.py` 第 21–25 行构造：

```python
self.model = tf.keras.models.Sequential([
    tf.keras.layers.Flatten(input_shape=[self.input_size]),
    tf.keras.layers.Dense(self.config['units'], activation='relu'),
    tf.keras.layers.Dense(output_size, activation='tanh'),
])
```

`mlp_config.yaml` 的 `simple_mlp` 配置中，`units: 160`。对本次七关节、位置加四元数的数据：

```text
7 个输入 → 1 个隐藏层（160 个神经元，各自执行 ReLU）→ 7 个输出（tanh）
```

Flatten 是形状处理，不是一个具有训练权重的隐藏 Dense 层。160 个神经元位于同一层，并排接收输入，不是依次串联的 160 层。

## DeepMlp：当前配置确实有 80 个隐藏层

同一个压缩包还提供了另一个模型 `deep_mlp_train.py`，第 21–24 行按配置循环添加层：

```python
for i in self.config['num_layers']:
    for j in range(i):
        self.model.add(tf.keras.layers.Dense(
            self.config['units'][self.config['num_layers'].index(i)], activation='relu'))
self.model.add(tf.keras.layers.Dense(output_size, activation='tanh'))
```

当前 `mlp_config.yaml` 的 `deep_mlp` 配置为：

```yaml
num_layers: [20, 40, 20]
units: [128, 256, 128]
```

因此结构是 20 个隐藏层各 128 神经元，接着 40 个隐藏层各 256 神经元，再接 20 个隐藏层各 128 神经元，总共 80 个隐藏层，最后另有一个输出 Dense 层。这个数字来自源码包当前配置，不表示论文所有实验都使用了同一个配置。

## 你的教学代码与这些模型是什么关系

- 已训练的 `runs/tiago_01` 对应一层 160 个神经元的隐藏结构，教学实现使用 NumPy、线性输出、Adam 以及只从训练集拟合的 robust scaling。
- 作者 SimpleMlp 的输出是 tanh，因此教程是教学改写，不是原代码逐行复现。
- 新增的“两层、每层 160”是容量对照实验，不是作者 80 层 DeepMlp 的复现。
- 单次验证对照中，一层 160 的平均位置误差为 120.82 mm，两层各 160 为 110.21 mm，约改善 8.8%。增加深度同时也增加参数量，所以还不能把全部改善单独归因于层数。
- 网络初值后加数值修正是另一条改进路线；其验证结果不能标为纯神经网络预测精度。

是否增加层数，应依据同一验证集的末端位置与朝向误差判断，不能仅凭神经元个数或“越深越好”决定。
