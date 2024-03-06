# ChatGLM3 源码解析

## `MLP`

```py
class MLP(torch.nn.Module):
    """
    MLP 把隐藏状态的尺寸从 HidSize 映射到 4HidSize，执行非线性激活，然后再映射回 HidSize
    """

    def __init__(self, config: ChatGLMConfig, device=None):
        super(MLP, self).__init__()

        # 控制是否添加偏置
        self.add_bias = config.add_bias_linear

        # 用于隐藏状态扩张的线性层
        # 由于激活采用 SwigLU，输出尺寸需要翻倍，也就是 8HidSize
        self.dense_h_to_4h = nn.Linear(
            config.hidden_size,
            config.ffn_hidden_size * 2,
            bias=self.add_bias,
            device=device,
            **_config_to_kwargs(config)
        )

        # SwigLU 的定义
        def swiglu(x):
            # 首先把输入最后一维平等分割
            # 然后计算第一部分的 SILU 乘上第二部分
            x = torch.chunk(x, 2, dim=-1)
            return F.silu(x[0]) * x[1]

        self.activation_func = swiglu

        # 用于收缩隐藏状态的线性层
        self.dense_4h_to_h = nn.Linear(
            config.ffn_hidden_size,
            config.hidden_size,
            bias=self.add_bias,
            device=device,
            **_config_to_kwargs(config)
        )

    def forward(self, hidden_states):
        # 输入隐藏状态的形状为 [SeqLen, BatchSize, HidSize]
        # 经过第一个线性层，变为 [SeqLen, BatchSize, 8HidSize]
        intermediate_parallel = self.dense_h_to_4h(hidden_states)
        # 激活后变为 [SeqLen, BatchSize, 4idSize]
        intermediate_parallel = self.activation_func(intermediate_parallel)
        # 经过第二个线性层，变为 [SeqLen, BatchSize, HidSize]
        output = self.dense_4h_to_h(intermediate_parallel)
        return output
```

## `GLMBlock`

```py
class GLMBlock(torch.nn.Module):
    """
    单个 GLM 块 包含注意力层和 MLP 层

    接受隐藏状态输入，尺寸为 [SeqLen, BatchSize, HidSize]，输出相同尺寸隐藏状态
    """

    def __init__(self, config: ChatGLMConfig, layer_number, device=None):
        super(GLMBlock, self).__init__()
        # GLM 块的序号
        self.layer_number = layer_number
        # 控制残差是否是在 LayerNorm 之前还是之后
        self.apply_residual_connection_post_layernorm = config.apply_residual_connection_post_layernorm
        # 控制残差连接是否是 FP32（好像没用到）
        self.fp32_residual_connection = config.fp32_residual_connection
        # 层标准化函数，可选用 RMSNorm 或者 LayerNorm
        LayerNormFunc = RMSNorm if config.rmsnorm else LayerNorm
        # 输入之后，注意力层之前的 LayerNorm
        self.input_layernorm = LayerNormFunc(config.hidden_size, eps=config.layernorm_epsilon, device=device,
                                             dtype=config.torch_dtype)

        # 自注意力
        self.self_attention = SelfAttention(config, layer_number, device=device)
        # Dropout 比例
        self.hidden_dropout = config.hidden_dropout

        # 注意力层之后，MLP 之前的 LayerNorm
        self.post_attention_layernorm = LayerNormFunc(config.hidden_size, eps=config.layernorm_epsilon, device=device,
                                                      dtype=config.torch_dtype)

        # MLP
        self.mlp = MLP(config, device=device)

    def forward(
            self, hidden_states, attention_mask, rotary_pos_emb, kv_cache=None, use_cache=True,
    ):
        # 输入尺寸 [DeqLen, BatchSize, HidSize]
        # 首先对输入使用 LayerNorm
        layernorm_output = self.input_layernorm(hidden_states)
        # 将中间结果传入注意力层，尺寸不变
        # kv_cache 是注意力层的 K 和 V 中间变量
        attention_output, kv_cache = self.self_attention(
            layernorm_output,
            attention_mask,
            rotary_pos_emb,
            kv_cache=kv_cache,
            use_cache=use_cache
        )

        # 根据【残差项是否在 LayerNorm 后面】，指定残差项
        if self.apply_residual_connection_post_layernorm:
            residual = layernorm_output
        else:
            residual = hidden_states

        # 对中间结果应用 Dropout
        layernorm_input = torch.nn.functional.dropout(attention_output, p=self.hidden_dropout, training=self.training)
        # 添加残差项
        layernorm_input = residual + layernorm_input

        # 对中间结果应用 LayerNorm
        layernorm_output = self.post_attention_layernorm(layernorm_input)

        # 将中间结果输入 MLP，尺寸不变
        mlp_output = self.mlp(layernorm_output)

        # 根据【残差项是否在 LayerNorm 后面】，指定第二个残差项
        if self.apply_residual_connection_post_layernorm:
            residual = layernorm_output
        else:
            residual = layernorm_input

        # 对中间结果应用 Dropout
        output = torch.nn.functional.dropout(mlp_output, p=self.hidden_dropout, training=self.training)
        # 添加残差
        output = residual + output

        return output, kv_cache
```

