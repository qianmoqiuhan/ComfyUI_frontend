# ComfyUI Frontend 开发指南

## 开发环境搭建

### 系统要求
- Node.js 18+ (推荐 Node.js 24+ 用于 Vite 开发服务器)
- PNPM 包管理器
- Git 版本控制
- 运行中的 ComfyUI 后端实例

### 快速开始
```bash
# 克隆项目
git clone https://github.com/Comfy-Org/ComfyUI_frontend.git
cd ComfyUI_frontend

# 安装依赖
pnpm install

# 启动开发服务器
pnpm dev
```

### 环境配置
创建 `.env` 文件配置开发环境：
```env
# ComfyUI 后端地址
DEV_SERVER_COMFYUI_URL=http://127.0.0.1:8188

# 启用远程访问 (移动设备调试)
VITE_REMOTE_DEV=false

# 禁用模板代理 (可选)
DISABLE_TEMPLATES_PROXY=false

# Sentry 配置
SENTRY_DSN=your_sentry_dsn

# Algolia 搜索配置
ALGOLIA_APP_ID=your_algolia_app_id
ALGOLIA_API_KEY=your_algolia_api_key
```

## 代码规范与风格

### TypeScript 规范
```typescript
// 使用严格的 TypeScript 配置
interface ComponentProps {
  title: string
  isVisible?: boolean
  onClose?: () => void
}

// 使用 const assertions 确保类型安全
const API_ENDPOINTS = {
  WORKFLOWS: '/api/workflows',
  NODES: '/api/nodes'
} as const

// 使用 Zod 进行运行时类型验证
import { z } from 'zod'

const WorkflowSchema = z.object({
  id: z.string(),
  name: z.string(),
  nodes: z.array(z.object({
    id: z.string(),
    type: z.string()
  }))
})

type Workflow = z.infer<typeof WorkflowSchema>
```

### Vue 3 组件规范
```vue
<template>
  <div class="component-wrapper">
    <!-- 使用语义化的 CSS 类名 -->
    <h2 class="component-title">{{ displayTitle }}</h2>
    
    <!-- 使用 v-show 而不是 v-if 来控制显示/隐藏 -->
    <div v-show="isVisible" class="component-content">
      <slot />
    </div>
  </div>
</template>

<script setup lang="ts">
// 使用 Vue 3.5 风格的 props 声明
interface Props {
  title: string
  isVisible?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  isVisible: true
})

// 定义 emits
const emit = defineEmits<{
  close: []
  update: [value: string]
}>()

// 使用 ref 和 computed
const internalValue = ref('')
const displayTitle = computed(() => 
  props.title.charAt(0).toUpperCase() + props.title.slice(1)
)

// 生命周期钩子
onMounted(() => {
  console.log('Component mounted')
})

// 方法定义
const handleClose = () => {
  emit('close')
}
</script>

<style scoped>
/* 使用 Tailwind CSS 类名 */
.component-wrapper {
  @apply flex flex-col space-y-4 p-4;
}

.component-title {
  @apply text-lg font-semibold text-gray-900 dark:text-gray-100;
}

.component-content {
  @apply rounded-lg border border-gray-200 dark:border-gray-700;
}
</style>
```

### Pinia Store 规范
```typescript
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'

export const useFeatureStore = defineStore('feature', () => {
  // State
  const items = ref<Item[]>([])
  const loading = ref(false)
  const error = ref<string | null>(null)

  // Getters (computed)
  const itemCount = computed(() => items.value.length)
  const hasItems = computed(() => itemCount.value > 0)

  // Actions
  const fetchItems = async () => {
    loading.value = true
    error.value = null
    
    try {
      const data = await api.fetchItems()
      items.value = data
    } catch (err) {
      error.value = err instanceof Error ? err.message : 'Unknown error'
      console.error('Failed to fetch items:', err)
    } finally {
      loading.value = false
    }
  }

  const addItem = (item: Item) => {
    items.value.push(item)
  }

  const removeItem = (id: string) => {
    const index = items.value.findIndex(item => item.id === id)
    if (index > -1) {
      items.value.splice(index, 1)
    }
  }

  return {
    // State
    items: readonly(items),
    loading: readonly(loading),
    error: readonly(error),
    
    // Getters
    itemCount,
    hasItems,
    
    // Actions
    fetchItems,
    addItem,
    removeItem
  }
})
```

## 扩展开发

