# 4. 数据展示模块 (Data Presentation Module)

## 4.1 模块概述

### 4.1.1 定义
数据展示模块负责将系统数据以用户友好的方式呈现，包括列表展示、详情页面、搜索功能和分页控制。

**形式化定义：**
```
DP = (D, V, F, P, S, R)
其中：
- D: 数据集合 {d₁, d₂, ..., dₙ}
- V: 视图集合 {v₁, v₂, ..., vₘ}
- F: 过滤函数 F: D × C → D'
- P: 分页函数 P: D → {page₁, page₂, ..., pageₖ}
- S: 排序函数 S: D × O → D''
- R: 渲染函数 R: D × V → HTML
```

### 4.1.2 哲学批判分析

#### 存在论层面
- **数据的本体地位**：数据是客观存在还是主观建构？
- **视图的真实性**：展示的视图是否反映数据的真实本质？

#### 认识论层面
- **数据理解的可靠性**：如何确保用户对数据的正确理解？
- **展示的准确性**：数据展示是否准确反映原始数据？

#### 价值论层面
- **简洁与完整的平衡**：如何在简洁性和完整性间找到平衡？
- **用户体验的优化**：如何优化用户的数据浏览体验？

## 4.2 子模块结构

### 4.2.1 列表展示 (list-display/)
- [列表组件设计](./list-display/component-design.md)
- [数据绑定](./list-display/data-binding.md)
- [动态更新](./list-display/dynamic-update.md)

### 4.2.2 详情页面 (detail-page/)
- [详情布局](./detail-page/layout.md)
- [数据展示](./detail-page/data-presentation.md)
- [交互功能](./detail-page/interaction.md)

### 4.2.3 搜索功能 (search/)
- [搜索算法](./search/algorithm.md)
- [搜索界面](./search/interface.md)
- [搜索结果](./search/results.md)

### 4.2.4 分页控制 (pagination/)
- [分页算法](./pagination/algorithm.md)
- [分页组件](./pagination/component.md)
- [分页状态](./pagination/state.md)

## 4.3 数学证明

### 4.3.1 数据一致性证明

**定理1：数据展示一致性**
```
对于任意数据 d ∈ D，视图 v ∈ V，如果 R(d, v) = html，
则 html 包含 d 的所有必要信息且格式正确
```

**证明：**
1. 假设 R(d, v) = html
2. 根据渲染函数定义，R 保持数据完整性
3. html 包含 d 的所有必要信息
4. 根据视图规范，html 格式正确
5. 证毕

### 4.3.2 分页完整性证明

**定理2：分页完整性**
```
对于任意数据集合 D，如果 P(D) = {page₁, page₂, ..., pageₙ}，
则 ∪ᵢ₌₁ⁿ pageᵢ = D 且 ∀i≠j, pageᵢ ∩ pageⱼ = ∅
```

## 4.4 代码示例

### 4.4.1 Go语言实现

