---
name: Bug Report
about: Report a bug in sdc-xml2graph
title: '[BUG] '
labels: bug
assignees: ''
---

## Bug Description

**Clear and concise description of the bug**

## Environment

- **sdc-xml2graph version**: (e.g., 4.0.0)
- **Python version**: (e.g., 3.11.0)
- **Operating System**: (e.g., Ubuntu 22.04, Windows 11, macOS 14)
- **sdcvalidator version**: (e.g., 4.0.1)
- **lxml version**: (e.g., 4.9.3)
- **rdflib version** (if applicable): (e.g., 6.3.2)

## Steps to Reproduce

```python
# Minimal code to reproduce
from sdcvalidator import SDC4Validator
from sdc_xml2graph import XML2Graph

validator = SDC4Validator('schema.xsd')
recovered = validator.validate_with_recovery('instance.xml')

converter = XML2Graph('schema.xsd')
# ... rest of code
```

1. Step 1
2. Step 2
3. Step 3

## Expected Behavior

What should happen:

## Actual Behavior

What actually happens:

## Error Output

<details>
<summary>Full error message and stack trace</summary>

```
Paste error output here
```

</details>

## Additional Context

**SDC4 Schema** (if relevant):
<details>
<summary>schema.xsd</summary>

```xml
<!-- Paste or attach schema -->
```

</details>

**XML Instance** (if relevant):
<details>
<summary>instance.xml</summary>

```xml
<!-- Paste or attach instance -->
```

</details>

**Generated Graph** (if relevant):
<details>
<summary>output.nq or output.gql</summary>

```
<!-- Paste generated graph output -->
```

</details>

## Graph Format

Which format were you generating?
- [ ] RDF (N-Quads)
- [ ] RDF (Turtle)
- [ ] RDF (JSON-LD)
- [ ] GQL (ISO GQL)
- [ ] Cypher (Neo4j)

## Possible Solution

If you have ideas for fixing this:
