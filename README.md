# AWS Transform Custom Workshop — Analyze, Transform, Scale, Measure

A hands-on exploration of **AWS Transform custom** for AI-assisted code modernization, covering the complete modernization lifecycle:

> **Analyze → Transform → Validate → Publish → Scale → Measure**

This workshop demonstrates how AWS Transform custom can be used to analyze legacy applications, execute AWS-managed transformations, author reusable custom transformation definitions, validate modernization results, and scale the same transformation workflow across multiple repositories using AWS Batch, AWS Fargate, Amazon S3, Lambda, API Gateway, and CloudWatch.

---

## Overview

Modernizing a large software portfolio is rarely a single-codebase problem. Organizations may have hundreds or thousands of applications running on outdated language versions, deprecated SDKs, legacy architectural patterns, or unsupported frameworks.

Running an AI-assisted transformation manually against every repository does not scale.

This workshop demonstrates a practical pattern for addressing that problem:

1. **Analyze** each codebase to understand its architecture, dependencies, technical debt, and modernization opportunities.
2. **Transform** applications using AWS-managed transformation definitions.
3. **Author** custom transformation definitions when the desired modernization goes beyond what a managed transformation provides.
4. **Validate** the resulting code with automated tests and transformation-specific exit criteria.
5. **Publish** validated custom transformations into the transformation registry for reuse.
6. **Scale** transformations across a fleet of repositories using containerized execution with AWS Batch and Fargate.
7. **Measure** execution, cost, code changes, success rates, and operational health using artifacts and CloudWatch.

The result is an **"Author Once, Roll Out Everywhere"** approach to AI-assisted modernization.

---

# What I Learned

By completing this workshop, I worked through the following capabilities:

- Running AWS Transform custom from the CLI
- Performing comprehensive codebase analysis
- Understanding transformation definitions (TDs)
- Applying AWS-managed transformation definitions
- Upgrading Python applications from older runtimes to Python 3.12
- Creating a multi-concern custom transformation definition
- Using existing codebase analysis as transformation context
- Applying repository and dependency modernization patterns
- Adding structured logging and Pydantic validation
- Automatically validating transformed applications
- Publishing a custom transformation to the registry
- Running transformations non-interactively
- Understanding containerized transformation execution
- Submitting multiple transformations through a REST API
- Monitoring AWS Batch jobs and Fargate tasks
- Collecting transformation output in Amazon S3
- Using CloudWatch for operational monitoring
- Designing a two-phase modernization strategy for large application fleets

---

# Workshop Architecture

The workshop used an AWS-hosted development environment running the AWS Code Editor IDE.

The environment contained:

```text
/workshop/
├── emoji-algebra/
├── qwords/
├── scientific-processor/
└── utilities/
    └── invoke-api.py
```

The three applications intentionally represented different modernization scenarios.

| Application | Original Runtime | Application Type | Modernization Focus |
|---|---:|---|---|
| `qwords` | Python 3.8 | CLI application | Python runtime and dependency modernization |
| `emoji-algebra` | Python 3.8 | CLI application | Python runtime and dependency modernization |
| `scientific-processor` | Python 3.9 | AWS Lambda / SAM | Runtime + architecture + validation + observability |

The workshop environment was hosted for approximately eight hours, which provided enough time to perform the interactive transformations and investigate the scaled execution workflow.

---

# 1. Environment Setup

The workshop environment was accessed through **AWS Workshop Studio** and its hosted Code Editor IDE.

The IDE provided:

- File Explorer
- Integrated terminal
- Source Control
- Editor
- Pre-installed development extensions
- AWS Transform custom CLI
- Python
- Node.js
- Git
- `uv`

The initial environment verification was:

```bash
atx --version
uv --version
python3 --version
node --version
git --version
```

AWS authentication was also verified through:

```bash
aws sts get-caller-identity
```

The workshop applications were already available under `/workshop`.

Each application was initialized as a Git repository because AWS Transform custom uses Git to track transformation changes and create result branches.

Example:

```bash
cd /workshop/qwords
git init
git add .
git commit -m "Initial commit"
```

The same process was performed for:

```text
qwords
emoji-algebra
scientific-processor
```

---

# 2. Baseline Application Testing

Before transforming anything, the applications were tested in their original state.

This is an important modernization practice:

> **Always establish a working baseline before allowing an automated transformation to modify the codebase.**

For example, `qwords` was tested using Python 3.8:

```bash
cd /workshop/qwords

uv venv --python 3.8 .venv
source .venv/bin/activate

uv pip install -r requirements.txt
python -m pytest tests/ -v

deactivate
```

The same approach was used for `emoji-algebra`.

The `scientific-processor` application was tested against its Python 3.9 configuration and its SAM infrastructure.

At this stage, the applications represented working legacy systems rather than broken applications.

That baseline becomes important later because transformation success is not simply:

> "The AI changed the code."

The real objective is:

> "The modernized application still satisfies its functional and architectural requirements."

---

# 3. Module 1 — Comprehensive Codebase Analysis

The first major step was understanding the code before transforming it.

For the `scientific-processor` application, AWS Transform custom's managed transformation definition was executed:

```bash
cd /workshop/scientific-processor

atx custom def exec \
  -n AWS/comprehensive-codebase-analysis \
  -p . \
  -x \
  -t
```

The important flags were:

| Flag | Purpose |
|---|---|
| `-n` | Selects the transformation definition |
| `-p` | Specifies the project path |
| `-x` | Executes without interactive confirmation |
| `-t` | Trusts tools for the execution |

The analysis was intentionally performed before creating the custom transformation.

## Why analysis first?

The analysis generated structured documentation covering areas such as:

- Application architecture
- Source structure
- Dependencies
- Infrastructure
- Testing
- Technical debt
- Migration considerations
- Transformation opportunities

The resulting `ATXDocumentation/` directory then became an important source of context for the custom transformation.

This established a key pattern:

```text
Source Code
     │
     ▼
Comprehensive Analysis
     │
     ▼
ATXDocumentation/
     │
     ▼
Transformation Planning
     │
     ▼
Transformation Execution
```

Instead of asking an AI agent to blindly modify a repository, the transformation process had structured architectural context available.

---

# 4. Module 2 — AWS-Managed Transformations

The next step was to use AWS-managed transformation definitions.

The workshop used:

```text
AWS/python-version-upgrade
```

against:

```text
qwords
emoji-algebra
```

The objective was to modernize the applications from Python 3.8 to Python 3.12.

---

## QWords Transformation

A transformation configuration was created:

```yaml
additionalPlanContext: |
  Target Python 3.12. Modernize all syntax from 3.8 to 3.12.
  Use uv for Python environment management.
  Ensure all existing tests continue to pass.
```

The transformation was then executed:

```bash
atx custom def exec \
  -n AWS/python-version-upgrade \
  -p . \
  -c "python -m pytest tests/ -v" \
  -g file://config.yaml \
  -x \
  -t
```

The important part of this command is the validation command:

```text
-c "python -m pytest tests/ -v"
```

The transformation was not treated as complete merely because the agent finished editing files.

The transformed application also had to pass its tests.

The workshop reference execution reported:

```text
qwords
53 tests passed
```

with approximately:

```text
37.86 agent-minutes
```

---

# 5. Emoji Algebra Transformation

The same managed transformation was applied to `emoji-algebra`.

The application was also upgraded from:

```text
Python 3.8
```

to:

```text
Python 3.12
```

The transformation was executed in a separate terminal so both transformations could run in parallel.

The workshop reference execution reported:

```text
emoji-algebra
48 tests passed
```

with approximately:

```text
31.74 agent-minutes
```

---

# 6. What the Managed Transformation Modernized

The transformation demonstrated several common Python modernization patterns.

Examples included:

| Legacy Pattern | Modernized Pattern |
|---|---|
| `Optional[str]` | `str \| None` |
| `List[str]` | `list[str]` |
| `Dict[str, int]` | `dict[str, int]` |
| `.format()` | f-strings |
| Long `if/elif` dispatch | `match/case` where appropriate |
| Manual data structures | More modern Python constructs |
| Older packaging configuration | Updated project configuration |

The transformation also handled dependency and project configuration changes required for the newer Python runtime.

The important observation was that AWS Transform custom was not being used as a simple text replacement engine.

It was reasoning about:

- source code
- dependencies
- tests
- project configuration
- runtime compatibility

The workshop material documents the managed transformation flow and validation process.

---

# 7. Git-Based Transformation Workflow

AWS Transform custom created separate transformation result branches rather than directly overwriting the original branch.

The resulting pattern was:

```text
main
 │
 ├── original application
 │
 └── atx-result-staging-<session-id>
          │
          ├── transformed source
          ├── dependency updates
          ├── configuration updates
          └── validation results
```

