---
name: file-pattern-detector
description: Detect file types, extract metadata, and identify code patterns. Invoke when you need to classify a file (React component, API endpoint, config, test, etc.) or extract functions, classes, imports, exports from code.
allowed-tools: Read, Grep
---

# File Pattern Detector Skill

You analyze files to detect their type and extract structural metadata for PR documentation.

## Capabilities

1. **File Type Detection**: Identify file category based on extension, path, and content patterns
2. **Code Structure Extraction**: Extract functions, classes, exports, imports
3. **Framework Detection**: Identify React, Next.js, Express, Terraform, etc.
4. **Configuration Recognition**: Detect config files and their purpose

## File Type Categories

- **Frontend**: React components (.tsx, .jsx), styles, Next.js pages/layouts
- **Backend**: API routes, controllers, services, middleware
- **Infrastructure**: Terraform (.tf), Kubernetes (.yaml), Docker files
- **Configuration**: JSON, YAML, ENV, package.json, tsconfig
- **Test**: .test.ts, .spec.ts, __tests__/
- **Documentation**: .md, README
- **Database**: Schema files, migrations, seeds

## Standard Output Format

Return structured metadata:

```json
{
  "file_type": "react_component",
  "category": "frontend",
  "framework": "next.js",
  "exports": ["HomePage", "getServerSideProps"],
  "imports": ["@/components/Header", "react"],
  "functions": ["handleSubmit", "validateForm"],
  "classes": [],
  "dependencies": ["react", "next"],
  "purpose": "Homepage component with server-side rendering"
}
```

## Pattern Recognition

**React Component:** Export of JSX, hooks usage, .tsx/.jsx extension
**API Endpoint:** Express routes, Next.js API routes, HTTP methods
**Config File:** .json/.yaml/.env in root or config/, known config patterns
**Infrastructure:** .tf extension, resource definitions, provider blocks
**Test File:** .test/.spec suffix, describe/it blocks, test assertions

## Usage

This skill is automatically invoked when analyzing files in PR workflows. Provide the file path and content, receive structured metadata.
