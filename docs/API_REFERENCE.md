# ComfyUI Frontend API 参考文档

## 概述

ComfyUI Frontend 提供了丰富的 API 系统，支持扩展开发、组件集成和自定义功能实现。本文档详细介绍了所有可用的 API 接口和使用方法。

## 核心 API

### 应用实例 (App)

应用实例是 ComfyUI Frontend 的核心入口点，提供了扩展注册、状态管理和服务访问等功能。

```typescript
// 获取应用实例
declare global {
  interface Window {
    app: ComfyApp
  }
}

const app = window.app
```

#### ComfyApp 接口

```typescript
interface ComfyApp {
  // 扩展管理器
  extensionManager: ExtensionManager
  
  // 注册扩展
  registerExtension(extension: ComfyExtension): void
  
  // 图形相关
  graph: LGraph
  canvas: LGraphCanvas
  
  // UI 组件
  ui: {
    settings: SettingsManager
    dialog: DialogService
    // ... 其他 UI 组件
  }
}
```

## 扩展系统 API

### 扩展注册

```typescript
interface ComfyExtension {
  // 扩展名称 (必须唯一)
  name: string
  
  // 初始化函数
  init?(): Promise<void> | void
  
  // 设置定义
  settings?: SettingDefinition[]
  
  // 命令定义
  commands?: CommandDefinition[]
  
  // 菜单命令
  menuCommands?: MenuCommandDefinition[]
  
  // 快捷键绑定
  keybindings?: KeybindingDefinition[]
  
  // 侧边栏标签页
  sidebarTabs?: SidebarTabDefinition[]
  
  // 底部面板标签页
  bottomPanelTabs?: BottomPanelTabDefinition[]
  
  // 关于页面徽章
  aboutPageBadges?: BadgeDefinition[]
  
  // 选择工具箱命令
  getSelectionToolboxCommands?(selectedItem: any): string[]
  
  // 节点工厂函数
  nodeFactory?: NodeFactoryFunction
  
  // 生命周期钩子
  beforeRegisterNodeDef?(nodeType: string, nodeData: any): void
  afterRegisterNodeDef?(nodeType: string, nodeData: any): void
}
```

### 扩展管理器

```typescript
interface ExtensionManager {
  // 设置管理
  setting: SettingAPI
  
  // 对话框服务
  dialog: DialogService
  
  // 消息提示
  toast: ToastManager
  
  // 注册侧边栏标签页
  registerSidebarTab(tab: SidebarTabDefinition): void
  
  // 注册底部面板标签页
  registerBottomPanelTab(tab: BottomPanelTabDefinition): void
}
```

## 设置 API

### 设置定义

```typescript
interface SettingDefinition {
  // 设置 ID (必须唯一)
  id: string
  
  // 显示名称
  name: string
  
  // 设置类型
  type: 'boolean' | 'number' | 'string' | 'slider' | 'combo' | 'text'
  
  // 默认值
  defaultValue: any
  
  // 描述信息
  tooltip?: string
  
  // 选项 (用于 combo 类型)
  options?: string[] | (() => string[])
  
  // 最小值 (用于 number/slider 类型)
  min?: number
  
  // 最大值 (用于 number/slider 类型)
  max?: number
  
  // 步长 (用于 number/slider 类型)
  step?: number
  
  // 变更回调
  onChange?(value: any, oldValue: any): void
}
```

### 设置 API 接口

```typescript
interface SettingAPI {
  // 获取设置值
  get<T = any>(key: string): T
  
  // 设置值
  set<T = any>(key: string, value: T): void
  
  // 监听设置变更
  addChangeListener(key: string, callback: (value: any, oldValue: any) => void): void
  
  // 移除变更监听器
  removeChangeListener(key: string, callback: Function): void
}
```

### 使用示例

```typescript
// 注册设置
app.registerExtension({
  name: 'MyExtension',
  settings: [
    {
      id: 'MyExtension.EnableFeature',
      name: 'Enable Custom Feature',
      type: 'boolean',
      defaultValue: true,
      tooltip: 'Enable or disable the custom feature'
    },
    {
      id: 'MyExtension.MaxItems',
      name: 'Maximum Items',
      type: 'slider',
      min: 1,
      max: 100,
      step: 1,
      defaultValue: 10
    }
  ]
})

// 使用设置
const isEnabled = app.extensionManager.setting.get('MyExtension.EnableFeature')
app.extensionManager.setting.set('MyExtension.MaxItems', 20)
```

## 命令系统 API

### 命令定义

