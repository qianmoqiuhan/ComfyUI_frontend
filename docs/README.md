# ComfyUI Frontend 技术文档索引

欢迎查阅 ComfyUI Frontend 完整技术文档。本文档集合提供了项目架构、开发指南、API 参考等全面信息。

## 📚 文档目录

### 🏗️ 架构设计
- **[前端架构设计文档](./FRONTEND_ARCHITECTURE.md)** - 完整的项目架构分析，包括技术栈、组件设计、数据流等
- **[ADR (架构决策记录)](./adr/)** - 重要架构决策的记录和说明

### 👨‍💻 开发指南
- **[开发指南](./DEVELOPMENT_GUIDE.md)** - 开发环境搭建、代码规范、最佳实践
- **[API 参考文档](./API_REFERENCE.md)** - 详细的 API 接口说明和使用示例
- **[扩展开发](./extensions/)** - 扩展系统开发文档

### ⚙️ 配置和设置
- **[设置系统](./SETTINGS.md)** - 应用设置系统说明
- **[功能开关](./FEATURE_FLAGS.md)** - 功能开关配置指南

## 🎯 快速导航

### 新手入门
如果您是首次接触 ComfyUI Frontend，建议按以下顺序阅读：

1. **[前端架构设计文档](./FRONTEND_ARCHITECTURE.md)** - 了解项目整体架构
2. **[开发指南](./DEVELOPMENT_GUIDE.md)** - 设置开发环境并学习基本概念
3. **[扩展开发文档](./extensions/development.md)** - 如果需要开发扩展

### 开发者参考
对于有经验的开发者，可以直接查阅：

- **[API 参考文档](./API_REFERENCE.md)** - 查找具体 API 接口
- **[组件开发最佳实践](./DEVELOPMENT_GUIDE.md#组件开发最佳实践)** 
- **[性能优化指南](./DEVELOPMENT_GUIDE.md#性能优化实践)**

### 架构师和技术负责人
如果您关注技术决策和架构演进：

- **[架构决策记录](./adr/)** - 了解重要技术决策的背景和影响
- **[前端架构未来发展](./FRONTEND_ARCHITECTURE.md#未来发展)** - 技术发展规划

## 🛠️ 技术栈概览

### 核心技术
- **前端框架**: Vue 3.5 + TypeScript
- **构建工具**: Vite 5.4 + NX 21.4
- **状态管理**: Pinia 2.1
- **UI 组件**: PrimeVue 4.2 + TailwindCSS 3.4
- **图形编辑**: litegraph.js (集成版)

### 开发工具
- **包管理**: PNPM
- **代码规范**: ESLint + Prettier
- **测试**: Vitest + Playwright + Storybook
- **国际化**: vue-i18n

### 云服务
- **认证**: Firebase Auth
- **存储**: Firebase Storage
- **搜索**: Algolia
- **监控**: Sentry

## 📖 核心概念

### 组件化架构
ComfyUI Frontend 采用 Vue 3 Composition API 构建模块化组件系统，支持：
- 原子级、分子级、页面级组件分层
- 可复用的组合式函数 (Composables)
- 类型安全的 props 和 emits 定义

### 状态管理模式
基于 Pinia 的集中式状态管理：
- 模块化 Store 设计
- 响应式数据流
- TypeScript 完整支持

### 扩展系统
灵活的插件化扩展机制：
- 核心扩展和第三方扩展支持
- 丰富的扩展 API
- 命令、设置、UI 组件等全方位扩展点

### 图形编辑器
集成的 litegraph.js 提供强大的节点编辑功能：
- 可视化工作流编辑
- 丰富的节点类型支持
- 实时预览和执行

## 🚀 主要特性

### 用户界面
- **现代化设计**: 基于 PrimeVue 的专业 UI 组件
- **响应式布局**: 支持桌面和移动设备
- **主题系统**: 深色/浅色模式切换
- **国际化**: 支持多种语言

### 开发体验
- **热重载**: Vite 提供快速开发体验
- **类型安全**: 完整的 TypeScript 支持
- **代码规范**: 自动化的代码格式化和检查
- **组件文档**: Storybook 组件开发环境

### 性能优化
- **代码分割**: 按需加载减少初始包大小
- **虚拟滚动**: 大列表性能优化
- **缓存策略**: 智能缓存提升用户体验
- **资源优化**: 图片压缩和 CDN 集成

### 扩展性
- **插件系统**: 支持第三方扩展开发
- **API 友好**: 丰富的开发者 API
- **模块化设计**: 易于维护和扩展
- **版本兼容**: 向后兼容性保障

## 🏃‍♂️ 快速开始

### 环境准备
```bash
# 安装 Node.js 18+ 和 PNPM
npm install -g pnpm

# 克隆项目
git clone https://github.com/Comfy-Org/ComfyUI_frontend.git
cd ComfyUI_frontend

# 安装依赖
pnpm install

# 启动开发服务器
pnpm dev
```

### 项目结构
```
ComfyUI_frontend/
├── src/
│   ├── components/     # Vue 组件
│   ├── stores/         # Pinia 状态管理
│   ├── services/       # 业务逻辑服务
│   ├── composables/    # 组合式函数
│   ├── extensions/     # 扩展系统
│   ├── lib/           # 第三方库 (litegraph)
│   └── views/         # 页面组件
├── docs/              # 项目文档
├── tests-ui/          # 测试文件
└── .storybook/        # Storybook 配置
```

## 🤝 贡献指南

### 如何贡献
1. **代码贡献**: 提交 Pull Request 修复问题或添加功能
2. **文档改进**: 完善文档内容和示例
3. **问题反馈**: 报告 Bug 或提出功能建议
4. **社区支持**: 在 Discord 中帮助其他用户

### 开发流程
1. Fork 项目并创建特性分支
2. 按照代码规范开发新功能
3. 编写相应的测试用例
4. 提交 Pull Request 并描述更改内容

### 代码规范
- 使用 Vue 3 Composition API
- 遵循 TypeScript 严格模式
- 使用 Tailwind CSS 进行样式开发
- 为新功能编写测试用例

## 🆘 获取帮助

### 官方资源
- **官方网站**: [https://www.comfy.org/](https://www.comfy.org/)
- **GitHub 仓库**: [https://github.com/Comfy-Org/ComfyUI_frontend](https://github.com/Comfy-Org/ComfyUI_frontend)
- **Discord 社区**: [https://www.comfy.org/discord](https://www.comfy.org/discord)

### 常见问题
- **安装问题**: 查看[开发指南](./DEVELOPMENT_GUIDE.md#开发环境搭建)
- **扩展开发**: 查看[扩展开发文档](./extensions/development.md)
- **API 使用**: 查看[API 参考文档](./API_REFERENCE.md)

### 问题报告
如果遇到问题，请在 GitHub Issues 中提交，包含：
- 问题描述和重现步骤
- 系统环境信息
- 相关错误日志
- 预期行为说明

## 📋 版本信息

- **当前版本**: v1.26.7
- **发布周期**: 每两周一个小版本
- **LTS 支持**: 主要版本提供长期支持
- **兼容性**: 支持现代浏览器和 Node.js 18+

## 📄 许可证

本项目采用 GPL-3.0 许可证。详情请查看 [LICENSE](../LICENSE) 文件。

## 🙏 致谢

感谢所有为 ComfyUI Frontend 项目贡献代码、文档、测试和反馈的开发者们。特别感谢：

- Vue.js 团队提供的优秀框架
- PrimeVue 团队的 UI 组件库
- litegraph.js 的原始开发者
- 所有社区贡献者的努力

---

*最后更新: 2025年1月*