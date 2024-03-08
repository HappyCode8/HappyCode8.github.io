# MiddleWare模式

```go
package main

import (
    "log"
    "net/http"
    "time"
)

// Handler defination
type Handler func(http.ResponseWriter, *http.Request)

// Middleware defination
type Middleware func(Handler) Handler

// 当多个middleware调用时，返回一个函数，这个函数需要输入一个handler,
func Chain(mws ...Middleware) Middleware {
    return func(handler Handler) Handler {
        for _, mw := range mws {
            handler = mw(handler)
        }
        return handler
    }
}

// Logging middleware
func Logging(handler Handler) Handler {
    return func(w http.ResponseWriter, r *http.Request) {
        log.Printf("[logging] %s", r.URL.String())
        handler(w, r)
    }
}

// Time middleware
func Time(handler Handler) Handler {
    return func(w http.ResponseWriter, r *http.Request) {
        start := time.Now()
        handler(w, r)
        log.Printf("[time cost] %s", time.Now().Sub(start).String())
    }
}

func Hello(w http.ResponseWriter, r *http.Request) {
    w.Write([]byte("hello"))
}

func main() {
    chain := Chain(Logging, Time)
    /*    mws=Logging, Time
        handler=Hello
        chain=func(handler Handler) Handler {
            for _, mw := range mws {
                handler = mw(handler)
            }
            return handler
        }

        当mw=Logging时，第一轮循环mw(Hello)以后：
        handler=func(w http.ResponseWriter, r *http.Request) {
            log.Printf("[access] %s", r.URL.String())
            Hello(w, r)
        }

        当mw=Time()时，第二轮循环mw(func(w http.ResponseWriter, r *http.Request) {
                log.Printf("[access] %s", r.URL.String())
                Hello(w, r)
            })以后：
        handler=func(w http.ResponseWriter, r *http.Request) {
            start := time.Now()
            log.Printf("[access] %s", r.URL.String())
            Hello(w, r)
            log.Printf("[time cost] %s", time.Now().Sub(start).String())
        }

        此时结束，然后返回上述handler然后给wr赋值以后触发真正的执行
    */
    http.HandleFunc("/hello", chain(Hello))
    log.Panic(http.ListenAndServe(":8080", nil))
}
```

# 状态机

