# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**sdc-xml2graph** is a planned Python tool (Q1-2026) for transforming validated SDC4 XML instances into knowledge graphs. It bridges SDC4 validation and graph database technologies by converting semantic data models into both RDF graphs (semantic web) and property graphs (GQL).

**Primary Use Case**: Enable SDC4-compliant data to be ingested into graph databases (Apache Jena/Fuseki, Neo4j, Amazon Neptune, etc.) while preserving semantic meaning and data quality indicators.

**Project Status**: 🚧 **Aspirational / Planning Phase**

This repository contains the vision and planned architecture for a future tool. Development will begin Q1-2026.

### Semantic Versioning Convention

Following the SDC ecosystem convention:

**Format**: `MAJOR.MINOR.PATCH`

- **MAJOR** = SDC Reference Model version (will be **4** for SDC4)
- **MINOR** = Feature releases
- **PATCH** = Bug fixes

**Planned Version**: `4.0.0` (initial release with SDC4 support)

## Planned Architecture

### Core Workflow

```
┌─────────────────┐
│  SDC4 Schema    │ (XSD)
│  + XML Instance │
└────────┬────────┘
         │
         ▼
┌─────────────────────────┐
│  sdcvalidator           │ ← Python validation library
│  ✓ Validate against XSD │
│  ✓ Inject ExceptionalValue │
└────────┬────────────────┘
         │
         ▼
┌─────────────────────────┐
│  sdc-xml2graph          │ ← This tool
│  • Parse validated XML  │
│  • Extract semantics    │
│  • Generate graph data  │
└────────┬────────────────┘
         │
         ├──────────┬──────────┐
         ▼          ▼          ▼
    ┌────────┐ ┌────────┐ ┌─────────┐
    │ N-Quads│ │  GQL   │ │ Cypher  │
    │  (RDF) │ │(ISO GQL)│ │ (Neo4j) │
    └────────┘ └────────┘ └─────────┘
         │          │          │
         ▼          ▼          ▼
    ┌────────┐ ┌────────┐ ┌─────────┐
    │ Fuseki │ │Neptune │ │  Neo4j  │
    └────────┘ └────────┘ └─────────┘
```

### Key Components (Planned)

1. **XML Parser** (`sdc_xml2graph/parser.py`)
   - Parse validated SDC4 XML instances
   - Extract component hierarchy (Cluster, Xd* types)
   - Identify ExceptionalValue elements
   - Build internal graph representation

2. **Graph Generator** (`sdc_xml2graph/generators/`)
   - `rdf_generator.py` - N-Quads/Turtle/JSON-LD output
   - `gql_generator.py` - ISO GQL CREATE statements
   - `cypher_generator.py` - Neo4j Cypher (optional)

3. **Semantic Mapper** (`sdc_xml2graph/semantic.py`)
   - Map SDC4 components to RDF ontology
   - Preserve ISO 21090 ExceptionalValue semantics
   - Link to external ontologies (SNOMED, LOINC, etc.)

4. **CLI Tool** (`sdc_xml2graph/cli.py`)
   - Command-line interface
   - Format selection (RDF, GQL, Cypher)
   - Batch processing support

### Technology Stack (Planned)

- **Python**: 3.9+ (matching sdcvalidator)
- **XML Processing**: `lxml` (consistent with sdcvalidator)
- **RDF Generation**: `rdflib` for RDF graph manipulation
- **Graph Query**: ISO GQL standard support
- **Testing**: `pytest` with 85% minimum coverage
- **CLI**: `click` or `argparse`
- **Packaging**: `setuptools` with `setup.py` + `pyproject.toml`

## Integration with SDC4 Ecosystem

### Dependency on sdcvalidator

**CRITICAL**: sdc-xml2graph depends on **sdcvalidator** for validation:

```python
from sdcvalidator import SDC4Validator

# Validate first
validator = SDC4Validator('schema.xsd')
recovered_tree = validator.validate_with_recovery('instance.xml')

# Then convert to graph
from sdc_xml2graph import XML2Graph

converter = XML2Graph(schema='schema.xsd')
graph_data = converter.to_rdf(recovered_tree)  # or .to_gql()
```

**Design Principle**: Never bypass validation. Always use sdcvalidator to:
1. Validate against SDC4 schema
2. Inject ExceptionalValue elements for invalid data
3. Ensure semantic quality before graph generation

### Ecosystem Integration

