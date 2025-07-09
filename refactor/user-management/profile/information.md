# 2.2.1.1 个人信息管理 (Profile Information Management)

## 2.2.1.1.1 设计概述

### 2.2.1.1.1.1 定义
个人信息管理模块负责用户档案的创建、维护和展示，是用户身份的重要组成部分。

**形式化定义：**
```
PI = (P, F, V, U, A)
其中：
- P: 档案集合 {p₁, p₂, ..., pₙ}
- F: 字段集合 {f₁, f₂, ..., fₘ}
- V: 验证规则集合 {v₁, v₂, ..., vₖ}
- U: 用户集合
- A: 访问控制函数 A: P × U → {read, write, none}
```

### 2.2.1.1.1.2 设计原则

#### 存在论层面
- **个人信息的本质**：用户信息是客观事实还是主观建构？
- **隐私的边界**：哪些信息属于个人隐私范畴？

#### 认识论层面
- **信息的准确性**：如何确保用户信息的真实性？
- **信息的完整性**：如何保证用户信息的完整性？

#### 价值论层面
- **个人尊严的保护**：如何在信息收集与个人尊严间平衡？
- **社会价值的实现**：如何通过个人信息实现社会价值？

## 2.2.1.1.2 档案结构设计

### 2.2.1.1.2.1 基本信息

```typescript
interface BasicProfile {
  // 身份信息
  id: string;
  username: string;
  email: string;
  phone?: string;
  
  // 个人信息
  firstName: string;
  lastName: string;
  displayName: string;
  gender?: 'male' | 'female' | 'other' | 'prefer_not_to_say';
  birthDate?: Date;
  
  // 位置信息
  country?: string;
  region?: string;
  city?: string;
  timezone?: string;
  
  // 语言偏好
  primaryLanguage: string;
  secondaryLanguages?: string[];
  
  // 时区设置
  timezone: string;
  dateFormat: 'MM/DD/YYYY' | 'DD/MM/YYYY' | 'YYYY-MM-DD';
  timeFormat: '12h' | '24h';
}
```

### 2.2.1.1.2.2 扩展信息

```typescript
interface ExtendedProfile {
  // 职业信息
  occupation?: string;
  company?: string;
  jobTitle?: string;
  industry?: string;
  
  // 教育背景
  education?: {
    level: 'high_school' | 'bachelor' | 'master' | 'phd' | 'other';
    institution?: string;
    field?: string;
    graduationYear?: number;
  };
  
  // 兴趣爱好
  interests?: string[];
  skills?: string[];
  
  // 社交媒体
  socialMedia?: {
    twitter?: string;
    linkedin?: string;
    github?: string;
    website?: string;
  };
  
  // 个人简介
  bio?: string;
  avatar?: string;
  coverImage?: string;
}
```

### 2.2.1.1.2.3 隐私设置

```typescript
interface PrivacySettings {
  // 可见性设置
  profileVisibility: 'public' | 'friends' | 'private';
  emailVisibility: 'public' | 'friends' | 'private';
  phoneVisibility: 'public' | 'friends' | 'private';
  
  // 搜索设置
  allowSearchByEmail: boolean;
  allowSearchByPhone: boolean;
  
  // 通知设置
  emailNotifications: {
    marketing: boolean;
    security: boolean;
    updates: boolean;
  };
  
  // 数据使用
  allowDataAnalytics: boolean;
  allowPersonalization: boolean;
}
```

## 2.2.1.1.3 验证规则设计

### 2.2.1.1.3.1 数据验证

**定理1：个人信息完整性**
```
对于任意档案 p ∈ P，如果 p 包含必填字段，
则 p 满足完整性约束
```

