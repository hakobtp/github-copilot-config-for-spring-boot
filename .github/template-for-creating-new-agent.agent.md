---
# =============================================================================
# AGENT FRONTMATTER TEMPLATE
# -----------------------------------------------------------------------------
# This file is a documented template for creating a new agent under
# `.github/agents/`. Copy it to `<your-agent>.agent.md`, fill in every field
# below, then replace this comment block with the agent's system prompt body
# (markdown after the closing `---`).
#
# File-naming convention: `<role-or-purpose>.agent.md`
#   examples: `pm.agent.md`, `architect.agent.md`, `code-review.agent.md`
# =============================================================================

# -----------------------------------------------------------------------------
# name (REQUIRED, string)
# -----------------------------------------------------------------------------
# Human-readable display name of the agent shown in the UI / agent picker.
# Use Title Case. Keep it short and descriptive of the role (NOT the file name).
#   good: 'Product Manager', 'Software Architect', 'Code Review Agent'
#   bad : 'pm', 'agent1', 'my-cool-agent'
name:

# -----------------------------------------------------------------------------
# description (REQUIRED, string — single paragraph)
# -----------------------------------------------------------------------------
# One-line elevator pitch of WHAT the agent does, WHAT it consumes as input,
# and WHAT it produces as output. This is the text other agents and the user
# see when deciding whether to invoke it, so be concrete about scope and
# explicit about what it does NOT do.
# Pattern: '<persona> persona. <consumes X> and produces <Y> for <consumer>.
#           Does not <out-of-scope thing>.'
#   example: 'Product Manager persona. Turns a raw idea or bug report into a
#             structured Markdown plan (problem, scope, user stories, acceptance
#             criteria) for the architect and dev agents. Does not write code.'
description:

# -----------------------------------------------------------------------------
# argument-hint (OPTIONAL, string)
# -----------------------------------------------------------------------------
# Short placeholder text shown next to the agent name to hint what argument
# the user should type when invoking it (e.g. a ticket id, a file path, a
# free-form prompt). Leave empty if the agent takes no positional argument.
#   examples: '<ticket-id>', '<path/to/file>', '<feature description>'
argument-hint:

# -----------------------------------------------------------------------------
# user-invocable (REQUIRED, boolean: true | false)
# -----------------------------------------------------------------------------
# Whether the END USER may invoke this agent directly from the agent picker
# / slash command.
#   true  -> appears in the user-facing agent list (e.g. pm, architect, dev).
#   false -> hidden from the user; can only be called as a sub-agent by
#            another agent via `handoffs` or `run_subagent`. Use this for
#            internal helper agents (e.g. a CVE-checker, a diagram generator).
user-invocable:

# -----------------------------------------------------------------------------
# tools (REQUIRED, list of strings — may be empty)
# -----------------------------------------------------------------------------
# Whitelist of tool names this agent is allowed to call. The agent CANNOT use
# any tool that is not in this list. Grant the minimum set needed for its job
# (principle of least privilege).
#
# Common tool groups for this repo:
#   read-only      : 'read_file', 'open_file', 'list_dir', 'file_search',
#                    'grep_search', 'get_errors', 'get_terminal_output'
#   write / edit   : 'insert_edit_into_file', 'replace_string_in_file',
#                    'create_file', 'apply_patch'
#   shell          : 'run_in_terminal'
#   user dialog    : 'ask_questions'
#   security       : 'validate_cves'
#   delegation     : 'run_subagent'
#
# Guidance:
#   - Review-only / read-only agents should NOT include write tools.
#   - Planning agents (pm, architect) typically need create_file + edit tools
#     to persist their plan/design under `@workspace/temp/`.
#   - Use `tools: []` ONLY if the agent should produce text in chat with no
#     tool access at all.
tools: []

# -----------------------------------------------------------------------------
# handoffs (OPTIONAL, list of handoff entries)
# -----------------------------------------------------------------------------
# Declares the next agents this one can hand control off to once its own work
# is approved by the user. The UI renders one button per entry. Keep handoffs
# aligned with this repo's gated workflow:
#       pm  →  architect  →  dev
# Omit this key entirely (or use `handoffs: []`) if the agent is terminal
# (e.g. code-review, terminal-helper).
#
# Each entry has four fields:
#   label  : Button text shown to the user.
#            example: 'Hand off to Architect'
#   agent  : File name (with extension) of the target agent to invoke.
#            example: 'architect.agent.md'
#   prompt : Initial prompt sent to the target agent. Reference the artifact
#            the current agent produced so the next agent knows where to read
#            from. Supports variables like `@workspace/temp/task.md`.
#            example: 'Read the approved plan at @workspace/temp/task.md and
#                      produce an architecture design.'
#   send   : Optional extra context/data appended to the prompt. Leave blank
#            if `prompt` already contains everything the next agent needs.
handoffs:
  - label:
    agent:
    prompt:
    send:
---

<!--
  Replace everything below this line with the agent's SYSTEM PROMPT BODY.

  Recommended sections (see pm.agent.md / architect.agent.md / dev.agent.md
  for full examples):

    # <Agent Name> Agent
    One-paragraph identity statement: "You are a <role> for this <repo>..."

    ## Workflow & user-approval gates
    Show where this agent sits in the pm → architect → dev pipeline and the
    rule that handoffs require an EXPLICIT user approval.

    ## Input artifact
    Where this agent READS from (e.g. `@workspace/temp/task*.md`) and the
    rules for treating it as read-only / single source of truth.

    ## Output artifact
    Where this agent WRITES its result (e.g. `@workspace/temp/design*.md`),
    the file-naming convention, and the rule to keep the file in sync with
    chat.

    ## Scope
    What the agent does and does NOT do. Cite the project rules it must
    obey (files under `.github/instructions/` and
    `.github/copilot-instructions.md`).

    ## What a good output contains
    Numbered sections / template the agent must follow when producing its
    artifact.

    ## Behavior rules
    Hard constraints (e.g. "no code", "one question at a time", "wait for
    approval before handoff", "never write to the previous agent's file").
-->