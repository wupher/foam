# iOS 机器学习 Apple 分享会笔记 

- Dimi 风格迁移，提取照片风格，从而创建定制滤镜 Style Transfer 
- 日记 App 日记分析，生成标签  Text Clasifier
- 爆光，饱和，黑点 批量调整，从过去的操作中学习，自动预测

动态 App

简单流程:

1. Context 交互的上下文：图片，类型，风格，内容 

Vision ~ Image  classfication  对照片进行分类 

2. Content 内容，精简图片，像素数组
3. Interaction: 交互，从用户输入中学习到什么 ？  三个参数（曝光，饱和，黑点）

三要素，生成个性化经验

Tabular Regressor  针对三个数值做三个回归器


VNClassfication

本地学习，增量单次学习（隐私，性能 ）

梯度提升决策树  四种回归器，如果选择回归器（如何选择最佳算法）

预测，一般从分类开始

回归器 像小学奥数题，无需关心算法，寻找线索

最佳实践心得


思考：
机器学习时代，智能化学习的应用场景 