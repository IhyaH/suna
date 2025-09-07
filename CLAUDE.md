# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Suna is an open-source AI agent platform for building, managing, and training sophisticated autonomous agents. The platform consists of four main components: a Python/FastAPI backend, a Next.js/React frontend, isolated Docker agent runtimes, and a Supabase-powered data layer.

## Development Setup

### Environment Requirements
- Use `mise.toml` for tool version management (Node.js 20, Python 3.11.10, UV 0.6.5)
- Set `PYTHONPATH` to include backend directory: `{{config_root}}/backend`

### Initial Setup
Run the setup wizard to configure the platform automatically:
```bash
python setup.py
```
This wizard guides through 14 steps with progress saving for Supabase, Redis, LLM providers, and other services.

### Starting the Platform
Use the start script to manage services based on setup method (Docker or manual):
```bash
python start.py
```

### Manual Development Commands

#### Backend Development
```bash
# Run Redis only (Docker)
docker compose up redis -d

# Run API locally
cd backend
uv run api.py

# Run background worker (in separate terminal)
cd backend
uv run dramatiq --processes 4 --threads 4 run_agent_background

# Full Docker restart
docker compose down && docker compose up --build
```

#### Frontend Development
```bash
cd frontend
npm install
npm run dev          # Development server with Turbopack
npm run build        # Production build
npm run start        # Production server
npm run lint         # ESLint
npm run format       # Prettier formatting
npm run format:check # Check formatting
```

### Testing
Backend tests use pytest with async support:
```bash
cd backend
uv run pytest backend/core/tests/
```

## Architecture Overview

### Core Components

**Backend (Python/FastAPI)**
- Main API server: `backend/api.py`
- Agent execution engine: `backend/core/run.py`
- Thread management: `backend/core/agentpress/thread_manager.py`
- Tool system: `backend/core/tools/`
- Database migrations: `backend/supabase/migrations/`
- Background worker: `backend/run_agent_background.py`

**Frontend (Next.js/React)**
- App Router structure: `frontend/src/app/`
- Components: `frontend/src/components/`
- State management: TanStack Query + Zustand
- Forms: React Hook Form + Zod validation
- UI framework: shadcn/ui with Tailwind CSS

**Agent Runtime (Docker)**
- Isolated execution environments via Daytona
- Browser automation, code interpreter, file system access
- Security sandboxing and resource limits
- Tool integration and execution

**Database & Storage (Supabase)**
- PostgreSQL with Row Level Security (RLS)
- Authentication via Supabase Auth
- Real-time subscriptions and webhooks
- File storage and agent configuration

### Key Architectural Patterns

**Agent System**
- Multi-version agent support with `agent_versions` table
- Thread-based conversation management
- Tool execution with timeout and sandbox isolation
- Background job processing via Dramatiq
- Structured tool schemas with OpenAPI + XML decorators

**Tool Development**
- Extend `AgentBuilderBaseTool` for agent builder tools
- Extend `Tool` for general agent tools
- Use `@openapi_schema` decorator for tool schemas
- Return `ToolResult` with consistent structure
- Register tools using `AgentBuilderToolRegistry`

**LLM Integration**
- LiteLLM for multi-provider support (Anthropic, OpenAI, OpenRouter, Gemini, etc.)
- Structured prompts and output handling
- Rate limiting and retry logic
- Integration with Langfuse for tracing

**Security Model**
- JWT validation without signature verification (Supabase)
- Row Level Security (RLS) for all user-accessible tables
- Fernet encryption for stored credentials
- Input validation via Pydantic models
- CORS policies and rate limiting

## Important Configuration

