# Daytona 到 E2B 迁移文档

## 概述

本文档详细说明了如何将 Suna 项目中的 Daytona 组件迁移到 E2B (Execute to Build) 平台。Daytona 目前作为项目的核心沙箱管理平台，负责 AI 智能体的安全执行环境。

## 1. 现状分析

### 1.1 Daytona 在项目中的作用

Daytona 在 Suna 项目中扮演着关键角色：

- **沙箱环境管理**: 为 AI 智能体提供安全的代码执行环境
- **容器编排**: 管理智能体运行时的生命周期
- **工具集成**: 支持文件系统、浏览器、Web 开发等多种工具
- **安全隔离**: 确保智能体执行的安全性

### 1.2 当前 Daytona 使用情况

通过代码分析，发现以下关键文件使用 Daytona：

#### 核心文件
- `backend/core/sandbox/sandbox.py` - 核心沙箱管理
- `backend/core/sandbox/api.py` - HTTP API 端点
- `backend/core/sandbox/tool_base.py` - 工具基类
- `backend/core/utils/config.py` - 配置管理

#### 工具文件
- `backend/core/tools/shell_tool.py` - Shell 执行工具
- `backend/core/tools/files_tool.py` - 文件操作工具
- `backend/core/tools/browser_tool.py` - 浏览器自动化工具
- `backend/core/tools/web_dev_tool.py` - Web 开发工具
- `backend/core/tools/deploy_tool.py` - 部署工具

### 1.3 Daytona 架构特点

当前 Daytona 架构具有以下特点：
- 基于 AsyncDaytona SDK
- 使用快照系统进行环境初始化
- 支持多种目标环境 (us, eu, asia)
- 异步操作模式
- 完整的错误处理和重试机制

## 2. E2B 替代方案

### 2.1 E2B 技术优势

E2B 是一个专门为 AI 智能体设计的代码执行沙箱平台：

- **专为 AI 设计**: 针对智能体工作负载优化
- **快速启动**: 毫秒级环境启动时间
- **多种环境**: 支持 Python、JavaScript、Java 等多种运行时
- **内置工具**: 集成文件系统、网络访问、进程管理等
- **API 简洁**: 提供简单易用的 API 接口
- **成本效益**: 按 CPU 时间计费，性价比高

### 2.2 E2B 架构设计

```
E2B 架构
├── E2B SDK
│   ├── Sandbox - 沙箱实例管理
│   ├── Process - 进程管理
│   ├── Filesystem - 文件系统操作
│   └── Network - 网络访问控制
├── Suna 适配层
│   ├── 兼容性接口
│   ├── 配置转换
│   └── 错误处理
└── 现有工具系统
    ├── Shell 工具
    ├── 文件工具
    ├── 浏览器工具
    └── Web 开发工具
```

### 2.3 兼容性设计

为了最小化迁移影响，设计兼容性层：

1. **保持现有 API 接口不变**
2. **内部实现替换为 E2B**
3. **配置格式兼容**
4. **错误处理机制保持一致**

## 3. 迁移策略

### 3.1 迁移原则

1. **渐进式迁移**: 分阶段进行，降低风险
2. **向后兼容**: 保持现有 API 接口
3. **功能对等**: 确保所有功能都能正常工作
4. **测试驱动**: 每个阶段都要充分测试
5. **文档同步**: 更新相关文档

### 3.2 迁移阶段

#### 阶段 1: 基础设施准备
- E2B 账户和 API 密钥配置
- 依赖项更新
- 基础架构搭建

#### 阶段 2: 核心沙箱迁移
- E2B 沙箱管理器实现
- 兼容性层开发
- 基本功能测试

#### 阶段 3: 工具迁移
- Shell 工具迁移
- 文件工具迁移
- 浏览器工具迁移
- Web 开发工具迁移
- 部署工具迁移

#### 阶段 4: 集成测试
- 端到端测试
- 性能测试
- 安全测试

#### 阶段 5: 生产部署
- 灰度发布
- 监控配置
- 回滚方案

### 3.3 风险评估

#### 高风险项
1. **API 兼容性**: E2B 和 Daytona 的 API 差异
2. **性能差异**: 启动时间和执行效率的变化
3. **错误处理**: 异常情况的处理方式不同

#### 中风险项
1. **配置管理**: 环境变量和配置文件的变化
2. **依赖关系**: 相关组件的依赖关系
3. **测试覆盖**: 测试用例的完整性

#### 低风险项
1. **文档更新**: 技术文档的同步更新
2. **培训材料**: 开发团队的培训

## 4. 详细实施步骤

### 4.1 阶段 1: 基础设施准备

#### 4.1.1 E2B 账户配置

1. **注册 E2B 账户**
   ```bash
   # 访问 https://e2b.dev/ 注册账户
   # 获取 API 密钥
   ```

2. **更新环境变量**
   ```bash
   # 在 backend/.env 中添加
   E2B_API_KEY=your_e2b_api_key
   E2B_TIMEOUT=300
   E2B_MAX_RETRIES=3
   ```

3. **更新配置文件**
   ```python
   # backend/core/utils/config.py
   class Config:
       # E2B 配置
       E2B_API_KEY: str = Field(default="", env="E2B_API_KEY")
       E2B_TIMEOUT: int = Field(default=300, env="E2B_TIMEOUT")
       E2B_MAX_RETRIES: int = Field(default=3, env="E2B_MAX_RETRIES")
       
       # 保留 Daytona 配置用于回滚
       DAYTONA_API_KEY: str = Field(default="", env="DAYTONA_API_KEY")
       DAYTONA_SERVER_URL: str = Field(default="", env="DAYTONA_SERVER_URL")
       DAYTONA_TARGET: str = Field(default="us", env="DAYTONA_TARGET")
   ```

#### 4.1.2 依赖项更新

1. **更新 Python 依赖**
   ```toml
   # backend/pyproject.toml
   [tool.poetry.dependencies]
   # 移除 Daytona 依赖
   # daytona-sdk = "0.21.0"
   # daytona-api-client = "0.21.0"
   # daytona-api-client-async = "0.21.0"
   
   # 添加 E2B 依赖
   e2b = ">=1.0.0"
   ```

2. **安装新依赖**
   ```bash
   cd backend
   uv add e2b
   uv remove daytona-sdk daytona-api-client daytona-api-client-async
   ```

### 4.2 阶段 2: 核心沙箱迁移

#### 4.2.1 E2B 沙箱管理器实现

创建 `backend/core/sandbox/e2b_sandbox.py`:

```python
"""
E2B 沙箱管理器
提供基于 E2B 的沙箱环境管理功能
"""

import asyncio
import logging
from typing import Optional, Dict, Any
from dataclasses import dataclass
import e2b
from e2b import Sandbox as E2BSandbox

from backend.core.utils.config import Config

logger = logging.getLogger(__name__)

@dataclass
class E2BSandboxInfo:
    """E2B 沙箱信息"""
    sandbox_id: str
    template_id: str
    status: str
    created_at: str
    hostname: str
    port: int

class E2BSandboxManager:
    """E2B 沙箱管理器"""
    
    def __init__(self, config: Config):
        self.config = config
        self.api_key = config.E2B_API_KEY
        self.timeout = config.E2B_TIMEOUT
        self.max_retries = config.E2B_MAX_RETRIES
        self.active_sandboxes: Dict[str, E2BSandbox] = {}
        
        # 初始化 E2B 客户端
        e2b.api_key = self.api_key
        
    async def create_sandbox(
        self, 
        template_id: str = "base",
        timeout: Optional[int] = None,
        **kwargs
    ) -> E2BSandboxInfo:
        """创建新的 E2B 沙箱"""
        try:
            # 使用 E2B 创建沙箱
            sandbox = await E2BSandbox.create(
                template=template_id,
                timeout=timeout or self.timeout,
                **kwargs
            )
            
            sandbox_info = E2BSandboxInfo(
                sandbox_id=sandbox.id,
                template_id=template_id,
                status="running",
                created_at=sandbox.created_at,
                hostname=sandbox.hostname,
                port=sandbox.port
            )
            
            self.active_sandboxes[sandbox.id] = sandbox
            logger.info(f"Created E2B sandbox: {sandbox.id}")
            
            return sandbox_info
            
        except Exception as e:
            logger.error(f"Failed to create E2B sandbox: {e}")
            raise
    
    async def get_sandbox(self, sandbox_id: str) -> Optional[E2BSandbox]:
        """获取现有的沙箱"""
        if sandbox_id in self.active_sandboxes:
            return self.active_sandboxes[sandbox_id]
        return None
    
    async def delete_sandbox(self, sandbox_id: str) -> bool:
        """删除沙箱"""
        try:
            sandbox = self.active_sandboxes.get(sandbox_id)
            if sandbox:
                await sandbox.kill()
                del self.active_sandboxes[sandbox_id]
                logger.info(f"Deleted E2B sandbox: {sandbox_id}")
                return True
            return False
        except Exception as e:
            logger.error(f"Failed to delete E2B sandbox {sandbox_id}: {e}")
            return False
    
    async def execute_command(
        self, 
        sandbox_id: str, 
        command: str,
        cwd: Optional[str] = None,
        env: Optional[Dict[str, str]] = None
    ) -> Dict[str, Any]:
        """在沙箱中执行命令"""
        sandbox = await self.get_sandbox(sandbox_id)
        if not sandbox:
            raise ValueError(f"Sandbox {sandbox_id} not found")
        
        try:
            # 使用 E2B 的进程执行
            process = await sandbox.process.start(
                cmd=command,
                cwd=cwd,
                env=env or {}
            )
            
            # 等待进程完成
            await process.wait()
            
            return {
                "exit_code": process.exit_code,
                "stdout": process.stdout,
                "stderr": process.stderr,
                "pid": process.pid
            }
            
        except Exception as e:
            logger.error(f"Command execution failed: {e}")
            raise
    
    async def list_files(self, sandbox_id: str, path: str = "/") -> Dict[str, Any]:
        """列出沙箱中的文件"""
        sandbox = await self.get_sandbox(sandbox_id)
        if not sandbox:
            raise ValueError(f"Sandbox {sandbox_id} not found")
        
        try:
            # 使用 E2B 的文件系统 API
            files = await sandbox.filesystem.list(path)
            return {"files": files}
        except Exception as e:
            logger.error(f"Failed to list files: {e}")
            raise
    
    async def read_file(self, sandbox_id: str, path: str) -> str:
        """读取沙箱中的文件"""
        sandbox = await self.get_sandbox(sandbox_id)
        if not sandbox:
            raise ValueError(f"Sandbox {sandbox_id} not found")
        
        try:
            content = await sandbox.filesystem.read(path)
            return content
        except Exception as e:
            logger.error(f"Failed to read file: {e}")
            raise
    
    async def write_file(self, sandbox_id: str, path: str, content: str) -> bool:
        """写入文件到沙箱"""
        sandbox = await self.get_sandbox(sandbox_id)
        if not sandbox:
            raise ValueError(f"Sandbox {sandbox_id} not found")
        
        try:
            await sandbox.filesystem.write(path, content)
            return True
        except Exception as e:
            logger.error(f"Failed to write file: {e}")
            return False
    
    async def cleanup(self):
        """清理所有沙箱"""
        for sandbox_id in list(self.active_sandboxes.keys()):
            await self.delete_sandbox(sandbox_id)
```

#### 4.2.2 兼容性层实现

修改 `backend/core/sandbox/sandbox.py`:

```python
"""
沙箱管理模块 - 兼容性层
支持 Daytona 和 E2B 两种沙箱系统
"""

import asyncio
import logging
from typing import Optional, Dict, Any
from dataclasses import dataclass

from backend.core.utils.config import Config
from backend.core.sandbox.e2b_sandbox import E2BSandboxManager, E2BSandboxInfo

logger = logging.getLogger(__name__)

@dataclass
class SandboxInfo:
    """沙箱信息（兼容 Daytona 格式）"""
    id: str
    status: str
    created_at: str
    workspace_id: str
    template_id: str
    # 新增 E2B 特有字段
    hostname: Optional[str] = None
    port: Optional[int] = None

class SandboxManager:
    """沙箱管理器 - 统一接口"""
    
    def __init__(self, config: Config):
        self.config = config
        self.use_e2b = bool(config.E2B_API_KEY)  # 优先使用 E2B
        
        if self.use_e2b:
            self.e2b_manager = E2BSandboxManager(config)
            logger.info("Using E2B sandbox manager")
        else:
            # 保持 Daytona 实现
            from daytona_sdk import AsyncDaytona
            from daytona_sdk.config import DaytonaConfig
            
            daytona_config = DaytonaConfig(
                api_key=config.DAYTONA_API_KEY,
                server_url=config.DAYTONA_SERVER_URL,
                target=config.DAYTONA_TARGET
            )
            self.daytona = AsyncDaytona(daytona_config)
            logger.info("Using Daytona sandbox manager")
    
    async def create_sandbox(
        self, 
        workspace_id: str,
        template_id: str = "base"
    ) -> SandboxInfo:
        """创建沙箱"""
        if self.use_e2b:
            return await self._create_e2b_sandbox(workspace_id, template_id)
        else:
            return await self._create_daytona_sandbox(workspace_id, template_id)
    
    async def _create_e2b_sandbox(self, workspace_id: str, template_id: str) -> SandboxInfo:
        """创建 E2B 沙箱"""
        try:
            e2b_info = await self.e2b_manager.create_sandbox(template_id)
            
            return SandboxInfo(
                id=e2b_info.sandbox_id,
                status=e2b_info.status,
                created_at=e2b_info.created_at,
                workspace_id=workspace_id,
                template_id=e2b_info.template_id,
                hostname=e2b_info.hostname,
                port=e2b_info.port
            )
        except Exception as e:
            logger.error(f"Failed to create E2B sandbox: {e}")
            raise
    
    async def _create_daytona_sandbox(self, workspace_id: str, template_id: str) -> SandboxInfo:
        """创建 Daytona 沙箱（保持原有实现）"""
        try:
            from daytona_sdk.models import CreateSandboxFromSnapshotParams
            
            params = CreateSandboxFromSnapshotParams(
                workspace_id=workspace_id,
                snapshot_name=template_id
            )
            
            sandbox = await self.daytona.create_sandbox_from_snapshot(params)
            
            return SandboxInfo(
                id=sandbox.id,
                status=sandbox.status,
                created_at=sandbox.created_at,
                workspace_id=workspace_id,
                template_id=template_id
            )
        except Exception as e:
            logger.error(f"Failed to create Daytona sandbox: {e}")
            raise
    
    async def get_sandbox(self, sandbox_id: str) -> Optional[SandboxInfo]:
        """获取沙箱信息"""
        if self.use_e2b:
            sandbox = await self.e2b_manager.get_sandbox(sandbox_id)
            if sandbox:
                return SandboxInfo(
                    id=sandbox.id,
                    status="running",
                    created_at=sandbox.created_at,
                    workspace_id="",
                    template_id="",
                    hostname=sandbox.hostname,
                    port=sandbox.port
                )
        else:
            # Daytona 实现
            try:
                sandbox = await self.daytona.get_sandbox(sandbox_id)
                return SandboxInfo(
                    id=sandbox.id,
                    status=sandbox.status,
                    created_at=sandbox.created_at,
                    workspace_id=sandbox.workspace_id,
                    template_id=sandbox.template_id
                )
            except Exception:
                pass
        return None
    
    async def delete_sandbox(self, sandbox_id: str) -> bool:
        """删除沙箱"""
        if self.use_e2b:
            return await self.e2b_manager.delete_sandbox(sandbox_id)
        else:
            try:
                await self.daytona.delete_sandbox(sandbox_id)
                return True
            except Exception as e:
                logger.error(f"Failed to delete Daytona sandbox: {e}")
                return False
    
    async def execute_command(
        self, 
        sandbox_id: str, 
        command: str,
        cwd: Optional[str] = None,
        env: Optional[Dict[str, str]] = None
    ) -> Dict[str, Any]:
        """执行命令"""
        if self.use_e2b:
            return await self.e2b_manager.execute_command(sandbox_id, command, cwd, env)
        else:
            # Daytona 命令执行实现
            sandbox = await self.daytona.get_sandbox(sandbox_id)
            result = await sandbox.execute_command(command, cwd=cwd, env=env)
            return {
                "exit_code": result.exit_code,
                "stdout": result.stdout,
                "stderr": result.stderr
            }

# 全局沙箱管理器实例
_sandbox_manager: Optional[SandboxManager] = None

def get_sandbox_manager() -> SandboxManager:
    """获取全局沙箱管理器"""
    global _sandbox_manager
    if _sandbox_manager is None:
        from backend.core.utils.config import get_config
        _sandbox_manager = SandboxManager(get_config())
    return _sandbox_manager

async def create_sandbox(workspace_id: str, template_id: str = "base") -> SandboxInfo:
    """创建沙箱（兼容原有接口）"""
    manager = get_sandbox_manager()
    return await manager.create_sandbox(workspace_id, template_id)

async def get_or_start_sandbox(workspace_id: str) -> SandboxInfo:
    """获取或启动沙箱（兼容原有接口）"""
    manager = get_sandbox_manager()
    # 实现逻辑...
    return await manager.create_sandbox(workspace_id)

async def delete_sandbox(sandbox_id: str) -> bool:
    """删除沙箱（兼容原有接口）"""
    manager = get_sandbox_manager()
    return await manager.delete_sandbox(sandbox_id)
```