## `GLMTransformer`


```py
class GLMTransformer(torch.nn.Module):
    """Transformer 包含 NLayer 个 GLM 块和嵌入层，但不包含输出层"""

    def __init__(self, config: ChatGLMConfig, device=None):
        super(GLMTransformer, self).__init__()
        # 控制残差连接是否是 FP32（好像没用到）
        self.fp32_residual_connection = config.fp32_residual_connection
        # 控制 GLM 块之后是否应用 LayerNorm
        self.post_layer_norm = config.post_layer_norm

        # 层数，NLayer
        self.num_layers = config.num_layers

        # 用于构建单个 GLM 块的函数
        def build_layer(layer_number):
            return GLMBlock(config, layer_number, device=device)

        # GLM 块的列表
        self.layers = torch.nn.ModuleList([build_layer(i + 1) for i in range(self.num_layers)])

        # 如果最后使用 LayerNorm 就创建它
        if self.post_layer_norm:
            LayerNormFunc = RMSNorm if config.rmsnorm else LayerNorm
            # Final layer norm before output.
            self.final_layernorm = LayerNormFunc(config.hidden_size, eps=config.layernorm_epsilon, device=device,
                                                 dtype=config.torch_dtype)
        # 控制是否开启梯度检查点（也就是不缓存中间张量）
        self.gradient_checkpointing = False

    def _get_layer(self, layer_number):
        return self.layers[layer_number]

    def forward(
            self, hidden_states, attention_mask, rotary_pos_emb, kv_caches=None,
            use_cache: Optional[bool] = True,
            output_hidden_states: Optional[bool] = False,
    ):
        # 如果没有传入 KVCache
        # 将其初始化为全 None 数组方便传给各个块
        if not kv_caches:
            kv_caches = [None for _ in range(self.num_layers)]
        # 如果设置了 `use_cache`，应当返回各层 KVCache
        # 初始化 `presents` 为空元组，之后存储它们
        presents = () if use_cache else None
        # 如果启用了梯度检查点，并且是训练模式，并且返回 KVCache，将其关闭
        if self.gradient_checkpointing and self.training:
            if use_cache:
                logger.warning_once(
                    "`use_cache=True` is incompatible with gradient checkpointing. Setting `use_cache=False`..."
                )
                use_cache = False

        # `all_self_attentions`应当储存各个块的注意力矩阵
        # 但是这里没有，仅仅将其初始化为 None
        all_self_attentions = None
        # 如果指定了`output_hidden_states`，应当返回所有隐藏状态
        # 也就是这个模块的输入以及各个块的输出
        # 初始化`all_hidden_states`为空元组，之后储存它们
        all_hidden_states = () if output_hidden_states else None
        # 遍历各个 GLM 块
        for index in range(self.num_layers):
            # 将当前隐藏状态储存到`all_hidden_states`
            if output_hidden_states:
                all_hidden_states = all_hidden_states + (hidden_states,)

            # 将隐藏状态传入每个 GLM 块，得到下一个隐藏状态
            # 如果启用了梯度检查点并处于训练模式
            # 就使用梯度检查点调用每个块，否则直接调用
            layer = self._get_layer(index)
            if self.gradient_checkpointing and self.training:
                layer_ret = torch.utils.checkpoint.checkpoint(
                    layer,
                    hidden_states,
                    attention_mask,
                    rotary_pos_emb,
                    kv_caches[index],
                    use_cache
                )
            else:
                layer_ret = layer(
                    hidden_states,
                    attention_mask,
                    rotary_pos_emb,
                    kv_cache=kv_caches[index],
                    use_cache=use_cache
                )
            hidden_states, kv_cache = layer_ret
            # 将该层的 KVCache 储存到`presents`中
            if use_cache:
                presents = presents + (kv_cache,)

        # 将最终隐藏状态储存到`all_hidden_states`
        if output_hidden_states:
            all_hidden_states = all_hidden_states + (hidden_states,)

        # 对最终隐藏状态使用 LayerNorm‘
        if self.post_layer_norm:
            hidden_states = self.final_layernorm(hidden_states)

        # 返回最终隐藏状态，所有层的 KVCache，所有隐藏状态，所有层的注意力矩阵（为 None）
        return hidden_states, presents, all_hidden_states, all_self_attentions
```