```go
package datapresentation

import (
    "sort"
    "strings"
)

// DataItem 数据项
type DataItem struct {
    ID       string                 `json:"id"`
    Title    string                 `json:"title"`
    Content  string                 `json:"content"`
    Metadata map[string]interface{} `json:"metadata"`
    Created  int64                 `json:"created"`
}

// ListView 列表视图
type ListView struct {
    Items       []DataItem `json:"items"`
    TotalCount  int        `json:"total_count"`
    PageSize    int        `json:"page_size"`
    CurrentPage int        `json:"current_page"`
    TotalPages  int        `json:"total_pages"`
}

// SearchCriteria 搜索条件
type SearchCriteria struct {
    Query      string            `json:"query"`
    Filters    map[string]string `json:"filters"`
    SortBy     string            `json:"sort_by"`
    SortOrder  string            `json:"sort_order"`
    Page       int               `json:"page"`
    PageSize   int               `json:"page_size"`
}

// DataPresentationService 数据展示服务
type DataPresentationService struct {
    data []DataItem
}

// NewDataPresentationService 创建数据展示服务
func NewDataPresentationService(data []DataItem) *DataPresentationService {
    return &DataPresentationService{
        data: data,
    }
}

// Search 搜索数据
func (dps *DataPresentationService) Search(criteria SearchCriteria) *ListView {
    // 过滤数据
    filteredData := dps.filterData(dps.data, criteria)
    
    // 排序数据
    sortedData := dps.sortData(filteredData, criteria.SortBy, criteria.SortOrder)
    
    // 分页数据
    paginatedData := dps.paginateData(sortedData, criteria.Page, criteria.PageSize)
    
    return &ListView{
        Items:       paginatedData,
        TotalCount:  len(filteredData),
        PageSize:    criteria.PageSize,
        CurrentPage: criteria.Page,
        TotalPages:  (len(filteredData) + criteria.PageSize - 1) / criteria.PageSize,
    }
}

// GetDetail 获取详情
func (dps *DataPresentationService) GetDetail(id string) (*DataItem, error) {
    for _, item := range dps.data {
        if item.ID == id {
            return &item, nil
        }
    }
    return nil, ErrItemNotFound
}

// filterData 过滤数据
func (dps *DataPresentationService) filterData(data []DataItem, criteria SearchCriteria) []DataItem {
    var filtered []DataItem
    
    for _, item := range data {
        // 文本搜索
        if criteria.Query != "" {
            if !strings.Contains(strings.ToLower(item.Title), strings.ToLower(criteria.Query)) &&
               !strings.Contains(strings.ToLower(item.Content), strings.ToLower(criteria.Query)) {
                continue
            }
        }
        
        // 属性过滤
        match := true
        for key, value := range criteria.Filters {
            if itemValue, exists := item.Metadata[key]; !exists || itemValue != value {
                match = false
                break
            }
        }
        
        if match {
            filtered = append(filtered, item)
        }
    }
    
    return filtered
}

// sortData 排序数据
func (dps *DataPresentationService) sortData(data []DataItem, sortBy, sortOrder string) []DataItem {
    sorted := make([]DataItem, len(data))
    copy(sorted, data)
    
    sort.Slice(sorted, func(i, j int) bool {
        var result bool
        
        switch sortBy {
        case "title":
            result = sorted[i].Title < sorted[j].Title
        case "created":
            result = sorted[i].Created < sorted[j].Created
        default:
            result = sorted[i].ID < sorted[j].ID
        }
        
        if sortOrder == "desc" {
            result = !result
        }
        
        return result
    })
    
    return sorted
}

// paginateData 分页数据
func (dps *DataPresentationService) paginateData(data []DataItem, page, pageSize int) []DataItem {
    start := (page - 1) * pageSize
    end := start + pageSize
    
    if start >= len(data) {
        return []DataItem{}
    }
    
    if end > len(data) {
        end = len(data)
    }
    
    return data[start:end]
}

// RenderListView 渲染列表视图
func (dps *DataPresentationService) RenderListView(view *ListView) string {
    html := "<div class='list-view'>\n"
    
    for _, item := range view.Items {
        html += dps.renderListItem(item)
    }
    
    html += dps.renderPagination(view)
    html += "</div>"
    
    return html
}

// renderListItem 渲染列表项
func (dps *DataPresentationService) renderListItem(item DataItem) string {
    return fmt.Sprintf(`
        <div class='list-item' data-id='%s'>
            <h3>%s</h3>
            <p>%s</p>
            <div class='metadata'>
                <span class='created'>%s</span>
            </div>
        </div>
    `, item.ID, item.Title, item.Content, time.Unix(item.Created, 0).Format("2006-01-02"))
}

// renderPagination 渲染分页
func (dps *DataPresentationService) renderPagination(view *ListView) string {
    if view.TotalPages <= 1 {
        return ""
    }
    
    html := "<div class='pagination'>\n"
    
    // 上一页
    if view.CurrentPage > 1 {
        html += fmt.Sprintf("<a href='?page=%d' class='prev'>上一页</a>\n", view.CurrentPage-1)
    }
    
    // 页码
    for i := 1; i <= view.TotalPages; i++ {
        if i == view.CurrentPage {
            html += fmt.Sprintf("<span class='current'>%d</span>\n", i)
        } else {
            html += fmt.Sprintf("<a href='?page=%d'>%d</a>\n", i, i)
        }
    }
    
    // 下一页
    if view.CurrentPage < view.TotalPages {
        html += fmt.Sprintf("<a href='?page=%d' class='next'>下一页</a>\n", view.CurrentPage+1)
    }
    
    html += "</div>"
    return html
}
```

### 4.4.2 Python实现

```python
from dataclasses import dataclass, field
from typing import List, Dict, Optional, Any
from datetime import datetime
import re

@dataclass
class DataItem:
    id: str
    title: str
    content: str
    metadata: Dict[str, Any] = field(default_factory=dict)
    created: int = field(default_factory=lambda: int(datetime.now().timestamp()))

@dataclass
class ListView:
    items: List[DataItem]
    total_count: int
    page_size: int
    current_page: int
    total_pages: int

@dataclass
class SearchCriteria:
    query: str = ""
    filters: Dict[str, str] = field(default_factory=dict)
    sort_by: str = "id"
    sort_order: str = "asc"
    page: int = 1
    page_size: int = 10

class DataPresentationService:
    def __init__(self, data: List[DataItem]):
        self.data = data
    
    def search(self, criteria: SearchCriteria) -> ListView:
        """搜索数据"""
        # 过滤数据
        filtered_data = self._filter_data(self.data, criteria)
        
        # 排序数据
        sorted_data = self._sort_data(filtered_data, criteria.sort_by, criteria.sort_order)
        
        # 分页数据
        paginated_data = self._paginate_data(sorted_data, criteria.page, criteria.page_size)
        
        total_pages = (len(filtered_data) + criteria.page_size - 1) // criteria.page_size
        
        return ListView(
            items=paginated_data,
            total_count=len(filtered_data),
            page_size=criteria.page_size,
            current_page=criteria.page,
            total_pages=total_pages
        )
    
    def get_detail(self, item_id: str) -> Optional[DataItem]:
        """获取详情"""
        for item in self.data:
            if item.id == item_id:
                return item
        return None
    
    def _filter_data(self, data: List[DataItem], criteria: SearchCriteria) -> List[DataItem]:
        """过滤数据"""
        filtered = []
        
        for item in data:
            # 文本搜索
            if criteria.query:
                query_lower = criteria.query.lower()
                if (query_lower not in item.title.lower() and 
                    query_lower not in item.content.lower()):
                    continue
            
            # 属性过滤
            match = True
            for key, value in criteria.filters.items():
                if key not in item.metadata or item.metadata[key] != value:
                    match = False
                    break
            
            if match:
                filtered.append(item)
        
        return filtered
    
    def _sort_data(self, data: List[DataItem], sort_by: str, sort_order: str) -> List[DataItem]:
        """排序数据"""
        sorted_data = data.copy()
        
        def sort_key(item: DataItem):
            if sort_by == "title":
                return item.title
            elif sort_by == "created":
                return item.created
            else:
                return item.id
        
        sorted_data.sort(key=sort_key, reverse=(sort_order == "desc"))
        return sorted_data
    
    def _paginate_data(self, data: List[DataItem], page: int, page_size: int) -> List[DataItem]:
        """分页数据"""
        start = (page - 1) * page_size
        end = start + page_size
        
        if start >= len(data):
            return []
        
        if end > len(data):
            end = len(data)
        
        return data[start:end]
    
    def render_list_view(self, view: ListView) -> str:
        """渲染列表视图"""
        html = "<div class='list-view'>\n"
        
        for item in view.items:
            html += self._render_list_item(item)
        
        html += self._render_pagination(view)
        html += "</div>"
        
        return html
    
    def _render_list_item(self, item: DataItem) -> str:
        """渲染列表项"""
        created_date = datetime.fromtimestamp(item.created).strftime("%Y-%m-%d")
        
        return f"""
        <div class='list-item' data-id='{item.id}'>
            <h3>{item.title}</h3>
            <p>{item.content}</p>
            <div class='metadata'>
                <span class='created'>{created_date}</span>
            </div>
        </div>
        """
    
    def _render_pagination(self, view: ListView) -> str:
        """渲染分页"""
        if view.total_pages <= 1:
            return ""
        
        html = "<div class='pagination'>\n"
        
        # 上一页
        if view.current_page > 1:
            html += f"<a href='?page={view.current_page-1}' class='prev'>上一页</a>\n"
        
        # 页码
        for i in range(1, view.total_pages + 1):
            if i == view.current_page:
                html += f"<span class='current'>{i}</span>\n"
            else:
                html += f"<a href='?page={i}'>{i}</a>\n"
        
        # 下一页
        if view.current_page < view.total_pages:
            html += f"<a href='?page={view.current_page+1}' class='next'>下一页</a>\n"
        
        html += "</div>"
        return html
    
    def render_detail_view(self, item: DataItem) -> str:
        """渲染详情视图"""
        created_date = datetime.fromtimestamp(item.created).strftime("%Y-%m-%d %H:%M:%S")
        
        metadata_html = ""
        for key, value in item.metadata.items():
            metadata_html += f"<tr><td>{key}</td><td>{value}</td></tr>"
        
        return f"""
        <div class='detail-view'>
            <h1>{item.title}</h1>
            <div class='content'>{item.content}</div>
            <div class='metadata'>
                <p><strong>创建时间:</strong> {created_date}</p>
                <table class='metadata-table'>
                    <thead>
                        <tr><th>属性</th><th>值</th></tr>
                    </thead>
                    <tbody>
                        {metadata_html}
                    </tbody>
                </table>
            </div>
        </div>
        """
```

## 4.5 相关性链接

- [返回主目录](../README.md)
- [用户认证模块](../authentication/README.md)
- [用户管理模块](../user-management/README.md)
- [支付系统模块](../payment/README.md)
- [交互控制模块](../interaction-control/README.md) 