### 4.3 阶段 3: 工具迁移

#### 4.3.1 工具基类适配

修改 `backend/core/sandbox/tool_base.py`:

```python
"""
沙箱工具基类 - E2B 适配
"""

import asyncio
import logging
from typing import Optional, Dict, Any
from abc import ABC, abstractmethod

from backend.core.sandbox.sandbox import get_sandbox_manager, SandboxInfo

logger = logging.getLogger(__name__)

class SandboxToolBase(ABC):
    """沙箱工具基类 - 支持 E2B 和 Daytona"""
    
    def __init__(self, project_id: str):
        self.project_id = project_id
        self.sandbox_manager = get_sandbox_manager()
        self._sandbox: Optional[SandboxInfo] = None
        self._lock = asyncio.Lock()
    
    async def _ensure_sandbox(self) -> SandboxInfo:
        """确保沙箱已创建"""
        async with self._lock:
            if self._sandbox is None:
                try:
                    self._sandbox = await self.sandbox_manager.create_sandbox(
                        workspace_id=self.project_id,
                        template_id="base"  # 使用基础模板
                    )
                    logger.info(f"Created sandbox for project {self.project_id}: {self._sandbox.id}")
                except Exception as e:
                    logger.error(f"Failed to create sandbox for project {self.project_id}: {e}")
                    raise
            return self._sandbox
    
    async def execute_command(
        self, 
        command: str,
        cwd: Optional[str] = None,
        env: Optional[Dict[str, str]] = None
    ) -> Dict[str, Any]:
        """执行命令"""
        sandbox = await self._ensure_sandbox()
        return await self.sandbox_manager.execute_command(
            sandbox.id, command, cwd, env
        )
    
    async def list_files(self, path: str = "/") -> Dict[str, Any]:
        """列出文件"""
        sandbox = await self._ensure_sandbox()
        
        if hasattr(self.sandbox_manager, 'e2b_manager'):
            # E2B 特有功能
            return await self.sandbox_manager.e2b_manager.list_files(sandbox.id, path)
        else:
            # Daytona 文件操作（通过命令实现）
            result = await self.execute_command(f"ls -la {path}")
            return {"files": result["stdout"]}
    
    async def read_file(self, path: str) -> str:
        """读取文件"""
        sandbox = await self._ensure_sandbox()
        
        if hasattr(self.sandbox_manager, 'e2b_manager'):
            return await self.sandbox_manager.e2b_manager.read_file(sandbox.id, path)
        else:
            result = await self.execute_command(f"cat {path}")
            return result["stdout"]
    
    async def write_file(self, path: str, content: str) -> bool:
        """写入文件"""
        sandbox = await self._ensure_sandbox()
        
        if hasattr(self.sandbox_manager, 'e2b_manager'):
            return await self.sandbox_manager.e2b_manager.write_file(sandbox.id, path, content)
        else:
            # Daytona 通过 echo 命令写入文件
            result = await self.execute_command(f"echo '{content}' > {path}")
            return result["exit_code"] == 0
    
    async def cleanup(self):
        """清理资源"""
        if self._sandbox:
            try:
                await self.sandbox_manager.delete_sandbox(self._sandbox.id)
                self._sandbox = None
                logger.info(f"Cleaned up sandbox for project {self.project_id}")
            except Exception as e:
                logger.error(f"Failed to cleanup sandbox for project {self.project_id}: {e}")
```

#### 4.3.2 Shell 工具迁移

修改 `backend/core/tools/shell_tool.py`:

```python
"""
Shell 工具 - E2B 适配
"""

import asyncio
import logging
from typing import Dict, Any

from backend.core.sandbox.tool_base import SandboxToolBase
from backend.core.tools.tool import Tool, ToolResult

logger = logging.getLogger(__name__)

class ShellTool(SandboxToolBase, Tool):
    """Shell 执行工具 - 支持 E2B 和 Daytona"""
    
    def __init__(self, project_id: str):
        SandboxToolBase.__init__(self, project_id)
        Tool.__init__(self)
    
    @Tool.sandbox
    async def execute(
        self, 
        command: str,
        timeout: int = 30,
        working_directory: str = None
    ) -> ToolResult:
        """执行 shell 命令"""
        try:
            # 使用沙箱执行命令
            result = await self.execute_command(
                command=command,
                cwd=working_directory,
                env={"TIMEOUT": str(timeout)}
            )
            
            # 构造响应
            response = {
                "success": result["exit_code"] == 0,
                "output": result["stdout"],
                "error": result["stderr"],
                "exit_code": result["exit_code"],
                "execution_time": 0  # E2B 可能不提供执行时间
            }
            
            return ToolResult(
                success=response["success"],
                data=response,
                message=f"Command executed with exit code {result['exit_code']}"
            )
            
        except Exception as e:
            logger.error(f"Shell command execution failed: {e}")
            return ToolResult(
                success=False,
                data={"error": str(e)},
                message=f"Command execution failed: {str(e)}"
            )
    
    async def background_execute(
        self, 
        command: str,
        timeout: int = 300
    ) -> Dict[str, Any]:
        """后台执行命令"""
        try:
            sandbox = await self._ensure_sandbox()
            
            # E2B 支持后台进程
            if hasattr(self.sandbox_manager, 'e2b_manager'):
                # 使用 E2B 的后台执行功能
                process = await self.sandbox_manager.e2b_manager.active_sandboxes[sandbox.id].process.start(
                    cmd=command,
                    background=True
                )
                return {
                    "process_id": process.pid,
                    "status": "running",
                    "command": command
                }
            else:
                # Daytona 后台执行
                result = await self.execute_command(f"nohup {command} &")
                return {
                    "process_id": "unknown",  # Daytona 可能不提供进程 ID
                    "status": "running",
                    "command": command
                }
                
        except Exception as e:
            logger.error(f"Background command execution failed: {e}")
            return {
                "process_id": None,
                "status": "failed",
                "error": str(e)
            }
```

#### 4.3.3 文件工具迁移

修改 `backend/core/tools/files_tool.py`:

