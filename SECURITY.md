# Security Policy

## Supported Versions

This project is currently in planning phase (Q1-2026 development start). Security updates will be provided for supported versions once released.

| Version | Supported          | Status |
| ------- | ------------------ | ------ |
| 4.0.x   | :white_check_mark: | Planned (Q1-2026) |

## Reporting a Vulnerability

We take security seriously. If you discover a security vulnerability, please report it responsibly.

### How to Report

**Email**: security@axius-sdc.com

**Include**:
- Description of the vulnerability
- Steps to reproduce
- Potential impact
- Suggested fix (if available)

**Response Time**:
- Initial response: Within 48 hours
- Status update: Within 7 days
- Fix timeline: Depends on severity

### What to Expect

1. **Acknowledgment** - We'll confirm receipt within 48 hours
2. **Investigation** - We'll assess the vulnerability
3. **Fix** - We'll develop and test a patch
4. **Disclosure** - We'll coordinate public disclosure with you
5. **Credit** - You'll be credited in the security advisory (if desired)

## Security Considerations

### XML Processing Vulnerabilities

**sdc-xml2graph** processes XML files and must protect against common XML attacks.

#### 1. XML External Entity (XXE) Attacks

**Risk**: Attackers can read local files or perform SSRF attacks via malicious XML.

**Mitigation** (planned):

```python
from lxml import etree

# ✅ SAFE: Disable external entity resolution
parser = etree.XMLParser(
    resolve_entities=False,  # Disable external entities
    no_network=True,         # Disable network access
    dtd_validation=False,    # Disable DTD validation
    load_dtd=False          # Don't load DTD
)

tree = etree.parse('instance.xml', parser)
```

**Status**: Will be enabled by default in all XML parsing.

#### 2. Billion Laughs Attack (XML Bomb)

**Risk**: Exponential entity expansion can exhaust memory.

**Mitigation** (planned):

```python
from lxml import etree

# Set resource limits
parser = etree.XMLParser(
    huge_tree=False,         # Limit tree size
    resolve_entities=False   # Prevent entity expansion
)

# Implement file size limits
MAX_FILE_SIZE = 100 * 1024 * 1024  # 100MB

def parse_xml_safely(xml_path: str) -> etree._Element:
    """Parse XML with size limits."""
    import os
    file_size = os.path.getsize(xml_path)
    if file_size > MAX_FILE_SIZE:
        raise ValueError(f"File too large: {file_size} bytes")

    parser = etree.XMLParser(
        resolve_entities=False,
        no_network=True,
        huge_tree=False
    )
    return etree.parse(xml_path, parser)
```

#### 3. XPath Injection

**Risk**: User-controlled XPath expressions can access unintended data.

**Mitigation** (planned):

```python
# ❌ DANGEROUS: User input in XPath
user_input = "../../sensitive-data"
xpath = f"//component[@name='{user_input}']"
results = tree.xpath(xpath)

# ✅ SAFE: Use parameterized XPath
from lxml import etree

def find_component_safe(tree: etree._Element, name: str) -> list:
    """Find component with safe XPath."""
    # Sanitize input
    safe_name = name.replace("'", "").replace('"', '')

    # Use XPath with validation
    xpath = f"//component[@name=$name]"
    results = tree.xpath(xpath, name=safe_name)
    return results
```

### Graph Injection Vulnerabilities

#### 1. SPARQL Injection

**Risk**: Malicious input can execute arbitrary SPARQL queries.

**Mitigation** (planned):

```python
from rdflib import Graph, Literal
from rdflib.plugins.sparql import prepareQuery

# ❌ DANGEROUS: String concatenation
user_input = "John'; DROP ALL; #"
query_string = f"""
    SELECT ?s WHERE {{
        ?s sdc:value "{user_input}"
    }}
"""

# ✅ SAFE: Parameterized queries
query = prepareQuery("""
    SELECT ?s WHERE {
        ?s sdc:value ?input
    }
""")

results = graph.query(query, initBindings={'input': Literal(user_input)})
```

#### 2. GQL Injection

**Risk**: Malicious input in GQL CREATE statements.

