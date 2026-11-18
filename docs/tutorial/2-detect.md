# 2. Detect

**Goal:** introduce a proposed Gemini model for driver questions and an MCP connection one
tracked change at a time, and read how the draft card and questionnaire change.

**You will need:** the repository from [page 1](1-setup.md), with its
checkpoint reached.

## Try it

1. **Add the Gemini dependency and model configuration.** Copy the example
   inputs into the tracked paths the scanner recognizes, commit, and scan:

   **PowerShell:**

   ```powershell
   Copy-Item workshop/gemini-requirements.example requirements.txt
   New-Item -ItemType Directory -Force config | Out-Null
   Copy-Item workshop/models.example config/models.yaml
   ```

   **bash (Linux and macOS):**

   ```bash
   cp workshop/gemini-requirements.example requirements.txt
   mkdir -p config
   cp workshop/models.example config/models.yaml
   ```

   Then commit and scan (the same in both shells):

   ```sh
   git add -A
   git commit -m "Record the proposed Gemini driver integration"
   trustbom scan
   ```

   [`trustbom scan`](../cli/scan.md#example) writes the same four artifacts
   and prints a **changed** fingerprint, because the model is now in the graph:

   ```text
   Graph fingerprint: <12 lowercase hex characters>
   Full fingerprint (SHA-256): <64 lowercase hex characters>
   Found 1 model(s), 0 tool(s), 2 finding(s). Review <repo>/trust/card.md.
   ```

2. **Read the model in `trust/card.json`** (selected fields). Each citation
   points at the exact file Trust will review:

   ```json
   {
     "models": [
       {
         "provider": "gemini",
         "model_id": "gemini-2.5-flash",
         "source": "inferred",
         "evidence": [
           { "path": "config/models.yaml", "start_line": 1 },
           { "path": "requirements.txt", "start_line": 1 }
         ]
       }
     ],
     "findings": [
       { "id": "region-unknown", "severity": "medium" },
       { "id": "subprocessor-undeclared", "severity": "medium" }
     ]
   }
   ```

   The two findings give Trust concrete questions: where will inference run,
   and which processor agreement covers it?

3. **Read the draft answers in `trust/questionnaire.yml`** (selected entries).
   They render the same graph, never an independent source:

   ```yaml
   status: "draft (unapproved)"
   graph_sha: "<64 lowercase hex characters>"
   questions:
     - id: Q2_models_and_vendors
       question: "Which models and vendors are used?"
       answer: "gemini: gemini-2.5-flash"
       cite:
         - "config/models.yaml"
         - "requirements.txt"
     - id: Q3_is_customer_data_used_to_train
       question: "Is customer data used to train?"
       answer: "unknown"
       cite: []
   ```

   `unknown` means the repository holds no evidence either way; absence of
   evidence is never read as "no".

4. **Add the MCP connection.** Our customers also ask how an AI client could
   reach driver earnings. Introduce a tracked MCP **configuration**:

   **PowerShell:**

   ```powershell
   Copy-Item workshop/mcp-config.example mcp.json
   ```

   **bash (Linux and macOS):**

   ```bash
   cp workshop/mcp-config.example mcp.json
   ```

   ```sh
   git add -A
   git commit -m "Record the proposed earnings MCP connection"
   trustbom scan
   ```

   The fingerprint changes again and the card gains an `mcp_servers` row, while
   the model from step 1 remains:

   ```json
   {
     "mcp_servers": [
       {
         "name": "synthetic-driver-earnings",
         "command": "driver-earnings-mcp",
         "source": "inferred",
         "evidence": [{ "path": "mcp.json", "start_line": 3 }]
       }
     ]
   }
   ```

5. **Commit the scan result**, so the card on this branch matches the graph
   you will review next:

   ```sh
   git add -A
   git commit -m "Rescan with the proposed Gemini model"
   ```

> **In production.** The scanner reads manifests, lockfiles and allowlisted
> configuration, not source code or runtime traffic, so a missing detection is
> unknown rather than absent. Before approval, a team verifies the MCP server
> implementation, its permissions and employee access, and records facts the
> scanner cannot see in `trust/manual.yml`
> ([declarations](../manual-declarations.md#declaring-data-handling)). Copilot,
> the next AI feature we might add, would be a separate integration with its own
> evidence, not an automatic second model binding.

## Checkpoint

- The model count went from 0 to 1 and the fingerprint changed twice.
- `trust/card.json` has one model (with two citations), one MCP server and the
  findings `region-unknown` and `subprocessor-undeclared`.
- `git status` is clean.

Next: [3. Approve](3-approve.md).
