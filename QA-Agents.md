## AI Agents that handle the routine QA work

Here are the AI agents I built to take the repetitive work off my plate. They connect to Jira and Confluence through the Atlassian MCP, help me review tickets, write test cases and report bugs, and always leave the final check to me.

<table>
  <thead>
    <tr>
      <th>#</th>
      <th>Agent</th>
      <th>What it does</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>1</td>
      <td><a href="#agent-1-qa-requirements-agent">QA Requirements Agent</a></td>
      <td>Spots what’s missing in a ticket before development starts, and writes acceptance criteria you can test</td>
    </tr>
    <tr>
      <td>2</td>
      <td><a href="#agent-2-test-case-generator">Test Case Generator</a></td>
      <td>Turns a ticket into structured test cases, ready for Zephyr Scale</td>
    </tr>
    <tr>
      <td>3</td>
      <td><a href="#agent-3-bug-report-agent">Bug Report Agent</a></td>
      <td>Turns my quick one-line description into a complete Jira bug report</td>
    </tr>
  </tbody>
</table>

### How the agents fit together

```mermaid
flowchart TD
    A["New Jira ticket"] --> B["Requirements Agent"]
    B -->|gaps + acceptance criteria| C["Test Case Generator"]
    C -->|test cases for Zephyr Scale| D["Testing"]
    D -->|bug found| E["Bug Report Agent"]
    E -->|structured Jira bug| F["Fix and retest"]
```

## Agent 1: QA Requirements Agent

### 1. Problem

- Some tickets were created by the tech team based on their own investigation, so it wasn't always clear what had actually changed.
- Acceptance criteria were missing.
- There was no information for testing: what to check, which roles or flows were affected, or what the expected behaviour was.
- Testing couldn't start until I tracked down the answers, which slowed down the whole release.

### 2. What it does

1. Reads a Jira ticket from start to finish.
2. Flags what's missing, unclear or conflicting, and explains why each gap matters.
3. Turns each gap into a question for the product owner and lists them in ticket notes.
4. Drafts acceptance criteria in a Given / When / Then format, and marks the ones that depend on an open question.

### 3. How I built it

1. Set it up as a shared Claude project, so the whole team uses the same agent and the same rules.
2. Wrote a checklist of what a complete ticket needs: What changed, scope, roles and permissions, statuses, data and system impact, acceptance criteria, and edge cases.
3. Told the agent never to fill a gap with a guess. If something is missing, it asks.
4. Fixed the output format, so every review looks the same and is quick to read.

### 4. How it looks in chat

<img width="592" height="565" alt="image" src="https://github.com/user-attachments/assets/942b3e18-5e76-4baa-acea-4833dcfbe25e" />


## Agent 2: Test Case Generator

### 1. Problem

- Every new ticket needed a full set of test cases, and writing them by hand took a big part of each sprint.
- With tight deadlines, there was a risk of skipping less obvious checks, like role access, boundaries or negative cases.

### 2. What it does

1. Reads a Jira ticket and generates structured test cases with steps and expected results.
2. Covers the happy path, data accuracy, boundaries, status changes, role access, export and negative cases.
3. Shows coverage metrics: requirement coverage, traceability and the share of functional, negative and edge cases.

### 3. How I built it

1. Gave the agent the test design techniques I use: equivalence partitioning, boundary values, decision tables, state transitions and role-based access testing.
2. Defined a fixed set of coverage areas, so no ticket skips negative or access tests.
3. Required every test case to point to an acceptance criterion, so gaps in coverage show up right away.
4. Added a metrics summary to every result, so I can check coverage before importing anything.
5. Matched the output to the Zephyr Scale import format.

### 4. How it looks in chat

<img width="588" height="600" alt="image" src="https://github.com/user-attachments/assets/2b0b8c58-ee94-4ad6-8acd-fceca4a396c0" />


---

## Agent 3: Bug Report Agent

### 1. Problem

Every bug needs the full structure: title format, preconditions, steps, actual and expected results, environment. Writing each one by hand was repetitive and took time away from testing.

### 2. What it does

1. Takes my short note and writes a full bug report: title, description, preconditions, steps, actual and expected results, logs, environment.
2. Asks before writing if something is missing, such as role, interface, rotation status, or environment
3. Uses context and technical evidence I give it (console errors, API responses) to suggest a likely cause.
4. Shows a draft first and creates the Jira ticket only after I approve.

### 3. How I built it

1. Gave the agent full project context: user roles, the app and admin panel, main flows, rotation statuses and how they change.
2. Defined the ticket structure, title format and default environment, so every report looks the same.
3. Set a list of required details (role, interface, rotation status, environment). If any of them is missing from my note, the agent asks short questions instead of guessing.
4. Connected the agent to Jira through the Atlassian MCP, so it can create tickets and add comments.
5. Added a review step: the agent shows the draft first and creates the ticket only after I approve it.

### 4. How it looks in chat

<img width="392" height="656" alt="image" src="https://github.com/user-attachments/assets/08291815-c42b-40a1-b9b1-478e6c1fe5a0" />
