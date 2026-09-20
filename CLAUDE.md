# Claude 全局配置

## 语言设置

**默认使用中文进行所有输出和思考过程**

- 所有回复、解释、说明都使用中文
- 内部思考过程使用中文
- 代码注释使用中文
- 文档和说明使用中文
- Git 提交信息使用中文

## 例外情况

以下情况可以使用英文或其他语言：

1. **代码本身**：变量名、函数名、类名等保持英文（遵循编程最佳实践）
2. **项目指定**：如果项目的 CLAUDE.md 或文档明确要求使用其他语言
3. **技术术语**：专业术语可保留英文，但需提供中文解释
4. **不支持中文的场景**：某些工具、API 或系统不支持中文时
5. **用户明确要求**：用户在对话中明确要求使用其他语言

## 代码风格

- 代码注释使用中文
- 变量和函数命名使用英文（符合国际惯例）
- 文档字符串使用中文
- README 等文档使用中文（除非项目要求英文）

## 代码质量要求

**核心原则：编写高可读性、高可维护性的代码**

### 命名规范
- **变量名**：使用有意义的描述性名称，避免单字母变量（除循环索引）
  - ✅ `user_count`, `max_iterations`, `learning_rate`
  - ❌ `uc`, `mi`, `lr`
- **函数名**：动词开头，清晰表达功能
  - ✅ `calculate_loss()`, `load_dataset()`, `validate_model()`
  - ❌ `calc()`, `load()`, `val()`
- **类名**：使用名词，PascalCase 风格
  - ✅ `DataLoader`, `ModelTrainer`, `ImageProcessor`
  - ❌ `loader`, `trainer`, `processor`
- **常量名**：全大写，下划线分隔
  - ✅ `MAX_EPOCHS`, `DEFAULT_BATCH_SIZE`, `MODEL_VERSION`

### 注释要求

**必须添加注释的场景**：
1. **函数/方法定义**：说明功能、参数、返回值
   ```python
   def train_model(model, dataloader, epochs=100):
       """
       训练模型
       
       参数:
           model: 待训练的神经网络模型
           dataloader: 数据加载器
           epochs: 训练轮数，默认 100
       
       返回:
           trained_model: 训练完成的模型
           loss_history: 训练损失历史记录
       """
   ```

2. **复杂逻辑**：解释算法思路、为什么这样实现
   ```python
   # 使用 Wing Loss 而非 MSE，因为它对小误差更敏感，适合关键点检测
   loss = wing_loss(predictions, targets, w=10.0, epsilon=2.0)
   ```

3. **关键参数**：说明参数的意义和取值范围
   ```python
   learning_rate = 0.001  # 初始学习率，经过多次实验确定
   patience = 40          # ReduceLROnPlateau 的耐心值，防止过早降低学习率
   ```

4. **数据格式**：说明输入输出的数据结构
   ```python
   # 输入形状: (batch_size, 3, 64, 64) - RGB 图像
   # 输出形状: (batch_size, 40) - 20 个关键点的 (x, y) 坐标
   landmarks = model(images)
   ```

5. **边界条件**：解释特殊情况的处理
   ```python
   # 裁剪坐标到 [0, 1] 范围，防止标注错误导致训练失败
   landmarks = np.clip(landmarks, 0.0, 1.0)
   ```

6. **魔法数字**：解释硬编码数值的来源
   ```python
   feature_dim = 4544  # 多尺度特征拼接后的维度: 64+128+128+256+256+3712
   ```

**不需要添加注释的场景**：
- 代码本身已经很清晰的简单操作
- 标准库函数的常规用法
- 变量名已经充分说明含义的赋值语句

### 代码结构
- **函数长度**：单个函数不超过 50 行（复杂函数拆分为多个子函数）
- **单一职责**：每个函数只做一件事
- **避免深层嵌套**：if/for 嵌套不超过 3 层
- **提前返回**：使用 early return 减少嵌套
  ```python
  # ✅ 推荐
  def process_data(data):
      if data is None:
          return None
      if len(data) == 0:
          return []
      return transform(data)
  
  # ❌ 不推荐
  def process_data(data):
      if data is not None:
          if len(data) > 0:
              return transform(data)
          else:
              return []
      else:
          return None
  ```

### 错误处理
- **明确的异常类型**：不使用裸 `except`
- **有意义的错误信息**：帮助快速定位问题
  ```python
  # ✅ 推荐
  try:
      landmarks = load_landmarks(label_path)
  except FileNotFoundError:
      raise FileNotFoundError(f"标签文件不存在: {label_path}")
  except ValueError as e:
      raise ValueError(f"标签格式错误 {label_path}: {e}")
  
  # ❌ 不推荐
  try:
      landmarks = load_landmarks(label_path)
  except:
      pass
  ```

### 代码组织
- **导入顺序**：标准库 → 第三方库 → 本地模块
- **逻辑分组**：用空行分隔不同功能模块
- **常量提取**：避免硬编码，集中定义配置参数
  ```python
  # 配置参数
  INPUT_SIZE = 64
  NUM_LANDMARKS = 20
  BATCH_SIZE = 256
  LEARNING_RATE = 0.0005
  ```

### 类型提示（推荐）
```python
from typing import Tuple, List, Optional
import torch

def calculate_loss(
    predictions: torch.Tensor,
    targets: torch.Tensor,
    weights: Optional[torch.Tensor] = None
) -> Tuple[torch.Tensor, float]:
    """计算加权损失"""
    pass
```