This makes it possible to inspect the changes before integrating them.

For example:

```bash
git log --oneline
```

and:

```bash
git diff HEAD~1 -- app.py
```

could be used to inspect the transformation.

This is an important safety characteristic for automated modernization:

> **Transformation output should be reviewable as code changes rather than treated as an opaque AI result.**

---

# 8. Module 2.2 — Creating a Custom Transformation

The managed Python upgrade solved one problem:

```text
Python 3.9 → Python 3.12
```

But the `scientific-processor` modernization requirements were broader.

The desired transformation included four coordinated architectural changes:

1. Python 3.9 → Python 3.12
2. Raw boto3 S3 calls → repository pattern
3. Unstructured logging → structured logging
4. No explicit input validation → Pydantic validation

No single AWS-managed transformation represented this complete combination.

This is where **custom transformation definitions** became useful.

The workshop specifically demonstrates custom TD authoring as a way to encode organization-specific modernization patterns into a reusable transformation.

---

# 9. Preparing the Custom Transformation

The first step was creating a Python 3.12 environment:

```bash
cd /workshop/scientific-processor

python3.12 -m venv .venv --clear
source .venv/bin/activate

pip install -r requirements.txt \
            -r tests/requirements.txt \
            -q
```

The initial dependency installation could fail because the original project pinned:

```text
numpy==1.21.6
```

which is not compatible with Python 3.12.

This failure was useful rather than unexpected.

It demonstrated one of the modernization problems the custom transformation needed to solve.

---

# 10. Authoring the Custom Transformation

AWS Transform custom was started interactively:

```bash
atx
```

The transformation request described the desired modernization:

```text
Python 3.9 Lambda → Python 3.12

Raw boto3 S3 calls
        ↓
S3 repository abstraction

Basic logging
        ↓
aws-lambda-powertools structured logging

Unvalidated input
        ↓
Pydantic validation
```

The existing:

```text
ATXDocumentation/
```

was explicitly provided as contextual reference.

This created a useful architecture:

```text
                 ┌─────────────────────┐
                 │   Source Repository │
                 └──────────┬──────────┘
                            │
                            ▼
                 ┌─────────────────────┐
                 │ ATXDocumentation/   │
                 │ Architecture       │
                 │ Dependencies       │
                 │ Technical Debt     │
                 └──────────┬──────────┘
                            │
                            ▼
                 ┌─────────────────────┐
                 │ Custom TD Authoring │
                 └──────────┬──────────┘
                            │
                            ▼
                 ┌─────────────────────┐
                 │     SKILL.md        │
                 └─────────────────────┘
```

---

# 11. Interactive Architecture Interview

Rather than immediately generating code, ATX asked architecture questions.

The interview covered decisions such as:

### Repository pattern

The recommendation was to create a concrete:

```python
S3Repository
```

with dependency injection.

### Input validation

The recommended approach was:

```text
ProcessorEvent
      │
      ├── operation A
      ├── operation B
      └── operation C
```

using Pydantic models and discriminated unions.

### Structured logging

The recommendation was to use:

```text
aws-lambda-powertools
```

with:

```python
@logger.inject_lambda_context
```

and correlation information.

For the workshop, the recommended defaults were accepted using:

```text
use defaults
```

This is an interesting part of the workflow because the AI was not simply generating a generic code migration.

It was gathering architectural requirements before generating the transformation definition.

---

# 12. Generated Transformation Definition

ATX generated a reusable transformation definition represented by a `SKILL.md`.

The generated transformation covered multiple phases, including:

1. Python runtime and dependency alignment
2. S3 repository abstraction
3. Pydantic validation
4. Structured logging
5. Lambda handler refactoring
6. Structured error handling
7. Test modernization
8. Configuration cleanup

The resulting transformation was conceptually:

```text
Python Lambda Modernization
        │
        ├── Runtime
        │     └── Python 3.12
        │
        ├── Architecture
        │     └── S3Repository
        │
        ├── Validation
        │     └── Pydantic
        │
        ├── Observability
        │     └── Lambda Powertools
        │
        ├── Error Handling
        │
        ├── Tests
        │
        └── Infrastructure
              ├── SAM
              └── build configuration
```

The transformation definition was saved as a draft before being applied.

This was an important workflow decision.

---

# 13. Safe Transformation Lifecycle

The workshop used the following lifecycle:

```text
Author
  ↓
Save as Draft
  ↓
Apply to Test Repository
  ↓
Run Validation
  ↓
Review Results
  ↓
Publish
```

Rather than publishing the transformation immediately, it was first applied to the source project.

The selected option was:

```text
Save as draft and apply
```

This ensured that the transformation was tested before being placed into the shared registry.

The workshop explicitly recommends validating before publishing.

---

# 14. Transformation Changes

The custom transformation modified the application architecture.

## Dependency modernization

The old NumPy dependency was replaced with a Python 3.12-compatible range.

Additional dependencies were introduced for the new architecture:

```text
aws-lambda-powertools
pydantic
```

The testing dependencies were also modernized.

---

## S3 Repository Pattern

Instead of directly calling boto3 from the Lambda handler:

```text
Lambda Handler
      │
      └── boto3
             │
             └── S3
```

the transformed architecture became:

```text
Lambda Handler
      │
      ▼
S3Repository
      │
      ▼
boto3 client
      │
      ▼
Amazon S3
```

This separates infrastructure access from business logic.

It also makes the S3 dependency easier to mock during testing.

---

# 15. Pydantic Input Validation

The custom transformation introduced Pydantic models to validate incoming Lambda events.

The architectural goal was:

```text
Incoming Event
      │
      ▼
Pydantic Validation
      │
      ├── Valid → Processing
      │
      └── Invalid → Controlled Error
```

This provides a stronger contract between the Lambda invocation mechanism and application code.

It also prevents malformed data from propagating deep into the processing logic.

---

# 16. Structured Logging

The transformed application adopted:

```text
aws-lambda-powertools
```

for structured logging.

The Lambda handler uses:

```python
@logger.inject_lambda_context
```

This improves observability by providing structured metadata and correlation information rather than relying solely on unstructured print statements.

The architecture therefore moved from:

```text
Lambda
  ↓
print()
  ↓
CloudWatch
```

toward:

```text
Lambda
  ↓
Structured Logger
  ↓
CloudWatch
  ↓
Searchable / correlated operational data
```

---

# 17. Automated Validation

After the transformation was applied, ATX ran:

```bash
python3.12 -m pytest tests/ -v
```

The transformation also evaluated explicit exit criteria.

The workshop reference validation included checks for:

- Tests passing
- Python 3.12 configured in SAM
- Python 3.12 configured in build configuration
- No direct S3 calls remaining in the processor
- `S3Repository` usage
- Structured logging
- Pydantic validation
- Modern dependency versions
- Updated test dependencies
- No unused imports
- Injectable repository client

The reference run produced:

```text
13 tests passed
```

with:

```text
5 original tests
+
8 new validation tests
=
13 passing tests
```

The exact test count can vary because the transformation is agentic and may generate additional tests based on the repository.

The important result is that the transformation satisfied its validation criteria.

The workshop documentation describes this validation-driven custom TD workflow in detail.

---

# 18. Publishing the Custom Transformation

Only after the transformation had been successfully applied and validated was it published.

The resulting transformation was conceptually named:

```text
python-lambda-modernization-3-12
```

with a description similar to:

```text
Upgrade Python 3.9 Lambda functions to 3.12 with repository
pattern for S3, structured logging via aws-lambda-powertools,
and Pydantic input validation.
```

Once published, the transformation became available in the AWS Transform custom registry.

This changes the transformation from a one-off AI experiment into a reusable modernization capability.

---

# 19. Author Once, Roll Out Everywhere

After publishing, the same custom transformation can be executed non-interactively:

```bash
atx custom def exec \
  -n python-lambda-modernization-3-12 \
  -p /path/to/other-project \
  -c "python3.12 -m pytest tests/ -v" \
  -x \
  -t
```

The conceptual workflow becomes:

```text
                   ┌──────────────────────┐
                   │ Custom Transformation│
                   │      Definition      │
                   └──────────┬───────────┘
                              │
              ┌───────────────┼───────────────┐
              ▼               ▼               ▼
          Repository A    Repository B    Repository C
              │               │               │
              ▼               ▼               ▼
          Python 3.12      Python 3.12      Python 3.12
          Repository       Repository       Repository
          Logging         Logging          Logging
          Validation      Validation       Validation
```

This is the core **"Author Once, Roll Out Everywhere"** concept.

---

# 20. Module 3 — Scaling Transformations

Interactive execution works well for one repository.

It becomes impractical when the organization has:

