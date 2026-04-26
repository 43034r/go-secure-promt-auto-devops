# go-secure-promt

**A secure AI agent runner for GitLab CI/CD pipelines.**

Written in Go, `secure-promt` embeds an LLM-powered DevOps assistant directly into a CI pipeline. The agent receives a high-level goal — such as investigating a deployment failure or auditing Kubernetes pod logs — and autonomously selects and executes the tools needed to accomplish it.

**Prompt encryption.** Before a task reaches the pipeline, the operator encrypts the goal using `secure-promt provision`. The resulting payload is sealed with AES-256-GCM using a root-only key stored on the runner host. At runtime, `secure-promt execute` decrypts it in memory — the plaintext prompt never appears in the CI job definition, logs, or environment variables.

**Build-time secret injection.** LLM API keys (OpenAI, Google Gemini) are compiled directly into the binary via `-ldflags` at build time. They do not exist as files or environment variables on the runner, reducing the surface for credential leakage.

**Command interception firewall.** A `bash` DEBUG trap intercepts every shell command executed during the CI job. Commands are checked against a strict allowlist before execution, blocking script-injection attacks that could otherwise hijack the agent's shell access.

**Dynamic tooling via MCP.** The agent discovers available tools at runtime from a catalog of MCP servers (Model Context Protocol — JSON-RPC over SSE). This gives it access to structured integrations like GitLab API calls, Kubernetes log retrieval, and Helm operations, without relying solely on raw shell commands. The LLM backend is provided by [langchaingo](https://github.com/tmc/langchaingo).