## `ChatGLMModel`

```py
class ChatGLMModel(ChatGLMPreTrainedModel):
    def __init__(self, config: ChatGLMConfig, device=None, empty_init=True):
        super().__init__(config)
        # 定义初始化方式
        if empty_init:
            init_method = skip_init
        else:
            init_method = default_init
        init_kwargs = {}
        if device is not None:
            init_kwargs["device"] = device
        # 词嵌入层
        self.embedding = init_method(Embedding, config, **init_kwargs)
        # NLayer：GLM 块数量
        self.num_layers = config.num_layers
        # NGroup：MQA 注意力的分组数
        self.multi_query_group_num = config.multi_query_group_num
        # HeadSize：每个头的维数
        self.kv_channels = config.kv_channels

        # SeqLen：序列长度
        self.seq_length = config.seq_length
        # 如果定义了 HeadSize 就使用它作为 ROPE 的尺寸
        # 如果没有，就从 HidSize // NHead 计算出来
        rotary_dim = (
            config.hidden_size // config.num_attention_heads if config.kv_channels is None else config.kv_channels
        )

        # 位置嵌入层
        self.rotary_pos_emb = RotaryEmbedding(rotary_dim // 2, rope_ratio=config.rope_ratio,
                                              original_impl=config.original_rope, device=device,
                                              dtype=config.torch_dtype)
        # GLM 块，也叫编码器
        self.encoder = init_method(GLMTransformer, config, **init_kwargs)
        # 输出层是个线性层，尺寸为 [HidSize, VocabSize]
        self.output_layer = init_method(nn.Linear, config.hidden_size, config.padded_vocab_size, bias=False,
                                        dtype=config.torch_dtype, **init_kwargs)
        # PreSeqLen：PTuning 前缀长度，为空时不启用 PTuning
        self.pre_seq_len = config.pre_seq_len
        # 控制前缀编码器中是否启用投影变换
        self.prefix_projection = config.prefix_projection
        if self.pre_seq_len is not None:
            # 如果启用了 PTuning，需要冻结除了前缀编码器的所有参数
            for param in self.parameters():
                param.requires_grad = False
            # 生成前缀 ID，是 0 - PreSeqLen 的数组
            self.prefix_tokens = torch.arange(self.pre_seq_len).long()
            # 前缀编码器，用于将前缀 ID 变成前缀嵌入
            self.prefix_encoder = PrefixEncoder(config)
            # 用于前缀嵌入的 Dropout
            self.dropout = torch.nn.Dropout(0.1)

    def get_input_embeddings(self):
        return self.embedding.word_embeddings

    # 使用前缀 ID 生成前缀嵌入
    def get_prompt(self, batch_size, device, dtype=torch.half):
        # 前缀 ID 保存在 self.prefix_tokens，尺寸为 [PreSeqLen]
        # 将其便成为 [BatchSize, PreSeqLen]
        prefix_tokens = self.prefix_tokens.unsqueeze(0).expand(batch_size, -1).to(device)
        # 将其传入前缀编码器，得到前缀嵌入，尺寸为 [BatchSize, PreSeqLen, NLayer x 2 x NGroup x HeadSize]
        past_key_values = self.prefix_encoder(prefix_tokens).type(dtype)
        # 将其变形为 [BatchSize, PreSeqLen, NLayer x 2,  NGroup, HeadSize]
        past_key_values = past_key_values.view(
            batch_size,
            self.pre_seq_len,
            self.num_layers * 2,
            self.multi_query_group_num,
            self.kv_channels
        )
        # 应用 Dropout
        past_key_values = self.dropout(past_key_values)
        # 将其变形为 [NLayer x 2, PreSeqLen, BatchSize,  NGroup, HeadSize]
        # 并且将第一维均等拆分，得到前缀嵌入 K 和 V
        past_key_values = past_key_values.permute([2, 1, 0, 3, 4]).split(2)
        return past_key_values

    def forward(
            self,
            input_ids,
            position_ids: Optional[torch.Tensor] = None,
            attention_mask: Optional[torch.BoolTensor] = None,
            full_attention_mask: Optional[torch.BoolTensor] = None,
            past_key_values: Optional[Tuple[Tuple[torch.Tensor, torch.Tensor], ...]] = None,
            inputs_embeds: Optional[torch.Tensor] = None,
            use_cache: Optional[bool] = None,
            output_hidden_states: Optional[bool] = None,
            return_dict: Optional[bool] = None,
    ):
        # 输入为单词 ID，尺寸为 [BatchSize, SeqLen]
        
        # 控制是否返回所有隐藏状态
        output_hidden_states = (
            output_hidden_states if output_hidden_states is not None else self.config.output_hidden_states
        )
        # 控制是否返回 KVCache
        use_cache = use_cache if use_cache is not None else self.config.use_cache
        # 控制返回字典还是元组
        return_dict = return_dict if return_dict is not None else self.config.use_return_dict
        # 从输入尺寸获取 BatchSize 和 SeqLen
        batch_size, seq_length = input_ids.shape

        # 如果传入了单词嵌入，则直接使用
        # 否则将单词 ID 传入嵌入层得到单词嵌入
        # 尺寸为 [SeqLen, BatchSize, HidSize]
        if inputs_embeds is None:
            inputs_embeds = self.embedding(input_ids)

        if self.pre_seq_len is not None:
            # 如果启用了 PTuning，并且没有传入 KVCache
            # 将前缀嵌入作为 KVCache，保证它们添加到单词 KV 的前面
            if past_key_values is None:
                past_key_values = self.get_prompt(batch_size=batch_size, device=input_ids.device,
                                                  dtype=inputs_embeds.dtype)
            
            # 如果传入了掩码数组
            # 为前缀生成相同尺寸的注意力掩码，全为 1
            # 然后按照 SeqLen 的维度拼接到现有掩码前面
            if attention_mask is not None:
                attention_mask = torch.cat([attention_mask.new_ones((batch_size, self.pre_seq_len)),
                                            attention_mask], dim=-1)

        # 如果没有传入掩码矩阵
        # 并且（传入了掩码数组，并且掩码数组不是全 1，
        #      或者传入 KVCache，并且 SeqLen 不等于 1）
        # 则使用单词 ID、KVCache 和掩码数组生成掩码矩阵
        if full_attention_mask is None:
            if (attention_mask is not None and not attention_mask.all()) or (past_key_values and seq_length != 1):
                full_attention_mask = self.get_masks(input_ids, past_key_values, padding_mask=attention_mask)

        # 应用位置嵌入
        # 首先获取长度为 SeqLen 的位置嵌入
        rotary_pos_emb = self.rotary_pos_emb(self.seq_length)
        # 如果传入了位置 ID，则按照位置 ID 索引
        # 否则获取位置嵌入的前 SeqLen 个
        if position_ids is not None:
            rotary_pos_emb = rotary_pos_emb[position_ids]
        else:
            rotary_pos_emb = rotary_pos_emb[None, :seq_length]
        # 前两维转至，尺寸为 [SeqLen, BatchSize, HidSize]
        rotary_pos_emb = rotary_pos_emb.transpose(0, 1).contiguous()

        # 将单词嵌入和其它东西传入编码器，得到最终隐藏状态，KVCache，所有隐藏状态和所有层的注意力矩阵（None）
        hidden_states, presents, all_hidden_states, all_self_attentions = self.encoder(
            inputs_embeds, full_attention_mask, rotary_pos_emb=rotary_pos_emb,
            kv_caches=past_key_values, use_cache=use_cache, output_hidden_states=output_hidden_states
        )

        # 如果不要求返回字典，将这四个东西打包成元组返回
        if not return_dict:
            return tuple(v for v in [hidden_states, presents, all_hidden_states, all_self_attentions] if v is not None)

        # 否则打包成字典返回
        return BaseModelOutputWithPast(
            last_hidden_state=hidden_states,
            past_key_values=presents,
            hidden_states=all_hidden_states,
            attentions=all_self_attentions,
        )
```