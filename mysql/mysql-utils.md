1. 配置校验规则条件
2. 每个规则进行定义 Predicate 类型以及子类,并且配置每个规则的Factory，由factory对Predicate进行实例化
3. 规则处理器，handler，包括串行 和并行。后期进行优化：支持stage、node
4. 针对每个规则条件校验的返回进行监听，并根据优化方案进行建议，标记
5. 