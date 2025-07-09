# 1.2.1 注册表单设计 (Registration Form Design)

## 1.2.1.1 设计概述

### 1.2.1.1.1 定义
注册表单是用户进入系统的第一个交互界面，负责收集用户基本信息并创建用户账户。

**形式化定义：**
```
RF = (F, V, S, U, C)
其中：
- F: 表单字段集合 {f₁, f₂, ..., fₙ}
- V: 验证规则集合 {v₁, v₂, ..., vₘ}
- S: 提交状态集合 {pending, success, failed}
- U: 用户输入集合
- C: 约束条件集合
```

### 1.2.1.1.2 设计原则

#### 存在论层面
- **信息收集的必要性**：哪些信息是系统运行必需的？
- **用户隐私的保护**：如何在收集信息与保护隐私间平衡？

#### 认识论层面
- **表单设计的可用性**：如何确保用户能够正确填写表单？
- **验证反馈的及时性**：如何提供即时的验证反馈？

#### 价值论层面
- **用户体验的优化**：如何减少用户的认知负担？
- **转化率的提升**：如何提高注册成功率？

## 1.2.1.2 表单字段设计

### 1.2.1.2.1 必填字段

#### 用户名 (Username)
```typescript
interface UsernameField {
  name: "username";
  type: "text";
  required: true;
  minLength: 3;
  maxLength: 20;
  pattern: "^[a-zA-Z0-9_]+$";
  validation: {
    unique: true;
    format: "alphanumeric_underscore";
  };
}
```

#### 邮箱 (Email)
```typescript
interface EmailField {
  name: "email";
  type: "email";
  required: true;
  pattern: "^[^@]+@[^@]+\\.[^@]+$";
  validation: {
    unique: true;
    format: "email";
    domain: "valid_domains";
  };
}
```

#### 密码 (Password)
```typescript
interface PasswordField {
  name: "password";
  type: "password";
  required: true;
  minLength: 8;
  maxLength: 128;
  pattern: "^(?=.*[a-z])(?=.*[A-Z])(?=.*\\d)(?=.*[@$!%*?&])[A-Za-z\\d@$!%*?&]{8,}$";
  validation: {
    strength: "strong";
    complexity: {
      lowercase: true;
      uppercase: true;
      numbers: true;
      symbols: true;
    };
  };
}
```

### 1.2.1.2.2 可选字段

#### 手机号码 (Phone)
```typescript
interface PhoneField {
  name: "phone";
  type: "tel";
  required: false;
  pattern: "^\\+?[1-9]\\d{1,14}$";
  validation: {
    format: "international";
    country: "auto_detect";
  };
}
```

#### 验证码 (Verification Code)
```typescript
interface VerificationCodeField {
  name: "verification_code";
  type: "text";
  required: true;
  length: 6;
  pattern: "^\\d{6}$";
  validation: {
    format: "numeric";
    expiry: "5_minutes";
  };
}
```

## 1.2.1.3 验证规则设计

### 1.2.1.3.1 实时验证

**定理1：实时验证的及时性**
```
对于任意字段 f ∈ F，如果用户输入发生变化，
则验证结果应在 300ms 内返回
```

**证明：**
1. 假设用户输入发生变化
2. 触发验证事件
3. 验证函数执行时间 ≤ 300ms
4. 返回验证结果
5. 证毕

### 1.2.1.3.2 强度验证

**定理2：密码强度验证**
```
对于任意密码 p，如果 strength(p) ≥ 0.8，
则 p 满足强密码要求
```

**强度计算算法：**
```python
def calculate_password_strength(password: str) -> float:
    score = 0.0
    
    # 长度分数 (0-25分)
    if len(password) >= 8:
        score += min(25, len(password) * 2)
    
    # 字符类型分数 (0-25分)
    if re.search(r'[a-z]', password):
        score += 5
    if re.search(r'[A-Z]', password):
        score += 5
    if re.search(r'\d', password):
        score += 5
    if re.search(r'[@$!%*?&]', password):
        score += 10
    
    # 复杂度分数 (0-50分)
    unique_chars = len(set(password))
    score += min(50, unique_chars * 2)
    
    return min(100, score) / 100
```

## 1.2.1.4 用户体验设计

### 1.2.1.4.1 渐进式披露

```typescript
interface ProgressiveDisclosure {
  steps: [
    {
      id: "basic_info";
      fields: ["username", "email"];
      validation: "real_time";
    },
    {
      id: "security_setup";
      fields: ["password", "confirm_password"];
      validation: "on_blur";
    },
    {
      id: "verification";
      fields: ["verification_code"];
      validation: "on_submit";
    }
  ];
}
```

### 1.2.1.4.2 错误处理

```typescript
interface ErrorHandling {
  display: {
    position: "inline";
    style: "error_message";
    timing: "immediate";
  };
  recovery: {
    suggestion: "auto_suggest";
    correction: "auto_correct";
    help: "contextual_help";
  };
}
```