```typescript
interface CommandDefinition {
  // 命令 ID (必须唯一)
  id: string
  
  // 显示标签
  label: string
  
  // 图标 (PrimeIcons 类名)
  icon?: string
  
  // 执行函数
  function: (...args: any[]) => void | Promise<void>
  
  // 启用条件
  enabled?: () => boolean
  
  // 显示条件
  visible?: () => boolean
  
  // 快捷键提示
  shortcut?: string
}
```

### 菜单命令定义

```typescript
interface MenuCommandDefinition {
  // 菜单路径
  path: string[]
  
  // 命令 ID 列表
  commands: string[]
  
  // 分隔符
  separator?: boolean
}
```

### 使用示例

```typescript
app.registerExtension({
  name: 'MyExtension',
  commands: [
    {
      id: 'my-extension.save-workflow',
      label: 'Save Workflow',
      icon: 'pi pi-save',
      function: async () => {
        await saveCurrentWorkflow()
      },
      enabled: () => hasUnsavedChanges()
    }
  ],
  menuCommands: [
    {
      path: ['File'],
      commands: ['my-extension.save-workflow']
    }
  ]
})
```

## 快捷键 API

### 快捷键定义

```typescript
interface KeybindingDefinition {
  // 快捷键组合
  combo: KeyCombo
  
  // 关联的命令 ID
  commandId: string
  
  // 启用条件
  when?: () => boolean
}

interface KeyCombo {
  // 主键
  key: string
  
  // 修饰键
  ctrl?: boolean
  shift?: boolean
  alt?: boolean
  meta?: boolean
}
```

### 使用示例

```typescript
app.registerExtension({
  name: 'MyExtension',
  keybindings: [
    {
      combo: { key: 'S', ctrl: true },
      commandId: 'my-extension.save-workflow'
    },
    {
      combo: { key: 'F1' },
      commandId: 'my-extension.show-help',
      when: () => app.ui.isGraphView()
    }
  ]
})
```

## 对话框 API

### 对话框服务

```typescript
interface DialogService {
  // 显示确认对话框
  confirm(options: ConfirmDialogOptions): Promise<boolean>
  
  // 显示提示输入对话框
  prompt(options: PromptDialogOptions): Promise<string | null>
  
  // 显示警告对话框
  alert(options: AlertDialogOptions): Promise<void>
  
  // 显示自定义对话框
  show(options: CustomDialogOptions): Promise<any>
}
```

### 对话框选项

```typescript
interface ConfirmDialogOptions {
  title: string
  message: string
  confirmLabel?: string
  cancelLabel?: string
  severity?: 'info' | 'warn' | 'error'
}

interface PromptDialogOptions {
  title: string
  message: string
  defaultValue?: string
  placeholder?: string
  inputType?: 'text' | 'password' | 'number'
}

interface AlertDialogOptions {
  title: string
  message: string
  severity?: 'info' | 'warn' | 'error'
}
```

### 使用示例

```typescript
// 确认对话框
const confirmed = await app.extensionManager.dialog.confirm({
  title: 'Delete Workflow',
  message: 'Are you sure you want to delete this workflow?',
  severity: 'warn'
})

if (confirmed) {
  deleteWorkflow()
}

// 输入对话框
const name = await app.extensionManager.dialog.prompt({
  title: 'Create New Workflow',
  message: 'Enter workflow name:',
  placeholder: 'My Workflow'
})

if (name) {
  createWorkflow(name)
}
```

## 消息提示 API

### Toast 管理器

```typescript
interface ToastManager {
  // 添加消息
  add(message: ToastMessage): void
  
  // 添加成功消息
  addSuccess(summary: string, detail?: string): void
  
  // 添加错误消息
  addError(summary: string, detail?: string): void
  
  // 添加警告消息
  addWarn(summary: string, detail?: string): void
  
  // 添加信息消息
  addInfo(summary: string, detail?: string): void
  
  // 添加警报消息
  addAlert(message: string): void
  
  // 清除所有消息
  clear(): void
}
```

### 消息定义

```typescript
interface ToastMessage {
  // 严重级别
  severity: 'success' | 'info' | 'warn' | 'error'
  
  // 摘要
  summary: string
  
  // 详细信息
  detail?: string
  
  // 显示时长 (毫秒)
  life?: number
  
  // 是否可关闭
  closable?: boolean
  
  // 是否粘性 (不自动消失)
  sticky?: boolean
}
```

### 使用示例

```typescript
// 简单消息
app.extensionManager.toast.addSuccess('Workflow saved successfully')

// 详细消息
app.extensionManager.toast.add({
  severity: 'info',
  summary: 'Processing',
  detail: 'Your request is being processed...',
  life: 5000
})

// 错误消息
app.extensionManager.toast.addError('Failed to load workflow', 'Please check your network connection')
```

## 侧边栏 API

