# yxizo-table 快速开始指南

> 10分钟快速上手 yxizo-table

---

## 📦 第一步：项目初始化

### 1.1 创建项目结构

```bash
# 创建项目目录
mkdir yxizo-table && cd yxizo-table

# 初始化 package.json
pnpm init

# 创建 workspace 配置
cat > pnpm-workspace.yaml << EOF
packages:
  - 'packages/*'
EOF

# 创建目录结构
mkdir -p packages/yxizo-table/src/{core,extensions,api,composables,components,utils}
```

### 1.2 安装依赖

```bash
cd packages/yxizo-table

# 安装核心依赖
pnpm add @tanstack/table-core @tanstack/vue-table

# 安装开发依赖
pnpm add -D \
  vue \
  typescript \
  @types/node \
  vite \
  rollup \
  @rollup/plugin-typescript \
  tslib
```

### 1.3 配置 package.json

```json
{
  "name": "@yxizo/table",
  "version": "1.0.0",
  "type": "module",
  "main": "./dist/index.js",
  "module": "./dist/index.js",
  "types": "./dist/index.d.ts",
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.js"
    },
    "./vue": {
      "types": "./dist/vue/index.d.ts",
      "import": "./dist/vue/index.js"
    }
  },
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "type-check": "tsc --noEmit"
  },
  "peerDependencies": {
    "@tanstack/vue-table": "^8.0.0",
    "vue": "^3.0.0"
  }
}
```

### 1.4 配置 TypeScript

```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "lib": ["ES2020", "DOM"],
    "moduleResolution": "bundler",
    "strict": true,
    "declaration": true,
    "declarationMap": true,
    "skipLibCheck": true,
    "esModuleInterop": true,
    "resolveJsonModule": true,
    "jsx": "preserve",
    "outDir": "./dist",
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    }
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

---

## 🛠️ 第二步：实现核心代码（3小时）

### 2.1 初始化层（30分钟）

#### 类型定义

```typescript
// src/core/types.ts
import type { ColumnDef } from '@tanstack/table-core'

export interface YxizoTableGlobalConfig {
  defaultPageSize?: number
  pageSizeOptions?: number[]
  theme?: 'light' | 'dark'
  locale?: 'zh-CN' | 'en-US'
  emptyText?: string
  loadingText?: string
}

export type RendererFunction<T = any> = (props: {
  value: any
  row: T
  column: ColumnDef<T>
}) => any

export type FormatterFunction = (value: any, row?: any) => string | number

export interface EnhancedColumnDef<T = any> extends ColumnDef<T> {
  formatter?: string
  renderer?: string
  width?: number
  minWidth?: number
  maxWidth?: number
}
```

#### 全局配置

```typescript
// src/core/setup.ts
import type { YxizoTableGlobalConfig, RendererFunction, FormatterFunction } from './types'

class YxizoTableConfig {
  private static instance: YxizoTableConfig

  private config: YxizoTableGlobalConfig = {
    defaultPageSize: 20,
    pageSizeOptions: [10, 20, 50, 100],
    theme: 'light',
    locale: 'zh-CN',
    emptyText: '暂无数据',
    loadingText: '加载中...',
  }

  private renderers = new Map<string, RendererFunction>()
  private formatters = new Map<string, FormatterFunction>()

  private constructor() {}

  static getInstance(): YxizoTableConfig {
    if (!YxizoTableConfig.instance) {
      YxizoTableConfig.instance = new YxizoTableConfig()
    }
    return YxizoTableConfig.instance
  }

  setConfig(config: Partial<YxizoTableGlobalConfig>): void {
    this.config = { ...this.config, ...config }
  }

  getConfig(): YxizoTableGlobalConfig {
    return { ...this.config }
  }

  registerRenderer(name: string, renderer: RendererFunction): void {
    this.renderers.set(name, renderer)
  }

  getRenderer(name: string): RendererFunction | undefined {
    return this.renderers.get(name)
  }

  registerFormatter(name: string, formatter: FormatterFunction): void {
    this.formatters.set(name, formatter)
  }

  getFormatter(name: string): FormatterFunction | undefined {
    return this.formatters.get(name)
  }
}

export const globalConfig = YxizoTableConfig.getInstance()

export function setupYxizoTable(config?: Partial<YxizoTableGlobalConfig>) {
  if (config) {
    globalConfig.setConfig(config)
  }
}
```

### 2.2 扩展层（30分钟）

#### 格式化器

```typescript
// src/extensions/formatters/index.ts
import { globalConfig } from '../../core/setup'

