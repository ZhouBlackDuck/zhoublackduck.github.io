# 工作流程
```mermaid
graph TD
    source["`***main*** /src`"]-->build["build"]
    build-- push -->dist["`***out*** /dist`"]
    dist-->deploy["deploy"]
```