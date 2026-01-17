---
name: contract-cr
description: 签约项目代码审查规范。当审查包含 contractReq.getPromiseInfo()、Optional 使用、空指针风险等签约业务相关的 Java 代码时，优先使用此规范进行专项检查
---

# 签约项目代码审查规范

## 核心检查项

### 空指针安全检查
- ❌ **禁止直接调用可能返回 null 的方法**：`contractReq.getPromiseInfo()` 必须使用 `Optional` 包装

## Code Review 输出模板

审查完成后，请按以下格式输出：

---

### 📋 审查概览
- **审查文件**: [文件路径]
- **代码行数**: [行数]
- **审查时间**: [时间]
- **严重程度**: 🔴高危 / 🟡中危 / 🟢低危

---

### ✅ 通过的检查项（符合规范）
1. ✅ `getPromiseInfo()` 调用已使用 `Optional` 包装
2. ✅ 其他检查项...

---

### ⚠️ 需要改进的地方（中等优先级）

#### 1. 建议优化 Optional 使用方式
- **位置**: `src/service/ContractService.java:45`
- **问题**: 虽然使用了 `Optional`，但可以使用更简洁的链式调用
- **建议**:
  ```java
  // 当前写法
  Optional<PromiseInfo> optInfo = Optional.ofNullable(contractReq.getPromiseInfo());
  if (optInfo.isPresent()) {
      PromiseInfo info = optInfo.get();
      processAmount(info.getAmount());
  }

  // 建议写法
  Optional.ofNullable(contractReq.getPromiseInfo())
      .ifPresent(info -> processAmount(info.getAmount()));
  ```

---

### ❌ 严重问题（必须修复）

#### 🔴 1. 未使用 Optional 包装可能为 null 的数据
- **位置**: `src/service/ContractService.java:89`
- **问题**: 直接调用 `contractReq.getPromiseInfo()` 未使用 `Optional` 包装
- **风险**: 可能导致 NullPointerException，影响系统稳定性
- **必须修复**:
  ```java
  // ❌ 错误写法
  PromiseInfo info = contractReq.getPromiseInfo();
  info.getAmount(); // 可能抛出 NPE

  // ✅ 正确写法 1：使用 ifPresent
  Optional.ofNullable(contractReq.getPromiseInfo())
      .ifPresent(info -> {
          // 安全地使用 info
          processAmount(info.getAmount());
      });

  // ✅ 正确写法 2：使用 map 链式调用
  BigDecimal amount = Optional.ofNullable(contractReq.getPromiseInfo())
      .map(PromiseInfo::getAmount)
      .orElse(BigDecimal.ZERO);

  // ✅ 正确写法 3：使用 orElseThrow
  PromiseInfo info = Optional.ofNullable(contractReq.getPromiseInfo())
      .orElseThrow(() -> new BusinessException("承诺信息不能为空"));
  ```

---

### 📊 代码质量评分

| 维度 | 评分 | 说明 |
|------|------|------|
| 空指针安全 | [X]/10 | 根据 Optional 使用情况评分 |

**总体评分**: [X]/10

---

### 💡 改进建议

#### 短期（1-2 天内必须完成）
1. 🔴 修复所有直接调用 `getPromiseInfo()` 的代码，使用 `Optional` 包装

#### 中期（1 周内完成）
1. 统一 `Optional` 的使用风格，优先使用链式调用
2. 在团队内推广 `Optional` 最佳实践

#### 长期（持续优化）
1. 在代码规范文档中明确 `Optional` 使用规范
2. 配置静态代码检查工具（如 SpotBugs）自动检测空指针风险

---

### 🎯 关键提醒
- ⚡ 本次审查专注于 `contractReq.getPromiseInfo()` 的空指针安全检查
- 📌 所有直接调用该方法的地方都必须使用 `Optional` 包装
- 🔒 建议在方法定义处添加 `@Nullable` 注解，明确告知调用方可能返回 null

---
