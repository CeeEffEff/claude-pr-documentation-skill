---
description: Base Neo4j schema with common entity and relation types for all workflows
globs: **/*
alwaysApply: true
---

# Base Neo4j Memory Server Schema

This document defines the common entity types and relation types that can be used with the Neo4j memory server across all workflows.

## Common Entity Types

| Type | Description |
|------|-------------|
| `Schema` | Represents data models or schemas |
| `File` | Represents a file in the repository |

## Common Relation Types

| Type | Description |
|------|-------------|
| `CONTAINS` | Indicates a container relationship |
| `REFERENCES` | Indicates a reference relationship |

## Usage Guidelines

- This base schema contains only the most common entity and relation types
- For workflow-specific entity and relation types, refer to the appropriate schema file:
  - PR Documentation: `.clinerules/neo4j-schema-pr.md`
  - Terraform Dependencies: `.clinerules/neo4j-schema-terraform.md`
- Any new types must get approval from a user before adding and use