```text
10 repositories
100 repositories
1,000 repositories
10,000 repositories
```

The workshop therefore introduced the **scaled execution containers** architecture.

The factory was already deployed in the AWS-hosted workshop account.

The architecture was:

```text
Developer / CI/CD
       │
       ▼
API Gateway
       │
       ▼
Lambda
       │
       ▼
AWS Batch
       │
       ▼
Fargate
       │
       ▼
ATX Container
       │
       ▼
AWS Transform custom
       │
       ├───────────────┐
       ▼               ▼
    Source S3       Output S3

CloudWatch
    ▲
    │
Logs + Metrics
```

---

# 21. Why Containerized Batch Execution?

The key architectural insight is that AWS Transform custom does not need to be turned into a traditional "cluster."

Each Fargate task acts as an execution host for the ATX CLI.

AWS Batch handles:

- Queuing
- Scheduling
- Parallel execution
- Retries
- Job isolation

Fargate provides:

- Ephemeral compute
- Container isolation
- Automatic lifecycle management

S3 provides:

- Source and result storage

CloudWatch provides:

- Centralized logs
- Operational metrics
- Dashboards

This creates a serverless modernization factory.

---

# 22. Phase 1 — Assess the Fleet

The scaled workflow begins with assessment rather than immediately modifying every repository.

This follows the principle:

> **Assess first, upgrade second.**

The API endpoint and output bucket were retrieved from the pre-provisioned CloudFormation stack:

```bash
export API_ENDPOINT=$(aws cloudformation describe-stacks \
  --stack-name atx-factory \
  --query 'Stacks[0].Outputs[?OutputKey==`ApiEndpoint`].OutputValue' \
  --output text \
  --region us-east-1)

export OUTPUT_BUCKET=$(aws cloudformation describe-stacks \
  --stack-name atx-factory \
  --query 'Stacks[0].Outputs[?OutputKey==`OutputBucket`].OutputValue' \
  --output text \
  --region us-east-1)
```

These values can also be found in the Workshop Studio Event Outputs.

---

# 23. Batch Assessment

The assessment submitted the same managed transformation used earlier:

```text
AWS/comprehensive-codebase-analysis
```

but this time against multiple repositories.

The REST API was called using:

```bash
python3 utilities/invoke-api.py
```

The request followed this structure:

```json
{
  "batchName": "codebase-analysis-2026",
  "jobs": [
    {
      "source": "https://github.com/example/repository-1",
      "command": "atx custom def exec -n AWS/comprehensive-codebase-analysis -p /source/repository-1 -x -t"
    },
    {
      "source": "https://github.com/example/repository-2",
      "command": "atx custom def exec -n AWS/comprehensive-codebase-analysis -p /source/repository-2 -x -t"
    }
  ]
}
```

The API immediately returned a:

```text
batchId
```

Each repository became an independent AWS Batch job.

---

# 24. Monitoring Batch Execution

The batch status was queried using:

```bash
AWS_DEFAULT_REGION=us-east-1 \
python3 utilities/invoke-api.py \
  --endpoint "$API_ENDPOINT" \
  --method GET \
  --path "/jobs/batch/$BATCH_ID"
```

A representative response looks like:

```json
{
  "status": "RUNNING",
  "progress": 45.5,
  "totalJobs": 3,
  "statusCounts": {
    "RUNNING": 1,
    "SUCCEEDED": 2,
    "FAILED": 0
  }
}
```

This allowed the batch to be monitored without manually inspecting each Fargate task.

---

# 25. CloudWatch Monitoring

The solution also streams Batch logs into CloudWatch.

Logs can be followed with:

```bash
aws logs tail \
  /aws/batch/atx-transform \
  --follow \
  --region us-east-1
```

The workshop environment also provided CloudWatch dashboards for monitoring:

- Completion rates
- Successful transformations
- Failed transformations
- Agent minutes
- Execution duration
- API Gateway health
- Lambda health

This provides an operational view of modernization rather than only a developer view.

---

# 26. Assessment Results

The assessment phase produces information that can be used to prioritize modernization waves.

For example:

| Repository | Finding | Recommended Transformation |
|---|---|---|
| Spring Petclinic | Java version modernization | `AWS/java-version-upgrade` |
| Python Lambda application | Python 3.8 EOL | `AWS/python-version-upgrade` |
| Node.js application | Node.js 16 EOL | `AWS/nodejs-version-upgrade` |

