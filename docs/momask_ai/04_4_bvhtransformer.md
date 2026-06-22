# 4. BVH转换工具模块（bvhtransformer）
**模块标识符**：bvhtransformer.py  
**编制员**：开发团队  
**修改完成日期**：2025年12月31日

### 4.1 模块功能
实现NPY→BVH格式转换，包含：NPY读取、BVH生成、模板支持、数据映射。

### 4.2 输入输出信息
- **输入**：NPY路径、BVH输出路径、模板路径
- **输出**：生成的BVH动作文件

### 4.3 处理描述
1. 读取NPY文件（numpy.load）
2. 解析模板BVH骨骼结构
3. NPY数据映射到BVH骨骼
4. 写入BVH的HIERARCHY和MOTION部分
5. 保存文件

**数据格式**：
- NPY：(frames, joints, 3)/(frames, joints, 4)数组
- BVH：标准Biovision Hierarchy格式

### 4.4 有关事项
- 依赖NumPy库
- 需要模板BVH文件
- 支持Euler/Quaternion旋转表示