### 创建扩展
```typescript
// src/extensions/custom/myExtension.ts
import { app } from '@/scripts/app'

app.registerExtension({
  name: 'MyCustomExtension',
  
  // 注册命令
  commands: [
    {
      id: 'my-extension.custom-action',
      label: 'Custom Action',
      icon: 'pi pi-star',
      function: () => {
        console.log('Custom action executed')
      }
    }
  ],
  
  // 注册菜单项
  menuCommands: [
    {
      path: ['Tools', 'Custom'],
      commands: ['my-extension.custom-action']
    }
  ],
  
  // 注册设置项
  settings: [
    {
      id: 'MyExtension.EnableFeature',
      name: 'Enable Custom Feature',
      type: 'boolean',
      defaultValue: true
    }
  ],
  
  // 注册快捷键
  keybindings: [
    {
      combo: { key: 'F', ctrl: true },
      commandId: 'my-extension.custom-action'
    }
  ],
  
  // 注册侧边栏标签
  async init() {
    app.extensionManager.registerSidebarTab({
      id: 'my-custom-tab',
      icon: 'pi pi-cog',
      title: 'Custom Tab',
      tooltip: 'Custom functionality',
      type: 'custom',
      render: (el) => {
        el.innerHTML = '<div class="p-4">Custom tab content</div>'
      }
    })
  }
})
```

### 扩展 API 使用
```typescript
// 使用 Toast 服务
app.extensionManager.toast.add({
  severity: 'success',
  summary: 'Success',
  detail: 'Operation completed successfully',
  life: 3000
})

// 使用对话框服务
const result = await app.extensionManager.dialog.confirm({
  title: 'Confirm Action',
  message: 'Are you sure you want to proceed?'
})

if (result) {
  // 用户确认操作
}

// 使用设置 API
const settingValue = app.extensionManager.setting.get('MyExtension.EnableFeature')
app.extensionManager.setting.set('MyExtension.EnableFeature', false)
```

## 组件开发最佳实践

### 1. 组件组合模式
```vue
<!-- ParentComponent.vue -->
<template>
  <div class="parent-component">
    <ComponentHeader>
      <template #title>
        {{ title }}
      </template>
      <template #actions>
        <Button @click="handleAction">Action</Button>
      </template>
    </ComponentHeader>
    
    <ComponentBody>
      <slot />
    </ComponentBody>
  </div>
</template>

<script setup lang="ts">
// 使用组合式函数
const { title, handleAction } = useComponentLogic()
</script>
```

### 2. 可复用组合式函数
```typescript
// composables/useAsyncData.ts
export function useAsyncData<T>(
  fetcher: () => Promise<T>,
  options: {
    immediate?: boolean
    onError?: (error: Error) => void
  } = {}
) {
  const data = ref<T | null>(null)
  const loading = ref(false)
  const error = ref<Error | null>(null)

  const execute = async () => {
    loading.value = true
    error.value = null
    
    try {
      data.value = await fetcher()
    } catch (err) {
      error.value = err instanceof Error ? err : new Error('Unknown error')
      options.onError?.(error.value)
    } finally {
      loading.value = false
    }
  }

  if (options.immediate !== false) {
    execute()
  }

  return {
    data: readonly(data),
    loading: readonly(loading),
    error: readonly(error),
    execute,
    refresh: execute
  }
}

// 使用示例
const { data: workflows, loading, error, refresh } = useAsyncData(
  () => api.getWorkflows(),
  {
    onError: (error) => {
      toast.add({
        severity: 'error',
        summary: 'Error',
        detail: error.message
      })
    }
  }
)
```

### 3. 状态管理模式
```typescript
// stores/featureStore.ts
export const useFeatureStore = defineStore('feature', () => {
  // 私有状态 (不导出)
  const _internalState = ref(new Map())
  
  // 公共状态
  const items = ref<Item[]>([])
  const selectedId = ref<string | null>(null)
  
  // 计算属性
  const selectedItem = computed(() => 
    items.value.find(item => item.id === selectedId.value) || null
  )
  
  const sortedItems = computed(() => 
    [...items.value].sort((a, b) => a.name.localeCompare(b.name))
  )
  
  // Actions with optimistic updates
  const updateItem = async (id: string, updates: Partial<Item>) => {
    const item = items.value.find(item => item.id === id)
    if (!item) return
    
    // Optimistic update
    const originalItem = { ...item }
    Object.assign(item, updates)
    
    try {
      await api.updateItem(id, updates)
    } catch (error) {
      // Rollback on error
      Object.assign(item, originalItem)
      throw error
    }
  }
  
  return {
    items: readonly(items),
    selectedId,
    selectedItem,
    sortedItems,
    updateItem
  }
})
```