This is much more scalable than blindly applying the same transformation to every repository.

The fleet can instead be divided into modernization waves:

```text
Fleet
 │
 ├── Critical / EOL
 │       └── Wave 1
 │
 ├── High priority
 │       └── Wave 2
 │
 ├── Medium priority
 │       └── Wave 3
 │
 └── Low priority
         └── Wave 4
```

---

# 27. Phase 2 — Targeted Upgrades

Once the assessment identifies what each repository needs, the second batch applies targeted transformations.

For example:

```text
Spring Petclinic
    → AWS/java-version-upgrade
    → Java 21

Python Lambda
    → AWS/python-version-upgrade
    → Python 3.13

Node.js application
    → AWS/nodejs-version-upgrade
    → Node.js 22
```

The important principle is:

> **Do not upgrade everything with the same transformation.**

Instead:

```text
Assessment
    ↓
Identify need
    ↓
Select appropriate TD
    ↓
Set target version
    ↓
Submit transformation
```

This reduces unnecessary transformation execution and makes modernization waves easier to control.

---

# 28. Troubleshooting the Batch API

One practical issue encountered during the workshop was an empty API endpoint.

The command initially failed with:

```text
ValueError: Invalid endpoint URL scheme.
Must start with https:// or http://.
Got:
```

The root cause was that:

```bash
$API_ENDPOINT
```

was empty.

The API helper therefore received:

```text
endpoint = ""
```

instead of:

```text
endpoint = https://...
```

The solution was to retrieve the stack output again:

```bash
export API_ENDPOINT=$(aws cloudformation describe-stacks \
  --stack-name atx-factory \
  --query 'Stacks[0].Outputs[?OutputKey==`ApiEndpoint`].OutputValue' \
  --output text \
  --region us-east-1)
```

Then verify:

```bash
echo "$API_ENDPOINT"
```

This is a useful troubleshooting lesson because the failure occurred **before AWS Batch was contacted**.

The sequence was:

```text
invoke-api.py
     │
     ▼
Validate endpoint
     │
     X
Empty API_ENDPOINT
     │
     ▼
Request never reaches API Gateway
     │
     ▼
No Batch job is created
```

---

# 29. Reviewing Results from S3

Once transformations completed, results were stored in the output bucket.

The complete output could be synchronized locally:

```bash
aws s3 sync \
  s3://$OUTPUT_BUCKET/transformations/ \
  ./results/ \
  --exclude "*/node_modules/*" \
  --exclude "*/.venv/*"
```

The structure follows a predictable pattern:

```text
results/
├── repository-1-comprehensive-codebase-analysis/
│   └── <timestamp>_<conversation-id>/
│       ├── code/
│       └── logs/
│
├── repository-1-python-version-upgrade/
│   └── <timestamp>_<conversation-id>/
│       ├── code/
│       └── logs/
│
└── repository-2-python-version-upgrade/
    └── <timestamp>_<conversation-id>/
        ├── code/
        └── logs/
```

Each job can therefore be reviewed independently.

---

# 30. Transformation Artifacts

The transformation workflow produces useful artifacts rather than only modified source code.

Examples include:

```text
worklog.log
validation_summary.md
git_instructions.md
SKILL.md
```

These artifacts provide different perspectives:

| Artifact | Purpose |
|---|---|
| `SKILL.md` | Transformation definition |
| `worklog.log` | Agent execution history |
| `validation_summary.md` | Validation and exit criteria |
| `git_instructions.md` | Git/PR integration guidance |

This is particularly valuable when AI-assisted transformations are used in engineering environments because the result can be audited and reviewed.

---

# 31. Scaling Custom Transformations

The most important extension of the workshop is that the same mechanism used for AWS-managed transformations can also execute a custom transformation.

Conceptually:

```text
Custom TD
    │
    ├── Repository A
    ├── Repository B
    ├── Repository C
    ├── Repository D
    └── Repository N
```

A custom modernization definition can therefore become an organizational standard.

For example:

```text
python-lambda-modernization-3-12
```

could be applied to an entire fleet of Python Lambda repositories.

That makes the custom TD more than an experiment.

It becomes a reusable modernization policy.

---

# 32. CI/CD Integration

The REST API is IAM authenticated, which makes it suitable for automation.

Possible triggers include:

### Pull Request Merge

```text
PR merged
   ↓
CI/CD pipeline
   ↓
Submit transformation
   ↓
Monitor batch/job
   ↓
Validate
```