export function formatDate(value: any): string {
  if (!value) return ''
  const date = new Date(value)
  if (isNaN(date.getTime())) return String(value)

  const year = date.getFullYear()
  const month = String(date.getMonth() + 1).padStart(2, '0')
  const day = String(date.getDate()).padStart(2, '0')

  return `${year}-${month}-${day}`
}

export function formatDateTime(value: any): string {
  if (!value) return ''
  const dateStr = formatDate(value)
  const date = new Date(value)
  const hours = String(date.getHours()).padStart(2, '0')
  const minutes = String(date.getMinutes()).padStart(2, '0')
  const seconds = String(date.getSeconds()).padStart(2, '0')

  return `${dateStr} ${hours}:${minutes}:${seconds}`
}

// 注册默认格式化器
globalConfig.registerFormatter('formatDate', formatDate)
globalConfig.registerFormatter('formatDateTime', formatDateTime)
```

#### 列增强

```typescript
// src/extensions/columns/column-helper.ts
import type { ColumnDef } from '@tanstack/table-core'
import type { EnhancedColumnDef } from '../../core/types'
import { globalConfig } from '../../core/setup'

export function enhanceColumn<T = any>(
  column: EnhancedColumnDef<T>
): ColumnDef<T> {
  const enhanced: ColumnDef<T> = { ...column }

  // 处理格式化器
  if (column.formatter) {
    const formatter = globalConfig.getFormatter(column.formatter)
    if (formatter) {
      enhanced.cell = (props) => {
        const value = props.getValue()
        return formatter(value, props.row.original)
      }
    }
  }

  // 处理渲染器
  if (column.renderer) {
    const renderer = globalConfig.getRenderer(column.renderer)
    if (renderer) {
      enhanced.cell = (props) => {
        return renderer({
          value: props.getValue(),
          row: props.row.original,
          column: column,
        })
      }
    }
  }

  // 处理列宽
  if (column.width !== undefined) enhanced.size = column.width
  if (column.minWidth !== undefined) enhanced.minSize = column.minWidth
  if (column.maxWidth !== undefined) enhanced.maxSize = column.maxWidth

  return enhanced
}

export function enhanceColumns<T = any>(
  columns: EnhancedColumnDef<T>[]
): ColumnDef<T>[] {
  return columns.map(col => enhanceColumn(col))
}
```

### 2.3 API 层（1小时）

```typescript
// src/api/types.ts
export interface QueryParams {
  page?: number
  pageSize?: number
  sortField?: string
  sortOrder?: 'asc' | 'desc'
  [key: string]: any
}

export interface QueryResponse<T = any> {
  items: T[]
  total: number
  page?: number
  pageSize?: number
}

export interface QueryConfig<T = any> {
  queryFn: (params: QueryParams) => Promise<QueryResponse<T>>
  immediate?: boolean
  initialParams?: QueryParams
  onSuccess?: (data: QueryResponse<T>) => void
  onError?: (error: Error) => void
}
```

```typescript
// src/api/table-api.ts
import type { Table } from '@tanstack/table-core'
import type { QueryConfig, QueryParams, QueryResponse } from './types'

export class YxizoTableApi<T = any> {
  private table: Table<T> | null = null
  private queryConfig: QueryConfig<T> | null = null
  private loading = false
  private latestParams: QueryParams = {}

  mount(table: Table<T>, queryConfig?: QueryConfig<T>): void {
    this.table = table
    this.queryConfig = queryConfig || null

    if (queryConfig?.immediate) {
      this.query(queryConfig.initialParams)
    }
  }

  async query(params?: QueryParams): Promise<void> {
    if (!this.queryConfig?.queryFn) return

    this.loading = true

    try {
      const finalParams: QueryParams = {
        ...this.latestParams,
        ...params,
      }

      if (this.table) {
        const state = this.table.getState()

        if (state.pagination) {
          finalParams.page = state.pagination.pageIndex + 1
          finalParams.pageSize = state.pagination.pageSize
        }

        if (state.sorting && state.sorting.length > 0) {
          const sort = state.sorting[0]
          finalParams.sortField = sort.id
          finalParams.sortOrder = sort.desc ? 'desc' : 'asc'
        }
      }

      this.latestParams = finalParams
      const response = await this.queryConfig.queryFn(finalParams)
      this.queryConfig.onSuccess?.(response)

      return response as any
    } catch (error) {
      this.queryConfig.onError?.(error as Error)
      throw error
    } finally {
      this.loading = false
    }
  }

  async reload(): Promise<void> {
    return this.query()
  }

  getSelectedRows(): T[] {
    if (!this.table) return []
    return this.table.getSelectedRowModel().rows.map(row => row.original)
  }

