# yxizo-table 架构设计文档

> 基于 @tanstack/table 的企业级表格组件封装方案

**文档版本**: v1.0.0
**最后更新**: 2026-01-09
**作者**: Architecture Team

---

## 📋 目录

- [一、项目概述](#一项目概述)
- [二、架构设计](#二架构设计)
- [三、技术选型](#三技术选型)
- [四、核心实现](#四核心实现)
- [五、使用指南](#五使用指南)
- [六、实施路线](#六实施路线)
- [七、最佳实践](#七最佳实践)
- [八、FAQ](#八faq)

---

## 一、项目概述

### 1.1 背景

在企业级应用开发中，表格组件是最常用的数据展示组件之一。现有方案存在以下问题：

- **完整组件库**（如 vxe-table）：功能完善但包体积大、UI 定制受限
- **从零实现**：开发周期长（6-12个月）、维护成本高
- **原生 @tanstack/table**：功能强大但 API 较底层，缺少业务层封装

### 1.2 目标

基于 @tanstack/table 构建一个：

- ✅ **框架无关**：支持 Vue/React/Solid 等多框架
- ✅ **业务友好**：封装常见业务场景（搜索+表格+分页）
- ✅ **高度可定制**：完全控制 UI 和样式
- ✅ **轻量级**：核心包 < 30KB gzipped
- ✅ **类型安全**：完整的 TypeScript 支持

### 1.3 核心优势

| 对比维度 | 从零实现 | 使用 vxe-table | 使用 yxizo-table |
|---------|---------|---------------|-----------------|
| 开发周期 | 6-12个月 | 1-2周 | 1-2周 |
| 包体积 | 自定义 | 200KB+ | 30KB |
| UI 自由度 | ⭐⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐⭐ |
| 框架支持 | 自己实现 | Vue | Vue/React/Solid |
| 维护成本 | 高 | 低 | 低 |
| 学习曲线 | 陡峭 | 中等 | 平缓 |

---

## 二、架构设计

### 2.1 整体架构图

```
┌─────────────────────────────────────────────────────────────┐
│                       业务代码层                              │
│         const [Table, api] = useYxizoTable(options)         │
└────────────────────────┬────────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────────┐
│                    组件封装层 (Component Layer)               │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  useYxizoTable.ts/tsx                                │  │
│  │  • 组合 @tanstack/table                              │  │
│  │  • 表单集成                                           │  │
│  │  • 工具栏构建                                         │  │
│  │  • 插槽系统                                           │  │
│  └──────────────────────────────────────────────────────┘  │
└────────────────────────┬────────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────────┐
│                     API 层 (API Layer)                       │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  YxizoTableApi.ts                                    │  │
│  │  • 统一的 API 接口                                    │  │
│  │  • 状态管理增强                                       │  │
│  │  • 方法封装（query/reload/export）                    │  │
│  │  • 生命周期管理                                       │  │
│  └──────────────────────────────────────────────────────┘  │
└────────────────────────┬────────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────────┐
│                    扩展层 (Extension Layer)                  │
│  ┌───────────┬───────────┬───────────┬───────────────┐    │
│  │ Renderer  │ Formatter │ Column    │ Row Model     │    │
│  │ 渲染器    │ 格式化器  │ 列增强    │ 行模型扩展    │    │
│  └───────────┴───────────┴───────────┴───────────────┘    │
└────────────────────────┬────────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────────┐
│                   初始化层 (Init Layer)                       │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  setup.ts                                            │  │
│  │  • 全局配置                                           │  │
│  │  • 主题配置                                           │  │
│  │  • 国际化                                             │  │
│  │  • 插件注册                                           │  │
│  └──────────────────────────────────────────────────────┘  │
└────────────────────────┬────────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────────┐
│                   @tanstack/table Core                       │
│       useReactTable | useVueTable | useSolidTable            │
└─────────────────────────────────────────────────────────────┘
```

### 2.2 分层职责

#### 初始化层（Init Layer）

**职责**：
- 全局配置管理（分页大小、主题、语言）
- 渲染器和格式化器注册
- 插件系统初始化

**关键文件**：
- `setup.ts` - 初始化入口
- `config.ts` - 配置管理
- `types.ts` - 类型定义

#### 扩展层（Extension Layer）

**职责**：
- 提供常用格式化器（日期、货币、百分比）
- 提供常用渲染器（图片、链接、标签）
- 列定义增强（支持自定义属性）

**关键文件**：
- `formatters/index.ts` - 格式化器
- `renderers/index.ts` - 渲染器
- `column-helper.ts` - 列增强工具

#### API 层（API Layer）

**职责**：
- 封装表格操作方法（query、reload、reset）
- 管理查询状态和参数
- 提供工具方法（导出、选择）

**关键文件**：
- `table-api.ts` - API 类
- `query-adapter.ts` - 查询适配器
- `types.ts` - API 类型定义

#### 组件层（Component Layer）

**职责**：
- 提供 Vue/React 组件封装
- 实现插槽系统
- 处理样式和主题

**关键文件**：
- `useYxizoTable.ts` - 组合函数
- `YxizoTable.vue` - Vue 组件
- `YxizoTable.tsx` - React 组件

### 2.3 数据流向

```
用户交互（点击排序、翻页）
    ↓
事件处理（onSortingChange, onPaginationChange）
    ↓
更新内部状态（sorting, pagination）
    ↓
触发查询（api.query()）
    ↓
调用 queryFn（用户提供的查询函数）
    ↓
更新数据（data.value = response.items）
    ↓
触发重新渲染
```

---

## 三、技术选型

### 3.1 核心依赖

| 依赖 | 版本 | 用途 |
|-----|------|------|
| @tanstack/table-core | ^8.0.0 | 框架无关核心 |
| @tanstack/vue-table | ^8.0.0 | Vue 适配器 |
| @tanstack/react-table | ^8.0.0 | React 适配器 |
| TypeScript | ^5.0.0 | 类型系统 |

### 3.2 为什么选择 @tanstack/table？

#### ✅ 优势

1. **Headless 设计**：只提供逻辑，完全控制 UI
2. **框架无关**：官方支持 React/Vue/Solid/Svelte
3. **类型安全**：原生 TypeScript，类型推断完善
4. **性能优秀**：虚拟滚动、增量更新
5. **生态成熟**：50k+ stars，活跃维护
6. **包体积小**：核心仅 20KB gzipped

#### ⚠️ 注意事项

1. **API 较底层**：需要封装才适合业务使用
2. **无内置 UI**：需要自己实现样式
3. **学习曲线**：需要理解 Row Model 概念

### 3.3 与 vxe-table 对比

| 特性 | vxe-table | @tanstack/table |
|------|-----------|-----------------|
| 定位 | 完整表格组件 | Headless 核心 |
| 包体积 | 200KB+ | 20KB |
| UI 自由度 | 受限 | 完全自由 |
| 框架支持 | Vue | Vue/React/Solid/Svelte |
| 学习成本 | 中 | 中 |
| 适用场景 | 快速开发 | 高度定制 |

---

## 四、核心实现

### 4.1 项目结构

```
packages/yxizo-table/
├── src/
│   ├── core/                          # 核心层
│   │   ├── setup.ts                   # 全局初始化
│   │   ├── config.ts                  # 默认配置
│   │   └── types.ts                   # 类型定义
│   │
│   ├── extensions/                    # 扩展层
│   │   ├── renderers/                 # 渲染器
│   │   │   ├── index.ts
│   │   │   ├── image-renderer.tsx
│   │   │   └── link-renderer.tsx
│   │   ├── formatters/                # 格式化器
│   │   │   ├── index.ts
│   │   │   ├── date-formatter.ts
│   │   │   └── currency-formatter.ts
│   │   ├── columns/                   # 列增强
│   │   │   ├── index.ts
│   │   │   └── column-helper.ts
│   │   └── index.ts
│   │
│   ├── api/                           # API 层
│   │   ├── table-api.ts               # 表格 API 类
│   │   ├── query-adapter.ts           # 查询适配器
│   │   └── types.ts
│   │
│   ├── composables/                   # 组合函数
│   │   ├── vue/
│   │   │   ├── useYxizoTable.ts       # Vue 主入口
│   │   │   └── useTableForm.ts        # 表单集成
│   │   └── react/
│   │       └── useYxizoTable.tsx      # React 主入口
│   │
│   ├── components/                    # 组件层
│   │   ├── vue/
│   │   │   ├── YxizoTable.vue         # Vue 表格组件
│   │   │   └── YxizoTableToolbar.vue  # 工具栏组件
│   │   └── react/
│   │       └── YxizoTable.tsx
│   │
│   ├── utils/                         # 工具函数
│   │   ├── merge.ts
│   │   ├── format.ts
│   │   └── export.ts
│   │
│   └── index.ts                       # 导出入口
│
├── package.json
├── tsconfig.json
├── vite.config.ts
└── README.md
```

### 4.2 核心代码实现

#### 4.2.1 初始化层

```typescript
// packages/yxizo-table/src/core/types.ts

import type { ColumnDef } from '@tanstack/table-core'

/**
 * 全局配置接口
 */
export interface YxizoTableGlobalConfig {
  /** 默认分页大小 */
  defaultPageSize?: number
  /** 可选分页大小列表 */
  pageSizeOptions?: number[]
  /** 默认主题 */
  theme?: 'light' | 'dark'
  /** 国际化语言 */
  locale?: 'zh-CN' | 'en-US'
  /** 空状态文本 */
  emptyText?: string
  /** 加载文本 */
  loadingText?: string
}

/**
 * 渲染器函数类型
 */
export type RendererFunction<T = any> = (props: {
  value: any
  row: T
  column: ColumnDef<T>
}) => any

/**
 * 格式化器函数类型
 */
export type FormatterFunction = (value: any, row?: any) => string | number
```

```typescript
// packages/yxizo-table/src/core/setup.ts

import type { YxizoTableGlobalConfig, RendererFunction, FormatterFunction } from './types'

/**
 * 全局配置管理类（单例模式）
 */
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

  /**
   * 设置全局配置
   */
  setConfig(config: Partial<YxizoTableGlobalConfig>): void {
    this.config = { ...this.config, ...config }
  }

  /**
   * 获取全局配置
   */
  getConfig(): YxizoTableGlobalConfig {
    return { ...this.config }
  }

  /**
   * 注册渲染器
   */
  registerRenderer(name: string, renderer: RendererFunction): void {
    if (this.renderers.has(name)) {
      console.warn(`[YxizoTable] Renderer "${name}" already exists`)
    }
    this.renderers.set(name, renderer)
  }

  /**
   * 获取渲染器
   */
  getRenderer(name: string): RendererFunction | undefined {
    return this.renderers.get(name)
  }

  /**
   * 注册格式化器
   */
  registerFormatter(name: string, formatter: FormatterFunction): void {
    if (this.formatters.has(name)) {
      console.warn(`[YxizoTable] Formatter "${name}" already exists`)
    }
    this.formatters.set(name, formatter)
  }

  /**
   * 获取格式化器
   */
  getFormatter(name: string): FormatterFunction | undefined {
    return this.formatters.get(name)
  }
}

export const globalConfig = YxizoTableConfig.getInstance()

/**
 * 初始化 YxizoTable
 *
 * @example
 * ```ts
 * setupYxizoTable({
 *   defaultPageSize: 20,
 *   locale: 'zh-CN'
 * })
 * ```
 */
export function setupYxizoTable(config?: Partial<YxizoTableGlobalConfig>) {
  if (config) {
    globalConfig.setConfig(config)
  }

  // 注册默认扩展
  registerDefaultExtensions()
}

/**
 * 注册默认扩展
 */
function registerDefaultExtensions() {
  // 默认格式化器在 extensions/formatters 中注册
}
```

#### 4.2.2 扩展层

```typescript
// packages/yxizo-table/src/extensions/formatters/index.ts

import { globalConfig } from '../../core/setup'

/**
 * 日期格式化 YYYY-MM-DD
 */
export function formatDate(value: any): string {
  if (!value) return ''
  const date = new Date(value)
  if (isNaN(date.getTime())) return String(value)

  const year = date.getFullYear()
  const month = String(date.getMonth() + 1).padStart(2, '0')
  const day = String(date.getDate()).padStart(2, '0')

  return `${year}-${month}-${day}`
}

/**
 * 日期时间格式化 YYYY-MM-DD HH:mm:ss
 */
export function formatDateTime(value: any): string {
  if (!value) return ''
  const date = new Date(value)
  if (isNaN(date.getTime())) return String(value)

  const dateStr = formatDate(value)
  const hours = String(date.getHours()).padStart(2, '0')
  const minutes = String(date.getMinutes()).padStart(2, '0')
  const seconds = String(date.getSeconds()).padStart(2, '0')

  return `${dateStr} ${hours}:${minutes}:${seconds}`
}

/**
 * 货币格式化 ¥1,234.56
 */
export function formatCurrency(value: any): string {
  if (value === null || value === undefined) return ''
  const num = Number(value)
  if (isNaN(num)) return String(value)

  return `¥${num.toFixed(2).replace(/\B(?=(\d{3})+(?!\d))/g, ',')}`
}

/**
 * 百分比格式化 12.34%
 */
export function formatPercent(value: any, decimals: number = 2): string {
  if (value === null || value === undefined) return ''
  const num = Number(value)
  if (isNaN(num)) return String(value)

  return `${(num * 100).toFixed(decimals)}%`
}

// 自动注册默认格式化器
globalConfig.registerFormatter('formatDate', formatDate)
globalConfig.registerFormatter('formatDateTime', formatDateTime)
globalConfig.registerFormatter('formatCurrency', formatCurrency)
globalConfig.registerFormatter('formatPercent', formatPercent)
```

```typescript
// packages/yxizo-table/src/extensions/columns/column-helper.ts

import type { ColumnDef } from '@tanstack/table-core'
import { globalConfig } from '../../core/setup'

/**
 * 增强列定义接口
 */
export interface EnhancedColumnDef<T = any> extends ColumnDef<T> {
  /** 格式化器名称 */
  formatter?: string
  /** 渲染器名称 */
  renderer?: string
  /** 是否显示溢出提示 */
  showOverflowTooltip?: boolean
  /** 列宽 */
  width?: number
  /** 最小列宽 */
  minWidth?: number
  /** 最大列宽 */
  maxWidth?: number
}

/**
 * 增强列定义
 * 将自定义配置转换为 @tanstack/table 的标准配置
 */
export function enhanceColumn<T = any>(
  column: EnhancedColumnDef<T>
): ColumnDef<T> {
  const enhanced: ColumnDef<T> = { ...column }

  // 处理格式化器
  if (column.formatter) {
    const formatter = globalConfig.getFormatter(column.formatter)
    if (formatter) {
      const originalCell = column.cell
      enhanced.cell = (props) => {
        const value = props.getValue()
        const formatted = formatter(value, props.row.original)
        if (originalCell && typeof originalCell === 'function') {
          return originalCell({ ...props, getValue: () => formatted })
        }
        return formatted
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
  if (column.width !== undefined) {
    enhanced.size = column.width
  }
  if (column.minWidth !== undefined) {
    enhanced.minSize = column.minWidth
  }
  if (column.maxWidth !== undefined) {
    enhanced.maxSize = column.maxWidth
  }

  return enhanced
}

/**
 * 批量增强列定义
 */
export function enhanceColumns<T = any>(
  columns: EnhancedColumnDef<T>[]
): ColumnDef<T>[] {
  return columns.map(col => enhanceColumn(col))
}
```

#### 4.2.3 API 层

```typescript
// packages/yxizo-table/src/api/types.ts

/**
 * 查询参数接口
 */
export interface QueryParams {
  /** 当前页 */
  page?: number
  /** 每页数量 */
  pageSize?: number
  /** 排序字段 */
  sortField?: string
  /** 排序方向 */
  sortOrder?: 'asc' | 'desc'
  /** 其他查询参数 */
  [key: string]: any
}

/**
 * 查询响应接口
 */
export interface QueryResponse<T = any> {
  /** 数据列表 */
  items: T[]
  /** 总数 */
  total: number
  /** 当前页 */
  page?: number
  /** 每页数量 */
  pageSize?: number
}

/**
 * 查询配置接口
 */
export interface QueryConfig<T = any> {
  /** 查询函数 */
  queryFn: (params: QueryParams) => Promise<QueryResponse<T>>
  /** 是否自动加载 */
  immediate?: boolean
  /** 初始查询参数 */
  initialParams?: QueryParams
  /** 成功回调 */
  onSuccess?: (data: QueryResponse<T>) => void
  /** 错误回调 */
  onError?: (error: Error) => void
}
```

```typescript
// packages/yxizo-table/src/api/table-api.ts

import type { Table } from '@tanstack/table-core'
import type { QueryConfig, QueryParams, QueryResponse } from './types'

/**
 * 表格 API 类
 * 封装常用的表格操作方法
 */
export class YxizoTableApi<T = any> {
  private table: Table<T> | null = null
  private queryConfig: QueryConfig<T> | null = null
  private loading = false
  private latestParams: QueryParams = {}

  /**
   * 挂载表格实例
   */
  mount(table: Table<T>, queryConfig?: QueryConfig<T>): void {
    this.table = table
    this.queryConfig = queryConfig || null

    // 自动加载
    if (queryConfig?.immediate) {
      this.query(queryConfig.initialParams)
    }
  }

  /**
   * 获取表格实例
   */
  getTable(): Table<T> | null {
    return this.table
  }

  /**
   * 查询数据
   * @param params - 额外的查询参数
   */
  async query(params?: QueryParams): Promise<void> {
    if (!this.queryConfig?.queryFn) {
      console.warn('[YxizoTable] No queryFn provided')
      return
    }

    this.loading = true

    try {
      // 合并参数
      const finalParams: QueryParams = {
        ...this.latestParams,
        ...params,
      }

      // 从表格状态中获取分页、排序信息
      if (this.table) {
        const state = this.table.getState()

        // 分页
        if (state.pagination) {
          finalParams.page = state.pagination.pageIndex + 1
          finalParams.pageSize = state.pagination.pageSize
        }

        // 排序
        if (state.sorting && state.sorting.length > 0) {
          const sort = state.sorting[0]
          finalParams.sortField = sort.id
          finalParams.sortOrder = sort.desc ? 'desc' : 'asc'
        }
      }

      this.latestParams = finalParams

      const response = await this.queryConfig.queryFn(finalParams)

      // 触发成功回调
      this.queryConfig.onSuccess?.(response)

      return response as any
    } catch (error) {
      this.queryConfig.onError?.(error as Error)
      throw error
    } finally {
      this.loading = false
    }
  }

  /**
   * 重新加载（使用最新参数）
   */
  async reload(): Promise<void> {
    return this.query()
  }

  /**
   * 重置并查询
   */
  async reset(): Promise<void> {
    if (this.table) {
      this.table.setPageIndex(0)
      this.table.resetSorting()
      this.table.resetColumnFilters()
      this.table.resetGlobalFilter()
    }

    this.latestParams = {}
    return this.query(this.queryConfig?.initialParams)
  }

  /**
   * 获取加载状态
   */
  isLoading(): boolean {
    return this.loading
  }

  /**
   * 获取选中的行
   */
  getSelectedRows(): T[] {
    if (!this.table) return []

    const selectedRowModel = this.table.getSelectedRowModel()
    return selectedRowModel.rows.map(row => row.original)
  }

  /**
   * 导出数据
   * @param format - 导出格式
   */
  exportData(format: 'csv' | 'json' = 'csv'): void {
    if (!this.table) return

    const rows = this.table.getRowModel().rows
    const columns = this.table.getAllColumns().filter(col => col.getIsVisible())

    if (format === 'json') {
      const data = rows.map(row => row.original)
      const json = JSON.stringify(data, null, 2)
      this.downloadFile(json, 'export.json', 'application/json')
    } else {
      // CSV 导出
      const headers = columns.map(col => col.columnDef.header).join(',')
      const csvRows = rows.map(row => {
        return columns.map(col => {
          const value = row.getValue(col.id)
          return `"${String(value ?? '').replace(/"/g, '""')}"`
        }).join(',')
      })
      const csv = [headers, ...csvRows].join('\n')
      this.downloadFile(csv, 'export.csv', 'text/csv;charset=utf-8;')
    }
  }

  /**
   * 下载文件辅助方法
   */
  private downloadFile(content: string, filename: string, mimeType: string): void {
    const blob = new Blob([content], { type: mimeType })
    const url = URL.createObjectURL(blob)
    const link = document.createElement('a')
    link.href = url
    link.download = filename
    link.click()
    URL.revokeObjectURL(url)
  }

  /**
   * 卸载
   */
  unmount(): void {
    this.table = null
    this.queryConfig = null
    this.latestParams = {}
  }
}
```

#### 4.2.4 组件层（Vue 实现）

```typescript
// packages/yxizo-table/src/composables/vue/useYxizoTable.ts

