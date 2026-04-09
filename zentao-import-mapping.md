# 禅道用例导入字段映射

## 禅道模板字段（13列）

| 序号 | 禅道字段 | 必填 | 说明 |
|:----:|---------|:----:|------|
| 1 | 所属产品 | ✅ | 产品名称（自动带入） |
| 2 | 平台/分支 | ❌ | 可留空 |
| 3 | 所属模块 | ❌ | 模块名称或路径，如：/因子管理 |
| 4 | 相关研发需求 | ❌ | 需求ID或名称 |
| 5 | 用例标题 | ✅ | 用例名称，必填 |
| 6 | 前置条件 | ❌ | 执行前条件 |
| 7 | 关键词 | ❌ | 标签 |
| 8 | 优先级 | ❌ | 1/2/3/4 对应 P1/P2/P3/P4 |
| 9 | 用例类型 | ✅ | 功能测试/性能测试等 |
| 10 | 适用阶段 | ❌ | 功能测试阶段等 |
| 11 | 用例状态 | ❌ | 正常/草稿 |
| 12 | 步骤 | ❌ | 测试步骤，用"1. 2. 3."格式 |
| 13 | 预期 | ❌ | 预期结果，用"1. 2. 3."格式 |

---

## 字段映射规则

### 生成的用例 → 禅道导入

| 生成的字段 | 禅道字段 | 映射规则 |
|-----------|---------|---------|
| 用例ID | - | 不导入，禅道自动生成 |
| 用例名称 | 用例标题 | 直接映射，必填 |
| 所属模块 | 所属模块 | 直接映射，如"因子管理" |
| 优先级 | 优先级 | P0→1, P1→2, P2→3, P3→4 |
| 前置条件 | 前置条件 | 直接映射 |
| 输入数据 | - | 不导入，可合并到步骤或忽略 |
| 测试步骤 | 步骤 | 格式化为"1. xxx\n2. xxx" |
| 预期结果 | 预期 | 格式化为"1. xxx\n2. xxx" |
| 备注 | 关键词 | 可选映射 |

### 禅道固定值

| 字段 | 值 | 说明 |
|------|---|------|
| 所属产品 | 环境基础信息管理系统 | 根据实际产品填写 |
| 平台/分支 | 空 | 可不填 |
| 用例类型 | 功能测试 | 默认值 |
| 适用阶段 | 功能测试阶段 | 默认值 |
| 用例状态 | 正常 | 默认值 |

---

## Excel 生成规则

### 文件格式

- **文件名**: `禅道导入_<模块名>_<日期>.xlsx`
- **Sheet名**: 用例
- **编码**: UTF-8

### 列顺序

```
A列: 所属产品
B列: 平台/分支
C列: 所属模块
D列: 相关研发需求
E列: 用例标题
F列: 前置条件
G列: 关键词
H列: 优先级
I列: 用例类型
J列: 适用阶段
K列: 用例状态
L列: 步骤
M列: 预期
```

### 步骤和预期格式

**格式要求**: 每个步骤用"数字."开头，换行分隔

**示例**:
```
步骤:
1. 登录系统
2. 点击因子管理菜单
3. 观察页面加载

预期:
1. 页面标题显示'因子管理'
2. 列表正常加载
```

---

## 优先级转换表

| 测试用例优先级 | 禅道优先级 | 说明 |
|:-------------:|:---------:|------|
| P0 | 1 | 阻塞级，核心流程 |
| P1 | 2 | 高级，重要功能 |
| P2 | 3 | 中级，次要功能 |
| P3 | 4 | 低级，优化建议 |

---

## 代码示例

### Python 生成禅道导入 Excel

```python
from openpyxl import Workbook
from openpyxl.styles import Font, PatternFill

def generate_zentao_excel(cases, product_name="环境基础信息管理系统"):
    """生成禅道导入格式 Excel"""
    wb = Workbook()
    ws = wb.active
    ws.title = "用例"
    
    # 表头
    headers = ["所属产品", "平台/分支", "所属模块", "相关研发需求", 
               "用例标题", "前置条件", "关键词", "优先级", 
               "用例类型", "适用阶段", "用例状态", "步骤", "预期"]
    
    for col, header in enumerate(headers, 1):
        ws.cell(row=1, column=col, value=header)
    
    # 优先级映射
    pri_map = {"P0": "1", "P1": "2", "P2": "3", "P3": "4"}
    
    # 数据行
    for row_idx, case in enumerate(cases, 2):
        ws.cell(row=row_idx, column=1, value=product_name)
        ws.cell(row=row_idx, column=2, value="")
        ws.cell(row=row_idx, column=3, value=case.get("module", ""))
        ws.cell(row=row_idx, column=4, value="")
        ws.cell(row=row_idx, column=5, value=case.get("title", ""))
        ws.cell(row=row_idx, column=6, value=case.get("precondition", ""))
        ws.cell(row=row_idx, column=7, value="")
        ws.cell(row=row_idx, column=8, value=pri_map.get(case.get("pri", "P2"), "3"))
        ws.cell(row=row_idx, column=9, value="功能测试")
        ws.cell(row=row_idx, column=10, value="功能测试阶段")
        ws.cell(row=row_idx, column=11, value="正常")
        
        # 步骤
        steps = case.get("steps", "")
        ws.cell(row=row_idx, column=12, value=steps)
        
        # 预期
        ws.cell(row=row_idx, column=13, value=case.get("expects", ""))
    
    # 保存
    wb.save("禅道导入_测试用例.xlsx")
    return "禅道导入_测试用例.xlsx"
```

---

## 导入注意事项

1. **必填字段**: 用例标题、用例类型不能为空，否则该行会被忽略
2. **步骤格式**: 使用"数字."格式，如"1. 步骤一\n2. 步骤二"
3. **模块路径**: 支持层级路径，如"/因子管理/因子库"
4. **优先级**: 必须是数字 1/2/3/4
5. **编码**: 确保 UTF-8 编码，中文不乱码
6. **行数限制**: 建议每次导入不超过 100 条

---

> 更新时间: 2026-04-01