  unmount(): void {
    this.table = null
    this.queryConfig = null
  }
}
```

### 2.4 组合函数（1小时）

```typescript
// src/composables/vue/useYxizoTable.ts
import { ref, onMounted, onUnmounted } from 'vue'
import type { Ref } from 'vue'
import {
  useVueTable,
  getCoreRowModel,
  getSortedRowModel,
  getPaginationRowModel,
} from '@tanstack/vue-table'
import type { PaginationState, SortingState } from '@tanstack/vue-table'

import { YxizoTableApi } from '../../api/table-api'
import type { QueryConfig } from '../../api/types'
import { enhanceColumns, type EnhancedColumnDef } from '../../extensions/columns/column-helper'
import { globalConfig } from '../../core/setup'

export interface UseYxizoTableOptions<T = any> {
  columns: EnhancedColumnDef<T>[]
  data?: Ref<T[]> | T[]
  queryConfig?: QueryConfig<T>
  enablePagination?: boolean
  enableSorting?: boolean
}

export function useYxizoTable<T = any>(options: UseYxizoTableOptions<T>) {
  const {
    columns: rawColumns,
    data: initialData,
    queryConfig,
    enablePagination = true,
    enableSorting = true,
  } = options

  const columns = enhanceColumns(rawColumns)
  const data = ref<T[]>(
    Array.isArray(initialData) ? initialData : (initialData?.value ?? [])
  ) as Ref<T[]>
  const total = ref(0)
  const loading = ref(false)

  const sorting = ref<SortingState>([])
  const pagination = ref<PaginationState>({
    pageIndex: 0,
    pageSize: globalConfig.getConfig().defaultPageSize ?? 20,
  })

  const api = new YxizoTableApi<T>()

  const enhancedQueryConfig: QueryConfig<T> | undefined = queryConfig ? {
    ...queryConfig,
    onSuccess: (response) => {
      data.value = response.items
      total.value = response.total
      loading.value = false
      queryConfig.onSuccess?.(response)
    },
    onError: (error) => {
      loading.value = false
      queryConfig.onError?.(error)
    },
  } : undefined

  const table = useVueTable({
    get data() {
      return data.value
    },
    columns,
    getCoreRowModel: getCoreRowModel(),
    ...(enableSorting && {
      getSortedRowModel: getSortedRowModel(),
      manualSorting: !!queryConfig,
    }),
    ...(enablePagination && {
      getPaginationRowModel: getPaginationRowModel(),
      manualPagination: !!queryConfig,
    }),
    state: {
      get sorting() {
        return sorting.value
      },
      get pagination() {
        return pagination.value
      },
    },
    onSortingChange: (updater) => {
      sorting.value =
        typeof updater === 'function' ? updater(sorting.value) : updater
      if (queryConfig) api.query()
    },
    onPaginationChange: (updater) => {
      pagination.value =
        typeof updater === 'function' ? updater(pagination.value) : updater
      if (queryConfig) api.query()
    },
  })

  onMounted(() => {
    api.mount(table, enhancedQueryConfig)
  })

  onUnmounted(() => {
    api.unmount()
  })

  return {
    table,
    api,
    loading,
    data,
    total,
  }
}
```

### 2.5 导出入口

```typescript
// src/index.ts
export { setupYxizoTable, globalConfig } from './core/setup'
export type { YxizoTableGlobalConfig } from './core/types'

export { formatDate, formatDateTime } from './extensions/formatters'
export { enhanceColumns } from './extensions/columns/column-helper'
export type { EnhancedColumnDef } from './core/types'

export { YxizoTableApi } from './api/table-api'
export type { QueryParams, QueryResponse, QueryConfig } from './api/types'

export { useYxizoTable } from './composables/vue/useYxizoTable'
export type { UseYxizoTableOptions } from './composables/vue/useYxizoTable'
```

---

## 🎯 第三步：测试验证（30分钟）

### 3.1 创建测试页面

```vue
<!-- examples/basic.vue -->
<script setup lang="ts">
import { ref } from 'vue'
import { useYxizoTable } from '../src/composables/vue/useYxizoTable'
import type { EnhancedColumnDef } from '../src/core/types'

interface User {
  id: number
  name: string
  age: number
  email: string
}

const users = ref<User[]>([
  { id: 1, name: '张三', age: 25, email: 'zhangsan@example.com' },
  { id: 2, name: '李四', age: 30, email: 'lisi@example.com' },
  { id: 3, name: '王五', age: 28, email: 'wangwu@example.com' },
])

const columns: EnhancedColumnDef<User>[] = [
  { accessorKey: 'id', header: 'ID', width: 80 },
  { accessorKey: 'name', header: '姓名', width: 120 },
  { accessorKey: 'age', header: '年龄', width: 100 },
  { accessorKey: 'email', header: '邮箱', width: 200 },
]