### Backend Environment (.env)
```bash
# Database/Auth
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_ANON_KEY=your-anon-key
SUPABASE_SERVICE_ROLE_KEY=your-service-role-key

# Infrastructure
REDIS_HOST=localhost   # Use 'redis' for Docker-to-Docker
REDIS_PORT=6379
REDIS_SSL=false

# LLM Providers (at least one required)
ANTHROPIC_API_KEY=your-key
OPENAI_API_KEY=your-key
OPENROUTER_API_KEY=your-key
MODEL_TO_USE=openrouter/moonshotai/kimi-k2

# Web Services
TAVILY_API_KEY=your-key
FIRECRAWL_API_KEY=your-key
RAPID_API_KEY=your-key

# Agent Execution
DAYTONA_API_KEY=your-key
DAYTONA_SERVER_URL=https://app.daytona.io/api
DAYTONA_TARGET=us

# Security
MCP_CREDENTIAL_ENCRYPTION_KEY=your-generated-key
WEBHOOK_BASE_URL=https://yourdomain.com
```

### Frontend Environment (.env.local)
```bash
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key
NEXT_PUBLIC_BACKEND_URL=http://localhost:8000/api
NEXT_PUBLIC_URL=http://localhost:3000
NEXT_PUBLIC_ENV_MODE=LOCAL
```

## Development Workflow

### Code Standards

**Python Backend**
- Follow PEP 8 with black formatting
- Use Python 3.11+ features with comprehensive type hints
- Async/await for all I/O operations
- Pydantic models for data validation
- Structured logging with `structlog`
- Context managers for resource management

**TypeScript Frontend**
- Strict TypeScript with no `any` types
- Next.js App Router with file-based routing
- shadcn/ui components for consistent UI
- TanStack Query for server state
- React Hook Form + Zod for form handling

### Database Migrations
Migrations are idempotent and located in `backend/supabase/migrations/`. Follow established patterns:
```sql
BEGIN;

-- Create table with UUID primary keys
CREATE TABLE IF NOT EXISTS example_table (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Create indexes
CREATE INDEX IF NOT EXISTS idx_example_table_user_id ON example_table(user_id);

-- Enable RLS
ALTER TABLE example_table ENABLE ROW LEVEL SECURITY;
CREATE POLICY "Users can manage their own records" ON example_table
    FOR ALL USING (auth.uid() = user_id);

COMMIT;
```

### Tool Development Patterns
Tools extend base classes and use schema decorators:
```python
class ExampleTool(AgentBuilderBaseTool):
    @openapi_schema({
        "type": "function",
        "function": {
            "name": "example_action",
            "description": "Perform an example action",
            "parameters": {
                "type": "object",
                "properties": {
                    "param1": {"type": "string", "description": "Parameter description"},
                    "param2": {"type": "integer", "default": 0}
                },
                "required": ["param1"]
            }
        }
    })
    async def example_action(self, param1: str, param2: int = 0) -> ToolResult:
        try:
            result = await self.perform_action(param1, param2)
            return self.success_response(result=result, message=f"Success: {param1}")
        except Exception as e:
            return self.fail_response(f"Failed: {str(e)}")
```

## Common Commands

### Quick Development Start
```bash
# Start Redis
docker compose up redis -d

# Start API (terminal 1)
cd backend && uv run api.py

# Start worker (terminal 2)
cd backend && uv run dramatiq --processes 4 --threads 4 run_agent_background

# Start frontend (terminal 3)
cd frontend && npm run dev
```

### Production Deployment
```bash
# For production with resource limits
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

### Testing and Validation
```bash
# Frontend linting and formatting
cd frontend && npm run lint && npm run format:check

# Backend testing
cd backend && uv run pytest

# Type checking (if available)
cd frontend && npm run type-check
```

## Important Notes

- Use `redis` as host when services communicate via Docker
- Use `localhost` as host when running API locally against Docker Redis
- At least one LLM provider key is functionally required
- Daytona keys are required for agent sandbox functionality
- MCP credential encryption key is recommended for secure credential storage
- Follow existing patterns in the codebase for consistency
- Refer to Cursor rules in `.cursor/rules/` for detailed domain-specific guidance