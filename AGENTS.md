# AGENTS.md - OpenTelemetry Collector Contrib 开发指南

## 构建和测试命令

### 主要构建命令
```bash
# 构建整个项目
make all

# 构建主要的Collector二进制文件
make otelcontribcol

# 构建testbed collector
make oteltestbedcol

# 构建telemetrygen工具
make telemetrygen

# 构建golden测试工具
make golden
```

### 测试命令
```bash
# 运行所有单元测试
make gotest

# 运行测试并生成覆盖率报告
make gotest-with-cover

# 运行集成测试
make integration-test

# 运行E2E测试
make e2e-test

# 运行两次测试（用于捕获间歇性失败）
make test-twice
```

### 单个测试运行方法
```bash
# 在特定模块中运行测试
cd exporter/elasticsearchexporter/
make test

# 运行特定测试函数
go test -run TestFunctionName ./...

# 运行带覆盖率的测试
go test -cover ./...

# 运行集成测试
go test -tags=integration ./...
```

## 代码风格指南

### 导入顺序
使用 `gci` 工具自动格式化，顺序为：
- 标准库
- 默认包
- 项目内部包 (`github.com/open-telemetry/opentelemetry-collector-contrib`)

### 代码格式化
```bash
# 格式化代码
make fmt

# 仅运行格式化检查（不修改文件）
make gogci
```

### 主要代码风格工具配置
- **gofumpt**: 代码格式化（启用额外规则）
- **gci**: 导入排序
- **goimports**: 导入管理
- **revive**: 代码质量检查（包含25条规则）
- **staticcheck**: 静态分析
- **testifylint**: 测试代码检查

### 关键命名约定
- 错误变量必须以 `err` 前缀
- Context 参数必须是函数的第一个参数
- 避免使用点导入（`.`）
- 接收者名称应反映结构名称
- 使用 `any` 代替 `interface{}`

### 错误处理规范
- 错误信息不应大写或标点符号结尾
- 错误应该是函数返回值的最后一个
- 优先使用 `fmt.Errorf` 而不是 `errors.New`

## 项目结构和开发指南

### 目录结构
```
├── cmd/                    # 主要命令行工具
│   ├── otelcontribcol/     # 主要Collector二进制文件
│   ├── oteltestbedcol/     # 测试环境Collector
│   ├── telemetrygen/       # 遥测数据生成工具
│   └── golden/            # Golden测试工具
├── exporter/              # 所有Exporter组件
├── receiver/              # 所有Receiver组件
├── processor/              # 所有Processor组件
├── extension/              # 所有Extension组件
├── connector/              # 所有Connector组件
├── pkg/                   # 共享包
├── internal/              # 内部包
├── docs/                  # 文档
└── examples/              # 示例配置
```

### 组件开发规范

每个组件必须包含：
- `README.md`: 组件文档和配置示例
- `metadata.yaml`: 组件元数据（稳定性、版本、负责人）
- `config.go`: 配置结构定义
- `factory.go`: 组件工厂实现
- `metrics.go`: 指标定义（如果需要）

### 元数据示例 (metadata.yaml)
```yaml
type: component-type
status:
  class: exporter/processor/receiver/extension
  stability:
    development: [traces, metrics, logs]
  distributions: []
  codeowners:
    active: [username1, username2]
```

### 开发工作流

#### 本地开发步骤
1. **构建**: `make otelcontribcol`
2. **测试**: `make gotest`
3. **格式化**: `make fmt`
4. **运行**: `./bin/otelcontribcol_<os>_<arch> --config config.yaml`

#### 工具链管理
```bash
make install-tools  # 安装所有开发工具
make generate      # 运行所有代码生成命令
make generate-tools # 仅生成工具代码
```

### 配置文件

#### 主要配置文件
- **Makefile**: 项目级目标（构建整个仓库）
- **Makefile.Common**: 模块级目标（单个模块的构建/测试）
- **Makefile.Weaver**: 用于代码生成的配置
- **go.mod**: Go模块配置 (Go 1.24)
- **.golangci.yml**: golangci-lint配置（26个检查器）
- **CONTRIBUTING.md**: 详细的开发指南

#### 代码质量配置
- **revive**: 25条代码质量规则
- **staticcheck**: 静态分析检查
- **gofumpt**: 严格代码格式化
- **testifylint**: 测试代码检查

### 关键开发原则

1. **模块化设计**: 每个组件独立开发，通过公共包共享代码
2. **向后兼容**: 保持API兼容性，重大变更需要标记
3. **文档先行**: README.md必须包含完整的使用示例
4. **测试覆盖**: 所有组件必须有单元测试和集成测试
5. **性能监控**: 包含必要的指标和日志
6. **错误处理**: 统一的错误处理和重试机制
7. **安全考虑**: 输入验证、敏感信息保护

### 特殊约定

- **组件类型**: 按功能分类（exporter、receiver、processor、extension、connector）
- **稳定性标签**: 使用development、alpha、beta、stable等状态
- **版本管理**: 遵循语义化版本控制
- **代码生成**: 使用Weaver进行模板代码生成
- **配置验证**: 必须包含配置验证逻辑

### 故障排除

#### 常见问题
1. **构建失败**: 检查Go版本（1.24）和依赖
2. **测试失败**: 运行 `make test-twice` 检查间歇性问题
3. **格式问题**: 运行 `make fmt` 和 `make gogci`
4. **依赖问题**: 运行 `go mod tidy` 更新依赖

#### 调试技巧
- 使用 `go test -v` 获取详细测试输出
- 使用 `go test -coverprofile=coverage.out` 生成覆盖率报告
- 查看每个组件的README.md获取特定组件的调试信息

## 总结

这个项目遵循Go语言的最佳实践，具有：
- 完善的构建系统（Makefile）
- 严格的代码质量工具链
- 清晰的模块化架构
- 标准化的组件开发流程
- 详细的开发文档和规范

所有开发工作都应该遵循这些约定，确保代码质量和一致性。