import type { Ref } from 'vue'
import type {
  ColumnDef,
  PaginationState,
  SortingState,
  ColumnFiltersState,
  VisibilityState,
} from '@tanstack/vue-table'
import { ref, onMounted, onUnmounted } from 'vue'
import {
  useVueTable,
  getCoreRowModel,
  getSortedRowModel,
  getFilteredRowModel,
  getPaginationRowModel,
} from '@tanstack/vue-table'

import { YxizoTableApi } from '../../api/table-api'
import type { QueryConfig } from '../../api/types'
import { enhanceColumns, type EnhancedColumnDef } from '../../extensions/columns/column-helper'
import { globalConfig } from '../../core/setup'

/**
 * 表格配置接口
 */
export interface UseYxizoTableOptions<T = any> {
  /** 列定义 */
  columns: EnhancedColumnDef<T>[]
  /** 数据（客户端模式） */
  data?: Ref<T[]> | T[]
  /** 查询配置（服务端模式） */
  queryConfig?: QueryConfig<T>
  /** 是否启用分页 */
  enablePagination?: boolean
  /** 是否启用排序 */
  enableSorting?: boolean
  /** 是否启用过滤 */
  enableFilters?: boolean
  /** 是否启用行选择 */
  enableRowSelection?: boolean
  /** 是否启用多列排序 */
  enableMultiSort?: boolean
  /** 初始分页状态 */
  initialPagination?: PaginationState
  /** 初始排序状态 */
  initialSorting?: SortingState
  /** 行唯一标识字段 */
  rowIdField?: string
}