### Nightly Modernization

```text
Nightly schedule
      ↓
Discover repositories
      ↓
Submit assessment batch
      ↓
Prioritize results
      ↓
Submit upgrade waves
```

### Compliance Deadline

```text
Compliance requirement
        ↓
Identify EOL workloads
        ↓
Assess fleet
        ↓
Submit prioritized upgrades
        ↓
Track completion
```

This turns modernization from a one-time project into an operational capability.

---

# 33. Measure — Understanding the Economics

One of the important lessons from the workshop is the difference between:

### Agent execution cost

and:

### Wall-clock execution time

Running jobs in parallel reduces elapsed time, but it does not make the underlying AI reasoning free.

Conceptually:

```text
Sequential

Repo A ──────────
Repo B              ──────────
Repo C                         ──────────

Long wall-clock time
```

versus:

```text
Parallel

Repo A ──────────
Repo B ──────────
Repo C ──────────

Much shorter wall-clock time
```

The agent work is still performed for each repository.

Therefore:

> **Parallelism primarily improves throughput and wall-clock time; it does not eliminate agent-minute consumption.**

This distinction is important when designing large-scale modernization programs.

---

# 34. Interactive vs Batch Execution

The workshop demonstrates two complementary execution models.

| Capability | Interactive | Batch |
|---|---|---|
| Typical scale | 1–5 repositories | Dozens to thousands |
| Execution | Developer terminal | AWS Batch/Fargate |
| Input | Local project | Git repository source |
| Output | Local ATX artifacts | S3 |
| Monitoring | Terminal/session | API + CloudWatch |
| CI/CD | Manual | API-native |
| Parallel execution | Limited | Designed for parallelism |
| Failure isolation | Repository/session | Per Batch job |

The key point is not that batch replaces interactive execution.

Instead:

> **Interactive execution is ideal for authoring and validating a transformation; batch execution is ideal for rolling out an already-understood transformation at scale.**

---

# 35. End-to-End Workflow

The complete workshop can be summarized as:

```text
                    ┌─────────────────┐
                    │ Legacy Codebase │
                    └────────┬────────┘
                             │
                             ▼
                  ┌─────────────────────┐
                  │ Comprehensive       │
                  │ Codebase Analysis   │
                  └──────────┬──────────┘
                             │
                             ▼
                     ATXDocumentation/
                             │
                             ▼
                  ┌─────────────────────┐
                  │ Transformation      │
                  │ Planning            │
                  └──────────┬──────────┘
                             │
                ┌────────────┴────────────┐
                │                         │
                ▼                         ▼
        AWS Managed TD             Custom TD
                │                         │
                │                         ▼
                │                Architecture Interview
                │                         │
                │                         ▼
                │                     SKILL.md
                │                         │
                └────────────┬────────────┘
                             ▼
                     Apply Transformation
                             │
                             ▼
                       Run Validation
                             │
                       ┌─────┴─────┐
                       │           │
                      FAIL        PASS
                       │           │
                       ▼           ▼
                    Refine       Publish
                                   │
                                   ▼
                           Transformation Registry
                                   │
                                   ▼
                         ┌────────────────────┐
                         │ Batch Modernization│
                         └─────────┬──────────┘
                                   │
                                   ▼
                              API Gateway
                                   │
                                   ▼
                                Lambda
                                   │
                                   ▼
                              AWS Batch
                                   │
                                   ▼
                                Fargate
                                   │
                                   ▼
                               ATX CLI
                                   │
                         ┌─────────┴─────────┐
                         ▼                   ▼
                       S3                 CloudWatch
                    Results               Logs/Metrics
```

---

# 36. Key Engineering Lessons

## 1. Analyze before transforming

The analysis phase gives the transformation engine architectural context and helps identify the correct modernization strategy.

---

## 2. Establish a working baseline

Testing the legacy application before transformation provides a reference point for determining whether modernization preserved functionality.

---

## 3. Prefer managed transformations when they fit

If AWS already provides a transformation for the required modernization, use it instead of creating unnecessary custom logic.

---

## 4. Use custom TDs for organizational patterns

Custom transformations become valuable when modernization requirements combine:

- Runtime upgrades
- Architecture changes
- Coding standards
- Security requirements
- Observability patterns
- Validation requirements

---

## 5. Validate before publishing

The safer lifecycle is:

