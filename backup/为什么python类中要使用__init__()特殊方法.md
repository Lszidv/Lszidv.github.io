今天看到如下代码（一个学习率调度器类），引发了笔者困扰已久的问题：__init__()特殊方法到底有什么用，为什么python类中要使用__init__()特殊方法？
```
class LRScheduler():
	"""
	Learning rate scheduler. If the validation loss does not decrease for the
	given number of `patience` epochs, then the learning rate will decrease by
	by given `factor`.
	"""
	def __init__(self, optimizer, patience=7, min_lr=1e-6, factor=0.5):
		"""
		new_lr = old_lr * factor
		:param optimizer: the optimizer we are using
		:param patience: how many epochs to wait before updating the lr
		:param min_lr: least lr value to reduce to while updating
		:param factor: factor by which the lr should be updated
		"""
		self.optimizer = optimizer
		self.patience = patience
		self.min_lr = min_lr
		self.factor = factor
		self.lr_scheduler = torch.optim.lr_scheduler.ReduceLROnPlateau(
				self.optimizer,
				mode='min',
				patience=self.patience,
				factor=self.factor,
				min_lr=self.min_lr,
				verbose=True
			)
	def __call__(self, val_loss):
		self.lr_scheduler.step(val_loss)
```

__init__是一个特殊方法，解释为类的初始化方法或构造器，功能也就不言而喻了，当创建一个类的实例时，python会自动调用这个方法。
__init__方法的第一个参数是self，表示调用它自身，之后定义的其他参数用于初始化其他变量。

为什么上面的代码不用 "def LRScheduler():" , 而要定义“ LRScheduler():”类呢？
  LRScheduler()类中有optimizer、patience、factor等配置参数，使用类将这些参数和逻辑进行封装，可以避免传入一大串参数，并且配合__call__(）特殊方法可以实现对类的实例的调用。

```
import torch
import torch.optim as optim

# 假设我们有一个模型
model = ...
# 创建一个优化器
optimizer = optim.Adam(model.parameters(), lr=0.001)

# 创建学习率调度器实例
scheduler = LRScheduler(optimizer, patience=5, min_lr=1e-7, factor=0.1)

# 在训练循环中
for epoch in range(num_epochs):
    # 训练模型...
    # 计算验证损失
    val_loss = ...
    # 更新学习率
    scheduler(val_loss)
```