**验证规则：**
```python
class ProfileValidator:
    def __init__(self):
        self.required_fields = ['username', 'email', 'firstName', 'lastName']
        self.validation_rules = {
            'username': UsernameRule(),
            'email': EmailRule(),
            'phone': PhoneRule(),
            'birthDate': BirthDateRule(),
        }
    
    def validate_profile(self, profile: dict) -> tuple[bool, List[str]]:
        """验证档案信息"""
        errors = []
        
        # 检查必填字段
        for field in self.required_fields:
            if not profile.get(field):
                errors.append(f"字段 {field} 为必填项")
        
        # 验证各字段格式
        for field, rule in self.validation_rules.items():
            if field in profile and profile[field]:
                if not rule.validate(profile[field]):
                    errors.append(rule.get_error_message())
        
        return len(errors) == 0, errors
```

### 2.2.1.1.3.2 数据一致性

**定理2：档案数据一致性**
```
对于任意档案 p ∈ P，如果 p 被更新，
则所有相关字段必须保持一致性
```

## 2.2.1.1.4 访问控制设计

### 2.2.1.1.4.1 权限模型

```typescript
interface ProfileAccessControl {
  // 所有者权限
  owner: {
    read: true;
    write: true;
    delete: true;
    share: true;
  };
  
  // 好友权限
  friends: {
    read: boolean; // 基于隐私设置
    write: false;
    delete: false;
    share: false;
  };
  
  // 公众权限
  public: {
    read: boolean; // 基于隐私设置
    write: false;
    delete: false;
    share: false;
  };
}
```

### 2.2.1.1.4.2 数据脱敏

```python
class DataMasking:
    @staticmethod
    def mask_email(email: str) -> str:
        """脱敏邮箱地址"""
        if not email:
            return ""
        
        parts = email.split('@')
        if len(parts) != 2:
            return email
        
        username, domain = parts
        if len(username) <= 2:
            masked_username = username
        else:
            masked_username = username[0] + '*' * (len(username) - 2) + username[-1]
        
        return f"{masked_username}@{domain}"
    
    @staticmethod
    def mask_phone(phone: str) -> str:
        """脱敏手机号码"""
        if not phone:
            return ""
        
        if len(phone) <= 4:
            return phone
        
        return phone[:2] + '*' * (len(phone) - 4) + phone[-2:]
    
    @staticmethod
    def mask_name(name: str) -> str:
        """脱敏姓名"""
        if not name:
            return ""
        
        if len(name) <= 1:
            return name
        
        return name[0] + '*' * (len(name) - 1)
```

## 2.2.1.1.5 代码实现

### 2.2.1.1.5.1 Go语言实现