## 测试策略

### 1. 组件测试
```typescript
// tests/components/MyComponent.test.ts
import { mount } from '@vue/test-utils'
import { describe, it, expect, vi } from 'vitest'
import MyComponent from '@/components/MyComponent.vue'

describe('MyComponent', () => {
  it('renders correctly with props', () => {
    const wrapper = mount(MyComponent, {
      props: {
        title: 'Test Title',
        isVisible: true
      }
    })
    
    expect(wrapper.find('.component-title').text()).toBe('Test Title')
    expect(wrapper.find('.component-content').isVisible()).toBe(true)
  })
  
  it('emits close event when close button is clicked', async () => {
    const wrapper = mount(MyComponent, {
      props: { title: 'Test' }
    })
    
    await wrapper.find('.close-button').trigger('click')
    
    expect(wrapper.emitted('close')).toBeTruthy()
    expect(wrapper.emitted('close')).toHaveLength(1)
  })
})
```

### 2. Store 测试
```typescript
// tests/stores/featureStore.test.ts
import { setActivePinia, createPinia } from 'pinia'
import { describe, it, expect, beforeEach, vi } from 'vitest'
import { useFeatureStore } from '@/stores/featureStore'

describe('FeatureStore', () => {
  beforeEach(() => {
    setActivePinia(createPinia())
  })
  
  it('initializes with empty state', () => {
    const store = useFeatureStore()
    
    expect(store.items).toEqual([])
    expect(store.selectedId).toBeNull()
    expect(store.selectedItem).toBeNull()
  })
  
  it('adds item correctly', () => {
    const store = useFeatureStore()
    const newItem = { id: '1', name: 'Test Item' }
    
    store.addItem(newItem)
    
    expect(store.items).toContain(newItem)
    expect(store.itemCount).toBe(1)
  })
})
```

### 3. E2E 测试
```typescript
// tests/e2e/workflow.spec.ts
import { test, expect } from '@playwright/test'

test.describe('Workflow Management', () => {
  test('should create a new workflow', async ({ page }) => {
    await page.goto('/')
    
    // 点击新建工作流按钮
    await page.click('[data-testid="new-workflow-button"]')
    
    // 填写工作流名称
    await page.fill('[data-testid="workflow-name-input"]', 'Test Workflow')
    
    // 保存工作流
    await page.click('[data-testid="save-workflow-button"]')
    
    // 验证工作流创建成功
    await expect(page.locator('[data-testid="workflow-title"]')).toHaveText('Test Workflow')
  })
  
  test('should load existing workflow', async ({ page }) => {
    await page.goto('/')
    
    // 打开工作流列表
    await page.click('[data-testid="workflows-menu"]')
    
    // 选择已存在的工作流
    await page.click('[data-testid="workflow-item"]:first-child')
    
    // 验证工作流加载成功
    await expect(page.locator('[data-testid="graph-canvas"]')).toBeVisible()
  })
})
```

## 性能优化实践

### 1. 组件懒加载
```typescript
// 路由级懒加载
const routes = [
  {
    path: '/settings',
    component: () => import('@/views/SettingsView.vue')
  }
]

// 组件级懒加载
const LazyComponent = defineAsyncComponent(() => import('@/components/HeavyComponent.vue'))
```

### 2. 虚拟滚动
```vue
<template>
  <div class="virtual-list-container">
    <RecycleScroller
      class="scroller"
      :items="items"
      :item-size="60"
      key-field="id"
      v-slot="{ item }"
    >
      <div class="item">
        {{ item.name }}
      </div>
    </RecycleScroller>
  </div>
</template>

<script setup lang="ts">
import { RecycleScroller } from 'vue-virtual-scroller'

const items = ref(generateLargeList())
</script>
```

### 3. 计算属性优化
```typescript
// 使用 computed 缓存复杂计算
const expensiveComputation = computed(() => {
  return items.value
    .filter(item => item.isActive)
    .map(item => processItem(item))
    .sort((a, b) => a.priority - b.priority)
})

// 使用 shallowRef 优化大型对象
const largeData = shallowRef({})

// 使用 markRaw 标记不需要响应式的对象
const staticData = markRaw({
  config: { /* 大型配置对象 */ }
})
```