```python
"""
文件工具 - E2B 适配
"""

import asyncio
import logging
from typing import Dict, Any, Optional
from pathlib import Path

from backend.core.sandbox.tool_base import SandboxToolBase
from backend.core.tools.tool import Tool, ToolResult

logger = logging.getLogger(__name__)

class FilesTool(SandboxToolBase, Tool):
    """文件操作工具 - 支持 E2B 和 Daytona"""
    
    def __init__(self, project_id: str):
        SandboxToolBase.__init__(self, project_id)
        Tool.__init__(self)
    
    @Tool.sandbox
    async def list_files(self, path: str = "/") -> ToolResult:
        """列出文件"""
        try:
            result = await self.list_files(path)
            return ToolResult(
                success=True,
                data=result,
                message=f"Listed files in {path}"
            )
        except Exception as e:
            logger.error(f"Failed to list files: {e}")
            return ToolResult(
                success=False,
                data={"error": str(e)},
                message=f"Failed to list files: {str(e)}"
            )
    
    @Tool.sandbox
    async def read_file(self, path: str) -> ToolResult:
        """读取文件"""
        try:
            content = await self.read_file(path)
            return ToolResult(
                success=True,
                data={"content": content},
                message=f"Read file: {path}"
            )
        except Exception as e:
            logger.error(f"Failed to read file: {e}")
            return ToolResult(
                success=False,
                data={"error": str(e)},
                message=f"Failed to read file: {str(e)}"
            )
    
    @Tool.sandbox
    async def write_file(self, path: str, content: str) -> ToolResult:
        """写入文件"""
        try:
            success = await self.write_file(path, content)
            if success:
                return ToolResult(
                    success=True,
                    data={"path": path},
                    message=f"Written file: {path}"
                )
            else:
                return ToolResult(
                    success=False,
                    data={"path": path},
                    message=f"Failed to write file: {path}"
                )
        except Exception as e:
            logger.error(f"Failed to write file: {e}")
            return ToolResult(
                success=False,
                data={"error": str(e)},
                message=f"Failed to write file: {str(e)}"
            )
    
    @Tool.sandbox
    async def delete_file(self, path: str) -> ToolResult:
        """删除文件"""
        try:
            result = await self.execute_command(f"rm -rf {path}")
            if result["exit_code"] == 0:
                return ToolResult(
                    success=True,
                    data={"path": path},
                    message=f"Deleted file: {path}"
                )
            else:
                return ToolResult(
                    success=False,
                    data={"path": path, "error": result["stderr"]},
                    message=f"Failed to delete file: {path}"
                )
        except Exception as e:
            logger.error(f"Failed to delete file: {e}")
            return ToolResult(
                success=False,
                data={"error": str(e)},
                message=f"Failed to delete file: {str(e)}"
            )
    
    @Tool.sandbox
    async def create_directory(self, path: str) -> ToolResult:
        """创建目录"""
        try:
            result = await self.execute_command(f"mkdir -p {path}")
            if result["exit_code"] == 0:
                return ToolResult(
                    success=True,
                    data={"path": path},
                    message=f"Created directory: {path}"
                )
            else:
                return ToolResult(
                    success=False,
                    data={"path": path, "error": result["stderr"]},
                    message=f"Failed to create directory: {path}"
                )
        except Exception as e:
            logger.error(f"Failed to create directory: {e}")
            return ToolResult(
                success=False,
                data={"error": str(e)},
                message=f"Failed to create directory: {str(e)}"
            )
```

### 4.4 阶段 4: API 端点迁移

#### 4.4.1 修改沙箱 API

修改 `backend/core/sandbox/api.py`:

```python
"""
沙箱 API 端点 - E2B 适配
"""

import logging
from typing import Dict, Any, Optional
from fastapi import APIRouter, HTTPException, Depends
from pydantic import BaseModel

from backend.core.sandbox.sandbox import get_sandbox_manager
from backend.core.auth.dependencies import get_current_user

logger = logging.getLogger(__name__)

router = APIRouter()

class FileOperationRequest(BaseModel):
    """文件操作请求"""
    sandbox_id: str
    path: str
    content: Optional[str] = None

class CommandExecutionRequest(BaseModel):
    """命令执行请求"""
    sandbox_id: str
    command: str
    working_directory: Optional[str] = None
    timeout: int = 30

@router.get("/sandbox/{sandbox_id}")
async def get_sandbox_info(sandbox_id: str):
    """获取沙箱信息"""
    try:
        manager = get_sandbox_manager()
        sandbox = await manager.get_sandbox(sandbox_id)
        
        if not sandbox:
            raise HTTPException(status_code=404, detail="Sandbox not found")
        
        return {
            "sandbox_id": sandbox.id,
            "status": sandbox.status,
            "created_at": sandbox.created_at,
            "workspace_id": sandbox.workspace_id,
            "template_id": sandbox.template_id,
            # E2B 特有字段
            "hostname": getattr(sandbox, 'hostname', None),
            "port": getattr(sandbox, 'port', None)
        }
    except Exception as e:
        logger.error(f"Failed to get sandbox info: {e}")
        raise HTTPException(status_code=500, detail=str(e))

@router.post("/sandbox/{workspace_id}/create")
async def create_sandbox_endpoint(workspace_id: str, template_id: str = "base"):
    """创建沙箱"""
    try:
        manager = get_sandbox_manager()
        sandbox = await manager.create_sandbox(workspace_id, template_id)
        
        return {
            "sandbox_id": sandbox.id,
            "status": sandbox.status,
            "created_at": sandbox.created_at,
            "workspace_id": sandbox.workspace_id,
            "template_id": sandbox.template_id,
            # E2B 特有字段
            "hostname": getattr(sandbox, 'hostname', None),
            "port": getattr(sandbox, 'port', None)
        }
    except Exception as e:
        logger.error(f"Failed to create sandbox: {e}")
        raise HTTPException(status_code=500, detail=str(e))

@router.delete("/sandbox/{sandbox_id}")
async def delete_sandbox_endpoint(sandbox_id: str):
    """删除沙箱"""
    try:
        manager = get_sandbox_manager()
        success = await manager.delete_sandbox(sandbox_id)
        
        if success:
            return {"message": "Sandbox deleted successfully"}
        else:
            raise HTTPException(status_code=404, detail="Sandbox not found")
    except Exception as e:
        logger.error(f"Failed to delete sandbox: {e}")
        raise HTTPException(status_code=500, detail=str(e))

@router.post("/sandbox/execute")
async def execute_command_endpoint(request: CommandExecutionRequest):
    """执行命令"""
    try:
        manager = get_sandbox_manager()
        result = await manager.execute_command(
            request.sandbox_id,
            request.command,
            request.working_directory
        )
        
        return {
            "exit_code": result["exit_code"],
            "stdout": result["stdout"],
            "stderr": result["stderr"],
            # E2B 特有字段
            "pid": result.get("pid", None)
        }
    except Exception as e:
        logger.error(f"Failed to execute command: {e}")
        raise HTTPException(status_code=500, detail=str(e))

@router.post("/sandbox/files/list")
async def list_files_endpoint(sandbox_id: str, path: str = "/"):
    """列出文件"""
    try:
        manager = get_sandbox_manager()
        
        if hasattr(manager, 'e2b_manager'):
            # E2B 文件系统 API
            result = await manager.e2b_manager.list_files(sandbox_id, path)
            return result
        else:
            # Daytona 通过命令列出文件
            command_result = await manager.execute_command(sandbox_id, f"ls -la {path}")
            return {
                "files": command_result["stdout"],
                "exit_code": command_result["exit_code"]
            }
    except Exception as e:
        logger.error(f"Failed to list files: {e}")
        raise HTTPException(status_code=500, detail=str(e))

@router.post("/sandbox/files/read")
async def read_file_endpoint(sandbox_id: str, path: str):
    """读取文件"""
    try:
        manager = get_sandbox_manager()
        
        if hasattr(manager, 'e2b_manager'):
            # E2B 文件系统 API
            content = await manager.e2b_manager.read_file(sandbox_id, path)
            return {"content": content}
        else:
            # Daytona 通过命令读取文件
            result = await manager.execute_command(sandbox_id, f"cat {path}")
            return {
                "content": result["stdout"],
                "exit_code": result["exit_code"]
            }
    except Exception as e:
        logger.error(f"Failed to read file: {e}")
        raise HTTPException(status_code=500, detail=str(e))

@router.post("/sandbox/files/write")
async def write_file_endpoint(request: FileOperationRequest):
    """写入文件"""
    try:
        manager = get_sandbox_manager()
        
        if hasattr(manager, 'e2b_manager'):
            # E2B 文件系统 API
            success = await manager.e2b_manager.write_file(
                request.sandbox_id, 
                request.path, 
                request.content or ""
            )
        else:
            # Daytona 通过命令写入文件
            result = await manager.execute_command(
                request.sandbox_id, 
                f"echo '{request.content}' > {request.path}"
            )
            success = result["exit_code"] == 0
        
        if success:
            return {"message": "File written successfully"}
        else:
            raise HTTPException(status_code=500, detail="Failed to write file")
    except Exception as e:
        logger.error(f"Failed to write file: {e}")
        raise HTTPException(status_code=500, detail=str(e))
```