## 1.2.1.5 代码实现

### 1.2.1.5.1 Go语言实现

```go
package registration

import (
    "regexp"
    "time"
)

// RegistrationForm 注册表单
type RegistrationForm struct {
    Username         string    `json:"username" validate:"required,min=3,max=20,alphanum"`
    Email            string    `json:"email" validate:"required,email"`
    Password         string    `json:"password" validate:"required,min=8"`
    ConfirmPassword  string    `json:"confirm_password" validate:"required,eqfield=Password"`
    Phone            string    `json:"phone" validate:"omitempty,phone"`
    VerificationCode string    `json:"verification_code" validate:"required,len=6,numeric"`
    CreatedAt        time.Time `json:"created_at"`
}

// FormValidator 表单验证器
type FormValidator struct {
    rules map[string]ValidationRule
}

// NewFormValidator 创建表单验证器
func NewFormValidator() *FormValidator {
    return &FormValidator{
        rules: map[string]ValidationRule{
            "username": &UsernameRule{},
            "email":    &EmailRule{},
            "password": &PasswordRule{},
            "phone":    &PhoneRule{},
        },
    }
}

// ValidateForm 验证表单
func (fv *FormValidator) ValidateForm(form RegistrationForm) (bool, []ValidationError) {
    var errors []ValidationError
    
    // 验证用户名
    if valid, err := fv.rules["username"].Validate(form.Username); !valid {
        errors = append(errors, err)
    }
    
    // 验证邮箱
    if valid, err := fv.rules["email"].Validate(form.Email); !valid {
        errors = append(errors, err)
    }
    
    // 验证密码
    if valid, err := fv.rules["password"].Validate(form.Password); !valid {
        errors = append(errors, err)
    }
    
    // 验证确认密码
    if form.Password != form.ConfirmPassword {
        errors = append(errors, ValidationError{
            Field:   "confirm_password",
            Message: "密码确认不匹配",
            Code:    "PASSWORD_MISMATCH",
        })
    }
    
    return len(errors) == 0, errors
}

// UsernameRule 用户名验证规则
type UsernameRule struct{}

func (ur *UsernameRule) Validate(value string) (bool, ValidationError) {
    if len(value) < 3 {
        return false, ValidationError{
            Message: "用户名至少需要3个字符",
            Code:    "USERNAME_TOO_SHORT",
        }
    }
    
    if len(value) > 20 {
        return false, ValidationError{
            Message: "用户名不能超过20个字符",
            Code:    "USERNAME_TOO_LONG",
        }
    }
    
    matched, _ := regexp.MatchString("^[a-zA-Z0-9_]+$", value)
    if !matched {
        return false, ValidationError{
            Message: "用户名只能包含字母、数字和下划线",
            Code:    "USERNAME_INVALID_CHARS",
        }
    }
    
    return true, ValidationError{}
}

// EmailRule 邮箱验证规则
type EmailRule struct{}

func (er *EmailRule) Validate(value string) (bool, ValidationError) {
    emailRegex := regexp.MustCompile(`^[^@]+@[^@]+\.[^@]+$`)
    if !emailRegex.MatchString(value) {
        return false, ValidationError{
            Message: "邮箱格式不正确",
            Code:    "EMAIL_INVALID_FORMAT",
        }
    }
    
    return true, ValidationError{}
}

// PasswordRule 密码验证规则
type PasswordRule struct{}

func (pr *PasswordRule) Validate(value string) (bool, ValidationError) {
    if len(value) < 8 {
        return false, ValidationError{
            Message: "密码至少需要8个字符",
            Code:    "PASSWORD_TOO_SHORT",
        }
    }
    
    // 检查密码强度
    strength := calculatePasswordStrength(value)
    if strength < 0.8 {
        return false, ValidationError{
            Message: "密码强度不够，请包含大小写字母、数字和特殊字符",
            Code:    "PASSWORD_TOO_WEAK",
        }
    }
    
    return true, ValidationError{}
}

// calculatePasswordStrength 计算密码强度
func calculatePasswordStrength(password string) float64 {
    score := 0.0
    
    // 长度分数
    if len(password) >= 8 {
        score += float64(min(25, len(password)*2))
    }
    
    // 字符类型分数
    if regexp.MustCompile(`[a-z]`).MatchString(password) {
        score += 5
    }
    if regexp.MustCompile(`[A-Z]`).MatchString(password) {
        score += 5
    }
    if regexp.MustCompile(`\d`).MatchString(password) {
        score += 5
    }
    if regexp.MustCompile(`[@$!%*?&]`).MatchString(password) {
        score += 10
    }
    
    // 复杂度分数
    charSet := make(map[rune]bool)
    for _, char := range password {
        charSet[char] = true
    }
    score += float64(min(50, len(charSet)*2))
    
    return min(100, score) / 100
}

func min(a, b int) int {
    if a < b {
        return a
    }
    return b
}
```