- **[SDCRM](https://github.com/SemanticDataCharter/SDCRM)** v4.0.0 - Provides SDC4 schemas and ontologies
- **[sdcvalidator](https://github.com/Axius-SDC/sdcvalidator)** v4.0.1 - Validation dependency
- **[SDCStudio](https://github.com/AxiusSDC/SDCStudio)** v4.0.0 - Will use this tool for graph export
- **[Obsidian Template](https://github.com/SemanticDataCharter/SDCObsidianTemplate)** v4.0.0 - Documentation templates

## RDF Graph Generation (Planned)

### SDC4 Ontology Mapping

Each SDC4 component type maps to RDF:

```turtle
# Example: XdString component
@prefix sdc: <http://semanticdatacharter.org/ns/sdc4/> .
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .

<urn:sdc:dm:patient-record:001>
  rdf:type sdc:DataModel ;
  sdc:hasCluster <urn:sdc:cluster:demographics> .

<urn:sdc:cluster:demographics>
  rdf:type sdc:Cluster ;
  sdc:hasComponent <urn:sdc:component:first-name> .

<urn:sdc:component:first-name>
  rdf:type sdc:XdString ;
  sdc:label "First Name"@en ;
  sdc:value "John" ;
  sdc:cardinality "1..1" .
```

### ExceptionalValue Preservation

Invalid data with ExceptionalValue elements must be preserved in the graph:

```turtle
<urn:sdc:component:age>
  rdf:type sdc:XdCount ;
  sdc:label "Age"@en ;
  sdc:hasExceptionalValue [
    rdf:type sdc:INV ;  # Invalid
    sdc:originalValue "not-a-number" ;
    sdc:reason "Value does not conform to xsd:integer" ;
    sdc:injectedBy "sdcvalidator/4.0.1"
  ] .
```

This ensures data quality issues are visible in graph queries.

## GQL Generation (Planned)

### ISO GQL Property Graph

Generate GQL CREATE statements for property graphs:

```gql
// Data Model node
CREATE (:DataModel {
  id: 'dm:patient-record:001',
  label: 'Patient Record',
  version: '1.0.0'
})

// Cluster node
CREATE (:Cluster {
  id: 'cluster:demographics',
  label: 'Demographics',
  cardinality: '1..1'
})

// Component node with ExceptionalValue
CREATE (:XdCount {
  id: 'component:age',
  label: 'Age',
  exceptional_value_type: 'INV',
  original_value: 'not-a-number',
  exceptional_value_reason: 'Value does not conform to xsd:integer'
})

// Relationships
CREATE (:DataModel {id: 'dm:patient-record:001'})
  -[:HAS_CLUSTER]->
  (:Cluster {id: 'cluster:demographics'})
  -[:HAS_COMPONENT]->
  (:XdCount {id: 'component:age'})
```

## Planned CLI Usage

```bash
# Install (future)
pip install sdc-xml2graph

# Validate and convert to RDF (N-Quads)
sdc-xml2graph instance.xml --schema schema.xsd --format nquads -o output.nq

# Convert to GQL
sdc-xml2graph instance.xml -s schema.xsd --format gql -o output.gql

# Convert to Cypher for Neo4j
sdc-xml2graph instance.xml -s schema.xsd --format cypher -o output.cypher

# Batch processing
sdc-xml2graph instances/*.xml -s schema.xsd --format nquads --output-dir graphs/

# Load directly to Fuseki
sdc-xml2graph instance.xml -s schema.xsd --fuseki http://localhost:3030/dataset
```

## Planned API Usage

```python
from sdc_xml2graph import XML2Graph

# Initialize converter
converter = XML2Graph(schema='schema.xsd')

# Convert to RDF graph (rdflib Graph object)
rdf_graph = converter.to_rdf('instance.xml')

# Serialize to N-Quads
nquads = rdf_graph.serialize(format='nquads')

# Convert to GQL statements
gql_statements = converter.to_gql('instance.xml')

# Direct upload to triple store
converter.to_fuseki(
    'instance.xml',
    endpoint='http://localhost:3030/dataset'
)
```

## Development Roadmap

### Phase 1: Foundation (Q1-2026)
- [ ] Project structure and package setup
- [ ] XML parser for SDC4 components
- [ ] Internal graph representation
- [ ] Integration with sdcvalidator

### Phase 2: RDF Generation (Q1-2026)
- [ ] RDF ontology mapping
- [ ] N-Quads output
- [ ] Turtle output
- [ ] JSON-LD output
- [ ] ExceptionalValue preservation

### Phase 3: GQL Generation (Q2-2026)
- [ ] ISO GQL CREATE statements
- [ ] Property graph schema
- [ ] Neo4j Cypher (optional)

### Phase 4: CLI and Tools (Q2-2026)
- [ ] Command-line interface
- [ ] Batch processing
- [ ] Fuseki integration
- [ ] Configuration files

### Phase 5: Testing & Documentation (Q2-2026)
- [ ] Comprehensive test suite (85% coverage)
- [ ] User guides
- [ ] API documentation
- [ ] Example notebooks

## Code Style and Conventions

### Python Best Practices

- **PEP 8** compliance (enforced by `flake8`)
- **Type hints** throughout (`mypy` checking)
- **Docstrings** in Google style
- **Black** for code formatting
- **isort** for import sorting

### Testing Requirements

- **pytest** for all tests
- **Minimum coverage**: 85% overall
- **Critical path coverage**: 95% for graph generation
- **Test structure**:
  ```
  tests/
  ├── test_parser.py
  ├── test_rdf_generator.py
  ├── test_gql_generator.py
  ├── test_cli.py
  └── fixtures/
      ├── schemas/
      └── instances/
  ```

### Error Handling

```python
class XML2GraphError(Exception):
    """Base exception for sdc-xml2graph."""
    pass

class ValidationError(XML2GraphError):
    """Raised when XML validation fails."""
    pass

class GraphGenerationError(XML2GraphError):
    """Raised when graph generation fails."""
    pass
```

## Dependencies (Planned)

### Required
- `sdcvalidator>=4.0.0` - SDC4 validation
- `lxml>=4.9.0` - XML processing
- `rdflib>=6.0.0` - RDF graph manipulation
- `click>=8.0.0` - CLI framework

### Optional
- `SPARQLWrapper>=2.0.0` - Fuseki integration
- `neo4j>=5.0.0` - Neo4j driver

### Development
- `pytest>=7.0.0`
- `pytest-cov>=4.0.0`
- `black>=23.0.0`
- `flake8>=6.0.0`
- `mypy>=1.0.0`
- `isort>=5.0.0`

## Security Considerations

### XML Processing
- Use `lxml` with external entity resolution disabled
- Validate input before processing
- Limit XML depth and size

### Graph Injection
- Sanitize all URIs
- Escape special characters in literals
- Prevent SPARQL/GQL injection

### Resource Limits
- Set maximum file size for processing
- Limit batch processing operations
- Implement timeout mechanisms

## Performance Targets

- **Small files** (< 1MB): < 1 second
- **Medium files** (1-10MB): < 10 seconds
- **Large files** (10-100MB): < 60 seconds
- **Memory**: < 2x file size

## Contributing (When Development Starts)

See [CONTRIBUTING.md](CONTRIBUTING.md) for:
- Development setup instructions
- Coding standards
- Testing requirements
- Pull request process

## Future Enhancements

Potential features for post-1.0:

- **Incremental updates** - Update graphs without full regeneration
- **SHACL validation** - Validate generated RDF against SHACL shapes
- **GraphQL API** - Query SDC4 data via GraphQL
- **Streaming** - Process large files in chunks
- **Parallel processing** - Multi-threaded batch operations
- **Plugin system** - Custom graph generators
- **OWL reasoning** - Infer relationships via OWL ontology

## References

- **SDC4 Specification**: [SDCRM Repository](https://github.com/SemanticDataCharter/SDCRM)
- **RDF 1.1**: [W3C Recommendation](https://www.w3.org/TR/rdf11-primer/)
- **ISO GQL**: [ISO/IEC 39075:2024](https://www.iso.org/standard/76120.html)
- **N-Quads**: [W3C Recommendation](https://www.w3.org/TR/n-quads/)
- **SPARQL 1.1**: [W3C Recommendation](https://www.w3.org/TR/sparql11-query/)
- **Apache Jena**: [Documentation](https://jena.apache.org/documentation/)
- **Neo4j**: [Documentation](https://neo4j.com/docs/)

## Questions?

This is a planned project. For questions or suggestions:

- **Open an issue**: Discuss architecture and features
- **GitHub Discussions**: Share ideas for graph mapping
- **Email**: contact@axius-sdc.com

---

**Status**: 🚧 Planning Phase - Development starts Q1-2026
