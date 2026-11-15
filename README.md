# MXM Penetrator

**AI-Powered Penetration Testing & Reverse Engineering Framework**

MXM Penetrator is an AI-powered, console-based penetration testing framework designed to automate web application security assessments through configurable local and cloud AI models.

Combining automated reconnaissance, intelligent security analysis, vulnerability assessment, and the capabilities of [REA (Reverse Engineer Anything)](https://github.com/morluto/rea), MXM aims to provide a unified environment for examining software, identifying security weaknesses, understanding application behavior, and generating comprehensive security reports.

Users will be able to select their preferred AI execution mode, provide an authorized target, choose a predefined assessment workflow, and let MXM coordinate the testing and analysis process.

---

## Overview

MXM Penetrator is built around a simple concept:

**Give MXM a target. Let it investigate. Understand the results. Generate a report.**

Instead of manually operating multiple security tools and interpreting disconnected results, MXM aims to coordinate different security assessment capabilities through a unified command-line interface.

The framework will combine:

- AI-powered analysis and orchestration.
- Automated web application reconnaissance.
- Security testing and vulnerability assessment.
- Reverse engineering through REA integration.
- Configurable local and cloud AI providers.
- Automated security report generation.

### Core Objectives

- Automate authorized web application security assessments.
- Integrate local and cloud AI models.
- Utilize REA for reverse engineering and software analysis.
- Coordinate multiple security tools through a unified CLI.
- Provide predefined and configurable assessment workflows.
- Generate evidence-based security reports.
- Maintain a modular and extensible architecture.

---

## Core Technologies

| Component | Technology |
|---|---|
| Primary language | Python |
| CLI framework | Typer |
| Terminal interface | Rich |
| HTTP client | HTTPX |
| AI integration | LiteLLM |
| Reverse engineering | REA |
| AI communication | MCP |
| Data validation | Pydantic |
| Asynchronous operations | asyncio |
| Testing | pytest |
| Package management | uv |

REA is maintained separately by [morluto](https://github.com/morluto/rea). MXM intends to integrate with REA rather than independently reimplement its reverse-engineering capabilities.

---

## AI Integration

MXM is designed to support two AI execution modes.

### Local AI

Run supported AI models locally through compatible inference providers.

Planned capabilities:

- Configure locally hosted models.
- Select available models.
- Perform AI-assisted security analysis.
- Keep model inference on the local machine when using local providers.
- Reduce dependence on external AI APIs.

### Cloud AI

Connect to supported cloud AI providers.

Planned capabilities:

- Configure cloud AI providers.
- Select supported models.
- Perform AI-assisted analysis and reasoning.
- Manage API credentials securely.
- Support different providers through a unified integration layer.

Users will be able to switch between local and cloud execution depending on their hardware, privacy requirements, and available resources.

**Important:** REA performs its analysis locally, but if MXM sends REA findings to a cloud AI provider for interpretation, those findings may be transmitted to that provider.

---

## REA Integration — Reverse Engineer Anything



MXM integrates with [REA (Reverse Engineer Anything)](https://github.com/morluto/rea), an external reverse-engineering framework that provides AI agents with tools for investigating software behavior, application structures, and compiled binaries.

REA exposes its capabilities through the Model Context Protocol (MCP), allowing compatible AI clients to interact with its analysis tools.

### What REA Brings to MXM

Depending on the selected analysis workflow and available tools, REA can provide capabilities for examining:

- Websites and web application behavior.
- JavaScript and Electron applications.
- Native binaries and executables.
- .NET assemblies.
- Android applications and APKs.
- Application structures and dependencies.
- Network captures and application communications.
- Other supported software artifacts.

### How MXM Uses REA

MXM will act as the primary interface and orchestration layer, while REA provides specialized reverse-engineering capabilities.

The intended integration workflow:

1. MXM receives an authorized target or software artifact.
2. MXM determines which analysis capabilities are relevant.
3. MXM communicates with REA through its MCP interface.
4. REA performs the requested analysis.
5. MXM receives structured findings and supporting evidence.
6. The selected AI provider interprets relevant results.
7. MXM incorporates the findings into its security assessment and report.

REA remains an independent dependency. Its installation, supported tools, and compatibility requirements are maintained by its upstream project.

**Official REA repository:** https://github.com/morluto/rea

---

## Automated Penetration Testing

MXM aims to coordinate security assessment workflows through predefined commands.

Intended capabilities include:

- Target reconnaissance.
- Endpoint and API discovery.
- Technology identification.
- HTTP request and response analysis.
- Authentication and authorization assessment.
- Input validation testing.
- Security configuration analysis.
- Vulnerability identification.
- AI-assisted interpretation of findings.

Testing will be restricted to explicitly authorized targets and configured assessment boundaries.

### Preset Assessment Workflows

One of MXM's central concepts is providing predefined commands that coordinate multiple testing operations.

Illustrative usage:

```bash
mxm penetrate http://localhost:8080
```

The intended workflow:

```text
TARGET PROVIDED
      |
      v
SCOPE VALIDATION
      |
      v
RECONNAISSANCE
      |
      v
APPLICATION ANALYSIS
      |
      v
SECURITY ASSESSMENT
      |
      v
REA INTEGRATION
      |
      v
AI-ASSISTED ANALYSIS
      |
      v
REPORT GENERATION
```

Users should also be able to execute individual operations when they want more control over the assessment process.

All automated testing must respect the configured target scope, request limits, and safety restrictions.

---

## Command-Line Interface

MXM is designed around a straightforward console-based experience.

Illustrative commands:

```bash
# Display available commands
mxm --help

# Perform an automated assessment
mxm penetrate http://localhost:8080

# Perform reconnaissance
mxm recon http://localhost:8080

# Analyze an authorized target
mxm analyze http://localhost:8080

# Generate a report from the latest assessment
mxm report --latest

# Display AI configuration
mxm config ai

# Select an AI execution mode
mxm config ai --mode local
mxm config ai --mode cloud
```

These commands illustrate the intended interface and are not necessarily available in the initial release.

---

## Automated Security Reporting

MXM aims to generate structured reports after completing an assessment.

Reports are intended to contain:

- Assessment overview.
- Target information.
- Discovered endpoints and services.
- Identified vulnerabilities.
- Severity and risk classifications.
- Supporting evidence.
- Relevant REA analysis results.
- Potential security implications.
- Recommended remediation steps.

AI-generated hypotheses will be distinguished from findings supported by verifiable evidence.

Planned report formats include Markdown, JSON, and HTML.

---

## Architecture

MXM is intended to follow a modular architecture that separates the CLI, core engine, AI providers, security testing modules, REA integration, and reporting system.

Proposed high-level architecture:

```text
                    MXM PENETRATOR
                           |
                    COMMAND INTERFACE
                           |
                       CORE ENGINE
                           |
          +----------------+----------------+
          |                |                |
      AI ENGINE       ASSESSMENT       CONFIGURATION
          |             ENGINE
          |                |
     +----+----+      +----+----+
     |         |      |         |
   LOCAL     CLOUD   RECON   SECURITY
    AI        AI     MODULES  MODULES
                           |
                    REA INTEGRATION
                           |
                      MCP CLIENT
                           |
                     REA FRAMEWORK
                           |
                   ANALYSIS RESULTS
                           |
                     AI ANALYSIS
                           |
                  REPORTING ENGINE
                           |
                   SECURITY REPORTS
```

The architecture is designed to support additional AI providers, testing modules, and external security tools without requiring major modifications to the core engine.

---

## Technology Requirements

### MXM

- Python (supported version to be established during implementation).
- uv package manager.
- Compatible local or cloud AI provider, depending on the selected mode.

### REA

REA has its own runtime and tool requirements.

According to its upstream documentation, its current setup requires compatible Node.js and npm versions. Certain analysis capabilities also depend on external tools such as Ghidra, Hopper, IDA, or other supported providers.

Some analysis operations require additional platform-specific dependencies.

Refer to the official documentation for installation and compatibility information:

https://github.com/morluto/rea

---

## Security and Responsible Usage

MXM Penetrator is intended exclusively for authorized penetration testing, security research, and educational purposes.

Users must have explicit permission to assess the systems they provide as targets.

The framework aims to incorporate:

- Explicit target scope configuration.
- Authorization checks before active testing.
- Configurable request and execution limits.
- Controlled handling of intrusive testing operations.
- Secure credential and API key management.
- Assessment logging and traceability.
- Clear reporting of verified and unverified findings.

Autonomous execution does not mean unrestricted execution. All testing must remain within the explicitly authorized scope and configured safety boundaries.

---

## Development Status

**Early development / Planning**

MXM Penetrator is an evolving project. Its architecture, capabilities, integrations, and supported technologies will develop throughout implementation.

The features described in this README represent the intended direction of the project and should not be interpreted as completed functionality.

---

## Roadmap

### Phase 1 — Foundation
- [ ] Initialize Python project.
- [ ] Establish modular architecture.
- [ ] Implement CLI.
- [ ] Implement configuration management.
- [ ] Establish logging and error handling.

### Phase 2 — AI Integration
- [ ] Local AI integration.
- [ ] Cloud AI integration.
- [ ] AI provider abstraction.
- [ ] Model selection.
- [ ] Structured AI responses.

### Phase 3 — REA Integration
- [ ] Study REA architecture and MCP interface.
- [ ] Establish MCP client integration.
- [ ] Connect MXM to the REA server.
- [ ] Implement REA tool discovery and invocation.
- [ ] Process and normalize REA analysis results.
- [ ] Integrate REA findings into assessment reports.

### Phase 4 — Web Security Assessment
- [ ] Target and scope management.
- [ ] Reconnaissance engine.
- [ ] Endpoint discovery.
- [ ] HTTP analysis.
- [ ] Security testing modules.
- [ ] Vulnerability classification.
- [ ] Evidence collection.

### Phase 5 — Intelligent Automation
- [ ] Preset assessment workflows.
- [ ] AI-assisted analysis.
- [ ] Assessment orchestration.
- [ ] Controlled autonomous execution.
- [ ] Finding prioritization.
- [ ] Assessment history.

### Phase 6 — Reporting
- [ ] Markdown reports.
- [ ] JSON reports.
- [ ] HTML reports.
- [ ] Severity classification.
- [ ] Evidence-backed findings.
- [ ] Remediation recommendations.

### Phase 7 — Expansion
- [ ] Additional reverse-engineering workflows.
- [ ] Additional external tool integrations.
- [ ] Expanded application analysis.
- [ ] More configurable assessment profiles.

---

## Installation

Installation instructions will be provided when the initial development version is available.

Planned repository setup:

```bash
git clone https://github.com/USERNAME/mxm-penetrator.git

cd mxm-penetrator

uv sync
```

REA integration will have separate setup and configuration instructions once the MCP connection is implemented.

---

## Configuration

MXM is intended to provide centralized configuration for AI providers and assessment settings.

Illustrative environment configuration:

```env
MXM_AI_MODE=local
MXM_AI_MODEL=your-local-model

MXM_CLOUD_PROVIDER=
MXM_CLOUD_MODEL=
MXM_API_KEY=

MXM_REA_ENABLED=true
MXM_REA_TRANSPORT=
MXM_REA_SERVER_COMMAND=
```

The actual configuration fields and supported providers will be documented as they are implemented.

Never commit credentials, API keys, or other secrets to version control.

---

## Contributing

Contributions, suggestions, and discussions are welcome.

As the project develops, contribution guidelines will be established to maintain code quality, architectural consistency, and security.

When working with REA, contributors should also respect its upstream license and attribution requirements.

---

## Third-Party Software and Licensing

MXM Penetrator's original code is intended to be distributed under the **Apache License 2.0**.

MXM integrates with external projects, including:

- [REA — Reverse Engineer Anything](https://github.com/morluto/rea) — MIT License.

Third-party software retains its respective license. MXM does not claim ownership of REA or its source code.

Third-party notices and applicable license terms will be maintained separately as integrations are implemented.

See [LICENSE](LICENSE) for the licensing terms applicable to MXM's original code.

---

## Project Philosophy

**MXM Penetrator — Intelligent Security Through Automation.**

The goal is not simply to create another collection of penetration-testing tools.

The vision is to develop an extensible, AI-powered framework capable of investigating applications, understanding their behavior, identifying security weaknesses, leveraging specialized reverse-engineering tools, and communicating its findings through a unified command-line experience.

**MXM. Analyze deeper. Understand better. Test smarter.**
