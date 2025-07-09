# 5. 交互控制模块 (Interaction Control Module)

## 5.1 模块概述

### 5.1.1 定义
交互控制模块负责处理用户与系统的交互，包括表单验证、状态管理、错误处理和用户反馈。

**形式化定义：**
```
IC = (E, V, S, H, F, R)
其中：
- E: 事件集合 {e₁, e₂, ..., eₙ}
- V: 验证函数 V: F → {valid, invalid}
- S: 状态函数 S: E → State
- H: 处理函数 H: E × S → S'
- F: 表单集合 {f₁, f₂, ..., fₘ}
- R: 响应函数 R: S → Response
```

### 5.1.2 哲学批判分析

#### 存在论层面
- **交互的本质**：用户交互是客观事件还是主观体验？
- **状态的真实性**：系统状态是否反映真实的系统状态？

#### 认识论层面
- **验证的确定性**：表单验证是否具有绝对确定性？
- **反馈的可靠性**：用户反馈机制是否可靠？

#### 价值论层面
- **用户体验的优化**：如何优化用户交互体验？
- **错误处理的友好性**：如何提供友好的错误处理？

## 5.2 子模块结构

### 5.2.1 表单验证 (form-validation/)
- [验证规则](./form-validation/rules.md)
- [验证流程](./form-validation/process.md)
- [错误提示](./form-validation/error-messages.md)

### 5.2.2 状态管理 (state-management/)
- [状态定义](./state-management/definition.md)
- [状态转换](./state-management/transitions.md)
- [状态持久化](./state-management/persistence.md)

### 5.2.3 错误处理 (error-handling/)
- [错误分类](./error-handling/classification.md)
- [错误处理策略](./error-handling/strategies.md)
- [错误恢复](./error-handling/recovery.md)

### 5.2.4 用户反馈 (user-feedback/)
- [反馈收集](./user-feedback/collection.md)
- [反馈分析](./user-feedback/analysis.md)
- [反馈响应](./user-feedback/response.md)

## 5.3 数学证明

### 5.3.1 状态一致性证明

**定理1：状态一致性**
```
对于任意事件 e ∈ E，状态 s ∈ S，如果 H(e, s) = s'，
则 s' 是 s 的有效后继状态
```

**证明：**
1. 假设 H(e, s) = s'
2. 根据状态转换函数定义，H 保持状态一致性
3. s' 是 s 的有效后继状态
4. 证毕

### 5.3.2 验证完整性证明

**定理2：验证完整性**
```
对于任意表单 f ∈ F，如果 V(f) = valid，则 f 满足所有验证规则
```

## 5.4 代码示例

### 5.4.1 Go语言实现

