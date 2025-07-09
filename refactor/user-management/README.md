# 2. 用户管理模块 (User Management Module)

## 2.1 模块概述

### 2.1.1 定义
用户管理模块负责用户信息的维护、权限控制和角色管理，是系统用户侧的核心管理组件。

**形式化定义：**
```
UM = (U, P, R, A, F)
其中：
- U: 用户集合 {u₁, u₂, ..., uₙ}
- P: 权限集合 {p₁, p₂, ..., pₘ}
- R: 角色集合 {r₁, r₂, ..., rₖ}
- A: 分配函数 A: U × R → P
- F: 用户信息函数 F: U → I
```

### 2.1.2 哲学批判分析

#### 存在论层面
- **用户信息的本体地位**：用户信息是客观存在还是主观建构？
- **权限的本质**：权限是自然属性还是社会建构？

#### 认识论层面
- **信息获取的可靠性**：如何确保用户信息的准确性？
- **权限验证的确定性**：权限检查是否具有绝对确定性？

#### 价值论层面
- **隐私与便利的平衡**：如何在保护用户隐私和提供便利服务间平衡？
- **权力分配正义**：权限分配是否公平合理？

## 2.2 子模块结构

### 2.2.1 个人设置 (profile/)
- [个人信息管理](./profile/information.md)
- [偏好设置](./profile/preferences.md)
- [隐私控制](./profile/privacy.md)

### 2.2.2 用户信息 (user-info/)
- [基本信息](./user-info/basic.md)
- [扩展信息](./user-info/extended.md)
- [统计信息](./user-info/statistics.md)

### 2.2.3 权限控制 (permissions/)
- [权限模型](./permissions/model.md)
- [权限检查](./permissions/check.md)
- [权限继承](./permissions/inheritance.md)

### 2.2.4 角色管理 (roles/)
- [角色定义](./roles/definition.md)
- [角色分配](./roles/assignment.md)
- [角色继承](./roles/inheritance.md)

## 2.3 数学证明

### 2.3.1 权限传递性证明

**定理1：权限传递性**
```
对于任意用户 u ∈ U，角色 r₁, r₂ ∈ R，权限 p ∈ P，
如果 A(u, r₁) = p 且 r₁ ⊆ r₂，则 A(u, r₂) = p
```

**证明：**
1. 假设 A(u, r₁) = p
2. 根据角色继承关系，r₁ ⊆ r₂
3. 根据权限传递性公理，A(u, r₂) = p
4. 证毕

### 2.3.2 用户信息一致性证明

**定理2：用户信息一致性**
```
对于任意用户 u ∈ U，如果 F(u) = i，则 i 满足完整性约束
```

## 2.4 代码示例

### 2.4.1 Go语言实现

```go
package usermanagement

import (
    "time"
    "sync"
)

// User 用户结构体
type User struct {
    ID        string                 `json:"id"`
    Username  string                 `json:"username"`
    Email     string                 `json:"email"`
    Profile   UserProfile            `json:"profile"`
    Roles     []string              `json:"roles"`
    Permissions map[string]bool     `json:"permissions"`
    CreatedAt time.Time             `json:"created_at"`
    UpdatedAt time.Time             `json:"updated_at"`
}

// UserProfile 用户档案
type UserProfile struct {
    FirstName string `json:"first_name"`
    LastName  string `json:"last_name"`
    Avatar    string `json:"avatar"`
    Bio       string `json:"bio"`
}

// Role 角色结构体
type Role struct {
    ID          string            `json:"id"`
    Name        string            `json:"name"`
    Permissions map[string]bool   `json:"permissions"`
    Parent      *string           `json:"parent,omitempty"`
}

// UserManagementService 用户管理服务
type UserManagementService struct {
    users map[string]*User
    roles map[string]*Role
    mu    sync.RWMutex
}

// NewUserManagementService 创建用户管理服务
func NewUserManagementService() *UserManagementService {
    return &UserManagementService{
        users: make(map[string]*User),
        roles: make(map[string]*Role),
    }
}

// CreateUser 创建用户
func (ums *UserManagementService) CreateUser(username, email string) (*User, error) {
    ums.mu.Lock()
    defer ums.mu.Unlock()
    
    user := &User{
        ID:        generateID(),
        Username:  username,
        Email:     email,
        Roles:     []string{"user"},
        Permissions: make(map[string]bool),
        CreatedAt: time.Now(),
        UpdatedAt: time.Now(),
    }
    
    ums.users[user.ID] = user
    return user, nil
}

// AssignRole 分配角色
func (ums *UserManagementService) AssignRole(userID, roleID string) error {
    ums.mu.Lock()
    defer ums.mu.Unlock()
    
    user, exists := ums.users[userID]
    if !exists {
        return ErrUserNotFound
    }
    
    role, exists := ums.roles[roleID]
    if !exists {
        return ErrRoleNotFound
    }
    
    // 检查角色是否已分配
    for _, existingRole := range user.Roles {
        if existingRole == roleID {
            return ErrRoleAlreadyAssigned
        }
    }
    
    user.Roles = append(user.Roles, roleID)
    user.UpdatedAt = time.Now()
    
    // 更新权限
    for permission := range role.Permissions {
        user.Permissions[permission] = true
    }
    
    return nil
}

// CheckPermission 检查权限
func (ums *UserManagementService) CheckPermission(userID, permission string) bool {
    ums.mu.RLock()
    defer ums.mu.RUnlock()
    
    user, exists := ums.users[userID]
    if !exists {
        return false
    }
    
    return user.Permissions[permission]
}
```

### 2.4.2 Python实现

```python
from dataclasses import dataclass, field
from typing import Dict, List, Optional
from datetime import datetime
import uuid

@dataclass
class UserProfile:
    first_name: str = ""
    last_name: str = ""
    avatar: str = ""
    bio: str = ""

@dataclass
class User:
    id: str
    username: str
    email: str
    profile: UserProfile = field(default_factory=UserProfile)
    roles: List[str] = field(default_factory=list)
    permissions: Dict[str, bool] = field(default_factory=dict)
    created_at: datetime = field(default_factory=datetime.now)
    updated_at: datetime = field(default_factory=datetime.now)

@dataclass
class Role:
    id: str
    name: str
    permissions: Dict[str, bool] = field(default_factory=dict)
    parent: Optional[str] = None

class UserManagementService:
    def __init__(self):
        self.users: Dict[str, User] = {}
        self.roles: Dict[str, Role] = {}
    
    def create_user(self, username: str, email: str) -> User:
        """创建用户"""
        user_id = str(uuid.uuid4())
        user = User(
            id=user_id,
            username=username,
            email=email,
            roles=["user"]
        )
        
        self.users[user_id] = user
        return user
    
    def assign_role(self, user_id: str, role_id: str) -> bool:
        """分配角色"""
        if user_id not in self.users:
            return False
        
        if role_id not in self.roles:
            return False
        
        user = self.users[user_id]
        role = self.roles[role_id]
        
        if role_id in user.roles:
            return False
        
        user.roles.append(role_id)
        user.updated_at = datetime.now()
        
        # 更新权限
        for permission in role.permissions:
            user.permissions[permission] = True
        
        return True
    
    def check_permission(self, user_id: str, permission: str) -> bool:
        """检查权限"""
        if user_id not in self.users:
            return False
        
        user = self.users[user_id]
        return user.permissions.get(permission, False)
```

## 2.5 相关性链接

- [返回主目录](../README.md)
- [用户认证模块](../authentication/README.md)
- [支付系统模块](../payment/README.md)
- [数据展示模块](../data-presentation/README.md)
- [交互控制模块](../interaction-control/README.md) 