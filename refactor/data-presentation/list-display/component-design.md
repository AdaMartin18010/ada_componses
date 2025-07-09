# 4.2.1.1 列表组件设计 (List Component Design)

## 4.2.1.1.1 设计概述

### 4.2.1.1.1.1 定义
列表组件是数据展示的核心组件，负责将数据以结构化的方式呈现给用户，支持排序、过滤、分页等功能。

**形式化定义：**
```
LC = (D, C, S, F, P, R)
其中：
- D: 数据集合 {d₁, d₂, ..., dₙ}
- C: 组件集合 {c₁, c₂, ..., cₘ}
- S: 状态集合 {loading, ready, error}
- F: 过滤函数 F: D × C → D'
- P: 分页函数 P: D → {page₁, page₂, ..., pageₖ}
- R: 渲染函数 R: D × C → HTML
```

### 4.2.1.1.1.2 设计原则

#### 存在论层面
- **数据的本质**：列表中的数据是客观存在还是主观建构？
- **组件的真实性**：组件是否真实反映数据的本质？

#### 认识论层面
- **用户理解的准确性**：如何确保用户正确理解列表数据？
- **交互反馈的及时性**：如何提供即时的用户交互反馈？

#### 价值论层面
- **信息密度与可读性**：如何在信息密度和可读性间平衡？
- **性能与用户体验**：如何在性能和用户体验间找到平衡？

## 4.2.1.1.2 组件架构设计

### 4.2.1.1.2.1 组件层次结构

```typescript
interface ListComponentHierarchy {
  ListContainer: {
    children: ['ListHeader', 'ListBody', 'ListFooter'];
    props: {
      data: DataItem[];
      config: ListConfig;
      state: ListState;
    };
  };
  
  ListHeader: {
    children: ['SearchBar', 'FilterPanel', 'SortControls'];
    props: {
      onSearch: (query: string) => void;
      onFilter: (filters: FilterCriteria) => void;
      onSort: (sortBy: string, order: 'asc' | 'desc') => void;
    };
  };
  
  ListBody: {
    children: ['ListItems', 'EmptyState', 'LoadingState'];
    props: {
      items: DataItem[];
      loading: boolean;
      empty: boolean;
    };
  };
  
  ListFooter: {
    children: ['Pagination', 'SummaryInfo'];
    props: {
      totalItems: number;
      currentPage: number;
      pageSize: number;
    };
  };
}
```

### 4.2.1.1.2.2 数据流设计

```typescript
interface ListDataFlow {
  // 数据输入
  input: {
    rawData: DataItem[];
    userPreferences: UserPreferences;
    systemConfig: SystemConfig;
  };
  
  // 数据处理
  processing: {
    filtering: FilterProcessor;
    sorting: SortProcessor;
    pagination: PaginationProcessor;
    transformation: DataTransformer;
  };
  
  // 数据输出
  output: {
    displayData: ProcessedDataItem[];
    metadata: ListMetadata;
    userFeedback: UserFeedback;
  };
}
```

## 4.2.1.1.3 列表项设计

### 4.2.1.1.3.1 列表项结构

```typescript
interface ListItem {
  // 基础信息
  id: string;
  type: 'default' | 'compact' | 'detailed' | 'card';
  
  // 内容区域
  content: {
    primary: string;      // 主要文本
    secondary?: string;   // 次要文本
    description?: string; // 描述信息
    metadata?: Record<string, any>; // 元数据
  };
  
  // 操作区域
  actions: {
    primary?: ActionItem[];
    secondary?: ActionItem[];
    context?: ActionItem[];
  };
  
  // 状态信息
  status: {
    selected: boolean;
    highlighted: boolean;
    disabled: boolean;
    loading: boolean;
  };
  
  // 样式配置
  styling: {
    theme: 'light' | 'dark' | 'auto';
    variant: 'default' | 'success' | 'warning' | 'error';
    size: 'small' | 'medium' | 'large';
  };
}

interface ActionItem {
  id: string;
  label: string;
  icon?: string;
  type: 'button' | 'link' | 'menu';
  disabled?: boolean;
  onClick: (item: ListItem) => void;
}
```

### 4.2.1.1.3.2 列表项渲染

**定理1：列表项渲染一致性**
```
对于任意数据项 d ∈ D，如果 render(d) = html，
则 html 包含 d 的所有必要信息且格式正确
```