### 侧边栏标签页定义

```typescript
interface SidebarTabDefinition {
  // 标签页 ID (必须唯一)
  id: string
  
  // 图标
  icon: string
  
  // 标题
  title: string
  
  // 工具提示
  tooltip?: string
  
  // 标签页类型
  type: 'vue' | 'custom'
  
  // Vue 组件 (type 为 'vue' 时)
  component?: Component
  
  // 自定义渲染函数 (type 为 'custom' 时)
  render?: (container: HTMLElement) => void
  
  // 显示条件
  visible?: () => boolean
}
```

### 使用示例

```typescript
// Vue 组件标签页
app.extensionManager.registerSidebarTab({
  id: 'my-custom-tab',
  icon: 'pi pi-cog',
  title: 'Settings',
  tooltip: 'Extension settings',
  type: 'vue',
  component: MySettingsComponent
})

// 自定义渲染标签页
app.extensionManager.registerSidebarTab({
  id: 'my-canvas-tab',
  icon: 'pi pi-image',
  title: 'Canvas',
  type: 'custom',
  render: (container) => {
    const canvas = document.createElement('canvas')
    canvas.width = 400
    canvas.height = 300
    container.appendChild(canvas)
    
    // 在 canvas 上绘制内容
    const ctx = canvas.getContext('2d')
    ctx.fillStyle = '#f0f0f0'
    ctx.fillRect(0, 0, canvas.width, canvas.height)
  }
})
```

## 节点 API

### 节点工厂函数

```typescript
type NodeFactoryFunction = (nodeType: string) => LGraphNode | null

// 使用示例
app.registerExtension({
  name: 'MyExtension',
  nodeFactory: (nodeType) => {
    if (nodeType === 'MyCustomNode') {
      return new MyCustomNode()
    }
    return null
  }
})
```

### 节点定义钩子

```typescript
app.registerExtension({
  name: 'MyExtension',
  beforeRegisterNodeDef: (nodeType, nodeData) => {
    // 在注册节点定义前修改节点数据
    if (nodeType === 'TargetNode') {
      nodeData.color = '#ff6b6b'
    }
  },
  afterRegisterNodeDef: (nodeType, nodeData) => {
    // 在注册节点定义后执行额外逻辑
    console.log(`Registered node: ${nodeType}`)
  }
})
```

## 工作流 API

### 工作流管理

```typescript
interface WorkflowAPI {
  // 获取当前工作流
  getCurrentWorkflow(): Workflow | null
  
  // 加载工作流
  loadWorkflow(workflow: Workflow): Promise<void>
  
  // 保存工作流
  saveWorkflow(name?: string): Promise<void>
  
  // 创建新工作流
  newWorkflow(): void
  
  // 导出工作流
  exportWorkflow(): string
  
  // 导入工作流
  importWorkflow(data: string): Promise<void>
}
```

### 图形 API

```typescript
interface GraphAPI {
  // 获取当前图形
  getGraph(): LGraph
  
  // 添加节点
  addNode(nodeType: string, position?: [number, number]): LGraphNode
  
  // 删除节点
  removeNode(node: LGraphNode): void
  
  // 连接节点
  connectNodes(outputNode: LGraphNode, outputSlot: number, inputNode: LGraphNode, inputSlot: number): void
  
  // 获取选中的节点
  getSelectedNodes(): LGraphNode[]
  
  // 清空图形
  clear(): void
}
```

## 主题和样式 API

### 主题管理

```typescript
interface ThemeAPI {
  // 获取当前主题
  getCurrentTheme(): 'light' | 'dark'
  
  // 设置主题
  setTheme(theme: 'light' | 'dark'): void
  
  // 切换主题
  toggleTheme(): void
  
  // 获取主题颜色
  getThemeColor(colorName: string): string
  
  // 监听主题变更
  onThemeChange(callback: (theme: string) => void): void
}
```

### 颜色调色板

```typescript
interface ColorPaletteAPI {
  // 获取节点颜色
  getNodeColor(nodeType: string): string
  
  // 设置节点颜色
  setNodeColor(nodeType: string, color: string): void
  
  // 获取预定义颜色
  getPredefinedColors(): string[]
  
  // 生成随机颜色
  generateRandomColor(): string
}
```

## 事件系统 API

### 事件发布订阅

```typescript
interface EventBus {
  // 订阅事件
  on(event: string, callback: Function): void
  
  // 取消订阅
  off(event: string, callback: Function): void
  
  // 发布事件
  emit(event: string, ...args: any[]): void
  
  // 一次性订阅
  once(event: string, callback: Function): void
}

// 使用示例
app.eventBus.on('workflow:loaded', (workflow) => {
  console.log('Workflow loaded:', workflow.name)
})

app.eventBus.emit('custom:event', { data: 'example' })
```