```go
package interactioncontrol

import (
    "regexp"
    "strings"
    "time"
)

// Event 事件结构体
type Event struct {
    ID        string                 `json:"id"`
    Type      string                 `json:"type"`
    Data      map[string]interface{} `json:"data"`
    Timestamp time.Time              `json:"timestamp"`
    UserID    string                 `json:"user_id"`
}

// State 状态结构体
type State struct {
    ID        string                 `json:"id"`
    Name      string                 `json:"name"`
    Data      map[string]interface{} `json:"data"`
    CreatedAt time.Time              `json:"created_at"`
    UpdatedAt time.Time              `json:"updated_at"`
}

// Form 表单结构体
type Form struct {
    ID       string                 `json:"id"`
    Fields   map[string]FormField   `json:"fields"`
    Rules    []ValidationRule       `json:"rules"`
    IsValid  bool                   `json:"is_valid"`
    Errors   []ValidationError      `json:"errors"`
}

// FormField 表单字段
type FormField struct {
    Name     string      `json:"name"`
    Type     string      `json:"type"`
    Value    interface{} `json:"value"`
    Required bool        `json:"required"`
    Pattern  string      `json:"pattern,omitempty"`
}

// ValidationError 验证错误
type ValidationError struct {
    Field   string `json:"field"`
    Message string `json:"message"`
    Code    string `json:"code"`
}

// ValidationRule 验证规则
type ValidationRule interface {
    Validate(field FormField) (bool, ValidationError)
}

// InteractionControlService 交互控制服务
type InteractionControlService struct {
    stateManager *StateManager
    formValidator *FormValidator
    errorHandler  *ErrorHandler
    feedbackCollector *FeedbackCollector
}

// NewInteractionControlService 创建交互控制服务
func NewInteractionControlService() *InteractionControlService {
    return &InteractionControlService{
        stateManager:     NewStateManager(),
        formValidator:    NewFormValidator(),
        errorHandler:     NewErrorHandler(),
        feedbackCollector: NewFeedbackCollector(),
    }
}

// ProcessEvent 处理事件
func (ics *InteractionControlService) ProcessEvent(event Event) (*State, error) {
    // 获取当前状态
    currentState := ics.stateManager.GetCurrentState()
    
    // 处理事件
    newState, err := ics.stateManager.Transition(currentState, event)
    if err != nil {
        return nil, ics.errorHandler.HandleError(err)
    }
    
    // 收集反馈
    ics.feedbackCollector.CollectFeedback(event, newState)
    
    return newState, nil
}

// ValidateForm 验证表单
func (ics *InteractionControlService) ValidateForm(form Form) (bool, []ValidationError) {
    return ics.formValidator.Validate(form)
}

// StateManager 状态管理器
type StateManager struct {
    states map[string]*State
    currentStateID string
}

// NewStateManager 创建状态管理器
func NewStateManager() *StateManager {
    return &StateManager{
        states: make(map[string]*State),
    }
}

// GetCurrentState 获取当前状态
func (sm *StateManager) GetCurrentState() *State {
    if sm.currentStateID == "" {
        return nil
    }
    return sm.states[sm.currentStateID]
}

// Transition 状态转换
func (sm *StateManager) Transition(currentState *State, event Event) (*State, error) {
    // 根据事件类型和当前状态确定新状态
    newState := sm.calculateNewState(currentState, event)
    
    if newState == nil {
        return nil, ErrInvalidStateTransition
    }
    
    // 更新状态
    sm.currentStateID = newState.ID
    sm.states[newState.ID] = newState
    
    return newState, nil
}

// calculateNewState 计算新状态
func (sm *StateManager) calculateNewState(currentState *State, event Event) *State {
    // 根据事件类型和当前状态计算新状态
    // 这里简化处理，实际应用中需要更复杂的状态机逻辑
    
    newState := &State{
        ID:        generateStateID(),
        Name:      event.Type + "_state",
        Data:      event.Data,
        CreatedAt: time.Now(),
        UpdatedAt: time.Now(),
    }
    
    return newState
}

// FormValidator 表单验证器
type FormValidator struct {
    rules []ValidationRule
}

// NewFormValidator 创建表单验证器
func NewFormValidator() *FormValidator {
    return &FormValidator{
        rules: []ValidationRule{
            &RequiredRule{},
            &PatternRule{},
            &LengthRule{},
        },
    }
}

// Validate 验证表单
func (fv *FormValidator) Validate(form Form) (bool, []ValidationError) {
    var errors []ValidationError
    
    for fieldName, field := range form.Fields {
        for _, rule := range fv.rules {
            if valid, err := rule.Validate(field); !valid {
                err.Field = fieldName
                errors = append(errors, err)
            }
        }
    }
    
    return len(errors) == 0, errors
}

// RequiredRule 必填规则
type RequiredRule struct{}

func (rr *RequiredRule) Validate(field FormField) (bool, ValidationError) {
    if field.Required {
        if field.Value == nil || field.Value == "" {
            return false, ValidationError{
                Message: "此字段为必填项",
                Code:    "REQUIRED",
            }
        }
    }
    return true, ValidationError{}
}

// PatternRule 模式规则
type PatternRule struct{}

func (pr *PatternRule) Validate(field FormField) (bool, ValidationError) {
    if field.Pattern != "" && field.Value != nil {
        strValue := toString(field.Value)
        matched, _ := regexp.MatchString(field.Pattern, strValue)
        if !matched {
            return false, ValidationError{
                Message: "格式不正确",
                Code:    "PATTERN_MISMATCH",
            }
        }
    }
    return true, ValidationError{}
}

// LengthRule 长度规则
type LengthRule struct{}

func (lr *LengthRule) Validate(field FormField) (bool, ValidationError) {
    if field.Value != nil {
        strValue := toString(field.Value)
        if len(strValue) > 1000 { // 最大长度限制
            return false, ValidationError{
                Message: "内容过长",
                Code:    "LENGTH_EXCEEDED",
            }
        }
    }
    return true, ValidationError{}
}

// ErrorHandler 错误处理器
type ErrorHandler struct {
    errorStrategies map[string]ErrorStrategy
}

// NewErrorHandler 创建错误处理器
func NewErrorHandler() *ErrorHandler {
    return &ErrorHandler{
        errorStrategies: map[string]ErrorStrategy{
            "validation": &ValidationErrorStrategy{},
            "system":     &SystemErrorStrategy{},
            "network":    &NetworkErrorStrategy{},
        },
    }
}

// HandleError 处理错误
func (eh *ErrorHandler) HandleError(err error) error {
    // 根据错误类型选择处理策略
    errorType := eh.classifyError(err)
    strategy := eh.errorStrategies[errorType]
    
    return strategy.Handle(err)
}

// classifyError 分类错误
func (eh *ErrorHandler) classifyError(err error) string {
    // 根据错误信息分类
    errMsg := err.Error()
    if strings.Contains(errMsg, "validation") {
        return "validation"
    } else if strings.Contains(errMsg, "network") {
        return "network"
    } else {
        return "system"
    }
}

// ErrorStrategy 错误处理策略
type ErrorStrategy interface {
    Handle(err error) error
}

// ValidationErrorStrategy 验证错误策略
type ValidationErrorStrategy struct{}

func (ves *ValidationErrorStrategy) Handle(err error) error {
    // 处理验证错误
    return err
}

// SystemErrorStrategy 系统错误策略
type SystemErrorStrategy struct{}

func (ses *SystemErrorStrategy) Handle(err error) error {
    // 处理系统错误
    return err
}

// NetworkErrorStrategy 网络错误策略
type NetworkErrorStrategy struct{}

func (nes *NetworkErrorStrategy) Handle(err error) error {
    // 处理网络错误
    return err
}

// FeedbackCollector 反馈收集器
type FeedbackCollector struct {
    feedbacks []Feedback
}

// NewFeedbackCollector 创建反馈收集器
func NewFeedbackCollector() *FeedbackCollector {
    return &FeedbackCollector{
        feedbacks: []Feedback{},
    }
}

// Feedback 反馈结构体
type Feedback struct {
    ID        string    `json:"id"`
    UserID    string    `json:"user_id"`
    EventID   string    `json:"event_id"`
    Type      string    `json:"type"`
    Content   string    `json:"content"`
    Rating    int       `json:"rating"`
    CreatedAt time.Time `json:"created_at"`
}

// CollectFeedback 收集反馈
func (fc *FeedbackCollector) CollectFeedback(event Event, state *State) {
    feedback := Feedback{
        ID:        generateFeedbackID(),
        UserID:    event.UserID,
        EventID:   event.ID,
        Type:      "interaction",
        Content:   "用户交互反馈",
        Rating:    5, // 默认评分
        CreatedAt: time.Now(),
    }
    
    fc.feedbacks = append(fc.feedbacks, feedback)
}

// 辅助函数
func generateStateID() string {
    return "state_" + time.Now().Format("20060102150405")
}

func generateFeedbackID() string {
    return "feedback_" + time.Now().Format("20060102150405")
}

func toString(value interface{}) string {
    if str, ok := value.(string); ok {
        return str
    }
    return ""
}
```