const { table } = useYxizoTable({
  columns,
  data: users,
  enablePagination: true,
  enableSorting: true,
})
</script>

<template>
  <div style="padding: 20px;">
    <h1>基础示例</h1>

    <table style="width: 100%; border-collapse: collapse;">
      <thead>
        <tr v-for="headerGroup in table.getHeaderGroups()" :key="headerGroup.id">
          <th
            v-for="header in headerGroup.headers"
            :key="header.id"
            @click="header.column.getToggleSortingHandler()?.($event)"
            style="border: 1px solid #ddd; padding: 8px; cursor: pointer;"
          >
            {{ header.column.columnDef.header }}
            <span v-if="header.column.getIsSorted()">
              {{ header.column.getIsSorted() === 'asc' ? '↑' : '↓' }}
            </span>
          </th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="row in table.getRowModel().rows" :key="row.id">
          <td
            v-for="cell in row.getVisibleCells()"
            :key="cell.id"
            style="border: 1px solid #ddd; padding: 8px;"
          >
            {{ cell.getValue() }}
          </td>
        </tr>
      </tbody>
    </table>

    <div style="margin-top: 20px;">
      <button @click="table.previousPage()" :disabled="!table.getCanPreviousPage()">
        上一页
      </button>
      <span style="margin: 0 10px;">
        第 {{ table.getState().pagination.pageIndex + 1 }} 页
      </span>
      <button @click="table.nextPage()" :disabled="!table.getCanNextPage()">
        下一页
      </button>
    </div>
  </div>
</template>
```

### 3.2 创建开发服务器

```typescript
// vite.config.ts
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [vue()],
  resolve: {
    alias: {
      '@': '/src',
    },
  },
})
```

### 3.3 运行测试

```bash
pnpm dev
```

打开浏览器访问 `http://localhost:5173`，测试：
- ✅ 表格能正常渲染
- ✅ 排序功能正常
- ✅ 分页功能正常

---

## 📦 第四步：构建发布（15分钟）

### 4.1 配置构建脚本

```typescript
// rollup.config.js
import typescript from '@rollup/plugin-typescript'
import { nodeResolve } from '@rollup/plugin-node-resolve'

export default {
  input: 'src/index.ts',
  external: ['vue', '@tanstack/vue-table', '@tanstack/table-core'],
  output: [
    {
      file: 'dist/index.js',
      format: 'es',
    },
  ],
  plugins: [
    nodeResolve(),
    typescript({
      tsconfig: './tsconfig.json',
    }),
  ],
}
```

### 4.2 构建

```bash
pnpm build
```

### 4.3 本地测试

```bash
# 链接到本地
pnpm link --global

# 在测试项目中使用
cd ../test-project
pnpm link --global @yxizo/table
```

---

## 🎉 完成检查清单

- [ ] ✅ 项目结构创建完成
- [ ] ✅ 核心代码实现完成
- [ ] ✅ 测试页面运行正常
- [ ] ✅ 构建输出正确
- [ ] ✅ 本地测试通过

---

## 📚 下一步

### 立即可做

1. **添加更多格式化器**
   ```typescript
   globalConfig.registerFormatter('formatCurrency', formatCurrency)
   globalConfig.registerFormatter('formatPercent', formatPercent)
   ```

2. **添加自定义渲染器**
   ```typescript
   globalConfig.registerRenderer('image', ({ value }) => {
     return h('img', { src: value, style: 'width: 40px;' })
   })
   ```

3. **集成到现有项目**
   - 替换 vxe-table 使用
   - 迁移现有表格页面

### 未来规划

- [ ] 实现虚拟滚动（@tanstack/virtual）
- [ ] 实现列拖拽调整
- [ ] 实现行选择功能
- [ ] 实现导出功能增强
- [ ] 支持 React 框架
- [ ] 完善文档和示例

---

## 💡 常见问题

### Q: 为什么表格不显示？

检查：
1. 是否正确引入 Vue
2. 是否正确传入 columns 和 data
3. 浏览器控制台是否有报错

### Q: 如何自定义样式？

两种方式：
1. 通过 CSS 覆盖默认样式
2. 完全自定义表格组件，使用 `table` 实例的方法

### Q: 如何处理大数据量？

使用虚拟滚动：
```typescript
import { useVirtualizer } from '@tanstack/virtual'

// 集成虚拟滚动逻辑
```

---

## 📞 获取帮助

- 查看完整架构文档：`yxizo-table-architecture.md`
- @tanstack/table 官方文档：https://tanstack.com/table/latest
- 提交 Issue：[项目地址]

---

**恭喜！你已经完成了 yxizo-table 的基础实现！🎉**