/**
 * 使用 YxizoTable
 *
 * @example
 * ```ts
 * const { table, api, loading } = useYxizoTable({
 *   columns,
 *   data: users,
 *   enablePagination: true
 * })
 * ```
 */
export function useYxizoTable<T = any>(options: UseYxizoTableOptions<T>) {
  const {
    columns: rawColumns,
    data: initialData,
    queryConfig,
    enablePagination = true,
    enableSorting = true,
    enableFilters = false,
    enableRowSelection = false,
    enableMultiSort = false,
    initialPagination,
    initialSorting,
    rowIdField = 'id',
  } = options

  // 增强列定义
  const columns = enhanceColumns(rawColumns)

  // 数据
  const data = ref<T[]>(
    Array.isArray(initialData) ? initialData : (initialData?.value ?? [])
  ) as Ref<T[]>

  const total = ref(0)

  // 状态
  const sorting = ref<SortingState>(initialSorting ?? [])
  const columnFilters = ref<ColumnFiltersState>([])
  const columnVisibility = ref<VisibilityState>({})
  const rowSelection = ref({})
  const pagination = ref<PaginationState>(
    initialPagination ?? {
      pageIndex: 0,
      pageSize: globalConfig.getConfig().defaultPageSize ?? 20,
    }
  )

  // 加载状态
  const loading = ref(false)

  // 创建 API 实例
  const api = new YxizoTableApi<T>()

  // 查询配置增强
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

  // 创建表格实例
  const table = useVueTable({
    get data() {
      return data.value
    },
    columns,
    getCoreRowModel: getCoreRowModel(),
    ...(enableSorting && {
      getSortedRowModel: getSortedRowModel(),
      enableSorting: true,
      enableMultiSort,
      manualSorting: !!queryConfig,
    }),
    ...(enableFilters && {
      getFilteredRowModel: getFilteredRowModel(),
      enableFilters: true,
      manualFiltering: !!queryConfig,
    }),
    ...(enablePagination && {
      getPaginationRowModel: getPaginationRowModel(),
      manualPagination: !!queryConfig,
    }),
    ...(enableRowSelection && {
      enableRowSelection: true,
      getRowId: (row) => String((row as any)[rowIdField]),
    }),
    state: {
      get sorting() {
        return sorting.value
      },
      get columnFilters() {
        return columnFilters.value
      },
      get columnVisibility() {
        return columnVisibility.value
      },
      get rowSelection() {
        return rowSelection.value
      },
      get pagination() {
        return pagination.value
      },
    },
    onSortingChange: (updater) => {
      sorting.value =
        typeof updater === 'function' ? updater(sorting.value) : updater

      if (queryConfig && enableSorting) {
        api.query()
      }
    },
    onColumnFiltersChange: (updater) => {
      columnFilters.value =
        typeof updater === 'function' ? updater(columnFilters.value) : updater

      if (queryConfig && enableFilters) {
        api.query()
      }
    },
    onColumnVisibilityChange: (updater) => {
      columnVisibility.value =
        typeof updater === 'function' ? updater(columnVisibility.value) : updater
    },
    onRowSelectionChange: (updater) => {
      rowSelection.value =
        typeof updater === 'function' ? updater(rowSelection.value) : updater
    },
    onPaginationChange: (updater) => {
      pagination.value =
        typeof updater === 'function' ? updater(pagination.value) : updater

      if (queryConfig && enablePagination) {
        api.query()
      }
    },
  })

  // 挂载 API
  onMounted(() => {
    api.mount(table, enhancedQueryConfig)
  })

  // 卸载
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

---

## 五、使用指南

### 5.1 安装

```bash
# 安装核心包
pnpm add @yxizo/table

# Vue 项目
pnpm add @tanstack/vue-table

# React 项目
pnpm add @tanstack/react-table
```

### 5.2 全局初始化

```typescript
// main.ts
import { setupYxizoTable, globalConfig } from '@yxizo/table'
import { h } from 'vue'

// 初始化配置
setupYxizoTable({
  defaultPageSize: 20,
  pageSizeOptions: [10, 20, 50, 100],
  locale: 'zh-CN',
})

// 注册自定义渲染器
globalConfig.registerRenderer('image', ({ value }) => {
  return h('img', {
    src: value,
    style: 'width: 40px; height: 40px; border-radius: 4px;'
  })
})

globalConfig.registerRenderer('tag', ({ value, column }) => {
  const colorMap = column.meta?.colorMap || {}
  return h('span', {
    class: 'tag',
    style: {
      padding: '2px 8px',
      borderRadius: '4px',
      backgroundColor: colorMap[value] || '#f0f0f0',
      color: '#333'
    }
  }, value)
})
```

### 5.3 基础使用（客户端模式）

```vue
<script setup lang="ts">
import { ref } from 'vue'
import { useYxizoTable } from '@yxizo/table/vue'
import type { EnhancedColumnDef } from '@yxizo/table'

interface User {
  id: number
  name: string
  age: number
  email: string
  status: 'active' | 'inactive'
  createdAt: string
}

const users = ref<User[]>([
  {
    id: 1,
    name: '张三',
    age: 25,
    email: 'zhangsan@example.com',
    status: 'active',
    createdAt: '2024-01-01T00:00:00Z',
  },
  // ... more data
])

const columns: EnhancedColumnDef<User>[] = [
  {
    accessorKey: 'id',
    header: 'ID',
    width: 80,
  },
  {
    accessorKey: 'name',
    header: '姓名',
    width: 120,
  },
  {
    accessorKey: 'age',
    header: '年龄',
    width: 100,
  },
  {
    accessorKey: 'email',
    header: '邮箱',
    width: 200,
  },
  {
    accessorKey: 'status',
    header: '状态',
    renderer: 'tag',
    meta: {
      colorMap: {
        active: '#52c41a',
        inactive: '#ff4d4f',
      },
    },
    width: 100,
  },
  {
    accessorKey: 'createdAt',
    header: '创建时间',
    formatter: 'formatDateTime',
    width: 180,
  },
]

const { table, api } = useYxizoTable({
  columns,
  data: users,
  enablePagination: true,
  enableSorting: true,
  enableRowSelection: true,
})
</script>

<template>
  <div class="table-container">
    <!-- 工具栏 -->
    <div class="toolbar">
      <button @click="api.exportData('csv')">导出 CSV</button>
      <span>已选择: {{ api.getSelectedRows().length }} 项</span>
    </div>

    <!-- 表格 -->
    <table class="yxizo-table">
      <thead>
        <tr v-for="headerGroup in table.getHeaderGroups()" :key="headerGroup.id">
          <th
            v-for="header in headerGroup.headers"
            :key="header.id"
            @click="header.column.getToggleSortingHandler()?.($event)"
          >
            {{ header.column.columnDef.header }}
            <span v-if="header.column.getCanSort()">
              {{ { asc: '↑', desc: '↓' }[header.column.getIsSorted()] || '⇅' }}
            </span>
          </th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="row in table.getRowModel().rows" :key="row.id">
          <td v-for="cell in row.getVisibleCells()" :key="cell.id">
            <component
              :is="cell.column.columnDef.cell"
              v-bind="cell.getContext()"
            />
          </td>
        </tr>
      </tbody>
    </table>

    <!-- 分页 -->
    <div class="pagination">
      <button @click="table.firstPage()">首页</button>
      <button @click="table.previousPage()">上一页</button>
      <span>第 {{ table.getState().pagination.pageIndex + 1 }} 页</span>
      <button @click="table.nextPage()">下一页</button>
      <button @click="table.lastPage()">末页</button>
    </div>
  </div>
</template>
```

### 5.4 服务端模式

```vue
<script setup lang="ts">
import { ref } from 'vue'
import { useYxizoTable } from '@yxizo/table/vue'
import type { QueryParams, QueryResponse, EnhancedColumnDef } from '@yxizo/table'

interface User {
  id: number
  name: string
  email: string
}

const columns: EnhancedColumnDef<User>[] = [
  { accessorKey: 'id', header: 'ID' },
  { accessorKey: 'name', header: '姓名' },
  { accessorKey: 'email', header: '邮箱' },
]

async function queryUsers(params: QueryParams): Promise<QueryResponse<User>> {
  const response = await fetch('/api/users', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(params),
  })
  return response.json()
}

const { table, api, loading } = useYxizoTable({
  columns,
  queryConfig: {
    queryFn: queryUsers,
    immediate: true,
    onSuccess: (data) => console.log('查询成功', data),
    onError: (error) => console.error('查询失败', error),
  },
  enablePagination: true,
  enableSorting: true,
})
</script>
```

### 5.5 与搜索表单集成

```vue
<script setup lang="ts">
import { ref, reactive } from 'vue'
import { useYxizoTable } from '@yxizo/table/vue'

const searchForm = reactive({
  name: '',
  email: '',
  status: '',
})

async function queryUsers(params: QueryParams): Promise<QueryResponse<User>> {
  return fetch('/api/users', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      ...params,
      ...searchForm, // 合并搜索表单
    }),
  }).then(res => res.json())
}

const tableRef = ref()
const { table, api } = useYxizoTable({
  columns,
  queryConfig: { queryFn: queryUsers, immediate: true },
})

const handleSearch = () => api.query()
const handleReset = () => {
  Object.assign(searchForm, { name: '', email: '', status: '' })
  api.reset()
}
</script>

<template>
  <div>
    <!-- 搜索表单 -->
    <div class="search-form">
      <input v-model="searchForm.name" placeholder="姓名" />
      <input v-model="searchForm.email" placeholder="邮箱" />
      <select v-model="searchForm.status">
        <option value="">全部</option>
        <option value="active">启用</option>
        <option value="inactive">禁用</option>
      </select>
      <button @click="handleSearch">搜索</button>
      <button @click="handleReset">重置</button>
    </div>

    <!-- 表格组件... -->
  </div>
</template>
```

---

## 六、实施路线

### 6.1 分阶段实施计划

#### 阶段1：基础搭建（1-2天）

**目标**：建立项目结构，完成基础配置

- [ ] 创建 monorepo 项目结构
- [ ] 安装依赖（@tanstack/table-core、@tanstack/vue-table）
- [ ] 配置 TypeScript、Rollup 构建
- [ ] 实现 `setup.ts`（全局配置管理）
- [ ] 实现 `types.ts`（核心类型定义）

#### 阶段2：扩展层实现（2-3天）

**目标**：实现渲染器和格式化器系统

- [ ] 实现格式化器（日期、货币、百分比）
- [ ] 实现列增强逻辑
- [ ] 注册默认扩展
- [ ] 编写单元测试

#### 阶段3：API 层实现（2-3天）

**目标**：封装表格操作 API

- [ ] 实现 `YxizoTableApi` 类
- [ ] 实现 query/reload/reset 方法
- [ ] 实现 getSelectedRows/exportData 方法
- [ ] 编写 API 测试用例

#### 阶段4：组件层实现（3-4天）

**目标**：完成 Vue 组件封装

- [ ] 实现 `useYxizoTable.ts`
- [ ] 实现 `YxizoTable.vue`
- [ ] 实现插槽系统
- [ ] 实现样式和主题

#### 阶段5：高级功能（3-5天）

**目标**：增加高级特性

- [ ] 虚拟滚动集成
- [ ] 列拖拽排序
- [ ] 列宽调整
- [ ] 行展开功能

#### 阶段6：文档和示例（2-3天）

**目标**：完善文档和示例

- [ ] 编写 API 文档
- [ ] 创建在线演示
- [ ] 编写迁移指南

### 6.2 时间估算

- **最小可行产品（MVP）**：6-8 天
- **基础功能完整版**：12-15 天
- **高级功能版本**：20-25 天

---

## 七、最佳实践

### 7.1 类型安全优先

```typescript
// ✅ 好的做法：使用泛型确保类型安全
export function useYxizoTable<T extends Record<string, any>>(
  options: UseYxizoTableOptions<T>
) {
  // T 的类型会在整个链路中传递
}

// ❌ 不好的做法：使用 any
export function useYxizoTable(options: any) {
  // 失去类型检查
}
```

### 7.2 渐进式增强

```typescript
// 保留对 @tanstack/table 原生 API 的访问
const { table, api } = useYxizoTable(options)

// 用户既可以使用封装的 API
api.query()

// 也可以直接使用原生 API
table.getRowModel().rows
```

### 7.3 性能优化

```typescript
// 使用 computed 缓存计算结果
const enhancedColumns = computed(() => enhanceColumns(rawColumns))

// 避免在渲染函数中创建新对象
const cellStyle = useMemo(() => ({
  width: column.getSize() + 'px'
}), [column])
```

### 7.4 文档优先

每个公开 API 都应该有完整的 TSDoc 注释：

```typescript
/**
 * 查询表格数据
 * @param params - 查询参数
 * @returns Promise<void>
 * @example
 * ```ts
 * await api.query({ page: 1, pageSize: 20 })
 * ```
 */
async query(params?: QueryParams): Promise<void> {
  // ...
}
```

---

## 八、FAQ

### Q1: 为什么选择 @tanstack/table 而不是从零实现？

**A**:
- **节省时间**：从零实现需要 6-12 个月，使用 @tanstack/table 只需 1-2 周
- **质量保证**：@tanstack/table 有 50k+ stars，经过大量生产环境验证
- **持续维护**：社区活跃，每月都有更新
- **生态完善**：与 @tanstack/virtual、@tanstack/query 等无缝集成

### Q2: 与 vxe-table 封装有什么区别？

**A**:

| 特性 | vxe-table | yxizo-table |
|------|-----------|-------------|
| 底层依赖 | vxe-table（200KB+）| @tanstack/table（20KB）|
| UI 自由度 | 受限 | 完全自由 |
| 框架支持 | Vue | Vue/React/Solid |
| 学习成本 | 中 | 中 |

### Q3: 如何迁移现有的 vxe-table 代码？

**A**: 参考迁移步骤：

1. 保留原有列定义，只需调整部分属性名
2. 用 `useYxizoTable` 替换 `useVbenVxeGrid`
3. 调整模板部分，使用新的插槽系统
4. 测试验证功能正常

### Q4: 是否支持虚拟滚动？

**A**: 是的，通过集成 `@tanstack/virtual` 即可支持虚拟滚动，适用于大数据量场景（10万+ 行）。

### Q5: 如何自定义单元格渲染？

**A**: 三种方式：

1. 使用格式化器：`{ formatter: 'formatDate' }`
2. 使用渲染器：`{ renderer: 'image' }`
3. 自定义 cell 函数：`{ cell: (props) => h('div', props.getValue()) }`

---

## 附录

### A. 参考资料

- [@tanstack/table 官方文档](https://tanstack.com/table/latest)
- [Vue 3 文档](https://vuejs.org/)
- [TypeScript 文档](https://www.typescriptlang.org/)
- [vxe-table 源码分析](https://github.com/x-extends/vxe-table)

### B. 相关工具

- [@tanstack/virtual](https://tanstack.com/virtual/latest) - 虚拟滚动
- [@tanstack/query](https://tanstack.com/query/latest) - 数据查询
- [ag-grid](https://www.ag-grid.com/) - 企业级表格（对比参考）
- [react-table v7](https://react-table-v7.tanstack.com/) - @tanstack/table 前身

### C. 更新日志

- **v1.0.0** (2026-01-09): 初始架构设计文档

---

**文档结束**

如有疑问或建议，请联系架构团队。