### 5.4.2 Python实现

```python
from dataclasses import dataclass, field
from typing import Dict, List, Optional, Any
from datetime import datetime
import re
import uuid

@dataclass
class Event:
    id: str
    type: str
    data: Dict[str, Any] = field(default_factory=dict)
    timestamp: datetime = field(default_factory=datetime.now)
    user_id: str = ""

@dataclass
class State:
    id: str
    name: str
    data: Dict[str, Any] = field(default_factory=dict)
    created_at: datetime = field(default_factory=datetime.now)
    updated_at: datetime = field(default_factory=datetime.now)

@dataclass
class FormField:
    name: str
    type: str
    value: Any = None
    required: bool = False
    pattern: str = ""

@dataclass
class Form:
    id: str
    fields: Dict[str, FormField] = field(default_factory=dict)
    rules: List[str] = field(default_factory=list)
    is_valid: bool = False
    errors: List[str] = field(default_factory=list)

@dataclass
class ValidationError:
    field: str = ""
    message: str = ""
    code: str = ""

@dataclass
class Feedback:
    id: str
    user_id: str
    event_id: str
    type: str
    content: str
    rating: int
    created_at: datetime = field(default_factory=datetime.now)

class InteractionControlService:
    def __init__(self):
        self.state_manager = StateManager()
        self.form_validator = FormValidator()
        self.error_handler = ErrorHandler()
        self.feedback_collector = FeedbackCollector()
    
    def process_event(self, event: Event) -> Optional[State]:
        """处理事件"""
        try:
            # 获取当前状态
            current_state = self.state_manager.get_current_state()
            
            # 处理事件
            new_state = self.state_manager.transition(current_state, event)
            
            # 收集反馈
            self.feedback_collector.collect_feedback(event, new_state)
            
            return new_state
        except Exception as e:
            self.error_handler.handle_error(e)
            return None
    
    def validate_form(self, form: Form) -> tuple[bool, List[ValidationError]]:
        """验证表单"""
        return self.form_validator.validate(form)

class StateManager:
    def __init__(self):
        self.states: Dict[str, State] = {}
        self.current_state_id: str = ""
    
    def get_current_state(self) -> Optional[State]:
        """获取当前状态"""
        if not self.current_state_id:
            return None
        return self.states.get(self.current_state_id)
    
    def transition(self, current_state: Optional[State], event: Event) -> State:
        """状态转换"""
        # 根据事件类型和当前状态确定新状态
        new_state = self._calculate_new_state(current_state, event)
        
        if new_state is None:
            raise ValueError("无效的状态转换")
        
        # 更新状态
        self.current_state_id = new_state.id
        self.states[new_state.id] = new_state
        
        return new_state
    
    def _calculate_new_state(self, current_state: Optional[State], event: Event) -> State:
        """计算新状态"""
        new_state = State(
            id=str(uuid.uuid4()),
            name=f"{event.type}_state",
            data=event.data
        )
        
        return new_state

class FormValidator:
    def __init__(self):
        self.rules = [
            RequiredRule(),
            PatternRule(),
            LengthRule()
        ]
    
    def validate(self, form: Form) -> tuple[bool, List[ValidationError]]:
        """验证表单"""
        errors = []
        
        for field_name, field in form.fields.items():
            for rule in self.rules:
                if not rule.validate(field):
                    errors.append(ValidationError(
                        field=field_name,
                        message=rule.get_error_message(),
                        code=rule.get_error_code()
                    ))
        
        return len(errors) == 0, errors

class ValidationRule:
    def validate(self, field: FormField) -> bool:
        """验证字段"""
        raise NotImplementedError
    
    def get_error_message(self) -> str:
        """获取错误消息"""
        raise NotImplementedError
    
    def get_error_code(self) -> str:
        """获取错误代码"""
        raise NotImplementedError

class RequiredRule(ValidationRule):
    def validate(self, field: FormField) -> bool:
        """验证必填字段"""
        if field.required:
            return field.value is not None and field.value != ""
        return True
    
    def get_error_message(self) -> str:
        return "此字段为必填项"
    
    def get_error_code(self) -> str:
        return "REQUIRED"

class PatternRule(ValidationRule):
    def validate(self, field: FormField) -> bool:
        """验证模式"""
        if field.pattern and field.value is not None:
            str_value = str(field.value)
            return bool(re.match(field.pattern, str_value))
        return True
    
    def get_error_message(self) -> str:
        return "格式不正确"
    
    def get_error_code(self) -> str:
        return "PATTERN_MISMATCH"

class LengthRule(ValidationRule):
    def validate(self, field: FormField) -> bool:
        """验证长度"""
        if field.value is not None:
            str_value = str(field.value)
            return len(str_value) <= 1000  # 最大长度限制
        return True
    
    def get_error_message(self) -> str:
        return "内容过长"
    
    def get_error_code(self) -> str:
        return "LENGTH_EXCEEDED"

class ErrorHandler:
    def __init__(self):
        self.error_strategies = {
            "validation": ValidationErrorStrategy(),
            "system": SystemErrorStrategy(),
            "network": NetworkErrorStrategy()
        }
    
    def handle_error(self, error: Exception) -> Exception:
        """处理错误"""
        error_type = self._classify_error(error)
        strategy = self.error_strategies.get(error_type, SystemErrorStrategy())
        
        return strategy.handle(error)
    
    def _classify_error(self, error: Exception) -> str:
        """分类错误"""
        error_msg = str(error).lower()
        if "validation" in error_msg:
            return "validation"
        elif "network" in error_msg:
            return "network"
        else:
            return "system"

class ErrorStrategy:
    def handle(self, error: Exception) -> Exception:
        """处理错误"""
        raise NotImplementedError

class ValidationErrorStrategy(ErrorStrategy):
    def handle(self, error: Exception) -> Exception:
        # 处理验证错误
        return error

class SystemErrorStrategy(ErrorStrategy):
    def handle(self, error: Exception) -> Exception:
        # 处理系统错误
        return error

class NetworkErrorStrategy(ErrorStrategy):
    def handle(self, error: Exception) -> Exception:
        # 处理网络错误
        return error

class FeedbackCollector:
    def __init__(self):
        self.feedbacks: List[Feedback] = []
    
    def collect_feedback(self, event: Event, state: State) -> None:
        """收集反馈"""
        feedback = Feedback(
            id=str(uuid.uuid4()),
            user_id=event.user_id,
            event_id=event.id,
            type="interaction",
            content="用户交互反馈",
            rating=5  # 默认评分
        )
        
        self.feedbacks.append(feedback)
    
    def get_feedbacks(self) -> List[Feedback]:
        """获取反馈列表"""
        return self.feedbacks
```

## 5.5 相关性链接

- [返回主目录](../README.md)
- [用户认证模块](../authentication/README.md)
- [用户管理模块](../user-management/README.md)
- [支付系统模块](../payment/README.md)
- [数据展示模块](../data-presentation/README.md) 