```typescript
class ListItemRenderer {
  private config: RenderConfig;
  private theme: Theme;
  
  constructor(config: RenderConfig, theme: Theme) {
    this.config = config;
    this.theme = theme;
  }
  
  render(item: ListItem): string {
    const template = this.getTemplate(item.type);
    const data = this.prepareData(item);
    const styles = this.getStyles(item.styling);
    
    return this.interpolate(template, data, styles);
  }
  
  private getTemplate(type: string): string {
    const templates = {
      default: this.getDefaultTemplate(),
      compact: this.getCompactTemplate(),
      detailed: this.getDetailedTemplate(),
      card: this.getCardTemplate()
    };
    
    return templates[type] || templates.default;
  }
  
  private getDefaultTemplate(): string {
    return `
      <div class="list-item {classNames}" data-id="{id}">
        <div class="list-item-content">
          <div class="list-item-primary">{primary}</div>
          {#if secondary}
          <div class="list-item-secondary">{secondary}</div>
          {/if}
          {#if description}
          <div class="list-item-description">{description}</div>
          {/if}
        </div>
        {#if actions}
        <div class="list-item-actions">
          {#each actions}
          <button class="action-button" onclick="{onClick}">
            {#if icon}<i class="{icon}"></i>{/if}
            {label}
          </button>
          {/each}
        </div>
        {/if}
      </div>
    `;
  }
  
  private prepareData(item: ListItem): Record<string, any> {
    return {
      id: item.id,
      primary: item.content.primary,
      secondary: item.content.secondary,
      description: item.content.description,
      actions: item.actions.primary,
      classNames: this.getClassNames(item)
    };
  }
  
  private getClassNames(item: ListItem): string {
    const classes = ['list-item'];
    
    if (item.status.selected) classes.push('selected');
    if (item.status.highlighted) classes.push('highlighted');
    if (item.status.disabled) classes.push('disabled');
    if (item.status.loading) classes.push('loading');
    
    classes.push(`theme-${item.styling.theme}`);
    classes.push(`variant-${item.styling.variant}`);
    classes.push(`size-${item.styling.size}`);
    
    return classes.join(' ');
  }
}
```

## 4.2.1.1.4 交互设计

### 4.2.1.1.4.1 选择机制

```typescript
interface SelectionManager {
  // 选择模式
  mode: 'single' | 'multiple' | 'range';
  
  // 选择状态
  selectedItems: Set<string>;
  
  // 选择事件
  onSelect: (itemId: string, selected: boolean) => void;
  onSelectAll: (selected: boolean) => void;
  onClearSelection: () => void;
  
  // 选择验证
  canSelect: (item: ListItem) => boolean;
  canDeselect: (item: ListItem) => boolean;
}

class ListSelectionManager implements SelectionManager {
  private selectedItems = new Set<string>();
  private mode: 'single' | 'multiple' | 'range' = 'single';
  private callbacks: SelectionCallbacks;
  
  constructor(mode: string, callbacks: SelectionCallbacks) {
    this.mode = mode as any;
    this.callbacks = callbacks;
  }
  
  select(itemId: string): void {
    if (!this.canSelect({ id: itemId } as ListItem)) {
      return;
    }
    
    if (this.mode === 'single') {
      this.selectedItems.clear();
    }
    
    this.selectedItems.add(itemId);
    this.callbacks.onSelect?.(itemId, true);
  }
  
  deselect(itemId: string): void {
    if (!this.canDeselect({ id: itemId } as ListItem)) {
      return;
    }
    
    this.selectedItems.delete(itemId);
    this.callbacks.onSelect?.(itemId, false);
  }
  
  selectAll(): void {
    // 实现全选逻辑
    this.callbacks.onSelectAll?.(true);
  }
  
  clearSelection(): void {
    this.selectedItems.clear();
    this.callbacks.onClearSelection?.();
  }
  
  canSelect(item: ListItem): boolean {
    return !item.status.disabled;
  }
  
  canDeselect(item: ListItem): boolean {
    return this.selectedItems.has(item.id);
  }
}
```

### 4.2.1.1.4.2 拖拽排序

```typescript
interface DragDropManager {
  // 拖拽状态
  dragging: boolean;
  draggedItem: ListItem | null;
  dropTarget: string | null;
  
  // 拖拽事件
  onDragStart: (item: ListItem, event: DragEvent) => void;
  onDragOver: (targetId: string, event: DragEvent) => void;
  onDrop: (sourceId: string, targetId: string) => void;
  onDragEnd: () => void;
  
  // 拖拽验证
  canDrag: (item: ListItem) => boolean;
  canDrop: (source: ListItem, target: ListItem) => boolean;
}

class ListDragDropManager implements DragDropManager {
  private dragging = false;
  private draggedItem: ListItem | null = null;
  private dropTarget: string | null = null;
  private callbacks: DragDropCallbacks;
  
  constructor(callbacks: DragDropCallbacks) {
    this.callbacks = callbacks;
  }
  
  startDrag(item: ListItem, event: DragEvent): void {
    if (!this.canDrag(item)) {
      return;
    }
    
    this.dragging = true;
    this.draggedItem = item;
    this.callbacks.onDragStart?.(item, event);
  }
  
  dragOver(targetId: string, event: DragEvent): void {
    if (!this.draggedItem) {
      return;
    }
    
    this.dropTarget = targetId;
    this.callbacks.onDragOver?.(targetId, event);
  }
  
  drop(sourceId: string, targetId: string): void {
    if (!this.canDrop(this.draggedItem!, { id: targetId } as ListItem)) {
      return;
    }
    
    this.callbacks.onDrop?.(sourceId, targetId);
    this.endDrag();
  }
  
  endDrag(): void {
    this.dragging = false;
    this.draggedItem = null;
    this.dropTarget = null;
    this.callbacks.onDragEnd?.();
  }
  
  canDrag(item: ListItem): boolean {
    return !item.status.disabled && item.type !== 'readonly';
  }
  
  canDrop(source: ListItem, target: ListItem): boolean {
    return source.id !== target.id;
  }
}
```

## 4.2.1.1.5 性能优化

### 4.2.1.1.5.1 虚拟滚动

**定理2：虚拟滚动性能**
```
对于包含 n 个项目的列表，如果使用虚拟滚动，
则渲染的DOM节点数量为 O(viewport_size) 而不是 O(n)
```

```typescript
interface VirtualScrollConfig {
  itemHeight: number;
  overscan: number;
  containerHeight: number;
}

class VirtualScrollManager {
  private config: VirtualScrollConfig;
  private items: ListItem[];
  private scrollTop = 0;
  private containerHeight = 0;
  
  constructor(config: VirtualScrollConfig) {
    this.config = config;
  }
  
  getVisibleRange(): { start: number; end: number } {
    const { itemHeight, overscan, containerHeight } = this.config;
    
    const start = Math.floor(this.scrollTop / itemHeight);
    const end = Math.ceil((this.scrollTop + containerHeight) / itemHeight);
    
    return {
      start: Math.max(0, start - overscan),
      end: Math.min(this.items.length, end + overscan)
    };
  }
  
  getVisibleItems(): ListItem[] {
    const { start, end } = this.getVisibleRange();
    return this.items.slice(start, end);
  }
  
  getScrollOffset(index: number): number {
    return index * this.config.itemHeight;
  }
  
  updateScrollPosition(scrollTop: number): void {
    this.scrollTop = scrollTop;
  }
  
  updateContainerHeight(height: number): void {
    this.containerHeight = height;
  }
}
```

### 4.2.1.1.5.2 懒加载

```typescript
interface LazyLoadConfig {
  threshold: number;
  batchSize: number;
  delay: number;
}

class LazyLoadManager {
  private config: LazyLoadConfig;
  private loadedItems: ListItem[] = [];
  private loading = false;
  private hasMore = true;
  
  constructor(config: LazyLoadConfig) {
    this.config = config;
  }
  
  async loadMore(): Promise<ListItem[]> {
    if (this.loading || !this.hasMore) {
      return [];
    }
    
    this.loading = true;
    
    try {
      const newItems = await this.fetchItems(
        this.loadedItems.length,
        this.config.batchSize
      );
      
      this.loadedItems.push(...newItems);
      this.hasMore = newItems.length === this.config.batchSize;
      
      return newItems;
    } finally {
      this.loading = false;
    }
  }
  
  private async fetchItems(offset: number, limit: number): Promise<ListItem[]> {
    // 实现数据获取逻辑
    return [];
  }
  
  isNearEnd(index: number): boolean {
    return index >= this.loadedItems.length - this.config.threshold;
  }
}
```

## 4.2.1.1.6 代码实现

### 4.2.1.1.6.1 Go语言实现

```go
package listcomponent

import (
    "html/template"
    "strings"
    "sync"
)

// ListComponent 列表组件
type ListComponent struct {
    config     *ListConfig
    data       []DataItem
    state      *ListState
    renderer   *ListItemRenderer
    selection  *SelectionManager
    dragDrop   *DragDropManager
    virtual    *VirtualScrollManager
    lazyLoad   *LazyLoadManager
    mu         sync.RWMutex
}

// NewListComponent 创建列表组件
func NewListComponent(config *ListConfig) *ListComponent {
    return &ListComponent{
        config:    config,
        data:      []DataItem{},
        state:     NewListState(),
        renderer:  NewListItemRenderer(config),
        selection: NewSelectionManager(config.SelectionMode),
        dragDrop:  NewDragDropManager(),
        virtual:   NewVirtualScrollManager(config.VirtualScroll),
        lazyLoad:  NewLazyLoadManager(config.LazyLoad),
    }
}

// ListConfig 列表配置
type ListConfig struct {
    // 基础配置
    ID          string            `json:"id"`
    Type        string            `json:"type"`
    Theme       string            `json:"theme"`
    
    // 功能配置
    SelectionMode string          `json:"selection_mode"`
    DragDrop      bool            `json:"drag_drop"`
    VirtualScroll *VirtualScrollConfig `json:"virtual_scroll"`
    LazyLoad      *LazyLoadConfig `json:"lazy_load"`
    
    // 样式配置
    ItemHeight    int             `json:"item_height"`
    ItemSpacing   int             `json:"item_spacing"`
    MaxHeight     int             `json:"max_height"`
    
    // 事件回调
    OnItemClick   func(item DataItem) `json:"-"`
    OnItemSelect  func(item DataItem, selected bool) `json:"-"`
    OnItemDrag    func(source, target DataItem) `json:"-"`
}

// DataItem 数据项
type DataItem struct {
    ID          string                 `json:"id"`
    Type        string                 `json:"type"`
    Content     ItemContent            `json:"content"`
    Actions     ItemActions            `json:"actions"`
    Status      ItemStatus             `json:"status"`
    Styling     ItemStyling            `json:"styling"`
    Metadata    map[string]interface{} `json:"metadata"`
}

// ItemContent 项目内容
type ItemContent struct {
    Primary     string `json:"primary"`
    Secondary   string `json:"secondary,omitempty"`
    Description string `json:"description,omitempty"`
}

// ItemActions 项目操作
type ItemActions struct {
    Primary   []ActionItem `json:"primary,omitempty"`
    Secondary []ActionItem `json:"secondary,omitempty"`
    Context   []ActionItem `json:"context,omitempty"`
}

// ActionItem 操作项
type ActionItem struct {
    ID       string `json:"id"`
    Label    string `json:"label"`
    Icon     string `json:"icon,omitempty"`
    Type     string `json:"type"`
    Disabled bool   `json:"disabled"`
}

// ItemStatus 项目状态
type ItemStatus struct {
    Selected    bool `json:"selected"`
    Highlighted bool `json:"highlighted"`
    Disabled    bool `json:"disabled"`
    Loading     bool `json:"loading"`
}

// ItemStyling 项目样式
type ItemStyling struct {
    Theme   string `json:"theme"`
    Variant string `json:"variant"`
    Size    string `json:"size"`
}

// ListState 列表状态
type ListState struct {
    Loading     bool   `json:"loading"`
    Error       string `json:"error,omitempty"`
    Empty       bool   `json:"empty"`
    TotalItems  int    `json:"total_items"`
    CurrentPage int    `json:"current_page"`
    PageSize    int    `json:"page_size"`
}

// NewListState 创建列表状态
func NewListState() *ListState {
    return &ListState{
        Loading:     false,
        Empty:       true,
        TotalItems:  0,
        CurrentPage: 1,
        PageSize:    20,
    }
}

// SetData 设置数据
func (lc *ListComponent) SetData(data []DataItem) {
    lc.mu.Lock()
    defer lc.mu.Unlock()
    
    lc.data = data
    lc.state.TotalItems = len(data)
    lc.state.Empty = len(data) == 0
}

// Render 渲染列表
func (lc *ListComponent) Render() string {
    lc.mu.RLock()
    defer lc.mu.RUnlock()
    
    if lc.state.Loading {
        return lc.renderLoading()
    }
    
    if lc.state.Error != "" {
        return lc.renderError()
    }
    
    if lc.state.Empty {
        return lc.renderEmpty()
    }
    
    return lc.renderList()
}

// renderList 渲染列表内容
func (lc *ListComponent) renderList() string {
    var items []string
    
    for _, item := range lc.data {
        itemHTML := lc.renderer.Render(item)
        items = append(items, itemHTML)
    }
    
    return fmt.Sprintf(`
        <div class="list-component" id="%s" data-type="%s" data-theme="%s">
            <div class="list-header">
                %s
            </div>
            <div class="list-body">
                %s
            </div>
            <div class="list-footer">
                %s
            </div>
        </div>
    `, lc.config.ID, lc.config.Type, lc.config.Theme,
       lc.renderHeader(),
       strings.Join(items, ""),
       lc.renderFooter())
}

// renderHeader 渲染头部
func (lc *ListComponent) renderHeader() string {
    return `
        <div class="list-header">
            <div class="list-search">
                <input type="text" placeholder="搜索..." class="search-input">
            </div>
            <div class="list-filters">
                <button class="filter-button">筛选</button>
            </div>
            <div class="list-sort">
                <select class="sort-select">
                    <option value="">排序</option>
                </select>
            </div>
        </div>
    `
}

// renderFooter 渲染底部
func (lc *ListComponent) renderFooter() string {
    return fmt.Sprintf(`
        <div class="list-footer">
            <div class="list-summary">
                显示 %d 项，共 %d 项
            </div>
            <div class="list-pagination">
                <button class="prev-button">上一页</button>
                <span class="page-info">第 %d 页</span>
                <button class="next-button">下一页</button>
            </div>
        </div>
    `, lc.state.PageSize, lc.state.TotalItems, lc.state.CurrentPage)
}

// renderLoading 渲染加载状态
func (lc *ListComponent) renderLoading() string {
    return `
        <div class="list-loading">
            <div class="loading-spinner"></div>
            <div class="loading-text">加载中...</div>
        </div>
    `
}

// renderError 渲染错误状态
func (lc *ListComponent) renderError() string {
    return fmt.Sprintf(`
        <div class="list-error">
            <div class="error-icon">⚠️</div>
            <div class="error-text">%s</div>
            <button class="retry-button">重试</button>
        </div>
    `, lc.state.Error)
}

// renderEmpty 渲染空状态
func (lc *ListComponent) renderEmpty() string {
    return `
        <div class="list-empty">
            <div class="empty-icon">📭</div>
            <div class="empty-text">暂无数据</div>
        </div>
    `
}

// ListItemRenderer 列表项渲染器
type ListItemRenderer struct {
    config *ListConfig
    theme  *Theme
}

// NewListItemRenderer 创建列表项渲染器
func NewListItemRenderer(config *ListConfig) *ListItemRenderer {
    return &ListItemRenderer{
        config: config,
        theme:  NewTheme(config.Theme),
    }
}

// Render 渲染列表项
func (lr *ListItemRenderer) Render(item DataItem) string {
    template := lr.getTemplate(item.Type)
    data := lr.prepareData(item)
    styles := lr.getStyles(item.Styling)
    
    return lr.interpolate(template, data, styles)
}

// getTemplate 获取模板
func (lr *ListItemRenderer) getTemplate(itemType string) string {
    templates := map[string]string{
        "default": lr.getDefaultTemplate(),
        "compact": lr.getCompactTemplate(),
        "detailed": lr.getDetailedTemplate(),
        "card": lr.getCardTemplate(),
    }
    
    if template, exists := templates[itemType]; exists {
        return template
    }
    
    return templates["default"]
}

// getDefaultTemplate 获取默认模板
func (lr *ListItemRenderer) getDefaultTemplate() string {
    return `
        <div class="list-item {classNames}" data-id="{id}">
            <div class="list-item-content">
                <div class="list-item-primary">{primary}</div>
                {#if secondary}
                <div class="list-item-secondary">{secondary}</div>
                {/if}
                {#if description}
                <div class="list-item-description">{description}</div>
                {/if}
            </div>
            {#if actions}
            <div class="list-item-actions">
                {#each actions}
                <button class="action-button" onclick="{onClick}">
                    {#if icon}<i class="{icon}"></i>{/if}
                    {label}
                </button>
                {/each}
            </div>
            {/if}
        </div>
    `
}

// prepareData 准备数据
func (lr *ListItemRenderer) prepareData(item DataItem) map[string]interface{} {
    return map[string]interface{}{
        "id":          item.ID,
        "primary":     item.Content.Primary,
        "secondary":   item.Content.Secondary,
        "description": item.Content.Description,
        "actions":     item.Actions.Primary,
        "classNames":  lr.getClassNames(item),
    }
}

// getClassNames 获取类名
func (lr *ListItemRenderer) getClassNames(item DataItem) string {
    classes := []string{"list-item"}
    
    if item.Status.Selected {
        classes = append(classes, "selected")
    }
    if item.Status.Highlighted {
        classes = append(classes, "highlighted")
    }
    if item.Status.Disabled {
        classes = append(classes, "disabled")
    }
    if item.Status.Loading {
        classes = append(classes, "loading")
    }
    
    classes = append(classes, "theme-"+item.Styling.Theme)
    classes = append(classes, "variant-"+item.Styling.Variant)
    classes = append(classes, "size-"+item.Styling.Size)
    
    return strings.Join(classes, " ")
}

// 其他辅助方法
func (lr *ListItemRenderer) getCompactTemplate() string {
    return `<div class="list-item compact">{primary}</div>`
}

func (lr *ListItemRenderer) getDetailedTemplate() string {
    return `<div class="list-item detailed">{primary} - {description}</div>`
}

func (lr *ListItemRenderer) getCardTemplate() string {
    return `<div class="list-item card">{primary}</div>`
}

func (lr *ListItemRenderer) getStyles(styling ItemStyling) map[string]string {
    return map[string]string{
        "theme":   styling.Theme,
        "variant": styling.Variant,
        "size":    styling.Size,
    }
}

func (lr *ListItemRenderer) interpolate(template string, data map[string]interface{}, styles map[string]string) string {
    // 实现模板插值逻辑
    return template
}

// 配置结构体
type VirtualScrollConfig struct {
    ItemHeight int `json:"item_height"`
    Overscan   int `json:"overscan"`
}

type LazyLoadConfig struct {
    Threshold int `json:"threshold"`
    BatchSize int `json:"batch_size"`
    Delay     int `json:"delay"`
}

type Theme struct {
    Name string
}

func NewTheme(name string) *Theme {
    return &Theme{Name: name}
}

// 管理器结构体
type SelectionManager struct {
    mode string
}

func NewSelectionManager(mode string) *SelectionManager {
    return &SelectionManager{mode: mode}
}

type DragDropManager struct{}

func NewDragDropManager() *DragDropManager {
    return &DragDropManager{}
}

type VirtualScrollManager struct {
    config *VirtualScrollConfig
}

func NewVirtualScrollManager(config *VirtualScrollConfig) *VirtualScrollManager {
    return &VirtualScrollManager{config: config}
}

type LazyLoadManager struct {
    config *LazyLoadConfig
}

func NewLazyLoadManager(config *LazyLoadConfig) *LazyLoadManager {
    return &LazyLoadManager{config: config}
}
```

### 4.2.1.1.5.2 Python实现

```python
from dataclasses import dataclass, field
from typing import List, Dict, Optional, Any
from abc import ABC, abstractmethod
import threading

@dataclass
class ListConfig:
    id: str
    type: str = "default"
    theme: str = "light"
    selection_mode: str = "single"
    drag_drop: bool = False
    virtual_scroll: Optional['VirtualScrollConfig'] = None
    lazy_load: Optional['LazyLoadConfig'] = None
    item_height: int = 60
    item_spacing: int = 8
    max_height: int = 400

@dataclass
class DataItem:
    id: str
    type: str = "default"
    content: 'ItemContent' = None
    actions: 'ItemActions' = None
    status: 'ItemStatus' = None
    styling: 'ItemStyling' = None
    metadata: Dict[str, Any] = field(default_factory=dict)
    
    def __post_init__(self):
        if self.content is None:
            self.content = ItemContent()
        if self.actions is None:
            self.actions = ItemActions()
        if self.status is None:
            self.status = ItemStatus()
        if self.styling is None:
            self.styling = ItemStyling()

@dataclass
class ItemContent:
    primary: str = ""
    secondary: str = ""
    description: str = ""

@dataclass
class ItemActions:
    primary: List['ActionItem'] = field(default_factory=list)
    secondary: List['ActionItem'] = field(default_factory=list)
    context: List['ActionItem'] = field(default_factory=list)

@dataclass
class ActionItem:
    id: str
    label: str
    icon: str = ""
    type: str = "button"
    disabled: bool = False

@dataclass
class ItemStatus:
    selected: bool = False
    highlighted: bool = False
    disabled: bool = False
    loading: bool = False

@dataclass
class ItemStyling:
    theme: str = "light"
    variant: str = "default"
    size: str = "medium"

@dataclass
class ListState:
    loading: bool = False
    error: str = ""
    empty: bool = True
    total_items: int = 0
    current_page: int = 1
    page_size: int = 20

class ListComponent:
    def __init__(self, config: ListConfig):
        self.config = config
        self.data: List[DataItem] = []
        self.state = ListState()
        self.renderer = ListItemRenderer(config)
        self.selection = SelectionManager(config.selection_mode)
        self.drag_drop = DragDropManager()
        self.virtual = VirtualScrollManager(config.virtual_scroll) if config.virtual_scroll else None
        self.lazy_load = LazyLoadManager(config.lazy_load) if config.lazy_load else None
        self._lock = threading.RLock()
    
    def set_data(self, data: List[DataItem]) -> None:
        """设置数据"""
        with self._lock:
            self.data = data
            self.state.total_items = len(data)
            self.state.empty = len(data) == 0
    
    def render(self) -> str:
        """渲染列表"""
        with self._lock:
            if self.state.loading:
                return self._render_loading()
            
            if self.state.error:
                return self._render_error()
            
            if self.state.empty:
                return self._render_empty()
            
            return self._render_list()
    
    def _render_list(self) -> str:
        """渲染列表内容"""
        items_html = []
        for item in self.data:
            item_html = self.renderer.render(item)
            items_html.append(item_html)
        
        return f"""
        <div class="list-component" id="{self.config.id}" data-type="{self.config.type}" data-theme="{self.config.theme}">
            <div class="list-header">
                {self._render_header()}
            </div>
            <div class="list-body">
                {''.join(items_html)}
            </div>
            <div class="list-footer">
                {self._render_footer()}
            </div>
        </div>
        """
    
    def _render_header(self) -> str:
        """渲染头部"""
        return """
        <div class="list-header">
            <div class="list-search">
                <input type="text" placeholder="搜索..." class="search-input">
            </div>
            <div class="list-filters">
                <button class="filter-button">筛选</button>
            </div>
            <div class="list-sort">
                <select class="sort-select">
                    <option value="">排序</option>
                </select>
            </div>
        </div>
        """
    
    def _render_footer(self) -> str:
        """渲染底部"""
        return f"""
        <div class="list-footer">
            <div class="list-summary">
                显示 {self.state.page_size} 项，共 {self.state.total_items} 项
            </div>
            <div class="list-pagination">
                <button class="prev-button">上一页</button>
                <span class="page-info">第 {self.state.current_page} 页</span>
                <button class="next-button">下一页</button>
            </div>
        </div>
        """
    
    def _render_loading(self) -> str:
        """渲染加载状态"""
        return """
        <div class="list-loading">
            <div class="loading-spinner"></div>
            <div class="loading-text">加载中...</div>
        </div>
        """
    
    def _render_error(self) -> str:
        """渲染错误状态"""
        return f"""
        <div class="list-error">
            <div class="error-icon">⚠️</div>
            <div class="error-text">{self.state.error}</div>
            <button class="retry-button">重试</button>
        </div>
        """
    
    def _render_empty(self) -> str:
        """渲染空状态"""
        return """
        <div class="list-empty">
            <div class="empty-icon">📭</div>
            <div class="empty-text">暂无数据</div>
        </div>
        """

class ListItemRenderer:
    def __init__(self, config: ListConfig):
        self.config = config
        self.theme = Theme(config.theme)
    
    def render(self, item: DataItem) -> str:
        """渲染列表项"""
        template = self._get_template(item.type)
        data = self._prepare_data(item)
        styles = self._get_styles(item.styling)
        
        return self._interpolate(template, data, styles)
    
    def _get_template(self, item_type: str) -> str:
        """获取模板"""
        templates = {
            "default": self._get_default_template(),
            "compact": self._get_compact_template(),
            "detailed": self._get_detailed_template(),
            "card": self._get_card_template()
        }
        
        return templates.get(item_type, templates["default"])
    
    def _get_default_template(self) -> str:
        """获取默认模板"""
        return """
        <div class="list-item {class_names}" data-id="{id}">
            <div class="list-item-content">
                <div class="list-item-primary">{primary}</div>
                {#if secondary}
                <div class="list-item-secondary">{secondary}</div>
                {/if}
                {#if description}
                <div class="list-item-description">{description}</div>
                {/if}
            </div>
            {#if actions}
            <div class="list-item-actions">
                {#each actions}
                <button class="action-button" onclick="{onClick}">
                    {#if icon}<i class="{icon}"></i>{/if}
                    {label}
                </button>
                {/each}
            </div>
            {/if}
        </div>
        """
    
    def _prepare_data(self, item: DataItem) -> Dict[str, Any]:
        """准备数据"""
        return {
            "id": item.id,
            "primary": item.content.primary,
            "secondary": item.content.secondary,
            "description": item.content.description,
            "actions": item.actions.primary,
            "class_names": self._get_class_names(item)
        }
    
    def _get_class_names(self, item: DataItem) -> str:
        """获取类名"""
        classes = ["list-item"]
        
        if item.status.selected:
            classes.append("selected")
        if item.status.highlighted:
            classes.append("highlighted")
        if item.status.disabled:
            classes.append("disabled")
        if item.status.loading:
            classes.append("loading")
        
        classes.append(f"theme-{item.styling.theme}")
        classes.append(f"variant-{item.styling.variant}")
        classes.append(f"size-{item.styling.size}")
        
        return " ".join(classes)
    
    def _get_styles(self, styling: ItemStyling) -> Dict[str, str]:
        """获取样式"""
        return {
            "theme": styling.theme,
            "variant": styling.variant,
            "size": styling.size
        }
    
    def _interpolate(self, template: str, data: Dict[str, Any], styles: Dict[str, str]) -> str:
        """模板插值"""
        # 简单的字符串替换实现
        result = template
        for key, value in data.items():
            result = result.replace(f"{{{key}}}", str(value))
        return result
    
    def _get_compact_template(self) -> str:
        """获取紧凑模板"""
        return '<div class="list-item compact">{primary}</div>'
    
    def _get_detailed_template(self) -> str:
        """获取详细模板"""
        return '<div class="list-item detailed">{primary} - {description}</div>'
    
    def _get_card_template(self) -> str:
        """获取卡片模板"""
        return '<div class="list-item card">{primary}</div>'

class SelectionManager:
    def __init__(self, mode: str):
        self.mode = mode
        self.selected_items = set()
    
    def select(self, item_id: str) -> None:
        """选择项目"""
        if self.mode == "single":
            self.selected_items.clear()
        
        self.selected_items.add(item_id)
    
    def deselect(self, item_id: str) -> None:
        """取消选择"""
        self.selected_items.discard(item_id)
    
    def clear_selection(self) -> None:
        """清除选择"""
        self.selected_items.clear()

class DragDropManager:
    def __init__(self):
        self.dragging = False
        self.dragged_item = None
        self.drop_target = None
    
    def start_drag(self, item: DataItem) -> None:
        """开始拖拽"""
        self.dragging = True
        self.dragged_item = item
    
    def end_drag(self) -> None:
        """结束拖拽"""
        self.dragging = False
        self.dragged_item = None
        self.drop_target = None

class VirtualScrollManager:
    def __init__(self, config: 'VirtualScrollConfig'):
        self.config = config
        self.scroll_top = 0
        self.container_height = 0
    
    def get_visible_range(self) -> tuple[int, int]:
        """获取可见范围"""
        item_height = self.config.item_height
        start = self.scroll_top // item_height
        end = (self.scroll_top + self.container_height) // item_height
        
        return start, end

class LazyLoadManager:
    def __init__(self, config: 'LazyLoadConfig'):
        self.config = config
        self.loaded_items = []
        self.loading = False
        self.has_more = True
    
    async def load_more(self) -> List[DataItem]:
        """加载更多数据"""
        if self.loading or not self.has_more:
            return []
        
        self.loading = True
        try:
            # 模拟异步加载
            new_items = []
            self.loaded_items.extend(new_items)
            return new_items
        finally:
            self.loading = False

@dataclass
class VirtualScrollConfig:
    item_height: int
    overscan: int

@dataclass
class LazyLoadConfig:
    threshold: int
    batch_size: int
    delay: int

class Theme:
    def __init__(self, name: str):
        self.name = name
```

## 4.2.1.1.7 相关性链接

- [返回数据展示模块](../README.md)
- [数据绑定](./data-binding.md)
- [动态更新](./dynamic-update.md)
- [详情布局](../detail-page/layout.md)
- [搜索算法](../search/algorithm.md) 