# Neo4j Schema Reference

## Entity Types

| Type | Properties |
|------|------------|
| `PullRequest` | title, description, number, author, createdAt, updatedAt, state |
| `File` | path, fileType, content |
| `Change` | linesAdded, linesRemoved, changeType, impact |
| `Component` | name, description, purpose |
| `Function` | name, signature, purpose, complexity |
| `Class` | name, properties, methods, purpose |
| `BusinessLogic` | name, purpose, impact, rules, validation |
| `Configuration` | name, parameters, values, environment |
| `Service` | name, endpoints, methods, authentication |
| `Dependency` | name, version, type, source |
| `Schema` | name, fields, relationships, validation |
| `Infrastructure` | name, resources, configurations, provider |
| `Pipeline` | name, stages, steps, triggers |

## Relation Types

| Type | From | To |
|------|------|-----|
| `MODIFIES` | PullRequest | File |
| `HAS_CHANGE` | File | Change |
| `CONTAINS` | File | Component/Function/Class |
| `DEPENDS_ON` | Component/Function/Class | Component/Function/Class |
| `REFERENCES` | Any | Any |
| `FIXES` | PullRequest | Issue |
| `IMPLEMENTS` | PullRequest | Component |
| `INTRODUCES` | PullRequest | Any |
| `MODIFIES_LOGIC` | PullRequest | BusinessLogic |
| `CONFIGURES` | PullRequest/Change | Configuration |
| `DEPENDS_ON_EXTERNAL` | Component/Service | Dependency |
| `IMPACTS` | Any | Any |

## Naming Prefixes

- `pr:{number}` - PullRequest
- `file:{path}` - File
- `bl:{name}` - BusinessLogic
- `config:{name}` - Configuration
- `service:{name}` - Service
- `dep:{name}` - Dependency
- `schema:{name}` - Schema
- `infra:{name}` - Infrastructure
