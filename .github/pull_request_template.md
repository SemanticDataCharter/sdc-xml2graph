# Pull Request

## Description

**What does this PR do?**

## Type of Change

- [ ] Bug fix (non-breaking)
- [ ] New feature (non-breaking)
- [ ] Breaking change
- [ ] Documentation update
- [ ] Performance improvement
- [ ] Code refactoring
- [ ] Test improvement
- [ ] Architecture implementation

## Related Issues

**Closes #(issue number)**
**Related to #(issue number)**

## Changes Made

1.
2.
3.

## Graph Format Impact

Which graph formats does this affect?

- [ ] RDF generation (N-Quads, Turtle, JSON-LD)
- [ ] GQL generation (ISO GQL)
- [ ] Cypher generation (Neo4j)
- [ ] Format-agnostic (core functionality)

## Testing

### Test Coverage

- [ ] All tests pass: `pytest`
- [ ] Coverage maintained/improved: `pytest --cov`
- [ ] Added tests for new code
- [ ] Integration tests with sdcvalidator

### Quality Checks

- [ ] Linting passes: `flake8 sdc_xml2graph/`
- [ ] Type checking passes: `mypy sdc_xml2graph/`
- [ ] Formatting correct: `black --check sdc_xml2graph/`
- [ ] Import sorting correct: `isort --check sdc_xml2graph/`

### Manual Testing

```python
# Example of manual testing performed
from sdcvalidator import SDC4Validator
from sdc_xml2graph import XML2Graph

validator = SDC4Validator('test.xsd')
recovered = validator.validate_with_recovery('test.xml')

converter = XML2Graph('test.xsd')
graph = converter.to_rdf(recovered)
# ... test output
```

**Test Results:**

## Integration Testing

### With sdcvalidator

- [ ] Tested with valid SDC4 instances
- [ ] Tested with invalid instances (ExceptionalValue)
- [ ] Tested with various SDC4 component types

### With Graph Databases (if applicable)

- [ ] Tested loading to Fuseki
- [ ] Tested loading to Neo4j
- [ ] Tested SPARQL/GQL queries on generated data

## Documentation

- [ ] Updated README if needed
- [ ] Updated docstrings (Google style)
- [ ] Updated architecture docs in `docs/`
- [ ] Added usage examples if applicable
- [ ] Updated CHANGELOG.md

## Checklist

- [ ] I have read [CONTRIBUTING.md](../CONTRIBUTING.md)
- [ ] I have read [CLAUDE.md](../CLAUDE.md)
- [ ] My code follows PEP 8 and project style guide
- [ ] I have added type hints to all functions
- [ ] I have added tests for my changes
- [ ] All tests pass locally
- [ ] I have updated documentation
- [ ] My code includes proper error handling
- [ ] I have considered security implications

## Security Considerations

**If this PR affects security-sensitive areas:**

- [ ] XML parsing: Safe parser configuration
- [ ] Graph generation: Injection prevention
- [ ] URI handling: Validation implemented
- [ ] Resource limits: Appropriate constraints
- [ ] No secrets in code or tests

**Security review needed?**
- [ ] Yes
- [ ] No

## Breaking Changes

**If breaking change, explain:**

- What breaks?
- Migration guide for users?
- Version bump required?

## Performance Impact

**If affects performance:**

- [ ] Benchmarked before/after
- [ ] Performance improved by: X%
- [ ] Performance regression justified because:
- [ ] Memory usage tested

**Benchmark Results:**
```
# Paste benchmark results if applicable
```

## Example Output

**If this changes graph generation, show example output:**

### RDF Output (if applicable)
<details>
<summary>example.nq</summary>

```turtle
<!-- Paste example RDF output -->
```

</details>

### GQL Output (if applicable)
<details>
<summary>example.gql</summary>

```gql
// Paste example GQL output
```

</details>

## Screenshots or Diagrams (if applicable)

**CLI output, architecture diagrams, etc.**

## Dependencies

**New dependencies added:**

- Package: version (reason)

**Dependency updates:**

- Package: old-version → new-version (reason)

## Deployment Notes

**Any special deployment considerations?**

## Reviewer Guidance

**What should reviewers focus on?**

1.
2.
3.

## Additional Notes

**Any other context for reviewers**

---

**For Maintainers:**

- [ ] Version bump needed (patch/minor/major)
- [ ] Changelog updated
- [ ] Release notes drafted
- [ ] Documentation site updated