### 内置事件

```typescript
// 工作流事件
'workflow:loaded'        // 工作流加载完成
'workflow:saved'         // 工作流保存完成
'workflow:changed'       // 工作流内容变更

// 节点事件
'node:created'           // 节点创建
'node:removed'           // 节点删除
'node:selected'          // 节点选中
'node:executed'          // 节点执行完成

// 图形事件
'graph:changed'          // 图形变更
'graph:cleared'          // 图形清空

// UI 事件
'ui:theme-changed'       // 主题变更
'ui:sidebar-toggled'     // 侧边栏切换
```

## 存储 API

### 本地存储

```typescript
interface StorageAPI {
  // 获取数据
  get<T = any>(key: string): T | null
  
  // 设置数据
  set<T = any>(key: string, value: T): void
  
  // 删除数据
  remove(key: string): void
  
  // 清空存储
  clear(): void
  
  // 获取所有键
  keys(): string[]
}

// 使用示例
app.storage.set('user:preferences', { theme: 'dark', language: 'en' })
const prefs = app.storage.get('user:preferences')
```

### 云端存储 (Firebase)

```typescript
interface CloudStorageAPI {
  // 上传文件
  uploadFile(file: File, path: string): Promise<string>
  
  // 下载文件
  downloadFile(path: string): Promise<Blob>
  
  // 删除文件
  deleteFile(path: string): Promise<void>
  
  // 列出文件
  listFiles(path: string): Promise<string[]>
}
```

## 实用工具 API

### 通用工具

```typescript
interface UtilsAPI {
  // 生成唯一 ID
  generateId(): string
  
  // 深度克隆
  deepClone<T>(obj: T): T
  
  // 防抖函数
  debounce<T extends (...args: any[]) => any>(func: T, delay: number): T
  
  // 节流函数
  throttle<T extends (...args: any[]) => any>(func: T, delay: number): T
  
  // 格式化文件大小
  formatFileSize(bytes: number): string
  
  // 格式化日期
  formatDate(date: Date, format: string): string
  
  // 验证邮箱
  isValidEmail(email: string): boolean
  
  // 下载文件
  downloadFile(content: string, filename: string, mimeType?: string): void
}
```

### 数学工具

```typescript
interface MathUtils {
  // 限制数值范围
  clamp(value: number, min: number, max: number): number
  
  // 线性插值
  lerp(start: number, end: number, factor: number): number
  
  // 角度转弧度
  degToRad(degrees: number): number
  
  // 弧度转角度
  radToDeg(radians: number): number
  
  // 随机数
  random(min?: number, max?: number): number
}
```

## 错误处理

### 错误监控

```typescript
interface ErrorHandler {
  // 捕获错误
  captureError(error: Error, context?: any): void
  
  // 捕获消息
  captureMessage(message: string, level?: 'info' | 'warning' | 'error'): void
  
  // 设置用户上下文
  setUserContext(user: { id: string; email?: string }): void
  
  // 设置标签
  setTag(key: string, value: string): void
  
  // 添加面包屑
  addBreadcrumb(message: string, category?: string): void
}

// 使用示例
try {
  riskyOperation()
} catch (error) {
  app.errorHandler.captureError(error, { operation: 'workflow-save' })
}
```

## API 版本兼容性

### 版本检查

```typescript
interface VersionAPI {
  // 获取前端版本
  getFrontendVersion(): string
  
  // 获取后端版本
  getBackendVersion(): Promise<string>
  
  // 检查版本兼容性
  checkCompatibility(): Promise<boolean>
  
  // 获取 API 版本
  getAPIVersion(): string
}
```

### 功能检测

```typescript
interface FeatureDetection {
  // 检查功能是否可用
  hasFeature(feature: string): boolean
  
  // 获取所有可用功能
  getAvailableFeatures(): string[]
  
  // 检查扩展是否已加载
  hasExtension(extensionName: string): boolean
}
```

## 总结

ComfyUI Frontend 提供了丰富而强大的 API 系统，支持各种扩展开发需求。通过这些 API，开发者可以：

1. **创建自定义扩展** - 使用扩展系统 API 添加新功能
2. **集成第三方服务** - 通过存储和网络 API 连接外部服务
3. **自定义用户界面** - 使用组件和主题 API 定制界面
4. **处理用户交互** - 通过命令和事件系统处理用户操作
5. **管理应用状态** - 使用状态管理 API 维护应用数据

所有 API 都经过精心设计，确保类型安全、性能优化和向后兼容性。开发者在使用这些 API 时，建议遵循文档中的最佳实践和使用示例。