## 国际化实践

### 1. 翻译文件组织
```
src/locales/
├── en/
│   ├── common.json          # 通用翻译
│   ├── components.json      # 组件翻译
│   ├── errors.json          # 错误信息翻译
│   └── main.json           # 主要功能翻译
├── zh-CN/
│   ├── common.json
│   ├── components.json
│   ├── errors.json
│   └── main.json
└── index.ts                # 国际化配置
```

### 2. 组件中使用翻译
```vue
<template>
  <div>
    <h1>{{ $t('components.title') }}</h1>
    <p>{{ $t('components.description', { name: userName }) }}</p>
    
    <!-- 复数形式 -->
    <span>{{ $tc('components.itemCount', itemCount, { count: itemCount }) }}</span>
  </div>
</template>

<script setup lang="ts">
import { useI18n } from 'vue-i18n'

const { t, tc } = useI18n()

// 在脚本中使用
const message = computed(() => t('components.message'))
</script>
```

### 3. 动态语言切换
```typescript
// stores/localeStore.ts
export const useLocaleStore = defineStore('locale', () => {
  const currentLocale = ref('en')
  const { locale, availableLocales } = useI18n()
  
  const setLocale = async (newLocale: string) => {
    if (availableLocales.includes(newLocale)) {
      locale.value = newLocale
      currentLocale.value = newLocale
      
      // 保存到本地存储
      localStorage.setItem('preferred-locale', newLocale)
      
      // 更新 HTML lang 属性
      document.documentElement.lang = newLocale
    }
  }
  
  return {
    currentLocale: readonly(currentLocale),
    setLocale
  }
})
```

## 调试和问题排查

### 1. Vue Devtools 使用
```typescript
// 开发环境启用 Vue Devtools
if (process.env.NODE_ENV === 'development') {
  app.config.performance = true
}

// 组件调试标记
defineOptions({
  name: 'MyComponent', // 在 devtools 中显示的名称
  inheritAttrs: false
})
```

### 2. 错误边界处理
```vue
<!-- ErrorBoundary.vue -->
<template>
  <div v-if="error" class="error-boundary">
    <h2>Something went wrong</h2>
    <details>
      <summary>Error details</summary>
      <pre>{{ error.stack }}</pre>
    </details>
    <button @click="retry">Retry</button>
  </div>
  <slot v-else />
</template>

<script setup lang="ts">
const error = ref<Error | null>(null)

const retry = () => {
  error.value = null
}

onErrorCaptured((err) => {
  error.value = err
  return false // 阻止错误传播
})
</script>
```

### 3. 性能监控
```typescript
// 性能标记
performance.mark('component-render-start')
// ... 组件渲染逻辑
performance.mark('component-render-end')
performance.measure('component-render', 'component-render-start', 'component-render-end')

// 内存使用监控
const memoryInfo = (performance as any).memory
if (memoryInfo) {
  console.log('Memory usage:', {
    used: memoryInfo.usedJSHeapSize,
    total: memoryInfo.totalJSHeapSize,
    limit: memoryInfo.jsHeapSizeLimit
  })
}
```

## 部署和发布

### 1. 构建优化
```typescript
// vite.config.mts
export default defineConfig({
  build: {
    // 启用压缩
    minify: 'esbuild',
    
    // 生成源码映射
    sourcemap: true,
    
    // 代码分割
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['vue', 'vue-router', 'pinia'],
          ui: ['primevue', '@primevue/themes'],
          utils: ['lodash-es', 'date-fns']
        }
      }
    },
    
    // 资源文件名策略
    assetsDir: 'assets',
    
    // 构建目标
    target: 'es2022'
  }
})
```

### 2. 环境配置
```bash
# 生产环境构建
NODE_ENV=production pnpm build

# 预览构建结果
pnpm preview

# 类型检查
pnpm typecheck
```

### 3. CI/CD 配置
```yaml
# .github/workflows/ci.yml
name: CI

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '20'
      - run: npm install -g pnpm
      - run: pnpm install
      - run: pnpm typecheck
      - run: pnpm lint
      - run: pnpm test:unit
      - run: pnpm build
```

这个开发指南涵盖了 ComfyUI Frontend 开发的各个方面，包括环境搭建、代码规范、组件开发、测试策略、性能优化等，为开发团队提供了完整的开发参考。