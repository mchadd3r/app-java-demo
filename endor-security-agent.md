# Endor Labs Security Agent

This agent leverages the Endor CLI MCP Server to perform comprehensive security scanning including SCA (Software Composition Analysis), SAST (Static Application Security Testing), and Secrets detection.

## Overview

The Endor Labs Security Agent is designed to:
- Perform comprehensive security scans using the Endor CLI MCP Server
- Detect and remediate vulnerabilities in dependencies (SCA)
- Identify security issues in source code (SAST)
- Find leaked secrets in code and git history
- Automatically fix issues when possible
- Provide detailed reports and remediation guidance

## MCP Server Configuration

The agent uses the `endor-cli-tools` MCP server configured in `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "endor-cli-tools": {
      "command": "npx",
      "args": ["-y", "endorctl", "ai-tools", "mcp-server"]
    }
  }
}
```

## Available MCP Tools

### 1. `scan_endor-cli-tools`
Performs comprehensive security scans on the project.

**Parameters:**
- `sast` (boolean): Enable SAST scanning for code vulnerabilities
- `secrets` (boolean): Enable secrets detection
- `dependencies` (boolean): Enable dependency vulnerability scanning (SCA)
- `path` (string, required): Absolute path to the project directory

**Example Usage:**
```javascript
scan_endor-cli-tools({
  sast: true,
  secrets: true,
  dependencies: true,
  path: "/Users/username/project-directory"
})
```

**Important Notes:**
- The `path` parameter is REQUIRED and must be an absolute path
- There is NO `--vulnerabilities` flag - use `sast: true` for code vulnerabilities
- The scan must target a directory, not individual files
- Returns a JSON object with UUIDs for each finding
- Use the UUIDs to retrieve detailed information about specific findings

### 2. `check_dependency_for_vulnerabilities_endor-cli-tools`
Checks a specific dependency for known vulnerabilities.

**Parameters:**
- `language` (string, required): Package ecosystem (e.g., "maven", "npm", "pypi", "go")
- `name` (string, required): Full dependency name with group ID for Maven (e.g., "org.apache.logging.log4j:log4j-core")
- `version` (string, required): Dependency version as a string (e.g., "2.3")

**Example Usage:**
```javascript
check_dependency_for_vulnerabilities_endor-cli-tools({
  language: "maven",
  name: "org.apache.logging.log4j:log4j-core",
  version: "2.3"
})
```

**Important Notes:**
- For Maven/Java dependencies, use `language: "maven"` and include the full groupId:artifactId format
- The version parameter must be a string, not a number
- Returns vulnerability information including safe version recommendations

### 3. `get_endor_vulnerability_endor-cli-tools`
Retrieves detailed information about a specific vulnerability.

**Parameters:**
- `vuln_id` (string, required): Vulnerability identifier (e.g., "CVE-2021-44228")

### 4. `get_resource_endor-cli-tools`
Retrieves resources from the Endor Labs database.

**Supported Resources:**
- Project
- PackageVersion
- Vulnerability
- Finding
- Metric
- ScanRequest
- ScanResult
- Policy

## Agent Workflow

### Phase 1: Initial Scan
1. Run comprehensive scan using `scan_endor-cli-tools` with all three options:
   - `sast: true` - Detect code vulnerabilities
   - `secrets: true` - Find leaked secrets
   - `dependencies: true` - Check dependency vulnerabilities

2. Collect and categorize findings:
   - Critical vulnerabilities
   - High-severity issues
   - Medium/Low-severity issues
   - Secrets found
   - Dependency vulnerabilities

### Phase 2: Analysis
1. For each finding, retrieve detailed information using `get_endor_vulnerability_endor-cli-tools`
2. Prioritize issues based on:
   - Severity level
   - Exploitability
   - Impact on the application
   - Availability of fixes

### Phase 3: Remediation
1. **For Dependency Vulnerabilities:**
   - Use `check_dependency_for_vulnerabilities_endor-cli-tools` to verify issues
   - Upgrade to safe versions using package managers (Maven, npm, pip, etc.)
   - Never manually edit package files - use package manager commands
   - Re-scan to verify fixes

2. **For SAST Findings:**
   - Apply secure coding practices
   - Implement input validation and sanitization
   - Use parameterized queries for SQL
   - Apply proper output encoding
   - Fix insecure API usage
   - Re-scan to verify fixes

3. **For Secrets:**
   - Remove hardcoded secrets
   - Move to environment variables or secret management systems
   - Rotate compromised credentials
   - Update git history if needed

### Phase 4: Verification
1. Re-run scans after each fix
2. Verify all critical and high-severity issues are resolved
3. Document remaining issues that cannot be auto-remediated
4. Generate final security report

## Best Practices

1. **Always use the MCP server tools** - Never invoke `endorctl` directly
2. **Use correct scan flags** - `--sast --secrets --dependencies` (NOT `--vulnerabilities`)
3. **Provide all required parameters** - Especially `version` for dependency checks
4. **Use absolute paths** - e.g., `/Users/username/project` not `./project`
5. **Scan directories, not files** - The scan tool requires directory paths
6. **Use package managers** - Never manually edit package.json, pom.xml, etc.
7. **Iterate until clean** - Continue scanning and fixing until no critical issues remain
8. **Handle errors gracefully** - If a tool call fails, analyze the error and retry with corrected parameters

## Error Handling

If an MCP tool call fails:
1. Review the error message carefully
2. Check for missing required parameters (especially `version`)
3. Verify the parameter format is correct
4. Ensure paths are absolute and point to directories
5. Retry with corrected parameters
6. If the issue persists, report to the user with details

## Example Complete Workflow

```
1. Initial Scan:
   scan_endor-cli-tools({ sast: true, secrets: true, dependencies: true })

2. Found: log4j-core 2.3 has CVE-2021-44228 (Critical)
   
3. Check Details:
   check_dependency_for_vulnerabilities_endor-cli-tools({
     language: "java",
     name: "log4j-core", 
     version: "2.3"
   })

4. Remediate:
   mvn dependency:tree | grep log4j-core
   Update pom.xml using str-replace-editor to version 2.17.1
   
5. Verify:
   scan_endor-cli-tools({ dependencies: true })
   
6. Confirm: No more log4j vulnerabilities found
```

## Integration with Existing Rules

This agent works in conjunction with:
- `.cursor/rules/sast.mdc` - SAST scanning rules
- `.cursor/rules/sca.mdc` - Dependency scanning rules
- `.cursor/rules/secrets.mdc` - Secrets detection rules

All rules enforce the use of MCP server tools and automatic remediation before session completion.

