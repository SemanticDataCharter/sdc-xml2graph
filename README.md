# sdc-xml2graph

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)
[![Python Version](https://img.shields.io/badge/python-3.9%2B-blue)](https://www.python.org)
[![Status](https://img.shields.io/badge/status-planning-yellow)](https://github.com/SemanticDataCharter/sdc-xml2graph)
[![Planned Release](https://img.shields.io/badge/release-Q1--2026-orange)](https://github.com/SemanticDataCharter/sdc-xml2graph)

**Transform validated SDC4 XML instances into knowledge graphs (RDF and property graphs)**

## 🚧 Project Status: Planning Phase

**Development Start**: Q1-2026

This repository contains the architecture, documentation, and planned implementation for **sdc-xml2graph**, a Python tool that bridges SDC4 validation and graph database technologies.

**Current Stage**: We're gathering requirements, refining the architecture, and welcoming community input. See [Use Case Submission](.github/ISSUE_TEMPLATE/use_case.md) to share how you plan to use this tool.

## Overview

**sdc-xml2graph** converts validated SDC4 (Semantic Data Charter Release 4) XML instances into graph formats for knowledge graph databases. It preserves semantic meaning, data quality indicators (ExceptionalValue), and hierarchical structure while transforming SDC4 data for graph-based analytics and queries.

### Key Features (Planned)

- ✅ **Multiple Graph Formats** - Generate RDF (N-Quads, Turtle, JSON-LD) and GQL (property graphs)
- ✅ **ExceptionalValue Preservation** - Maintain ISO 21090 data quality indicators in graphs
- ✅ **Validation-First** - Tight integration with [sdcvalidator](https://github.com/Axius-SDC/sdcvalidator)
- ✅ **Semantic Mapping** - Map SDC4 components to graph ontology
- ✅ **Database Integration** - Direct upload to Fuseki, Neo4j, Neptune
- ✅ **CLI Tool** - Command-line interface for batch processing
- ✅ **Performance** - Handle large SDC4 instances efficiently

### Use Cases

1. **Healthcare Analytics** - Transform validated patient data into queryable graphs
2. **Research Data Management** - Convert SDC4 research datasets for semantic web
3. **Regulatory Compliance** - Maintain data quality provenance in graph databases
4. **Multi-Database Integration** - Bridge SDC4 XML and graph database ecosystems
5. **Semantic Enrichment** - Link SDC4 data to external ontologies (SNOMED, LOINC)

## Architecture Overview

```
┌─────────────────┐
│  SDC4 Schema    │ (XSD from SDCRM)
│  + XML Instance │
└────────┬────────┘
         │
         ▼
┌─────────────────────────┐
│  sdcvalidator           │ ← Validation library
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

## Planned Usage

### CLI (Planned for Q1-2026)

```bash
# Install (when available)
pip install sdc-xml2graph

# Validate and convert to RDF (N-Quads)
sdc-xml2graph instance.xml --schema schema.xsd --format nquads -o output.nq

# Convert to GQL for property graphs
sdc-xml2graph instance.xml -s schema.xsd --format gql -o output.gql

# Batch processing
sdc-xml2graph instances/*.xml -s schema.xsd --format nquads --output-dir graphs/

# Load directly to Fuseki
sdc-xml2graph instance.xml -s schema.xsd --fuseki http://localhost:3030/dataset
```

### Python API (Planned)

```python
from sdcvalidator import SDC4Validator
from sdc_xml2graph import XML2Graph

# Validate first (always!)
validator = SDC4Validator('schema.xsd')
recovered = validator.validate_with_recovery('patient.xml')

# Convert to RDF graph
converter = XML2Graph('schema.xsd')
rdf_graph = converter.to_rdf(recovered)

# Serialize to N-Quads
nquads = rdf_graph.serialize(format='nquads')
with open('output.nq', 'w') as f:
    f.write(nquads)

# Or convert to GQL
gql_statements = converter.to_gql(recovered)
with open('output.gql', 'w') as f:
    f.write('\n'.join(gql_statements))
```

## ExceptionalValue Preservation

One of sdc-xml2graph's unique features is preserving data quality indicators in graphs:

**XML Input (with ExceptionalValue)**:
```xml
<XdCount label="Age" cardinality="1..1">
  <ExceptionalValue type="INV">
    not-a-number
  </ExceptionalValue>
</XdCount>
```

**RDF Output (N-Quads)**:
```turtle
<urn:sdc:component:age> rdf:type sdc:XdCount .
<urn:sdc:component:age> sdc:label "Age"@en .
<urn:sdc:component:age> sdc:hasExceptionalValue _:ev1 .
_:ev1 rdf:type sdc:INV .
_:ev1 sdc:originalValue "not-a-number" .
_:ev1 sdc:reason "Value does not conform to xsd:integer" .
```

**GQL Output (Property Graph)**:
```gql
CREATE (:XdCount {
  id: 'component:age',
  label: 'Age',
  exceptional_value_type: 'INV',
  original_value: 'not-a-number',
  exceptional_value_reason: 'Value does not conform to xsd:integer'
})
```

This allows downstream systems to query, analyze, and report on data quality issues.

## Technology Stack (Planned)

- **Python**: 3.9+ (matching sdcvalidator)
- **XML Processing**: `lxml` (consistent with sdcvalidator)
- **RDF Generation**: `rdflib` for graph manipulation
- **Graph Formats**: ISO GQL standard, N-Quads, Turtle, JSON-LD
- **Testing**: `pytest` with 85% minimum coverage
- **CLI**: `click` framework
- **Security**: Safe XML parsing, injection prevention, resource limits

## SDC4 Ecosystem Integration

**sdc-xml2graph** is part of the SDC4 ecosystem:

- **[SDCRM](https://github.com/SemanticDataCharter/SDCRM)** v4.0.0 - Reference model and schemas
- **[SDCStudio](https://github.com/AxiusSDC/SDCStudio)** v4.0.0 - Web application for model generation
- **[sdcvalidator (Python)](https://github.com/Axius-SDC/sdcvalidator)** v4.0.1 - **Required dependency** for validation
- **[sdcvalidatorJS (TypeScript)](https://github.com/Axius-SDC/sdcvalidatorJS)** v4.0.0 - JavaScript implementation
- **[Obsidian Template](https://github.com/SemanticDataCharter/SDCObsidianTemplate)** v4.0.0 - Markdown templates

All SDC4 projects use **4.x.x** versioning where the MAJOR version (4) represents the SDC generation.

### Workflow Integration

Typical SDC4 → Graph Database workflow:

```
1. Design model in SDCStudio → Generate XSD schema
2. Create XML instances from your data
3. Validate with sdcvalidator → Get recovered XML with ExceptionalValue
4. Convert with sdc-xml2graph → Generate RDF or GQL
5. Load to graph database (Fuseki, Neo4j, Neptune)
6. Query and analyze with SPARQL or GQL
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
- [ ] Turtle and JSON-LD formats
- [ ] ExceptionalValue preservation in RDF

### Phase 3: GQL Generation (Q2-2026)
- [ ] ISO GQL CREATE statements
- [ ] Property graph schema
- [ ] Neo4j Cypher (optional)

### Phase 4: CLI and Tools (Q2-2026)
- [ ] Command-line interface
- [ ] Batch processing
- [ ] Direct database integration
- [ ] Configuration files

### Phase 5: Testing & Documentation (Q2-2026)
- [ ] Comprehensive test suite (85% coverage)
- [ ] User guides and tutorials
- [ ] API documentation
- [ ] Example notebooks

## Contributing

We welcome contributions, feedback, and use case submissions!

### Ways to Contribute

- **[Share Your Use Case](.github/ISSUE_TEMPLATE/use_case.md)** - Help us understand real-world needs
- **[Architecture Discussion](.github/ISSUE_TEMPLATE/architecture_discussion.md)** - Provide feedback on design
- **[Feature Requests](.github/ISSUE_TEMPLATE/feature_request.md)** - Suggest capabilities
- **Early Code Contributions** - Help build the foundation (Q1-2026)

### Development Setup (When Code Available)

```bash
git clone https://github.com/SemanticDataCharter/sdc-xml2graph.git
cd sdc-xml2graph

# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install in development mode
pip install -e ".[dev]"

# Run tests
pytest

# Check code quality
black sdc_xml2graph/ tests/
flake8 sdc_xml2graph/
mypy sdc_xml2graph/
```

See [CONTRIBUTING.md](CONTRIBUTING.md) for detailed guidelines.

## Documentation

- **[CLAUDE.md](CLAUDE.md)** - Developer guide and architecture documentation
- **[CONTRIBUTING.md](CONTRIBUTING.md)** - Contribution guidelines and workflow
- **[SECURITY.md](SECURITY.md)** - Security policy and best practices
- **[Use Cases](.github/ISSUE_TEMPLATE/use_case.md)** - Submit your use case

## Related Resources

- **[Semantic Data Charter](https://semanticdatacharter.github.io)** - SDC4 specification and documentation
- **[SDCRM Repository](https://github.com/SemanticDataCharter/SDCRM)** - Schemas, ontologies, examples
- **[Apache Jena](https://jena.apache.org/)** - RDF database (Fuseki)
- **[Neo4j](https://neo4j.com/)** - Property graph database
- **[ISO GQL](https://www.iso.org/standard/76120.html)** - Graph Query Language standard
- **[W3C RDF](https://www.w3.org/RDF/)** - RDF specifications

## Support and Discussion

- **Issues**: [GitHub Issues](https://github.com/SemanticDataCharter/sdc-xml2graph/issues) - Bug reports and feature requests
- **Discussions**: [GitHub Discussions](https://github.com/SemanticDataCharter/sdc-xml2graph/discussions) - Questions and ideas
- **Email**: contact@axius-sdc.com
- **Security**: security@axius-sdc.com (see [SECURITY.md](SECURITY.md))

## License

MIT License - Copyright (c) 2025 Axius SDC

See [LICENSE](LICENSE) file for details.

## Versioning

This package will follow SDC ecosystem semantic versioning:
- **Major version** = SDC reference model version (4.x.x for SDC4)
- **Minor version** = New features (4.1.0, 4.2.0, etc.)
- **Patch version** = Bug fixes (4.0.1, 4.0.2, etc.)

## Frequently Asked Questions

**Q: When will this be available?**
A: Development starts Q1-2026. We're currently in the planning phase and gathering community input.

**Q: How does this differ from existing RDF converters?**
A: sdc-xml2graph is specifically designed for SDC4 with ExceptionalValue preservation, semantic validation, and tight integration with sdcvalidator.

**Q: Can I use this without sdcvalidator?**
A: Not recommended. Always validate with sdcvalidator first to ensure data quality and inject ExceptionalValue elements.

**Q: Which graph databases will be supported?**
A: Initially Apache Jena/Fuseki (RDF) and Neo4j (property graphs), with support for Amazon Neptune, Stardog, and GraphDB planned.

**Q: Will this support custom ontology mappings?**
A: Yes, extensibility for custom ontology alignment is in the roadmap (post-1.0).

## Acknowledgments

- **SDC4 Community** for the semantic data charter specification
- **sdcvalidator contributors** for the validation foundation
- **Apache Jena** and **rdflib** projects for RDF tools
- **ISO GQL Working Group** for graph query language standardization

---

**Status**: 🚧 Planning Phase - [Submit Your Use Case](.github/ISSUE_TEMPLATE/use_case.md) to help shape development!
