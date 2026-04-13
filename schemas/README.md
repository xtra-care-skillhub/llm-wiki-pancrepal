# Schemas

## Medical AST v0

- `medical_ast.node.schema.json`
- `medical_ast.edge.schema.json`
- `medical_ast.snapshot.schema.json`

设计原则：
1. graph 是派生层，必须可回溯到 `source_id + evidence_span`  
2. 强制 `provenance`：`extracted / inferred / ambiguous`  
3. 保留 `source_authority + evidence_grade` 用于治理与检索过滤  
