# Security Analysis

STRIDE threat modeling, adversarial attack simulation, and AI red team analysis for technical specifications.

> **Part of the spec-review suite.** Can run standalone (performs its own discovery) or as a sub-command of `/pmp:spec-review` (discovery already done). When standalone, first execute [discovery.md](../../spec-review/references/discovery.md).

## Standalone Mode

When invoked directly (not dispatched by orchestrator):
1. Read [discovery.md](../../spec-review/references/discovery.md) and execute Phase 0 + Phase 1
2. Proceed to the analysis phases below
3. Produce a standalone report using [spec-security-output.md](../assets/spec-security-output.md)

When dispatched by `/pmp:spec-review`, skip directly to the analysis phases — the system model is already in context.

---

### Phase 6: Threat Modeling

Use the STRIDE model. Analyze threats for:
- **Spoofing** — identity forgery, token replay
- **Tampering** — data modification, parameter manipulation
- **Repudiation** — missing audit trails, unsigned actions
- **Information Disclosure** — data leaks, timing side-channels
- **Denial of Service** — resource exhaustion, amplification
- **Elevation of Privilege** — access control bypass, role confusion

Also evaluate **AI/Agentic-specific threats** if the system involves LLMs or agents:
- prompt injection (direct and indirect)
- tool misuse and unsafe tool chaining
- sensitive data leakage via model output
- prompt/system prompt exfiltration
- policy bypass through crafted prompts
- jailbreak attacks

For each threat provide: **Threat**, **Attack path**, **Impact**, **Severity**, **Mitigation**.

---

### Phase 7: Attack Simulation

Simulate adversarial scenarios:

**Infrastructure attacks:**
1. Malicious client flooding the proxy
2. Attacker attempting policy bypass
3. Malicious upstream response
4. Request smuggling
5. Resource exhaustion attacks

**AI/Agentic attacks** (if applicable):
1. Prompt injection propagation
2. Tool execution abuse
3. System prompt leakage
4. Model output exfiltration
5. Jailbreak bypass
6. Unsafe tool chaining
7. Indirect prompt injection via documents

For each scenario: explain how the system would behave and identify weaknesses.

---

### Phase 13: AI Red Team Attack Analysis (Conditional)

> **Trigger:** Run this phase ONLY when the system under review involves AI/LLM features — AI gateways, LLM proxies, agentic systems, tool-calling architectures, RAG pipelines, prompt routing, or any component where an LLM processes user-influenced input. Skip entirely for non-AI systems.

Switch mindset: you are now an **adversarial AI security researcher** performing a red-team attack against the specification. Your objective is to **break the system design**. Think like a creative attacker, not a defender. Focus on discovering realistic exploit paths.

Assume the system will be deployed in enterprise environments with adversarial users attempting to bypass policy enforcement, execute unauthorized tools, exfiltrate sensitive data, override system prompts, perform denial-of-service, escalate privileges, and leak secrets.

#### 13.1 Attack Surface Mapping

Identify every entry point where an attacker could influence system behavior:

- User prompts and conversation history
- Tool arguments and tool definitions
- System prompt injection vectors
- RAG document inputs and retrieval results
- External URLs referenced in prompts or tool calls
- Streaming response channels
- Model outputs consumed by downstream systems
- Logs and observability pipelines
- Metadata fields (headers, request IDs, tenant IDs)
- HTTP headers and authentication tokens
- Request batching and queuing systems
- Configuration and policy definition interfaces

For each entry point, map it to the internal components it reaches and the trust boundary it crosses.

#### 13.2 Prompt Injection Attacks

Design attacks that could override system instructions:

- **Instruction hierarchy attacks** — exploit ambiguity between system, user, and assistant roles
- **Hidden instructions in long prompts** — bury override directives in lengthy context
- **Multi-step prompt manipulation** — build up context across turns to shift model behavior
- **Context poisoning via RAG documents** — inject instructions into retrieved content
- **Tool override prompts** — craft prompts that redefine tool behavior or arguments
- **Jailbreak attempts** — bypass safety filters through roleplay, encoding, or framing

For each attack, document:

| Field | Content |
|-------|---------|
| **Attack Prompt** | The crafted input |
| **Attack Goal** | What the attacker wants to achieve |
| **Expected System Behavior** | How the system should respond |
| **Possible Bypass** | How the attack might circumvent defenses |

#### 13.3 Tool Execution Exploits

Attempt to exploit the tool execution system:

- **Argument injection** — craft tool arguments that execute unintended operations
- **Command chaining** — chain multiple tool calls to achieve unauthorized outcomes
- **Unsafe shell arguments** — inject shell metacharacters through tool parameters
- **Malicious URLs** — pass URLs that trigger SSRF, redirect to internal services, or exfiltrate data
- **SSRF via tool requests** — use tools to probe internal network topology
- **Indirect tool triggers** — manipulate model output so downstream parsing triggers tool execution

Evaluate whether the model could be manipulated into calling tools it should not, calling tools with arguments it should not, or calling tools in sequences that bypass individual-call validation.

#### 13.4 Data Exfiltration Attacks

Attempt to extract sensitive information:

- **System prompt extraction** — trick the model into revealing its instructions
- **Internal policy leakage** — extract policy rules, guardrails, or filtering logic
- **Cross-conversation leakage** — access data from other users' sessions
- **API key and secret extraction** — probe for credentials in model context
- **Tool secret leakage** — extract authentication tokens used by tools
- **Training data extraction** — attempt to surface memorized sensitive data

For each vector, design a concrete prompt that attempts the extraction and assess whether the spec's defenses would prevent it.

#### 13.5 Policy Bypass Attacks

Attempt to bypass the policy enforcement system:

- **Prompt obfuscation** — rewrite prohibited content using synonyms, metaphors, or coded language
- **Encoding attacks** — use Base64, ROT13, Unicode escapes, or other encodings to evade text-based filters
- **Multi-step reasoning attacks** — split a prohibited request across multiple benign-looking turns
- **Token splitting** — break sensitive keywords across token boundaries
- **Instruction shadowing** — place conflicting instructions that override policy checks
- **Indirect tool triggering** — achieve a prohibited outcome through a sequence of individually-permitted tool calls

For each attack, explain exactly which policy check it bypasses and why.

#### 13.6 Model Output Exploits

Evaluate whether model output could be weaponized:

- **Malicious links in output** — model generates phishing or exploit URLs
- **Unsafe tool execution instructions** — output that, if consumed by automation, triggers dangerous actions
- **Generated prompts that bypass filters** — model produces content that poisons its own future context
- **Recursive prompt loops** — output designed to create infinite processing loops
- **XSS/injection via output** — model output rendered in UI without sanitization

Determine whether downstream systems (UIs, APIs, logging pipelines, other agents) could be exploited by model-generated content.

#### 13.7 Resource Exhaustion Attacks

Attempt to cause system failure through resource abuse:

- **Token flooding** — submit prompts designed to maximize token consumption
- **Extremely long prompts** — test context window limits and memory allocation
- **Streaming abuse** — hold streaming connections open indefinitely
- **Recursive tool calls** — trigger tool chains that amplify into unbounded execution
- **Concurrent request floods** — overwhelm rate limiting or connection pools
- **Slow client attacks** — send requests at minimum speed to exhaust server resources
- **Cost amplification** — small attacker input triggers expensive downstream operations (LLM calls, API calls, database queries)

Evaluate whether the spec defines defenses for each vector.

#### 13.8 Multi-Tenant Attacks

Attempt to exploit tenant isolation boundaries:

- **Prompt leakage across tenants** — access another tenant's system prompts or conversation history
- **Shared cache poisoning** — inject content into caches that affects other tenants' responses
- **Shared policy bypass** — exploit policy definitions that bleed across tenant boundaries
- **Shared model context leaks** — influence model behavior for other tenants through shared state
- **Tenant ID spoofing** — forge tenant identifiers to access other tenants' resources

For each attack, assess whether the spec enforces sufficient isolation.

#### 13.9 Advanced Attacks

Attempt creative and emerging attack vectors:

- **Indirect prompt injection via documents** — embed instructions in files, emails, or web pages that the system retrieves
- **Adversarial prompt encoding** — use Unicode homoglyphs, zero-width characters, or bidirectional text markers to hide instructions
- **Chain-of-thought manipulation** — influence the model's reasoning process to reach attacker-desired conclusions
- **Hidden Unicode attacks** — embed invisible characters that alter parsing or filtering behavior
- **Context window poisoning** — gradually fill the context with attacker-controlled content that shifts model behavior
- **Malicious prompt compression** — craft inputs that expand dramatically during tokenization or processing
- **Model fingerprinting** — probe the system to identify the underlying model, version, and configuration for targeted attacks
- **Timing side-channels** — infer internal state from response latency patterns