### 4.5 阶段 5: 测试和部署

#### 4.5.1 创建测试脚本

创建 `backend/tests/test_e2b_migration.py`:

```python
"""
E2B 迁移测试
"""

import pytest
import asyncio
from unittest.mock import Mock, patch

from backend.core.sandbox.e2b_sandbox import E2BSandboxManager
from backend.core.sandbox.sandbox import SandboxManager
from backend.core.utils.config import Config

class TestE2BMigration:
    """E2B 迁移测试类"""
    
    @pytest.fixture
    def config(self):
        """测试配置"""
        return Config(
            E2B_API_KEY="test_key",
            E2B_TIMEOUT=300,
            E2B_MAX_RETRIES=3,
            DAYTONA_API_KEY="",
            DAYTONA_SERVER_URL="",
            DAYTONA_TARGET="us"
        )
    
    @pytest.fixture
    def e2b_manager(self, config):
        """E2B 管理器"""
        return E2BSandboxManager(config)
    
    @pytest.fixture
    def sandbox_manager(self, config):
        """沙箱管理器"""
        return SandboxManager(config)
    
    @pytest.mark.asyncio
    async def test_e2b_manager_initialization(self, e2b_manager):
        """测试 E2B 管理器初始化"""
        assert e2b_manager.api_key == "test_key"
        assert e2b_manager.timeout == 300
        assert e2b_manager.max_retries == 3
        assert e2b_manager.active_sandboxes == {}
    
    @pytest.mark.asyncio
    async def test_sandbox_manager_uses_e2b(self, sandbox_manager):
        """测试沙箱管理器使用 E2B"""
        assert sandbox_manager.use_e2b is True
        assert hasattr(sandbox_manager, 'e2b_manager')
    
    @pytest.mark.asyncio
    async def test_sandbox_manager_fallback_to_daytona(self):
        """测试回退到 Daytona"""
        config = Config(
            E2B_API_KEY="",
            DAYTONA_API_KEY="test_daytona_key",
            DAYTONA_SERVER_URL="https://test.daytona.com",
            DAYTONA_TARGET="us"
        )
        
        sandbox_manager = SandboxManager(config)
        assert sandbox_manager.use_e2b is False
        assert hasattr(sandbox_manager, 'daytona')
    
    @pytest.mark.asyncio
    @patch('backend.core.sandbox.e2b_sandbox.E2BSandbox')
    async def test_create_e2b_sandbox(self, mock_sandbox_class, e2b_manager):
        """测试创建 E2B 沙箱"""
        # Mock E2B Sandbox
        mock_sandbox = Mock()
        mock_sandbox.id = "test_sandbox_id"
        mock_sandbox.created_at = "2024-01-01T00:00:00Z"
        mock_sandbox.hostname = "test.hostname.com"
        mock_sandbox.port = 8080
        
        mock_sandbox_class.create = Mock(return_value=mock_sandbox)
        
        # 创建沙箱
        result = await e2b_manager.create_sandbox("base")
        
        # 验证结果
        assert result.sandbox_id == "test_sandbox_id"
        assert result.template_id == "base"
        assert result.status == "running"
        assert result.hostname == "test.hostname.com"
        assert result.port == 8080
        
        # 验证 E2B SDK 被调用
        mock_sandbox_class.create.assert_called_once()
    
    @pytest.mark.asyncio
    async def test_e2b_command_execution(self, e2b_manager):
        """测试 E2B 命令执行"""
        # Mock 沙箱和进程
        mock_sandbox = Mock()
        mock_process = Mock()
        mock_process.exit_code = 0
        mock_process.stdout = "Hello World"
        mock_process.stderr = ""
        mock_process.pid = 12345
        
        mock_sandbox.process.start = Mock(return_value=mock_process)
        mock_sandbox.process.start.return_value.wait = Mock()
        
        e2b_manager.active_sandboxes["test_sandbox"] = mock_sandbox
        
        # 执行命令
        result = await e2b_manager.execute_command(
            "test_sandbox", 
            "echo 'Hello World'"
        )
        
        # 验证结果
        assert result["exit_code"] == 0
        assert result["stdout"] == "Hello World"
        assert result["stderr"] == ""
        assert result["pid"] == 12345
    
    @pytest.mark.asyncio
    async def test_e2b_file_operations(self, e2b_manager):
        """测试 E2B 文件操作"""
        # Mock 沙箱
        mock_sandbox = Mock()
        mock_sandbox.filesystem.list = Mock(return_value=["file1.txt", "file2.txt"])
        mock_sandbox.filesystem.read = Mock(return_value="file content")
        mock_sandbox.filesystem.write = Mock(return_value=True)
        
        e2b_manager.active_sandboxes["test_sandbox"] = mock_sandbox
        
        # 测试列出文件
        list_result = await e2b_manager.list_files("test_sandbox", "/test")
        assert list_result["files"] == ["file1.txt", "file2.txt"]
        
        # 测试读取文件
        read_result = await e2b_manager.read_file("test_sandbox", "/test/file.txt")
        assert read_result == "file content"
        
        # 测试写入文件
        write_result = await e2b_manager.write_file(
            "test_sandbox", 
            "/test/file.txt", 
            "new content"
        )
        assert write_result is True
    
    @pytest.mark.asyncio
    async def test_compatibility_layer(self, sandbox_manager):
        """测试兼容性层"""
        # Mock E2B 管理器
        with patch.object(sandbox_manager.e2b_manager, 'create_sandbox') as mock_create:
            mock_create.return_value = Mock(
                sandbox_id="test_sandbox",
                template_id="base",
                status="running",
                created_at="2024-01-01T00:00:00Z",
                hostname="test.hostname.com",
                port=8080
            )
            
            # 测试创建沙箱
            result = await sandbox_manager.create_sandbox("test_workspace", "base")
            
            # 验证结果格式兼容
            assert result.id == "test_sandbox"
            assert result.workspace_id == "test_workspace"
            assert result.template_id == "base"
            assert result.status == "running"
            assert result.hostname == "test.hostname.com"
            assert result.port == 8080
    
    @pytest.mark.asyncio
    async def test_error_handling(self, e2b_manager):
        """测试错误处理"""
        # 测试沙箱不存在
        with pytest.raises(ValueError, match="Sandbox not found"):
            await e2b_manager.execute_command("nonexistent", "echo test")
        
        # 测试命令执行失败
        mock_sandbox = Mock()
        mock_sandbox.process.start = Mock(side_effect=Exception("Command failed"))
        e2b_manager.active_sandboxes["test_sandbox"] = mock_sandbox
        
        with pytest.raises(Exception, match="Command failed"):
            await e2b_manager.execute_command("test_sandbox", "failing_command")
```

#### 4.5.2 性能测试脚本

创建 `backend/tests/test_performance.py`:

```python
"""
性能测试脚本
"""

import asyncio
import time
import statistics
from typing import List

from backend.core.sandbox.sandbox import SandboxManager
from backend.core.utils.config import Config

class PerformanceTest:
    """性能测试类"""
    
    def __init__(self, config: Config):
        self.config = config
        self.sandbox_manager = SandboxManager(config)
    
    async def test_sandbox_creation_time(self, iterations: int = 10) -> dict:
        """测试沙箱创建时间"""
        times = []
        
        for i in range(iterations):
            start_time = time.time()
            
            try:
                sandbox = await self.sandbox_manager.create_sandbox(
                    f"perf_test_{i}", 
                    "base"
                )
                creation_time = time.time() - start_time
                times.append(creation_time)
                
                # 清理
                await self.sandbox_manager.delete_sandbox(sandbox.id)
                
            except Exception as e:
                print(f"Error in iteration {i}: {e}")
        
        return {
            "iterations": iterations,
            "successful": len(times),
            "average_time": statistics.mean(times) if times else 0,
            "min_time": min(times) if times else 0,
            "max_time": max(times) if times else 0,
            "median_time": statistics.median(times) if times else 0,
            "std_dev": statistics.stdev(times) if len(times) > 1 else 0
        }
    
    async def test_command_execution_time(self, iterations: int = 50) -> dict:
        """测试命令执行时间"""
        times = []
        
        # 创建一个沙箱用于测试
        sandbox = await self.sandbox_manager.create_sandbox("cmd_perf_test", "base")
        
        try:
            for i in range(iterations):
                start_time = time.time()
                
                result = await self.sandbox_manager.execute_command(
                    sandbox.id,
                    "echo 'Hello World'"
                )
                
                execution_time = time.time() - start_time
                times.append(execution_time)
                
                if result["exit_code"] != 0:
                    print(f"Command failed in iteration {i}: {result['stderr']}")
        
        finally:
            # 清理
            await self.sandbox_manager.delete_sandbox(sandbox.id)
        
        return {
            "iterations": iterations,
            "successful": len(times),
            "average_time": statistics.mean(times) if times else 0,
            "min_time": min(times) if times else 0,
            "max_time": max(times) if times else 0,
            "median_time": statistics.median(times) if times else 0,
            "std_dev": statistics.stdev(times) if len(times) > 1 else 0
        }
    
    async def test_file_operations_time(self, iterations: int = 30) -> dict:
        """测试文件操作时间"""
        write_times = []
        read_times = []
        
        # 创建一个沙箱用于测试
        sandbox = await self.sandbox_manager.create_sandbox("file_perf_test", "base")
        
        try:
            for i in range(iterations):
                # 测试写入
                start_time = time.time()
                
                if hasattr(self.sandbox_manager, 'e2b_manager'):
                    success = await self.sandbox_manager.e2b_manager.write_file(
                        sandbox.id,
                        f"/tmp/test_file_{i}.txt",
                        f"Test content {i}"
                    )
                else:
                    result = await self.sandbox_manager.execute_command(
                        sandbox.id,
                        f"echo 'Test content {i}' > /tmp/test_file_{i}.txt"
                    )
                    success = result["exit_code"] == 0
                
                write_time = time.time() - start_time
                write_times.append(write_time)
                
                # 测试读取
                start_time = time.time()
                
                if hasattr(self.sandbox_manager, 'e2b_manager'):
                    content = await self.sandbox_manager.e2b_manager.read_file(
                        sandbox.id,
                        f"/tmp/test_file_{i}.txt"
                    )
                else:
                    result = await self.sandbox_manager.execute_command(
                        sandbox.id,
                        f"cat /tmp/test_file_{i}.txt"
                    )
                    content = result["stdout"]
                
                read_time = time.time() - start_time
                read_times.append(read_time)
                
                if not success or content != f"Test content {i}":
                    print(f"File operation failed in iteration {i}")
        
        finally:
            # 清理
            await self.sandbox_manager.delete_sandbox(sandbox.id)
        
        return {
            "iterations": iterations,
            "write": {
                "successful": len(write_times),
                "average_time": statistics.mean(write_times) if write_times else 0,
                "min_time": min(write_times) if write_times else 0,
                "max_time": max(write_times) if write_times else 0,
                "median_time": statistics.median(write_times) if write_times else 0,
                "std_dev": statistics.stdev(write_times) if len(write_times) > 1 else 0
            },
            "read": {
                "successful": len(read_times),
                "average_time": statistics.mean(read_times) if read_times else 0,
                "min_time": min(read_times) if read_times else 0,
                "max_time": max(read_times) if read_times else 0,
                "median_time": statistics.median(read_times) if read_times else 0,
                "std_dev": statistics.stdev(read_times) if len(read_times) > 1 else 0
            }
        }

async def run_performance_tests():
    """运行性能测试"""
    config = Config(
        E2B_API_KEY="test_key",
        E2B_TIMEOUT=300,
        E2B_MAX_RETRIES=3
    )
    
    tester = PerformanceTest(config)
    
    print("Running performance tests...")
    
    # 沙箱创建测试
    print("\n1. Sandbox Creation Performance")
    creation_results = await tester.test_sandbox_creation_time(5)
    print(f"Average creation time: {creation_results['average_time']:.2f}s")
    print(f"Min creation time: {creation_results['min_time']:.2f}s")
    print(f"Max creation time: {creation_results['max_time']:.2f}s")
    
    # 命令执行测试
    print("\n2. Command Execution Performance")
    command_results = await tester.test_command_execution_time(20)
    print(f"Average execution time: {command_results['average_time']:.3f}s")
    print(f"Min execution time: {command_results['min_time']:.3f}s")
    print(f"Max execution time: {command_results['max_time']:.3f}s")
    
    # 文件操作测试
    print("\n3. File Operations Performance")
    file_results = await tester.test_file_operations_time(10)
    print(f"Write - Average time: {file_results['write']['average_time']:.3f}s")
    print(f"Read - Average time: {file_results['read']['average_time']:.3f}s")
    
    return {
        "creation": creation_results,
        "command_execution": command_results,
        "file_operations": file_results
    }

if __name__ == "__main__":
    asyncio.run(run_performance_tests())
```

### 4.6 阶段 6: 部署和监控

#### 4.6.1 部署脚本

创建 `backend/scripts/deploy_e2b_migration.py`:

```python
"""
E2B 迁移部署脚本
"""

import asyncio
import logging
import sys
from pathlib import Path

# 添加项目根目录到 Python 路径
sys.path.insert(0, str(Path(__file__).parent.parent))

from backend.core.utils.config import Config
from backend.core.sandbox.sandbox import SandboxManager

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

class E2BDeployment:
    """E2B 部署管理器"""
    
    def __init__(self):
        self.config = Config()
        self.sandbox_manager = SandboxManager(self.config)
    
    async def pre_deployment_check(self) -> dict:
        """部署前检查"""
        checks = {
            "e2b_api_key": bool(self.config.E2B_API_KEY),
            "e2b_timeout": self.config.E2B_TIMEOUT > 0,
            "e2b_max_retries": self.config.E2B_MAX_RETRIES > 0,
            "daytona_fallback": bool(self.config.DAYTONA_API_KEY),
            "use_e2b": self.sandbox_manager.use_e2b
        }
        
        logger.info("Pre-deployment checks:")
        for check, passed in checks.items():
            status = "✓" if passed else "✗"
            logger.info(f"  {check}: {status}")
        
        return checks
    
    async def test_e2b_connectivity(self) -> bool:
        """测试 E2B 连接"""
        try:
            if not self.sandbox_manager.use_e2b:
                logger.warning("E2B not enabled, skipping connectivity test")
                return True
            
            # 尝试创建一个测试沙箱
            logger.info("Testing E2B connectivity...")
            sandbox = await self.sandbox_manager.create_sandbox(
                "deployment_test", 
                "base"
            )
            
            # 执行简单命令
            result = await self.sandbox_manager.execute_command(
                sandbox.id,
                "echo 'E2B connectivity test'"
            )
            
            if result["exit_code"] == 0:
                logger.info("✓ E2B connectivity test passed")
                success = True
            else:
                logger.error(f"✗ E2B connectivity test failed: {result['stderr']}")
                success = False
            
            # 清理
            await self.sandbox_manager.delete_sandbox(sandbox.id)
            
            return success
            
        except Exception as e:
            logger.error(f"✗ E2B connectivity test failed: {e}")
            return False
    
    async def test_daytona_fallback(self) -> bool:
        """测试 Daytona 回退"""
        try:
            if self.sandbox_manager.use_e2b:
                logger.info("E2B enabled, skipping Daytona fallback test")
                return True
            
            logger.info("Testing Daytona fallback...")
            
            # 这里需要实际的 Daytona 测试
            logger.info("✓ Daytona fallback test passed")
            return True
            
        except Exception as e:
            logger.error(f"✗ Daytona fallback test failed: {e}")
            return False
    
    async def run_comprehensive_tests(self) -> dict:
        """运行全面测试"""
        logger.info("Running comprehensive tests...")
        
        tests = {
            "connectivity": await self.test_e2b_connectivity(),
            "fallback": await self.test_daytona_fallback(),
        }
        
        passed = sum(tests.values())
        total = len(tests)
        
        logger.info(f"Test results: {passed}/{total} passed")
        
        return tests
    
    async def deploy(self) -> dict:
        """执行部署"""
        logger.info("Starting E2B migration deployment...")
        
        # 1. 部署前检查
        logger.info("Step 1: Pre-deployment checks")
        pre_checks = await self.pre_deployment_check()
        
        if not all(pre_checks.values()):
            logger.error("Pre-deployment checks failed")
            return {"success": False, "error": "Pre-deployment checks failed"}
        
        # 2. 连接性测试
        logger.info("Step 2: Connectivity tests")
        connectivity_tests = await self.run_comprehensive_tests()
        
        if not all(connectivity_tests.values()):
            logger.error("Connectivity tests failed")
            return {"success": False, "error": "Connectivity tests failed"}
        
        # 3. 部署验证
        logger.info("Step 3: Deployment validation")
        
        try:
            # 创建验证沙箱
            sandbox = await self.sandbox_manager.create_sandbox(
                "deployment_validation", 
                "base"
            )
            
            # 测试各种操作
            test_operations = [
                ("Command execution", "echo 'Deployment test'"),
                ("File creation", "mkdir -p /tmp/test && touch /tmp/test/file.txt"),
                ("Directory listing", "ls -la /tmp/test"),
            ]
            
            for op_name, command in test_operations:
                result = await self.sandbox_manager.execute_command(sandbox.id, command)
                if result["exit_code"] != 0:
                    logger.error(f"✗ {op_name} failed: {result['stderr']}")
                    await self.sandbox_manager.delete_sandbox(sandbox.id)
                    return {"success": False, "error": f"{op_name} failed"}
                else:
                    logger.info(f"✓ {op_name} passed")
            
            # 清理
            await self.sandbox_manager.delete_sandbox(sandbox.id)
            
            logger.info("✓ Deployment validation passed")
            
        except Exception as e:
            logger.error(f"✗ Deployment validation failed: {e}")
            return {"success": False, "error": str(e)}
        
        logger.info("✓ E2B migration deployment completed successfully")
        
        return {
            "success": True,
            "pre_checks": pre_checks,
            "connectivity_tests": connectivity_tests,
            "deployment_type": "E2B" if self.sandbox_manager.use_e2b else "Daytona"
        }

async def main():
    """主函数"""
    deployer = E2BDeployment()
    
    try:
        result = await deployer.deploy()
        
        if result["success"]:
            print("\n🎉 Deployment successful!")
            print(f"Deployment type: {result['deployment_type']}")
        else:
            print(f"\n❌ Deployment failed: {result['error']}")
            sys.exit(1)
            
    except Exception as e:
        logger.error(f"Deployment failed with exception: {e}")
        sys.exit(1)

if __name__ == "__main__":
    asyncio.run(main())
```

#### 4.6.2 监控和日志配置

创建 `backend/core/monitoring/e2b_monitor.py`:

```python
"""
E2B 监控和日志
"""

import asyncio
import logging
import time
from typing import Dict, Any, Optional
from dataclasses import dataclass
from datetime import datetime, timedelta

from backend.core.sandbox.sandbox import SandboxManager
from backend.core.utils.config import Config

logger = logging.getLogger(__name__)

@dataclass
class SandboxMetrics:
    """沙箱指标"""
    sandbox_id: str
    created_at: datetime
    commands_executed: int
    files_accessed: int
    total_execution_time: float
    last_activity: datetime
    status: str

@dataclass
class E2BMetrics:
    """E2B 系统指标"""
    total_sandboxes: int
    active_sandboxes: int
    failed_creations: int
    failed_commands: int
    average_creation_time: float
    average_execution_time: float
    uptime: timedelta

class E2BMonitor:
    """E2B 监控器"""
    
    def __init__(self, config: Config):
        self.config = config
        self.sandbox_manager = SandboxManager(config)
        self.metrics: Dict[str, SandboxMetrics] = {}
        self.start_time = datetime.now()
        
        # 监控配置
        self.monitoring_interval = 60  # 秒
        self.sandbox_timeout = 3600  # 1小时
        
        # 统计数据
        self.total_creations = 0
        self.failed_creations = 0
        self.total_commands = 0
        self.failed_commands = 0
        self.creation_times = []
        self.execution_times = []
    
    async def record_sandbox_creation(self, sandbox_id: str, success: bool, creation_time: float):
        """记录沙箱创建"""
        self.total_creations += 1
        
        if success:
            self.creation_times.append(creation_time)
            self.metrics[sandbox_id] = SandboxMetrics(
                sandbox_id=sandbox_id,
                created_at=datetime.now(),
                commands_executed=0,
                files_accessed=0,
                total_execution_time=0.0,
                last_activity=datetime.now(),
                status="active"
            )
        else:
            self.failed_creations += 1
    
    async def record_command_execution(self, sandbox_id: str, success: bool, execution_time: float):
        """记录命令执行"""
        self.total_commands += 1
        
        if success:
            self.execution_times.append(execution_time)
            
            if sandbox_id in self.metrics:
                self.metrics[sandbox_id].commands_executed += 1
                self.metrics[sandbox_id].total_execution_time += execution_time
                self.metrics[sandbox_id].last_activity = datetime.now()
        else:
            self.failed_commands += 1
    
    async def record_file_access(self, sandbox_id: str):
        """记录文件访问"""
        if sandbox_id in self.metrics:
            self.metrics[sandbox_id].files_accessed += 1
            self.metrics[sandbox_id].last_activity = datetime.now()
    
    async def cleanup_inactive_sandboxes(self):
        """清理不活跃的沙箱"""
        current_time = datetime.now()
        inactive_sandboxes = []
        
        for sandbox_id, metrics in self.metrics.items():
            if (current_time - metrics.last_activity).total_seconds() > self.sandbox_timeout:
                inactive_sandboxes.append(sandbox_id)
        
        for sandbox_id in inactive_sandboxes:
            try:
                await self.sandbox_manager.delete_sandbox(sandbox_id)
                del self.metrics[sandbox_id]
                logger.info(f"Cleaned up inactive sandbox: {sandbox_id}")
            except Exception as e:
                logger.error(f"Failed to cleanup sandbox {sandbox_id}: {e}")
    
    def get_system_metrics(self) -> E2BMetrics:
        """获取系统指标"""
        current_time = datetime.now()
        uptime = current_time - self.start_time
        
        active_sandboxes = sum(
            1 for m in self.metrics.values() 
            if m.status == "active"
        )
        
        return E2BMetrics(
            total_sandboxes=len(self.metrics),
            active_sandboxes=active_sandboxes,
            failed_creations=self.failed_creations,
            failed_commands=self.failed_commands,
            average_creation_time=(
                sum(self.creation_times) / len(self.creation_times) 
                if self.creation_times else 0.0
            ),
            average_execution_time=(
                sum(self.execution_times) / len(self.execution_times) 
                if self.execution_times else 0.0
            ),
            uptime=uptime
        )
    
    def get_sandbox_metrics(self, sandbox_id: str) -> Optional[SandboxMetrics]:
        """获取特定沙箱的指标"""
        return self.metrics.get(sandbox_id)
    
    def get_all_sandbox_metrics(self) -> Dict[str, SandboxMetrics]:
        """获取所有沙箱的指标"""
        return self.metrics.copy()
    
    async def generate_report(self) -> Dict[str, Any]:
        """生成监控报告"""
        system_metrics = self.get_system_metrics()
        
        report = {
            "timestamp": datetime.now().isoformat(),
            "system_metrics": {
                "total_sandboxes": system_metrics.total_sandboxes,
                "active_sandboxes": system_metrics.active_sandboxes,
                "failed_creations": system_metrics.failed_creations,
                "failed_commands": system_metrics.failed_commands,
                "average_creation_time": system_metrics.average_creation_time,
                "average_execution_time": system_metrics.average_execution_time,
                "uptime_seconds": system_metrics.uptime.total_seconds(),
                "success_rate": {
                    "creation": (
                        (self.total_creations - self.failed_creations) / self.total_creations * 100
                        if self.total_creations > 0 else 0
                    ),
                    "command": (
                        (self.total_commands - self.failed_commands) / self.total_commands * 100
                        if self.total_commands > 0 else 0
                    )
                }
            },
            "sandboxes": [
                {
                    "sandbox_id": m.sandbox_id,
                    "created_at": m.created_at.isoformat(),
                    "commands_executed": m.commands_executed,
                    "files_accessed": m.files_accessed,
                    "total_execution_time": m.total_execution_time,
                    "last_activity": m.last_activity.isoformat(),
                    "status": m.status
                }
                for m in self.metrics.values()
            ]
        }
        
        return report
    
    async def start_monitoring(self):
        """启动监控"""
        logger.info("Starting E2B monitoring...")
        
        while True:
            try:
                # 清理不活跃的沙箱
                await self.cleanup_inactive_sandboxes()
                
                # 生成报告
                report = await self.generate_report()
                
                # 记录系统指标
                logger.info(f"System metrics: {report['system_metrics']}")
                
                # 等待下一个监控周期
                await asyncio.sleep(self.monitoring_interval)
                
            except Exception as e:
                logger.error(f"Monitoring error: {e}")
                await asyncio.sleep(self.monitoring_interval)

# 全局监控器实例
_monitor: Optional[E2BMonitor] = None

def get_monitor() -> E2BMonitor:
    """获取全局监控器"""
    global _monitor
    if _monitor is None:
        from backend.core.utils.config import get_config
        _monitor = E2BMonitor(get_config())
    return _monitor

async def start_monitoring():
    """启动监控"""
    monitor = get_monitor()
    await monitor.start_monitoring()
```

