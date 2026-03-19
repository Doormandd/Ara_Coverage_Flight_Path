# AGENTS.md - Ara 航带规划飞行路径开发指南

本指南适用于使用 CesiumJS 的 Ara 航带规划飞行路径应用程序的编程代理。

## 项目概述

基于 CesiumJS 的 Web 应用程序，用于航空测量的飞行路径规划。
- **框架**: 原生 JavaScript 配合 CesiumJS 1.115
- **目的**: 无人机测量的交互式飞行路径生成
- **核心功能**: ROI 绘制、边界矩形生成、Z字形飞行路径、路径裁剪
- **架构**: 模块化设计，分离关注点（app, geometry, sensor, UI 管理）

## 构建和开发命令

```bash
# 本地开发（简单 HTTP 服务器）
python -m http.server 8000
# 或者
npx http-server

# 无需构建过程 - 直接提供文件服务
# 依赖：CesiumJS（从 CDN 加载）
```

## 测试和验证

```bash
# 手动测试 - 在浏览器中加载 planning.html
# 测试用例：
# 1. ROI 多边形绘制和保存
# 2. 最长边识别
# 3. 边界矩形生成
# 4. 不同重叠设置的飞行路径生成
# 5. 路径裁剪算法
# 6. 测区保存/加载功能
```

## 代码风格指南

### 模块结构
- 使用 ES6 类和模块
- 将关注点分离到专门的模块：
  - `app.js` - 主应用程序逻辑
  - `geometry-utils.js` - 几何计算
  - `flight-path.js` - 路径生成算法
  - `sensor-utils.js` - 相机/传感器计算
  - `ui-manager.js` - UI 交互
  - `config.js` - 配置常量

### 命名约定
- 类：PascalCase（`FlightPlanningApp`, `GeometryUtils`）
- 常量：UPPER_SNAKE_CASE（`CONFIG`, `Cesium`）
- 函数：camelCase（`calculatePolygonArea`, `generateBoundingRectangle`）
- 变量：使用描述性的 camelCase（`roiPolygon`, `currentFlightPath`）
- HTML ID：保持一致的 camelCase（`cesiumContainer`, `controlPanel`）

### 导入模式
- 在 HTML 中使用 script 标签加载模块
- 对于这个项目规模，全局命名空间污染是可以接受的
- 常量应在模块级别定义（例如 `CONFIG`）

### 代码组织
- 保持函数专注于单一职责
- 为所有公共方法和函数添加 JSDoc 注释
- 在模块中组合相关函数
- 使用一致的错误处理模式

### 错误处理
- 在实用函数中验证输入参数
- 对初始化和异步操作使用 try-catch
- 通过 UI 状态更新向用户提供有意义的错误消息
- 使用控制台错误进行调试

### UI 集成
- 对所有 DOM 操作和用户反馈使用 `UIManager`
- 状态更新应通过 `updateStatusBar(message, type)`
- 在处理前应验证用户输入
- 为所有用户操作提供视觉反馈

### CesiumJS 最佳实践
- 对地面级实体使用 `HeightReference.CLAMP_TO_GROUND`
- 为标签设置 `disableDepthTestDistance: Number.POSITIVE_INFINITY`
- 在删除/更新时正确清理实体
- 使用带前缀的一致实体命名模式

### 性能考虑
- 尽可能批量处理实体操作
- 使用实体属性进行分组和过滤
- 对频繁计算实施防抖
- 在大场景中限制可见实体数量

### 浏览器兼容性
- 面向现代浏览器（ES6+ 特性）
- 此项目无需构建步骤
- CesiumJS 处理大多数 WebGL 兼容性问题

## 常见任务

```javascript
// 添加新的几何实用函数
const GeometryUtils = {
    // 现有方法...
    
    /**
     * 新的实用函数
     * @param {type} param - 描述
     * @returns {type} 描述
     */
    newFunction(param) {
        // 实现
    }
};

// 添加新的 UI 元素管理
const UIManager = {
    // 现有方法...
    
    /**
     * 更新新元素显示
     */
    updateNewElement(value) {
        const element = document.getElementById('newElement');
        if (element) {
            element.value = value;
        }
    }
};

// 添加新的 Cesium 实体
viewer.entities.add({
    id: 'unique-entity-id',
    polygon: {
        hierarchy: new Cesium.PolygonHierarchy(points),
        material: Cesium.Color.BLUE.withAlpha(0.3),
        outline: true,
        outlineColor: Cesium.Color.BLUE,
        outlineWidth: 2
    }
});
```

## 重要文件和目录

- `planning.html` - 主应用程序页面和 UI
- `app.js` - 核心应用程序逻辑和事件处理
- `config.js` - 应用程序配置常量
- `geometry-utils.js` - 数学/几何计算
- `flight-path.js` - 飞行路径生成算法
- `sensor-utils.js` - 相机和传感器参数计算
- `ui-manager.js` - 用户界面管理

## 测试清单

添加或修改代码时：

1. **几何函数**
   - [ ] 测试边缘情况（共线点、退化多边形）
   - [ ] 验证大/小值的数值稳定性
   - [ ] 检查边界条件

2. **UI 组件**
   - [ ] 验证按钮状态正确启用/禁用
   - [ ] 测试输入验证和错误消息
   - [ ] 检查响应行为

3. **Cesium 集成**
   - [ ] 验证实体正确创建和删除
   - [ ] 测试不同地形配置
   - [ ] 检查实体可见性和交互

4. **工作流集成**
   - [ ] 测试完整的测区创建工作流
   - [ ] 验证会话期间数据持久性
   - [ ] 检查所有 UI 反馈机制

## 调试技巧

- 使用浏览器开发工具进行 JavaScript 调试
- 检查 Cesium 内置调试工具以解决 3D 可视化问题
- 为飞行路径裁剪启用详细日志：将 `CONFIG.tolerance` 设置为更大的值进行调试
- 使用 viewer 实体面板检查创建的实体
- 大多数主要操作已实现控制台日志记录

## 性能指南

- 尽可能限制可见实体数量
- 对频繁创建/删除的实体使用实体池化
- 批量处理 DOM 更新以最小化回流
- 考虑使用 web workers 进行繁重计算（尚未实现）