**Mitigation** (planned):

```python
# ❌ DANGEROUS: Direct string interpolation
user_label = "'; DROP DATABASE; --"
gql = f"CREATE (:Component {{label: '{user_label}'}})"

# ✅ SAFE: Escape and validate
import re

def sanitize_gql_string(value: str) -> str:
    """Sanitize string for GQL."""
    # Escape single quotes
    value = value.replace("'", "\\'")
    # Remove control characters
    value = re.sub(r'[\x00-\x1f\x7f-\x9f]', '', value)
    return value

safe_label = sanitize_gql_string(user_label)
gql = f"CREATE (:Component {{label: '{safe_label}'}})"
```

#### 3. URI Manipulation

**Risk**: Malicious URIs can cause issues in RDF graphs.

**Mitigation** (planned):

```python
from rdflib import URIRef
from urllib.parse import quote

def create_safe_uri(namespace: str, local_part: str) -> URIRef:
    """Create safe URI with validation."""
    # Validate namespace
    if not namespace.startswith(('http://', 'https://', 'urn:')):
        raise ValueError(f"Invalid namespace: {namespace}")

    # Sanitize local part
    safe_local = quote(local_part, safe='')

    # Create URI
    uri = URIRef(f"{namespace}{safe_local}")
    return uri
```

### Dependency Vulnerabilities

We rely on external libraries that may have vulnerabilities.

#### Dependency Security (Planned)

```bash
# Regular security audits
pip-audit

# Check for known vulnerabilities
safety check

# Keep dependencies updated
pip install --upgrade sdcvalidator lxml rdflib
```

**Automated Scanning**:
- GitHub Dependabot enabled
- Weekly security scans in CI/CD
- Automated PRs for security updates

### Resource Exhaustion

#### File Size Limits

```python
# Planned defaults
MAX_FILE_SIZE = 100 * 1024 * 1024  # 100MB
MAX_TREE_DEPTH = 1000
MAX_COMPONENTS = 100000

def validate_file_size(file_path: str) -> None:
    """Ensure file is within size limits."""
    import os
    size = os.path.getsize(file_path)
    if size > MAX_FILE_SIZE:
        raise ValueError(f"File exceeds maximum size: {size} bytes")
```

#### Memory Limits

```python
# Planned: Streaming for large files
def process_large_file(xml_path: str) -> None:
    """Process large XML files in chunks."""
    from lxml import etree

    context = etree.iterparse(
        xml_path,
        events=('start', 'end'),
        tag='component'
    )

    for event, elem in context:
        if event == 'end':
            # Process component
            process_component(elem)
            # Clear memory
            elem.clear()
            while elem.getprevious() is not None:
                del elem.getparent()[0]
```

#### Timeout Mechanisms

```python
import signal
from contextlib import contextmanager

@contextmanager
def timeout(seconds: int):
    """Context manager for operation timeout."""
    def timeout_handler(signum, frame):
        raise TimeoutError(f"Operation exceeded {seconds} seconds")

    signal.signal(signal.SIGALRM, timeout_handler)
    signal.alarm(seconds)
    try:
        yield
    finally:
        signal.alarm(0)

# Usage
with timeout(30):
    graph = converter.to_rdf('large_file.xml')
```

## Best Practices for Users

### Validate Input

Always validate XML before processing:

```python
from sdcvalidator import SDC4Validator
from sdc_xml2graph import XML2Graph

# ✅ Validate first
validator = SDC4Validator('schema.xsd')
try:
    recovered = validator.validate_with_recovery('instance.xml')
except Exception as e:
    # Handle validation errors
    print(f"Validation failed: {e}")
    exit(1)

# Then convert
converter = XML2Graph('schema.xsd')
graph = converter.to_rdf(recovered)
```

### Limit File Sources

Only process XML from trusted sources:

```python
ALLOWED_DIRECTORIES = [
    '/var/sdc/data',
    '/home/user/sdc/validated'
]

def is_safe_path(file_path: str) -> bool:
    """Check if file is from allowed directory."""
    import os
    abs_path = os.path.abspath(file_path)
    return any(abs_path.startswith(d) for d in ALLOWED_DIRECTORIES)

if not is_safe_path(input_file):
    raise ValueError("File must be from allowed directory")
```