## 虚拟环境管理

**⚠️ 重要：禁止在全局环境中安装包**

在运行代码、测试或安装依赖之前，必须遵循以下规范：

### 1. 检查虚拟环境
运行任何 Python 代码或安装包之前，必须先检查：
- 项目是否有虚拟环境（venv, conda, virtualenv 等）
- 当前是否已激活虚拟环境

**检查方法**：
```bash
# 检查是否在虚拟环境中
which python  # Linux/Mac
where python  # Windows

# 查看当前 Python 路径
python --version
pip --version

# 检查常见虚拟环境目录
ls -la | grep -E "venv|env|.venv|conda"
```

### 2. 虚拟环境优先级
按以下顺序查找和使用虚拟环境：
1. **项目本地虚拟环境**：`venv/`, `env/`, `.venv/`, `.conda/`
2. **Conda 环境**：检查 `environment.yml` 或 `conda-meta/`
3. **Poetry/Pipenv**：检查 `poetry.lock` 或 `Pipfile.lock`
4. **询问用户**：如果找不到虚拟环境，询问用户是否需要创建

### 3. 禁止的操作
**绝对禁止**在以下情况下安装包：
- 未检查虚拟环境状态
- 确认是全局 Python 环境
- 用户未明确授权在全局环境安装

### 4. 安装包流程
```
1. 检查虚拟环境 → 2. 激活虚拟环境 → 3. 询问用户确认 → 4. 安装包
```

**正确示例**：
```bash
# 1. 检查是否有虚拟环境
ls venv/

# 2. 如果存在，提示用户激活
# 助手：检测到项目有 venv 虚拟环境，但当前未激活。
#       请运行以下命令激活：
#       Windows: venv\Scripts\activate
#       Linux/Mac: source venv/bin/activate

# 3. 如果不存在，询问用户是否创建
# 助手：未检测到虚拟环境，是否需要创建？
#       方案1：使用 venv（Python 内置）
#       方案2：使用 conda
#       方案3：使用现有全局环境（不推荐）
```

### 5. 创建虚拟环境建议
如果项目没有虚拟环境，推荐创建：
```bash
# 方案1：Python venv（推荐）
python -m venv venv

# 方案2：Conda
conda create -n project_name python=3.9

# 方案3：Poetry
poetry install
```

### 6. 依赖安装
安装依赖时优先使用项目的依赖文件：
- `requirements.txt` → `pip install -r requirements.txt`
- `environment.yml` → `conda env create -f environment.yml`
- `pyproject.toml` → `poetry install`
- `Pipfile` → `pipenv install`

## 代码修改流程

**⚠️ 重要：禁止未经确认直接修改代码**

在对任何现有代码进行修改之前，必须遵循以下流程：

### 1. 分析现状
- 阅读相关代码文件
- 理解当前实现逻辑
- 识别需要修改的部分

### 2. 提出技术方案
向用户说明：
- **现状描述**：当前代码的实现方式
- **问题分析**：为什么需要修改（如果是修复）
- **技术方案**：提供 2-3 个可行的实现方案，包括：
  - 方案描述（具体如何实现）
  - 优点和缺点
  - 推荐方案及理由
  - 涉及的文件和代码范围
  - 潜在风险和注意事项

### 3. 等待用户确认
- 用户明确选择某个方案后，才开始编写代码
- 如果用户提出修改意见，调整方案后再次确认
- 禁止在用户未明确同意的情况下直接修改代码

### 4. 实施修改
- 严格按照确认的方案执行
- 修改过程中如发现新问题，暂停并再次向用户说明
- 完成后说明修改内容和验证结果

### 例外情况（可以直接修改）
以下情况可以不经过确认流程直接修改：
- **明显的错误修复**：拼写错误、语法错误、明显的 bug
- **用户明确指示**：用户已经给出具体的修改指令（如"把这个变量名改成 xxx"）
- **新建文件**：创建全新的文件（非修改现有代码）
- **文档更新**：修改 README、注释等文档内容
- **格式调整**：代码格式化、缩进调整等不改变逻辑的操作

### 示例对话流程

❌ **错误做法**：
```
用户：优化一下训练代码的性能
助手：[直接修改 train.py，添加混合精度训练]
```

✅ **正确做法**：
```
用户：优化一下训练代码的性能
助手：我分析了 train.py，发现可以从以下几个方面优化性能：

【现状】
- 当前使用 FP32 训练，batch_size=256
- 数据加载使用单进程（workers=0）
- 未使用梯度累积

【优化方案】
方案1：启用混合精度训练（推荐）
- 使用 torch.cuda.amp.autocast 和 GradScaler
- 优点：速度提升约 2-3 倍，显存占用减半
- 缺点：极少数情况下可能影响精度
- 修改文件：train.py（约 15 行新增代码）
- 风险：低，可随时回退

方案2：增加数据加载并行度
- 设置 workers=4，启用 pin_memory
- 优点：减少数据加载等待时间
- 缺点：占用更多 CPU 和内存
- 修改文件：train.py（2 行代码）
- 风险：极低

方案3：两者结合
- 同时启用方案1和方案2
- 优点：性能提升最大
- 缺点：需要更多资源

我推荐先使用方案1（混合精度训练），这是最有效的优化手段。您希望采用哪个方案？
```

## 沟通风格

- 保持专业、清晰、简洁
- 提供完整的上下文
- **修改代码前必须先提出技术方案并获得确认**
- 遇到阻塞时才寻求澄清