### 1.2.1.5.2 Python实现

```python
import re
from dataclasses import dataclass
from typing import List, Optional
from datetime import datetime

@dataclass
class RegistrationForm:
    username: str
    email: str
    password: str
    confirm_password: str
    phone: Optional[str] = None
    verification_code: Optional[str] = None
    created_at: datetime = None
    
    def __post_init__(self):
        if self.created_at is None:
            self.created_at = datetime.now()

@dataclass
class ValidationError:
    field: str = ""
    message: str = ""
    code: str = ""

class FormValidator:
    def __init__(self):
        self.rules = {
            "username": UsernameRule(),
            "email": EmailRule(),
            "password": PasswordRule(),
            "phone": PhoneRule(),
        }
    
    def validate_form(self, form: RegistrationForm) -> tuple[bool, List[ValidationError]]:
        """验证表单"""
        errors = []
        
        # 验证用户名
        if not self.rules["username"].validate(form.username):
            errors.append(ValidationError(
                field="username",
                message=self.rules["username"].get_error_message(),
                code=self.rules["username"].get_error_code()
            ))
        
        # 验证邮箱
        if not self.rules["email"].validate(form.email):
            errors.append(ValidationError(
                field="email",
                message=self.rules["email"].get_error_message(),
                code=self.rules["email"].get_error_code()
            ))
        
        # 验证密码
        if not self.rules["password"].validate(form.password):
            errors.append(ValidationError(
                field="password",
                message=self.rules["password"].get_error_message(),
                code=self.rules["password"].get_error_code()
            ))
        
        # 验证确认密码
        if form.password != form.confirm_password:
            errors.append(ValidationError(
                field="confirm_password",
                message="密码确认不匹配",
                code="PASSWORD_MISMATCH"
            ))
        
        return len(errors) == 0, errors

class ValidationRule:
    def validate(self, value: str) -> bool:
        """验证字段"""
        raise NotImplementedError
    
    def get_error_message(self) -> str:
        """获取错误消息"""
        raise NotImplementedError
    
    def get_error_code(self) -> str:
        """获取错误代码"""
        raise NotImplementedError

class UsernameRule(ValidationRule):
    def validate(self, value: str) -> bool:
        """验证用户名"""
        if len(value) < 3:
            return False
        
        if len(value) > 20:
            return False
        
        return bool(re.match(r'^[a-zA-Z0-9_]+$', value))
    
    def get_error_message(self) -> str:
        return "用户名长度应为3-20个字符，只能包含字母、数字和下划线"
    
    def get_error_code(self) -> str:
        return "USERNAME_INVALID"

class EmailRule(ValidationRule):
    def validate(self, value: str) -> bool:
        """验证邮箱"""
        return bool(re.match(r'^[^@]+@[^@]+\.[^@]+$', value))
    
    def get_error_message(self) -> str:
        return "邮箱格式不正确"
    
    def get_error_code(self) -> str:
        return "EMAIL_INVALID"

class PasswordRule(ValidationRule):
    def validate(self, value: str) -> bool:
        """验证密码"""
        if len(value) < 8:
            return False
        
        # 检查密码强度
        strength = self._calculate_strength(value)
        return strength >= 0.8
    
    def _calculate_strength(self, password: str) -> float:
        """计算密码强度"""
        score = 0.0
        
        # 长度分数
        if len(password) >= 8:
            score += min(25, len(password) * 2)
        
        # 字符类型分数
        if re.search(r'[a-z]', password):
            score += 5
        if re.search(r'[A-Z]', password):
            score += 5
        if re.search(r'\d', password):
            score += 5
        if re.search(r'[@$!%*?&]', password):
            score += 10
        
        # 复杂度分数
        unique_chars = len(set(password))
        score += min(50, unique_chars * 2)
        
        return min(100, score) / 100
    
    def get_error_message(self) -> str:
        return "密码至少8个字符，需包含大小写字母、数字和特殊字符"
    
    def get_error_code(self) -> str:
        return "PASSWORD_TOO_WEAK"

class PhoneRule(ValidationRule):
    def validate(self, value: str) -> bool:
        """验证手机号"""
        if not value:  # 可选字段
            return True
        
        return bool(re.match(r'^\+?[1-9]\d{1,14}$', value))
    
    def get_error_message(self) -> str:
        return "手机号格式不正确"
    
    def get_error_code(self) -> str:
        return "PHONE_INVALID"
```

## 1.2.1.6 相关性链接

- [返回认证模块](../README.md)
- [数据验证逻辑](./validation.md)
- [用户协议处理](./agreement.md)
- [登录界面设计](../login/interface.md)
- [密码验证算法](../login/password-verification.md) 