```go
package profile

import (
    "time"
    "regexp"
    "strings"
)

// Profile 用户档案
type Profile struct {
    ID        string    `json:"id"`
    UserID    string    `json:"user_id"`
    
    // 基本信息
    Username     string    `json:"username"`
    Email        string    `json:"email"`
    Phone        string    `json:"phone,omitempty"`
    FirstName    string    `json:"first_name"`
    LastName     string    `json:"last_name"`
    DisplayName  string    `json:"display_name"`
    Gender       string    `json:"gender,omitempty"`
    BirthDate    *time.Time `json:"birth_date,omitempty"`
    
    // 位置信息
    Country      string    `json:"country,omitempty"`
    Region       string    `json:"region,omitempty"`
    City         string    `json:"city,omitempty"`
    Timezone     string    `json:"timezone"`
    
    // 语言设置
    PrimaryLanguage    string   `json:"primary_language"`
    SecondaryLanguages []string `json:"secondary_languages,omitempty"`
    
    // 格式设置
    DateFormat string `json:"date_format"`
    TimeFormat string `json:"time_format"`
    
    // 扩展信息
    Occupation  string `json:"occupation,omitempty"`
    Company     string `json:"company,omitempty"`
    JobTitle    string `json:"job_title,omitempty"`
    Industry    string `json:"industry,omitempty"`
    
    // 个人简介
    Bio         string `json:"bio,omitempty"`
    Avatar      string `json:"avatar,omitempty"`
    CoverImage  string `json:"cover_image,omitempty"`
    
    // 隐私设置
    ProfileVisibility string `json:"profile_visibility"`
    EmailVisibility   string `json:"email_visibility"`
    PhoneVisibility   string `json:"phone_visibility"`
    
    // 时间戳
    CreatedAt time.Time `json:"created_at"`
    UpdatedAt time.Time `json:"updated_at"`
}

// ProfileService 档案服务
type ProfileService struct {
    profiles map[string]*Profile
    validator *ProfileValidator
    accessControl *AccessControl
}

// NewProfileService 创建档案服务
func NewProfileService() *ProfileService {
    return &ProfileService{
        profiles: make(map[string]*Profile),
        validator: NewProfileValidator(),
        accessControl: NewAccessControl(),
    }
}

// CreateProfile 创建档案
func (ps *ProfileService) CreateProfile(userID string, data map[string]interface{}) (*Profile, error) {
    // 验证数据
    if valid, errors := ps.validator.ValidateProfile(data); !valid {
        return nil, &ValidationError{Errors: errors}
    }
    
    profile := &Profile{
        ID:        generateProfileID(),
        UserID:    userID,
        Username:  data["username"].(string),
        Email:     data["email"].(string),
        FirstName: data["first_name"].(string),
        LastName:  data["last_name"].(string),
        Timezone:  "UTC",
        DateFormat: "YYYY-MM-DD",
        TimeFormat: "24h",
        ProfileVisibility: "public",
        EmailVisibility:   "private",
        PhoneVisibility:   "private",
        CreatedAt: time.Now(),
        UpdatedAt: time.Now(),
    }
    
    // 设置可选字段
    if phone, ok := data["phone"].(string); ok {
        profile.Phone = phone
    }
    if displayName, ok := data["display_name"].(string); ok {
        profile.DisplayName = displayName
    }
    
    ps.profiles[profile.ID] = profile
    return profile, nil
}

// UpdateProfile 更新档案
func (ps *ProfileService) UpdateProfile(profileID string, userID string, data map[string]interface{}) (*Profile, error) {
    profile, exists := ps.profiles[profileID]
    if !exists {
        return nil, ErrProfileNotFound
    }
    
    // 检查权限
    if !ps.accessControl.CanWrite(profile, userID) {
        return nil, ErrAccessDenied
    }
    
    // 验证更新数据
    if valid, errors := ps.validator.ValidateUpdate(data); !valid {
        return nil, &ValidationError{Errors: errors}
    }
    
    // 更新字段
    for key, value := range data {
        switch key {
        case "first_name":
            profile.FirstName = value.(string)
        case "last_name":
            profile.LastName = value.(string)
        case "email":
            profile.Email = value.(string)
        case "phone":
            profile.Phone = value.(string)
        case "bio":
            profile.Bio = value.(string)
        case "avatar":
            profile.Avatar = value.(string)
        }
    }
    
    profile.UpdatedAt = time.Now()
    return profile, nil
}

// GetProfile 获取档案
func (ps *ProfileService) GetProfile(profileID string, userID string) (*Profile, error) {
    profile, exists := ps.profiles[profileID]
    if !exists {
        return nil, ErrProfileNotFound
    }
    
    // 检查访问权限
    if !ps.accessControl.CanRead(profile, userID) {
        return nil, ErrAccessDenied
    }
    
    // 根据权限脱敏数据
    return ps.accessControl.MaskProfile(profile, userID), nil
}

// ProfileValidator 档案验证器
type ProfileValidator struct {
    rules map[string]ValidationRule
}

// NewProfileValidator 创建档案验证器
func NewProfileValidator() *ProfileValidator {
    return &ProfileValidator{
        rules: map[string]ValidationRule{
            "email": &EmailRule{},
            "phone": &PhoneRule{},
            "birth_date": &BirthDateRule{},
        },
    }
}

// ValidateProfile 验证档案
func (pv *ProfileValidator) ValidateProfile(data map[string]interface{}) (bool, []string) {
    var errors []string
    
    // 检查必填字段
    requiredFields := []string{"username", "email", "first_name", "last_name"}
    for _, field := range requiredFields {
        if value, exists := data[field]; !exists || value == "" {
            errors = append(errors, fmt.Sprintf("字段 %s 为必填项", field))
        }
    }
    
    // 验证各字段格式
    for field, rule := range pv.rules {
        if value, exists := data[field]; exists && value != "" {
            if valid, err := rule.Validate(value); !valid {
                errors = append(errors, err.Message)
            }
        }
    }
    
    return len(errors) == 0, errors
}

// ValidateUpdate 验证更新数据
func (pv *ProfileValidator) ValidateUpdate(data map[string]interface{}) (bool, []string) {
    var errors []string
    
    // 验证更新的字段
    for field, rule := range pv.rules {
        if value, exists := data[field]; exists && value != "" {
            if valid, err := rule.Validate(value); !valid {
                errors = append(errors, err.Message)
            }
        }
    }
    
    return len(errors) == 0, errors
}

// AccessControl 访问控制
type AccessControl struct{}

// NewAccessControl 创建访问控制器
func NewAccessControl() *AccessControl {
    return &AccessControl{}
}

// CanRead 检查读取权限
func (ac *AccessControl) CanRead(profile *Profile, userID string) bool {
    // 所有者可以读取
    if profile.UserID == userID {
        return true
    }
    
    // 根据隐私设置判断
    switch profile.ProfileVisibility {
    case "public":
        return true
    case "friends":
        // 检查是否为好友关系
        return ac.isFriend(profile.UserID, userID)
    case "private":
        return false
    default:
        return false
    }
}

// CanWrite 检查写入权限
func (ac *AccessControl) CanWrite(profile *Profile, userID string) bool {
    return profile.UserID == userID
}

// MaskProfile 脱敏档案数据
func (ac *AccessControl) MaskProfile(profile *Profile, userID string) *Profile {
    masked := *profile
    
    // 如果不是所有者，进行脱敏
    if profile.UserID != userID {
        // 根据隐私设置脱敏邮箱
        if profile.EmailVisibility == "private" {
            masked.Email = maskEmail(profile.Email)
        }
        
        // 根据隐私设置脱敏手机号
        if profile.PhoneVisibility == "private" {
            masked.Phone = maskPhone(profile.Phone)
        }
    }
    
    return &masked
}

// 辅助函数
func generateProfileID() string {
    return "profile_" + time.Now().Format("20060102150405")
}

func maskEmail(email string) string {
    if email == "" {
        return ""
    }
    
    parts := strings.Split(email, "@")
    if len(parts) != 2 {
        return email
    }
    
    username, domain := parts[0], parts[1]
    if len(username) <= 2 {
        return email
    }
    
    maskedUsername := string(username[0]) + strings.Repeat("*", len(username)-2) + string(username[len(username)-1])
    return maskedUsername + "@" + domain
}

func maskPhone(phone string) string {
    if phone == "" || len(phone) <= 4 {
        return phone
    }
    
    return phone[:2] + strings.Repeat("*", len(phone)-4) + phone[len(phone)-2:]
}

func (ac *AccessControl) isFriend(userID1, userID2 string) bool {
    // 实现好友关系检查逻辑
    return false
}
```