1. 基本的状态机定义base
   
   ```go
   package statemachine
   
   import (
       "xxxxxxx/motor/bfsm"
   )
   
   // 状态定义
   var (
       InitSt          = bfsm.NewState(0, "init", bfsm.WithStateTypeOption(bfsm.Begin))
       InEditSt        = bfsm.NewState(1, "草稿中")
       WaitEffectiveSt = bfsm.NewState(2, "待生效")
       EffectiveSt     = bfsm.NewState(3, "生效中")
       EffectiveLockSt = bfsm.NewState(4, "生效锁定中")
       ExpiredSt       = bfsm.NewState(5, "已经过期", bfsm.WithStateTypeOption(bfsm.End))
       CancelSt        = bfsm.NewState(6, "已经取消", bfsm.WithStateTypeOption(bfsm.Cancel))
       AutoCancelSt    = bfsm.NewState(7, "自动作废", bfsm.WithStateTypeOption(bfsm.Cancel))
   )
   
   // 动作定义
   var (
       CreateNewPolicyEt        = bfsm.NewEvent("Create", "新建")
       CreateNewPolicyVersionEt = bfsm.NewEvent("CreateVersion", "新增变更版本")
       SubmitModifyEt           = bfsm.NewEvent("SubmitModify", "提交修改")
       BeginEffectEt            = bfsm.NewEvent("BeginEffect", "开始生效")
       EndEffectEt              = bfsm.NewEvent("EndEffect", "结束生效")
       FreezePolicyEt           = bfsm.NewEvent("FreezePolicy", "冻结政策")
       UnFreezePolicyEt         = bfsm.NewEvent("UnFreezePolicy", "解冻")
       DirectCancelPolicyEt     = bfsm.NewEvent("DirectCancel", "直接作废")
       AutoCancelPolicyEt       = bfsm.NewEvent("AutoCancelPolicy", "自动作废")
   )
   
   const (
       PolicySMName            = "policy"
       BeforeEffectiveTimeCond = "beforeEffectiveTime"
       InEffectiveTime         = "inEffectiveTime"
       AfterEffectiveTime      = "AfterEffectiveTime"
   )
   
   func InitStateMachine() {
       // 初始化
       if err := bfsm.Init(stateMachineConfig, nil); err != nil {
           panic(err)
       }
   }
   
   var stateMachineConfig = map[string]bfsm.BizDesc{
       PolicySMName: PolicySM,
   }
   
   var PolicySM = bfsm.BizDesc{
       TransDescV2List: []bfsm.TransDescV2{
           // 新建
           {
               Event: CreateNewPolicyEt,
               Src:   []bfsm.State{InitSt},
               Matchers: []bfsm.MatcherV2{
                   {
                       Dst:          InEditSt,
                       IsMasterFlow: true,
                   },
               },
           },
           // 变更政策(新增版本)
           {
               Event: CreateNewPolicyVersionEt,
               Src:   []bfsm.State{InitSt},
               Matchers: []bfsm.MatcherV2{
                   {
                       Dst:          InEditSt,
                       IsMasterFlow: true,
                   },
               },
           },
           // 提交修改
           {
               Event: SubmitModifyEt,
               Src:   []bfsm.State{InEditSt},
               Matchers: []bfsm.MatcherV2{
                   {
                       Dst:       WaitEffectiveSt,
                       Condition: "beforeEffectiveTime",
                   },
                   {
                       Dst:       WaitEffectiveSt,
                       Condition: "inEffectiveTime",
                   },
                   {
                       Dst:       WaitEffectiveSt,
                       Condition: "AfterEffectiveTime",
                   },
               },
           },
           // 开始生效
           {
               Event: BeginEffectEt,
               Src:   []bfsm.State{WaitEffectiveSt},
               Matchers: []bfsm.MatcherV2{
                   {
                       Dst:          EffectiveSt,
                       IsMasterFlow: true,
                   },
               },
           },
           // 冻结政策
           {
               Event: FreezePolicyEt,
               Src:   []bfsm.State{EffectiveSt},
               Matchers: []bfsm.MatcherV2{
                   {
                       Dst: EffectiveLockSt,
                   },
               },
           },
           // 解冻结政策
           {
               Event: UnFreezePolicyEt,
               Src:   []bfsm.State{EffectiveLockSt},
               Matchers: []bfsm.MatcherV2{
                   {
                       Dst:       ExpiredSt,
                       Condition: "AfterEffectiveTime",
                   },
                   {
                       Dst:       EffectiveSt,
                       Condition: "inEffectiveTime",
                   },
               },
           },
           // 结束生效
           {
               Event: EndEffectEt,
               Src:   []bfsm.State{EffectiveSt},
               Matchers: []bfsm.MatcherV2{
                   {
                       Dst:          ExpiredSt,
                       IsMasterFlow: true,
                   },
               },
           },
           // 直接作废
           {
               Event: DirectCancelPolicyEt,
               Src:   []bfsm.State{WaitEffectiveSt, EffectiveSt},
               Matchers: []bfsm.MatcherV2{
                   {
                       Dst: CancelSt,
                   },
               },
           },
           // 自动作废
           {
               Event: AutoCancelPolicyEt,
               Src:   []bfsm.State{WaitEffectiveSt, EffectiveSt, EffectiveLockSt},
               Matchers: []bfsm.MatcherV2{
                   {
                       Dst: AutoCancelSt,
                   },
               },
           },
       },
   }
   ```

2. 生成状态机图以及测试状态机
   
   ```go
   package statemachine
   
   import (
       "xxxxxxx/motor/bfsm"
       "context"
       "fmt"
       "log"
       "os"
       "os/exec"
       "strings"
       "testing"
   )
   
   func TestGraph(t *testing.T) {
       _ = PolicySM.ExportDOTV2("./dots/test", 0)
       files, err := os.ReadDir("./dots")
       if err != nil {
           log.Fatal(err)
       }
   
       for _, f := range files {
           fileNameWithoutSuffix, _ := strings.CutSuffix(f.Name(), ".dot")
           if strings.HasSuffix(f.Name(), ".dot") {
               exec.Command("dot",
                   fmt.Sprintf("./dots/%s", f.Name()),
                   "-Tpng",
                   "-o",
                   fmt.Sprintf("./png/%s.png", fileNameWithoutSuffix),
               ).Run()
           }
       }
   
   }
   
   type testExecution struct {
       *BaseExecution
       stateMachine *bfsm.FSM
   }
   
   func NewTestExecution(ctx context.Context) *testExecution {
       return &testExecution{
           BaseExecution: NewBaseExecution(ctx, SubmitModifyEt.Value()),
       }
   }
   
   func MakeCondition() map[string]interface{} {
       var res = make(map[string]interface{})
       res[BeforeEffectiveTimeCond] = true
       res[AfterEffectiveTime] = true
       res[InEffectiveTime] = true
       return res
   }
   
   func (e *testExecution) Init(ctx context.Context) error {
       InitStateMachine()
       fsm, err := bfsm.NewFSM(PolicySMName, 1, nil)
       if err != nil {
           fmt.Println("error init")
       }
       fmt.Println("当前状态是：", fsm.CurState())
       fmt.Println("当前状态能否进行到下一个状态：", fsm.Can(SubmitModifyEt.Value()))
       fsm.Fire(ctx, SubmitModifyEt.Value(), MakeCondition())
       fmt.Println("转换后的状态是：", fsm.CurState())
       err = fsm.Fire(ctx, SubmitModifyEt.Value(), MakeCondition())
       if err != nil {
           fmt.Println("转换不成，", err.Error())
       }
       return nil
   }
   
   func TestStateMachine(t *testing.T) {
       ctx := context.Background()
       execution := NewTestExecution(ctx)
       execution.Init(ctx)
   }
   ```

