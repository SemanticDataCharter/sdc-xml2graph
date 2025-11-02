---
name: Use Case Submission
about: Share how you plan to use sdc-xml2graph
title: '[USE CASE] '
labels: use-case, requirements
assignees: ''
---

## Use Case Overview

**Brief description of your intended use:**

## Your Context

**Industry/Domain:**
(e.g., Healthcare, Finance, Research, Government, etc.)

**Organization Type:**
(e.g., Hospital, University, Enterprise, Startup, etc.)

**Team Size:**
(e.g., Solo developer, 2-5 person team, 10+ person team)

## SDC4 Data

**What kind of SDC4 data will you process?**

**Data Volume:**
- Number of XML instances: (e.g., 100s, 1000s, millions)
- Average file size: (e.g., 10KB, 1MB, 100MB)
- Total data volume: (e.g., 1GB, 100GB, 1TB)

**Data Characteristics:**
- [ ] Highly structured (few components)
- [ ] Complex hierarchies (many nested Clusters)
- [ ] Large datasets (quantified types with many values)
- [ ] Multilingual content
- [ ] ExceptionalValue-heavy (data quality issues)

**Update Frequency:**
- [ ] One-time migration
- [ ] Daily batch processing
- [ ] Real-time streaming
- [ ] Ad-hoc conversions

## Graph Database Target

**Which graph database will you use?**

- [ ] Apache Jena/Fuseki (RDF)
- [ ] Amazon Neptune (RDF or Property Graph)
- [ ] Neo4j (Property Graph)
- [ ] Stardog (RDF)
- [ ] GraphDB (RDF)
- [ ] Other: __________

**Graph Format Preference:**
- [ ] RDF (N-Quads)
- [ ] RDF (Turtle)
- [ ] RDF (JSON-LD)
- [ ] GQL (ISO GQL)
- [ ] Cypher (Neo4j)
- [ ] Other: __________

## Workflow

**How will sdc-xml2graph fit into your workflow?**

```
Example:
1. Receive patient data as CSV
2. Use SDCStudio to create SDC4 model
3. Generate XML instances
4. Validate with sdcvalidator
5. Convert to RDF with sdc-xml2graph
6. Load into Fuseki
7. Query with SPARQL for analytics
```

1.
2.
3.
4.
5.

## Integration Requirements

**What integrations do you need?**

- [ ] CLI batch processing
- [ ] Python API integration
- [ ] Direct database upload
- [ ] RESTful API
- [ ] Cloud service (AWS, GCP, Azure)
- [ ] CI/CD pipeline integration
- [ ] Other: __________

## Performance Requirements

**What are your performance needs?**

**Processing Speed:**
- Acceptable processing time per file: (e.g., < 1 second, < 10 seconds)
- Batch processing requirements: (e.g., 1000 files in < 1 hour)

**Resource Constraints:**
- Available memory: (e.g., 4GB, 16GB, 64GB)
- CPU cores: (e.g., 2, 8, 32)
- Storage: (e.g., SSD, HDD, cloud)

## Semantic Requirements

**What semantic features are important?**

- [ ] Ontology alignment (SNOMED, LOINC, etc.)
- [ ] OWL reasoning
- [ ] SHACL validation
- [ ] Custom predicates/properties
- [ ] Multilingual labels
- [ ] Versioning/provenance
- [ ] ExceptionalValue querying

## Key Features Needed

**Rank these features by importance (1=most important):**

- [ ] RDF generation
- [ ] GQL/property graph generation
- [ ] ExceptionalValue preservation
- [ ] Performance/scalability
- [ ] Direct database integration
- [ ] Batch processing
- [ ] Streaming support
- [ ] Custom mapping rules
- [ ] Validation/error handling
- [ ] Documentation

## Challenges You Anticipate

**What concerns or challenges do you foresee?**

1.
2.
3.

## Timeline

**When do you need this?**

- [ ] Q1-2026 (initial release)
- [ ] Q2-2026
- [ ] Q3-2026
- [ ] Later
- [ ] No specific timeline

## Willingness to Contribute

**Would you be willing to:**

- [ ] Test beta versions
- [ ] Provide feedback on API design
- [ ] Contribute code
- [ ] Contribute documentation
- [ ] Provide test data/schemas
- [ ] Share success stories

## Example Data

**Can you provide example SDC4 schemas or instances?**
(Anonymized/synthetic data is fine)

<details>
<summary>Example Schema (optional)</summary>

```xml
<!-- Paste example schema -->
```

</details>

<details>
<summary>Example Instance (optional)</summary>

```xml
<!-- Paste example instance -->
```

</details>

## Additional Context

**Any other information about your use case:**

## Contact (Optional)

**If you're willing to discuss further:**

- Name:
- Email:
- Organization:

---

Thank you for sharing your use case! This helps us prioritize features and ensure sdc-xml2graph meets real-world needs.