### 2.2.1.1.5.2 Python实现

```python
from dataclasses import dataclass, field
from typing import Dict, List, Optional, Any
from datetime import datetime, date
import re

@dataclass
class Profile:
    id: str
    user_id: str
    username: str
    email: str
    first_name: str
    last_name: str
    display_name: str = ""
    phone: Optional[str] = None
    gender: Optional[str] = None
    birth_date: Optional[date] = None
    country: Optional[str] = None
    region: Optional[str] = None
    city: Optional[str] = None
    timezone: str = "UTC"
    primary_language: str = "zh-CN"
    secondary_languages: List[str] = field(default_factory=list)
    date_format: str = "YYYY-MM-DD"
    time_format: str = "24h"
    occupation: Optional[str] = None
    company: Optional[str] = None
    job_title: Optional[str] = None
    industry: Optional[str] = None
    bio: Optional[str] = None
    avatar: Optional[str] = None
    cover_image: Optional[str] = None
    profile_visibility: str = "public"
    email_visibility: str = "private"
    phone_visibility: str = "private"
    created_at: datetime = field(default_factory=datetime.now)
    updated_at: datetime = field(default_factory=datetime.now)

class ProfileService:
    def __init__(self):
        self.profiles: Dict[str, Profile] = {}
        self.validator = ProfileValidator()
        self.access_control = AccessControl()
    
    def create_profile(self, user_id: str, data: Dict[str, Any]) -> Profile:
        """创建档案"""
        # 验证数据
        is_valid, errors = self.validator.validate_profile(data)
        if not is_valid:
            raise ValidationError(errors)
        
        profile = Profile(
            id=self._generate_profile_id(),
            user_id=user_id,
            username=data["username"],
            email=data["email"],
            first_name=data["first_name"],
            last_name=data["last_name"],
            display_name=data.get("display_name", ""),
            phone=data.get("phone"),
            timezone=data.get("timezone", "UTC"),
            primary_language=data.get("primary_language", "zh-CN")
        )
        
        self.profiles[profile.id] = profile
        return profile
    
    def update_profile(self, profile_id: str, user_id: str, data: Dict[str, Any]) -> Profile:
        """更新档案"""
        if profile_id not in self.profiles:
            raise ProfileNotFoundError()
        
        profile = self.profiles[profile_id]
        
        # 检查权限
        if not self.access_control.can_write(profile, user_id):
            raise AccessDeniedError()
        
        # 验证更新数据
        is_valid, errors = self.validator.validate_update(data)
        if not is_valid:
            raise ValidationError(errors)
        
        # 更新字段
        for key, value in data.items():
            if hasattr(profile, key):
                setattr(profile, key, value)
        
        profile.updated_at = datetime.now()
        return profile
    
    def get_profile(self, profile_id: str, user_id: str) -> Profile:
        """获取档案"""
        if profile_id not in self.profiles:
            raise ProfileNotFoundError()
        
        profile = self.profiles[profile_id]
        
        # 检查访问权限
        if not self.access_control.can_read(profile, user_id):
            raise AccessDeniedError()
        
        # 根据权限脱敏数据
        return self.access_control.mask_profile(profile, user_id)
    
    def _generate_profile_id(self) -> str:
        """生成档案ID"""
        return f"profile_{datetime.now().strftime('%Y%m%d%H%M%S')}"

class ProfileValidator:
    def __init__(self):
        self.rules = {
            "email": EmailRule(),
            "phone": PhoneRule(),
            "birth_date": BirthDateRule(),
        }
    
    def validate_profile(self, data: Dict[str, Any]) -> tuple[bool, List[str]]:
        """验证档案"""
        errors = []
        
        # 检查必填字段
        required_fields = ["username", "email", "first_name", "last_name"]
        for field in required_fields:
            if not data.get(field):
                errors.append(f"字段 {field} 为必填项")
        
        # 验证各字段格式
        for field, rule in self.rules.items():
            if field in data and data[field]:
                if not rule.validate(data[field]):
                    errors.append(rule.get_error_message())
        
        return len(errors) == 0, errors
    
    def validate_update(self, data: Dict[str, Any]) -> tuple[bool, List[str]]:
        """验证更新数据"""
        errors = []
        
        # 验证更新的字段
        for field, rule in self.rules.items():
            if field in data and data[field]:
                if not rule.validate(data[field]):
                    errors.append(rule.get_error_message())
        
        return len(errors) == 0, errors

class ValidationRule:
    def validate(self, value: Any) -> bool:
        """验证字段"""
        raise NotImplementedError
    
    def get_error_message(self) -> str:
        """获取错误消息"""
        raise NotImplementedError

class EmailRule(ValidationRule):
    def validate(self, value: str) -> bool:
        """验证邮箱"""
        return bool(re.match(r'^[^@]+@[^@]+\.[^@]+$', value))
    
    def get_error_message(self) -> str:
        return "邮箱格式不正确"

class PhoneRule(ValidationRule):
    def validate(self, value: str) -> bool:
        """验证手机号"""
        return bool(re.match(r'^\+?[1-9]\d{1,14}$', value))
    
    def get_error_message(self) -> str:
        return "手机号格式不正确"

class BirthDateRule(ValidationRule):
    def validate(self, value: Any) -> bool:
        """验证出生日期"""
        try:
            if isinstance(value, str):
                datetime.strptime(value, "%Y-%m-%d")
            elif isinstance(value, date):
                return True
            return True
        except ValueError:
            return False
    
    def get_error_message(self) -> str:
        return "出生日期格式不正确"

class AccessControl:
    def can_read(self, profile: Profile, user_id: str) -> bool:
        """检查读取权限"""
        # 所有者可以读取
        if profile.user_id == user_id:
            return True
        
        # 根据隐私设置判断
        if profile.profile_visibility == "public":
            return True
        elif profile.profile_visibility == "friends":
            return self._is_friend(profile.user_id, user_id)
        elif profile.profile_visibility == "private":
            return False
        
        return False
    
    def can_write(self, profile: Profile, user_id: str) -> bool:
        """检查写入权限"""
        return profile.user_id == user_id
    
    def mask_profile(self, profile: Profile, user_id: str) -> Profile:
        """脱敏档案数据"""
        masked = Profile(
            id=profile.id,
            user_id=profile.user_id,
            username=profile.username,
            email=profile.email,
            first_name=profile.first_name,
            last_name=profile.last_name,
            display_name=profile.display_name,
            phone=profile.phone,
            gender=profile.gender,
            birth_date=profile.birth_date,
            country=profile.country,
            region=profile.region,
            city=profile.city,
            timezone=profile.timezone,
            primary_language=profile.primary_language,
            secondary_languages=profile.secondary_languages,
            date_format=profile.date_format,
            time_format=profile.time_format,
            occupation=profile.occupation,
            company=profile.company,
            job_title=profile.job_title,
            industry=profile.industry,
            bio=profile.bio,
            avatar=profile.avatar,
            cover_image=profile.cover_image,
            profile_visibility=profile.profile_visibility,
            email_visibility=profile.email_visibility,
            phone_visibility=profile.phone_visibility,
            created_at=profile.created_at,
            updated_at=profile.updated_at
        )
        
        # 如果不是所有者，进行脱敏
        if profile.user_id != user_id:
            # 根据隐私设置脱敏邮箱
            if profile.email_visibility == "private":
                masked.email = self._mask_email(profile.email)
            
            # 根据隐私设置脱敏手机号
            if profile.phone_visibility == "private":
                masked.phone = self._mask_phone(profile.phone)
        
        return masked
    
    def _mask_email(self, email: str) -> str:
        """脱敏邮箱"""
        if not email:
            return ""
        
        parts = email.split('@')
        if len(parts) != 2:
            return email
        
        username, domain = parts
        if len(username) <= 2:
            return email
        
        masked_username = username[0] + '*' * (len(username) - 2) + username[-1]
        return f"{masked_username}@{domain}"
    
    def _mask_phone(self, phone: str) -> str:
        """脱敏手机号"""
        if not phone or len(phone) <= 4:
            return phone
        
        return phone[:2] + '*' * (len(phone) - 4) + phone[-2:]
    
    def _is_friend(self, user_id1: str, user_id2: str) -> bool:
        """检查是否为好友"""
        # 实现好友关系检查逻辑
        return False

class ValidationError(Exception):
    def __init__(self, errors: List[str]):
        self.errors = errors
        super().__init__("验证失败")

class ProfileNotFoundError(Exception):
    pass

class AccessDeniedError(Exception):
    pass
```

## 2.2.1.1.6 相关性链接

- [返回用户管理模块](../README.md)
- [偏好设置](./preferences.md)
- [隐私控制](./privacy.md)
- [基本信息](../user-info/basic.md)
- [扩展信息](../user-info/extended.md) 