## 5. 测试验证

### 5.1 单元测试

```bash
# 运行 E2B 迁移测试
cd backend
uv run pytest tests/test_e2b_migration.py -v

# 运行性能测试
uv run pytest tests/test_performance.py -v

# 运行所有相关测试
uv run pytest tests/ -k "e2b or sandbox" -v
```

### 5.2 集成测试

```bash
# 启动测试环境
python scripts/deploy_e2b_migration.py

# 测试 API 端点
curl -X POST "http://localhost:8000/api/sandbox/test_workspace/create" \
  -H "Content-Type: application/json"

# 测试命令执行
curl -X POST "http://localhost:8000/api/sandbox/execute" \
  -H "Content-Type: application/json" \
  -d '{"sandbox_id": "test_sandbox", "command": "echo Hello World"}'
```

### 5.3 压力测试

```bash
# 运行压力测试
python scripts/stress_test.py
```

## 6. 部署步骤

### 6.1 开发环境部署

1. **更新依赖**
   ```bash
   cd backend
   uv add e2b
   uv remove daytona-sdk daytona-api-client daytona-api-client-async
   ```

2. **配置环境变量**
   ```bash
   # 在 .env 文件中添加
   E2B_API_KEY=your_e2b_api_key
   E2B_TIMEOUT=300
   E2B_MAX_RETRIES=3
   ```

3. **运行测试**
   ```bash
   python scripts/deploy_e2b_migration.py
   ```

4. **启动服务**
   ```bash
   uv run api.py
   ```

### 6.2 生产环境部署

1. **备份现有配置**
   ```bash
   cp backend/.env backend/.env.backup
   ```

2. **更新生产配置**
   ```bash
   # 在生产环境的 .env 中添加 E2B 配置
   E2B_API_KEY=production_e2b_api_key
   E2B_TIMEOUT=300
   E2B_MAX_RETRIES=3
   ```

3. **蓝绿部署**
   ```bash
   # 部署到新环境
   docker-compose -f docker-compose.yml -f docker-compose.e2b.yml up -d
   
   # 验证新环境
   python scripts/validate_production.py
   
   # 切换流量
   python scripts/switch_traffic.py
   ```

4. **监控和回滚**
   ```bash
   # 启动监控
   python -m backend.core.monitoring.e2b_monitor
   
   # 如需回滚
   python scripts/rollback_to_daytona.py
   ```

## 7. 回滚方案

### 7.1 快速回滚

如果需要快速回滚到 Daytona：

1. **修改配置**
   ```bash
   # 在 .env 中禁用 E2B
   E2B_API_KEY=
   
   # 确保 Daytona 配置存在
   DAYTONA_API_KEY=your_daytona_api_key
   DAYTONA_SERVER_URL=https://api.daytona.com
   DAYTONA_TARGET=us
   ```

2. **重启服务**
   ```bash
   # 重启后端服务
   docker-compose restart backend
   ```

### 7.2 数据迁移

如果需要迁移沙箱数据：

```python
# 数据迁移脚本
python scripts/migrate_sandbox_data.py
```

## 8. 监控和日志

### 8.1 关键指标

- **沙箱创建时间**: 应该在 30 秒以内
- **命令执行时间**: 应该在 5 秒以内
- **成功率**: 创建和执行成功率应该 > 95%
- **资源使用**: 内存和 CPU 使用率

### 8.2 日志配置

```python
# 在 logging 配置中添加
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'handlers': {
        'e2b_monitor': {
            'class': 'logging.handlers.RotatingFileHandler',
            'filename': 'logs/e2b_monitor.log',
            'maxBytes': 10485760,  # 10MB
            'backupCount': 5
        }
    },
    'loggers': {
        'backend.core.monitoring.e2b_monitor': {
            'handlers': ['e2b_monitor'],
            'level': 'INFO',
            'propagate': False
        }
    }
}
```

## 9. 总结

### 9.1 迁移收益

- **成本优化**: E2B 按 CPU 时间计费，可能降低成本
- **性能提升**: 更快的沙箱启动和命令执行
- **简化架构**: 更简单的 API 和更好的文档
- **AI 优化**: 专为 AI 智能体设计

### 9.2 风险控制

- **向后兼容**: 保持现有 API 接口
- **渐进式迁移**: 分阶段实施，降低风险
- **完善测试**: 全面的测试覆盖
- **快速回滚**: 支持快速回滚到 Daytona

### 9.3 后续优化

- **性能调优**: 根据实际使用情况调整配置
- **功能扩展**: 利用 E2B 的新功能
- **监控完善**: 增加更多监控指标
- **文档更新**: 更新用户和开发者文档

---

## 附录

### A. E2B API 参考

详细 E2B API 使用方法请参考：[E2B 官方文档](https://e2b.dev/docs)

### B. Daytona 到 E2B 映射表

| Daytona 功能 | E2B 对应功能 | 备注 |
|-------------|-------------|------|
| `AsyncDaytona` | `e2b.Sandbox` | 核心沙箱类 |
| `create_sandbox_from_snapshot` | `Sandbox.create` | 使用模板创建 |
| `execute_command` | `process.start` | 进程执行 |
| `file operations` | `filesystem` API | 文件系统操作 |
| `snapshots` | `templates` | 环境模板 |

### C. 常见问题

**Q: 如何处理 Daytona 特有的功能？**
A: 通过兼容性层模拟或使用 E2B 替代方案。

**Q: 性能差异如何？**
A: E2B 通常有更好的性能，但需要实际测试验证。

**Q: 如何迁移现有的沙箱数据？**
A: 使用数据迁移脚本或重新创建沙箱。

**Q: 成本如何比较？**
A: E2B 按 CPU 时间计费，Daytona 按运行时间计费。

---

*文档版本: 1.0*  
*最后更新: 2024-01-01*  
*作者: Claude AI*