3. 接口
   
   ```go
   package statemachine
   
   import (
       "context"
       "time"
   )
   
   type IDistributedLock interface {
       LockTarget() string
       LockTimeout() time.Duration
   }
   
   type ITransaction interface {
       OpenTX() bool
   }
   
   /*
   *
   result是返回结果
   */
   type IExecution interface {
       Init(ctx context.Context) error                                         // 初始化参数 配置, 本领域数据
       CheckParams(ctx context.Context) error                                  // 参数校验
       PreProcess(ctx context.Context) error                                   // 政策数据模型，外部依赖数据
       ProcessEditVersion(ctx context.Context) ([]TransactionFunc, error)      // 处理编辑版本变更 草稿中的
       ProcessCurVersion(ctx context.Context) ([]TransactionFunc, error)       // 处理当前版本变更 待生效/生效中
       ProcessMaster(ctx context.Context) ([]TransactionFunc, error)           // 处理全局数据变更
       SaveData(ctx context.Context, transactionFuncs []TransactionFunc) error // 保存数据
       PostProcess(ctx context.Context) error                                  // 后置处理
       IDistributedLock
       Name() string
       Result() interface{}
   }
   ```

4. 执行
   
   ```go
   package statemachine
   
   import (
       "xxxxxxx/aurora/block"
       "xxxxxxx/logs"
       "xxxxxxx"
       "context"
   )
   
   type Executor struct{}
   
   func NewExecutor() *Executor {
       return &Executor{}
   }
   
   // 执行所有的execution，在savedata里执行包含了3个process函数，此外返回了最后的result结果
   func (e *Executor) Exec(ctx context.Context, execution IExecution, tags ...metrics.T) (interface{}, error) {
       result, err := func() (interface{}, error) {
           logs.CtxInfo(ctx, "[Executor] execution name = %s", execution.Name())
           if err := execution.Init(ctx); err != nil {
               logs.CtxError(ctx, "execution=%s init step failed error=%v", execution.Name(), err)
               tags = append(tags, metrics.T{Name: "fail_stage", Value: "Init"})
               return execution.Result(), err
           }
           if err := execution.CheckParams(ctx); err != nil {
               logs.CtxError(ctx, "execution=%s check param step failed error=%v", execution.Name(), err)
               tags = append(tags, metrics.T{Name: "fail_stage", Value: "CheckParams"})
               return execution.Result(), err
           }
           if len(execution.LockTarget()) > 0 {
               locker, err := block.NewLock(execution.LockTarget())
               if err != nil {
                   logs.CtxError(ctx, "[Executor] unlock lock failed err = %v, lock = %s", err, locker.LockKey())
                   tags = append(tags, metrics.T{Name: "fail_stage", Value: "NewLock"})
                   return execution.Result(), err
               }
               success, err := locker.TryLock(ctx, execution.LockTimeout())
               if err != nil {
                   logs.CtxError(ctx, "[Executor] try lock failed error=%v", err)
                   tags = append(tags, metrics.T{Name: "fail_stage", Value: "TryLock"})
                   return execution.Result(), err
               }
               if !success {
                   logs.CtxWarn(ctx, "[Executor] try lock failed lock already lock key：%s", execution.LockTarget())
                   tags = append(tags, metrics.T{Name: "fail_stage", Value: "TryLock"})
                   return execution.Result(), err
               }
               defer func() {
                   err = locker.UnLock(ctx)
                   if err != nil {
                       tags = append(tags, metrics.T{Name: "fail_stage", Value: "UnLock"})
                       logs.CtxError(ctx, "[Executor] unlock lock failed err = %v, lock = %s", err, locker.LockKey())
                   }
               }()
           }
           if err := execution.PreProcess(ctx); err != nil {
               logs.CtxError(ctx, "execution=%s pre process step failed error=%v", execution.Name(), err)
               tags = append(tags, metrics.T{Name: "fail_stage", Value: "PreProcess"})
               return execution.Result(), err
           }
           targetFuncs := make([]TransactionFunc, 0)
           funcs1, err := execution.ProcessEditVersion(ctx)
           if err != nil {
               logs.CtxError(ctx, "execution=%s ProcessEditVersion step failed error=%v", execution.Name(), err)
               tags = append(tags, metrics.T{Name: "fail_stage", Value: "ProcessEditVersion"})
               return execution.Result(), err
           }
           targetFuncs = append(targetFuncs, funcs1...)
           funcs2, err := execution.ProcessCurVersion(ctx)
           if err != nil {
               logs.CtxError(ctx, "execution=%s ProcessCurVersion step failed error=%v", execution.Name(), err)
               tags = append(tags, metrics.T{Name: "fail_stage", Value: "ProcessCurVersion"})
               return execution.Result(), err
           }
           targetFuncs = append(targetFuncs, funcs2...)
           funcs3, err := execution.ProcessMaster(ctx)
           if err != nil {
               logs.CtxError(ctx, "execution=%s ProcessMaster step failed error=%v", execution.Name(), err)
               tags = append(tags, metrics.T{Name: "fail_stage", Value: "ProcessMaster"})
               return execution.Result(), err
           }
           targetFuncs = append(targetFuncs, funcs3...)
           if err := execution.SaveData(ctx, targetFuncs); err != nil {
               logs.CtxError(ctx, "execution=%s SaveData step failed error=%v", execution.Name(), err)
               tags = append(tags, metrics.T{Name: "fail_stage", Value: "SaveData"})
               return execution.Result(), err
           }
           if err := execution.PostProcess(ctx); err != nil {
               logs.CtxError(ctx, "execution=%s post process step failed error=%v", execution.Name(), err)
               tags = append(tags, metrics.T{Name: "fail_stage", Value: "PostProcess"})
               return execution.Result(), err
           }
           return execution.Result(), nil
       }()
       return result, err
   }
   ```