```text
Draft
  ↓
Apply
  ↓
Test
  ↓
Validate
  ↓
Publish
```

rather than:

```text
Create
  ↓
Publish
  ↓
Discover that it doesn't work
```

---

## 6. Git is part of the safety model

Transformation branches and commits make automated AI changes inspectable and reviewable.

---

## 7. Assessment should precede fleet-wide upgrades

A large fleet should not be treated as homogeneous.

Different repositories may need:

```text
Java upgrade
Python upgrade
Node.js upgrade
Dependency modernization
Custom architecture transformation
No action
```

Assessment allows modernization waves to be prioritized intelligently.

---

## 8. Parallelism changes throughput

AWS Batch and Fargate allow many repositories to be transformed concurrently.

The primary benefit is:

```text
Lower wall-clock time
+
Higher throughput
+
Failure isolation
```

not elimination of AI execution cost.

---

# 37. Final Results

The workshop demonstrated modernization across three applications.

### QWords

```text
Python 3.8
     ↓
Python 3.12
     ↓
AWS/python-version-upgrade
     ↓
53 tests passing
```

### Emoji Algebra

```text
Python 3.8
     ↓
Python 3.12
     ↓
AWS/python-version-upgrade
     ↓
48 tests passing
```

### Scientific Processor

```text
Python 3.9 Lambda
        ↓
Python 3.12
        +
S3 Repository Pattern
        +
Structured Logging
        +
Pydantic Validation
        +
Updated Dependencies
        +
Expanded Tests
        ↓
Custom Transformation
        ↓
13 tests passing in the reference run
        ↓
Transformation Published
```

The custom transformation was then conceptually ready for reuse across other compatible repositories.

---

# 38. What This Demonstrates for Real Organizations

The most interesting aspect of the workshop is not simply upgrading Python.

The larger pattern is an approach to **AI-assisted modernization at organizational scale**.

A centralized team can define modernization standards such as:

```text
Python 3.12
+
AWS Lambda Powertools
+
Pydantic
+
Repository pattern
+
Testing requirements
+
Security requirements
```

and encode those standards into a reusable transformation definition.

The transformation can then be validated against a representative application and subsequently applied to an entire fleet.

This changes the modernization model from:

```text
Engineer-by-engineer manual migration
```

to:

```text
Centralized modernization policy
              ↓
        Transformation TD
              ↓
       Automated validation
              ↓
          Fleet rollout
```

---

# 39. Repository Structure

A GitHub repository documenting this workshop can be organized as:

```text
aws-transform-custom-workshop/
│
├── README.md
│
├── applications/
│   ├── qwords/
│   ├── emoji-algebra/
│   └── scientific-processor/
│
├── transformations/
│   └── python-lambda-modernization-3-12/
│       ├── SKILL.md
│       └── references/
│
├── batch/
│   ├── assessment.json
│   ├── upgrade.json
│   └── invoke-api.py
│
├── results/
│   ├── validation/
│   └── screenshots/
│
└── docs/
    ├── architecture.md
    ├── troubleshooting.md
    └── scaling.md
```

Actual source code, transformation definitions, credentials, private repository URLs, account identifiers, and temporary workshop artifacts should be reviewed before publishing.

---

# 40. Conclusion

This workshop demonstrated a complete AI-assisted modernization lifecycle using AWS Transform custom.

The progression was:

```text
ANALYZE
Understand the application
        ↓
TRANSFORM
Apply managed transformations
        ↓
AUTHOR
Create organization-specific transformation definitions
        ↓
VALIDATE
Run tests and transformation exit criteria
        ↓
PUBLISH
Register validated transformations
        ↓
SCALE
Execute transformations across a fleet
        ↓
MEASURE
Track execution, quality, cost, and operational health
```

The key takeaway is that AI-assisted modernization becomes significantly more useful when it is treated as an **engineering workflow**, rather than simply an AI coding exercise.

The combination of:

- structured codebase analysis,
- reusable transformation definitions,
- Git-based change isolation,
- automated validation,
- registry-based reuse,
- containerized execution,
- AWS Batch parallelism,
- S3 artifact storage,
- CloudWatch monitoring,
- and API/CI-CD integration

provides a foundation for turning individual modernization experiments into repeatable modernization programs.

**The central pattern demonstrated by this workshop is:**

> **Analyze once, encode the modernization intent into a reusable transformation, validate it against a real application, publish it, and then scale that transformation across the fleet.**
