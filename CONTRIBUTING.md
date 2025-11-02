# Contributing to sdc-xml2graph

Thank you for your interest in contributing to **sdc-xml2graph**! This document provides guidelines for contributing to this project.

## Project Status

🚧 **This project is currently in the planning phase** (Q1-2026 development start).

We welcome:
- **Architecture feedback** - Suggestions on the planned design
- **Use case input** - How you plan to use this tool
- **Graph mapping ideas** - RDF/GQL generation strategies
- **Early code contributions** - Help us build the foundation

## Table of Contents

- [Code of Conduct](#code-of-conduct)
- [Getting Started](#getting-started)
- [Development Setup](#development-setup)
- [Coding Standards](#coding-standards)
- [Testing Requirements](#testing-requirements)
- [Pull Request Process](#pull-request-process)
- [Architecture Guidelines](#architecture-guidelines)
- [Release Process](#release-process)

## Code of Conduct

We are committed to providing a welcoming and inclusive environment. Please be respectful and professional in all interactions.

## Getting Started

### Prerequisites

- **Python**: 3.9 or higher
- **git**: For version control
- **sdcvalidator**: Understanding of SDC4 validation
- **Graph databases**: Familiarity with RDF or property graphs helpful

### Project Goals

1. **Validate First**: Always use sdcvalidator before graph generation
2. **Preserve Semantics**: Maintain SDC4 semantic meaning in graphs
3. **Quality Indicators**: Preserve ExceptionalValue information
4. **Multiple Formats**: Support RDF (N-Quads) and GQL (property graphs)
5. **Performance**: Handle large SDC4 instances efficiently

## Development Setup

### 1. Fork and Clone

```bash
# Fork on GitHub first, then:
git clone git@github.com:YOUR-USERNAME/sdc-xml2graph.git
cd sdc-xml2graph
```

### 2. Create Virtual Environment

```bash
# Using venv
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Or using conda
conda create -n sdc-xml2graph python=3.11
conda activate sdc-xml2graph
```

### 3. Install Dependencies (When Available)

```bash
# Install in development mode
pip install -e ".[dev]"

# Or install from requirements files
pip install -r requirements.txt
pip install -r requirements-dev.txt
```

### 4. Verify Installation

```bash
# Run tests (when available)
pytest

# Check code style
black --check sdc_xml2graph/
flake8 sdc_xml2graph/
mypy sdc_xml2graph/
```

## Coding Standards

### Python Style

We follow **PEP 8** with these specific guidelines:

#### Code Formatting

- **Black** for automatic formatting (line length: 88)
- **isort** for import sorting
- **flake8** for linting

```bash
# Format code
black sdc_xml2graph/ tests/
isort sdc_xml2graph/ tests/

# Check compliance
flake8 sdc_xml2graph/
```

#### Type Hints

All code must include type hints:

```python
from typing import Optional, Dict, List
from rdflib import Graph
from lxml import etree

def parse_xml_instance(
    xml_path: str,
    schema_path: str
) -> etree._Element:
    """Parse and validate SDC4 XML instance.

    Args:
        xml_path: Path to XML instance file
        schema_path: Path to SDC4 schema (XSD)

    Returns:
        Validated XML element tree

    Raises:
        ValidationError: If XML doesn't conform to schema
    """
    ...
```

Run type checking:
```bash
mypy sdc_xml2graph/
```

#### Docstrings

Use **Google style** docstrings:

```python
def to_rdf(
    xml_tree: etree._Element,
    format: str = "nquads"
) -> Graph:
    """Convert SDC4 XML to RDF graph.

    Args:
        xml_tree: Validated SDC4 XML element tree
        format: Output format (nquads, turtle, json-ld)

    Returns:
        RDF graph with SDC4 semantics

    Raises:
        GraphGenerationError: If conversion fails

    Example:
        >>> from sdc_xml2graph import XML2Graph
        >>> converter = XML2Graph('schema.xsd')
        >>> graph = converter.to_rdf('instance.xml')
        >>> print(graph.serialize(format='nquads'))
    """
    ...
```

### Project Structure

```
sdc-xml2graph/
├── sdc_xml2graph/          # Main package
│   ├── __init__.py
│   ├── parser.py           # XML parsing
│   ├── semantic.py         # Semantic mapping
│   ├── generators/         # Graph generators
│   │   ├── __init__.py
│   │   ├── rdf_generator.py
│   │   ├── gql_generator.py
│   │   └── cypher_generator.py
│   ├── cli.py              # Command-line interface
│   └── utils.py            # Utilities
├── tests/                  # Test suite
│   ├── test_parser.py
│   ├── test_rdf_generator.py
│   ├── test_gql_generator.py
│   ├── test_cli.py
│   └── fixtures/           # Test data
│       ├── schemas/
│       └── instances/
├── docs/                   # Documentation
│   ├── architecture.md
│   ├── rdf-mapping.md
│   └── gql-mapping.md
├── examples/               # Usage examples
├── setup.py                # Package configuration
├── pyproject.toml          # Build configuration
├── requirements.txt        # Production dependencies
├── requirements-dev.txt    # Development dependencies
└── README.md
```

### Import Ordering

Use **isort** with these settings (in `pyproject.toml`):

```toml
[tool.isort]
profile = "black"
line_length = 88
```

Example:
```python
# Standard library
import os
from typing import Dict, List

# Third-party
from lxml import etree
from rdflib import Graph, Namespace, URIRef, Literal

# Local
from sdc_xml2graph.parser import SDC4Parser
from sdc_xml2graph.exceptions import GraphGenerationError
```

## Testing Requirements

### Test Framework

We use **pytest** for all testing.

### Coverage Requirements

- **Minimum**: 85% overall coverage
- **Graph generators**: 95% coverage (critical path)
- **CLI**: 80% coverage

### Running Tests

```bash
# Run all tests
pytest

# Run with coverage
pytest --cov=sdc_xml2graph --cov-report=html

# Run specific test file
pytest tests/test_rdf_generator.py

# Run specific test
pytest tests/test_rdf_generator.py::test_xdstring_to_rdf

# Run with verbose output
pytest -v

# Run failed tests only
pytest --lf
```

### Writing Tests

#### Unit Tests

```python
import pytest
from lxml import etree
from sdc_xml2graph.generators.rdf_generator import RDFGenerator

def test_xdstring_component_to_rdf():
    """Test RDF generation for XdString component."""
    # Arrange
    xml_string = """
    <XdString label="First Name">
        <value>John</value>
    </XdString>
    """
    element = etree.fromstring(xml_string)
    generator = RDFGenerator()

    # Act
    graph = generator.component_to_rdf(element)

    # Assert
    assert len(graph) > 0
    # Verify specific triples
    assert (subject, RDF.type, SDC.XdString) in graph
```

#### Integration Tests

```python
from sdcvalidator import SDC4Validator
from sdc_xml2graph import XML2Graph

def test_full_workflow_with_validation():
    """Test complete workflow: validate then convert."""
    # Validate with sdcvalidator
    validator = SDC4Validator('tests/fixtures/schemas/test.xsd')
    recovered = validator.validate_with_recovery(
        'tests/fixtures/instances/patient.xml'
    )

    # Convert to RDF
    converter = XML2Graph('tests/fixtures/schemas/test.xsd')
    graph = converter.to_rdf(recovered)

    # Verify graph contains expected triples
    assert len(graph) > 0
    # Check for ExceptionalValue preservation
    assert 'INV' in str(graph.serialize())
```

#### Fixtures

Use pytest fixtures for common test data:

```python
import pytest
from pathlib import Path

@pytest.fixture
def schema_path():
    """Provide test schema path."""
    return Path('tests/fixtures/schemas/test.xsd')

@pytest.fixture
def valid_instance():
    """Provide valid XML instance."""
    return Path('tests/fixtures/instances/valid.xml')

@pytest.fixture
def invalid_instance():
    """Provide invalid XML instance with ExceptionalValue."""
    return Path('tests/fixtures/instances/invalid.xml')

def test_with_fixtures(schema_path, valid_instance):
    """Test using fixtures."""
    converter = XML2Graph(schema_path)
    graph = converter.to_rdf(valid_instance)
    assert graph is not None
```

### Test Organization

Group related tests:

```python
class TestRDFGenerator:
    """Tests for RDF graph generation."""

    def test_xdstring_mapping(self):
        ...

    def test_xdcount_mapping(self):
        ...

    def test_cluster_hierarchy(self):
        ...

    def test_exceptional_value_preservation(self):
        ...

class TestGQLGenerator:
    """Tests for GQL property graph generation."""

    def test_node_creation(self):
        ...

    def test_relationship_creation(self):
        ...
```

## Pull Request Process

### 1. Create Feature Branch

```bash
git checkout -b feature/your-feature-name
# or
git checkout -b fix/bug-description
```

### 2. Make Changes

- Write code following style guidelines
- Add tests for new functionality
- Update documentation as needed
- Ensure all tests pass

### 3. Commit Changes

Use conventional commit messages:

```bash
git commit -m "feat: add RDF generation for XdTemporal components"
git commit -m "fix: handle empty ExceptionalValue elements"
git commit -m "docs: update RDF mapping documentation"
git commit -m "test: add integration tests for GQL generation"
```

**Commit types**:
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation only
- `test`: Adding/updating tests
- `refactor`: Code refactoring
- `perf`: Performance improvement
- `chore`: Maintenance tasks

### 4. Push and Create PR

```bash
git push origin feature/your-feature-name
```

Then create a Pull Request on GitHub.

### 5. PR Checklist

Ensure your PR includes:

- [ ] **Code changes** with proper style
- [ ] **Tests** for new functionality (85% coverage)
- [ ] **Documentation** updates (docstrings, guides)
- [ ] **Type hints** for all functions
- [ ] **CHANGELOG** entry (for significant changes)
- [ ] All **tests passing**
- [ ] All **linters passing** (black, flake8, mypy)

### 6. Code Review

- Address review feedback promptly
- Keep discussions focused and professional
- Update tests and documentation as requested

### 7. Merge

Once approved, maintainers will merge your PR.

## Architecture Guidelines

### Integration with sdcvalidator

**CRITICAL**: Always validate before generating graphs.

```python
# ❌ BAD - Bypasses validation
converter = XML2Graph('schema.xsd')
graph = converter.to_rdf('raw_instance.xml')

# ✅ GOOD - Validates first
from sdcvalidator import SDC4Validator

validator = SDC4Validator('schema.xsd')
recovered = validator.validate_with_recovery('instance.xml')

converter = XML2Graph('schema.xsd')
graph = converter.to_rdf(recovered)
```

### ExceptionalValue Preservation

Always preserve ExceptionalValue information in generated graphs:

**RDF Example**:
```python
def map_exceptional_value_to_rdf(
    component_uri: URIRef,
    ev_element: etree._Element,
    graph: Graph
) -> None:
    """Add ExceptionalValue triples to graph."""
    ev_type = ev_element.get('type')  # e.g., 'INV'
    original_value = ev_element.text

    ev_node = BNode()  # Blank node for ExceptionalValue
    graph.add((component_uri, SDC.hasExceptionalValue, ev_node))
    graph.add((ev_node, RDF.type, SDC[ev_type]))
    graph.add((ev_node, SDC.originalValue, Literal(original_value)))
```

### Graph URI Strategy

Use consistent URI patterns:

```python
# Data Model
f"urn:sdc:dm:{dm_id}"

# Cluster
f"urn:sdc:cluster:{cluster_id}"

# Component
f"urn:sdc:component:{component_id}"

# ExceptionalValue (blank node or)
f"urn:sdc:ev:{component_id}:{index}"
```

### Performance Optimization

- **Stream large files** instead of loading entirely into memory
- **Batch triple generation** for RDF graphs
- **Use lxml efficiently** (avoid unnecessary tree traversals)
- **Profile critical paths** with `cProfile`

## Release Process

### Versioning

Follow SDC ecosystem semantic versioning:

- **MAJOR**: SDC reference model version (4 for SDC4)
- **MINOR**: New features
- **PATCH**: Bug fixes

Example: `4.0.0` → `4.1.0` (new feature) → `4.1.1` (bug fix)

### Release Checklist

1. Update version in `setup.py` and `sdc_xml2graph/__init__.py`
2. Update `CHANGELOG.md`
3. Run full test suite: `pytest`
4. Build package: `python -m build`
5. Tag release: `git tag v4.0.0`
6. Push tag: `git push origin v4.0.0`
7. Publish to PyPI: `python -m twine upload dist/*`
8. Create GitHub release

## Documentation

### User Documentation

Update relevant documentation in `docs/`:

- **Architecture** - System design decisions
- **RDF Mapping** - How SDC4 maps to RDF
- **GQL Mapping** - How SDC4 maps to property graphs
- **CLI Usage** - Command-line examples

### API Documentation

Generate API docs from docstrings:

```bash
# Using sphinx (when set up)
cd docs/
make html
```

### Examples

Add working examples to `examples/`:

```python
# examples/basic_rdf_generation.py
from sdcvalidator import SDC4Validator
from sdc_xml2graph import XML2Graph

# Validate
validator = SDC4Validator('schema.xsd')
recovered = validator.validate_with_recovery('patient.xml')

# Convert to RDF
converter = XML2Graph('schema.xsd')
graph = converter.to_rdf(recovered)

# Save as N-Quads
with open('output.nq', 'w') as f:
    f.write(graph.serialize(format='nquads'))
```

## Getting Help

- **GitHub Issues**: [Report bugs or request features](https://github.com/SemanticDataCharter/sdc-xml2graph/issues)
- **GitHub Discussions**: [Ask questions and share ideas](https://github.com/SemanticDataCharter/sdc-xml2graph/discussions)
- **Email**: contact@axius-sdc.com

## Recognition

Contributors will be recognized in:
- `CONTRIBUTORS.md` file
- GitHub releases
- Project documentation

Thank you for contributing to the SDC4 ecosystem!

---

**Questions?** Open an issue or start a discussion. We're here to help!