5. 实现
   
   ```go
   package statemachine
   
   import (
       "xxxxxxx/gopkg/logs"
       "context"
       "fmt"
       "time"
   )
   
   type BaseExecution struct {
       action string
   }
   
   type TransactionFunc func(ctx context.Context) error
   
   func NewBaseExecution(ctx context.Context, action string) *BaseExecution {
       return &BaseExecution{
           action: action,
       }
   }
   
   func (e *BaseExecution) Init(ctx context.Context) error {
       return nil
   }
   
   func (e *BaseExecution) CheckParams(ctx context.Context) error {
       return nil
   }
   
   func (e *BaseExecution) PreProcess(ctx context.Context) error {
       return nil
   }
   
   func (e *BaseExecution) ProcessEditVersion(ctx context.Context) ([]TransactionFunc, error) {
       return nil, nil
   }
   
   func (e *BaseExecution) ProcessCurVersion(ctx context.Context) ([]TransactionFunc, error) {
       return nil, nil
   }
   
   func (e *BaseExecution) ProcessMaster(ctx context.Context) ([]TransactionFunc, error) {
       return nil, nil
   }
   
   func (e *BaseExecution) SaveData(ctx context.Context, transactionFuncs []TransactionFunc) error {
       // 开启事务保存数据
       for _, transactionFunc := range transactionFuncs {
           bizErr := transactionFunc(ctx)
           if bizErr != nil {
               logs.CtxError(ctx, "[BaseExecution-SaveData] do TransactionFunc error, err = %v", bizErr.Error())
               return bizErr
           }
       }
       return nil
   }
   
   func (e *BaseExecution) PostProcess(ctx context.Context) error {
       return nil
   }
   
   func (e *BaseExecution) LockTarget() string {
       return ""
   }
   
   func (e *BaseExecution) LockTimeout() time.Duration {
       return 5 * time.Second
   }
   
   func (e *BaseExecution) Name() string {
       return fmt.Sprintf("%s", e.action)
   }
   
   func (e *BaseExecution) Result() interface{} {
       return nil
   }
   
   func (e *BaseExecution) GetAction() string {
       return e.action
   }
   ```

# Option

```go
package option

import (
    "fmt"
    "testing"
)

// ServerConfig 定义服务器配置
type ServerConfig struct {
    Port    int
    Timeout int
}

// Option 定义函数选项类型
type Option func(*ServerConfig)

// WithPort 设置服务器端口
func WithPort(port int) Option {
    return func(cfg *ServerConfig) {
        cfg.Port = port
    }
}

// WithTimeout 设置超时时间
func WithTimeout(timeout int) Option {
    return func(cfg *ServerConfig) {
        cfg.Timeout = timeout
    }
}

// NewServer 创建一个新的服务器实例
func NewServer(options ...Option) *ServerConfig {
    // 如果不传，就使用默认值
    cfg := &ServerConfig{
        Port: 8080,
    }
    for _, opt := range options {
        opt(cfg)
    }
    return cfg
}

func Test_option(t *testing.T) {
    // 创建服务器实例并指定选项
    server := NewServer(
        WithPort(9090),
        WithTimeout(60),
    )
    fmt.Printf("Server Port: %d, Timeout: %d\n", server.Port, server.Timeout)
}
```