### Use Resource Limits

Set appropriate limits for your environment:

```python
from sdc_xml2graph import XML2Graph

converter = XML2Graph(
    schema='schema.xsd',
    max_file_size=50 * 1024 * 1024,  # 50MB
    max_components=50000,
    timeout=60  # 60 seconds
)
```

### Secure Graph Storage

Protect generated graphs:

```python
import os

# Set restrictive permissions on output files
output_file = 'output.nq'
with open(output_file, 'w') as f:
    f.write(graph.serialize(format='nquads'))

os.chmod(output_file, 0o600)  # rw------- (owner only)
```

### Validate URIs

Check URIs before using in graphs:

```python
from urllib.parse import urlparse

def is_safe_uri(uri: str) -> bool:
    """Validate URI format and scheme."""
    try:
        parsed = urlparse(uri)
        # Only allow http, https, urn schemes
        return parsed.scheme in ('http', 'https', 'urn')
    except Exception:
        return False

if not is_safe_uri(user_provided_uri):
    raise ValueError("Invalid URI")
```

## Security Checklist for Contributors

When contributing code, ensure:

- [ ] **XML parsing** uses safe parser settings
- [ ] **External entities** are disabled
- [ ] **File size limits** are enforced
- [ ] **User input** is sanitized before use in queries
- [ ] **URIs** are validated before adding to graphs
- [ ] **Error messages** don't leak sensitive information
- [ ] **Temporary files** are cleaned up
- [ ] **Secrets** are not hardcoded or logged
- [ ] **Dependencies** are up to date
- [ ] **Tests** include security test cases

## CI/CD Security (Planned)

### GitHub Actions Security

```yaml
# .github/workflows/security.yml
name: Security

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 0 * * 1'  # Weekly

jobs:
  pip-audit:
    name: Dependency Audit
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.11'
      - name: Install pip-audit
        run: pip install pip-audit
      - name: Run pip-audit
        run: pip-audit

  codeql:
    name: CodeQL Analysis
    runs-on: ubuntu-latest
    permissions:
      security-events: write
    steps:
      - uses: actions/checkout@v4
      - uses: github/codeql-action/init@v3
        with:
          languages: python
      - uses: github/codeql-action/analyze@v3
```

## Known Security Considerations

### lxml Library

**lxml** is a C extension with potential memory safety issues.

- Keep updated to latest version
- Monitor CVE database for lxml vulnerabilities
- Consider fallback to pure Python XML parser for untrusted input

### rdflib Library

**rdflib** processes RDF data and SPARQL queries.

- Use parameterized queries
- Validate all URIs
- Limit query complexity
- Set execution timeouts

## Security Roadmap

### Phase 1 (Q1-2026)
- [ ] Implement safe XML parsing defaults
- [ ] Add file size and resource limits
- [ ] Create security test suite
- [ ] Set up automated vulnerability scanning

### Phase 2 (Q2-2026)
- [ ] Add SPARQL injection protection
- [ ] Implement GQL sanitization
- [ ] Add URI validation
- [ ] Create security documentation

### Phase 3 (Q3-2026)
- [ ] Security audit by external firm
- [ ] Penetration testing
- [ ] Security hardening guide
- [ ] Bug bounty program (if applicable)

## References

- **OWASP XML Security**: https://cheatsheetseries.owasp.org/cheatsheets/XML_Security_Cheat_Sheet.html
- **OWASP Injection Prevention**: https://cheatsheetseries.owasp.org/cheatsheets/Injection_Prevention_Cheat_Sheet.html
- **lxml Security**: https://lxml.de/FAQ.html#security
- **SPARQL Injection**: https://www.w3.org/TR/sparql11-query/#security

## Contact

For security concerns:
- **Email**: security@axius-sdc.com
- **PGP Key**: (Will be provided upon request)

---

**Remember**: Security is everyone's responsibility. Thank you for helping keep sdc-xml2graph secure!
