# Ansible Variables Tests

This repository contains test scenarios to verify variable/fact persistence behavior across different playbook inclusion methods in Ansible and Ansible Automation Platform (AAP).

## Purpose

The main goal is to test variable persistence behavior across different playbook inclusion methods. This repository demonstrates how different types of variables behave:
- **Facts** set with `set_fact` (host-scoped)
- **Play variables** defined with `vars:` (play-scoped)
- Different inclusion methods (`include_tasks` vs `import_playbook`)

Understanding these differences is crucial for building complex Ansible automation, especially in Ansible Automation Platform (AAP).

## Test Scenarios

### Scenario 1: Using `include_tasks`
- **Main playbook**: `scenario_1.yml`
- **Method**: Uses `include_tasks` to include task files
- **Structure**: Single play with multiple included task lists
- **Target hosts**: Uses `hosts: all` to test against all hosts in the inventory
- **Files**:
  - `scenario_1/first_playbook.yml` - Sets `example_variable` fact and prints it
  - `scenario_1/second_playbook.yml` - Attempts to print the `example_variable` from first playbook

### Scenario 2: Using `import_playbook` with `set_fact`
- **Main playbook**: `scenario_2.yml`
- **Method**: Uses `import_playbook` to include complete playbooks
- **Variable type**: Uses `set_fact` to create host-bound facts
- **Structure**: Multiple plays, each from a separate playbook file
- **Target hosts**: Uses `hosts: all` to test against all hosts in the inventory
- **Files**:
  - `scenario_2/first_playbook.yml` - Sets `example_variable` fact with `set_fact` and prints it
  - `scenario_2/second_playbook.yml` - Attempts to print the `example_variable` from first playbook

### Scenario 3: Using `import_playbook` with play variables
- **Main playbook**: `scenario_3.yml`
- **Method**: Uses `import_playbook` to include complete playbooks
- **Variable type**: Uses play variables (`vars:`) instead of `set_fact`
- **Structure**: Multiple plays, each from a separate playbook file
- **Target hosts**: Uses `hosts: all` to test against all hosts in the inventory
- **Files**:
  - `scenario_3/first_playbook.yml` - Sets `example_variable` as a play variable and prints it
  - `scenario_3/second_playbook.yml` - Attempts to print the `example_variable` from first playbook

## Running Tests

### With ansible-playbook (Traditional CLI)

```bash
# Test Scenario 1 (requires inventory)
ansible-playbook -i inventory scenario_1.yml
# Or use a simple inline inventory
ansible-playbook -i "localhost," scenario_1.yml

# Test Scenario 2 (requires inventory)
ansible-playbook -i inventory scenario_2.yml
# Or use a simple inline inventory
ansible-playbook -i "localhost," scenario_2.yml

# Test Scenario 3 (requires inventory)
ansible-playbook -i inventory scenario_3.yml
# Or use a simple inline inventory
ansible-playbook -i "localhost," scenario_3.yml
```

### With ansible-navigator (Execution Environments)

```bash
# Test Scenario 1 (requires inventory)
ansible-navigator run scenario_1.yml -i inventory -m stdout
# Or use a simple inline inventory
ansible-navigator run scenario_1.yml -i "localhost," -m stdout

# Test Scenario 2 (requires inventory)
ansible-navigator run scenario_2.yml -i inventory -m stdout
# Or use a simple inline inventory
ansible-navigator run scenario_2.yml -i "localhost," -m stdout

# Test Scenario 3 (requires inventory)
ansible-navigator run scenario_3.yml -i inventory -m stdout
# Or use a simple inline inventory
ansible-navigator run scenario_3.yml -i "localhost," -m stdout

# Or with interactive mode
ansible-navigator run scenario_1.yml -i "localhost,"
ansible-navigator run scenario_2.yml -i "localhost,"
ansible-navigator run scenario_3.yml -i "localhost,"
```

### With Ansible Automation Platform (AAP)

1. **Create a new Project** in AAP pointing to this repository
2. **Create Job Templates** for each scenario:
   - **Job Template 1**:
     - Name: "Variables Test - Scenario 1 (include_tasks)"
     - Playbook: `scenario_1.yml`
     - Inventory: Select an inventory with target hosts (playbook targets all hosts)
   - **Job Template 2**:
     - Name: "Variables Test - Scenario 2 (import_playbook + set_fact)"
     - Playbook: `scenario_2.yml`
     - Inventory: Select an inventory with target hosts (playbook targets all hosts)
   - **Job Template 3**:
     - Name: "Variables Test - Scenario 3 (import_playbook + play vars)"
     - Playbook: `scenario_3.yml`
     - Inventory: Select an inventory with target hosts (playbook targets all hosts)
3. **Launch the Job Templates** and review the output logs
4. **Compare the results** to understand how different variable types persist across playbooks

## Expected Behavior

### Scenario 1 (include_tasks with set_fact)
**Result: ✅ SUCCESS**

Variables persist because all tasks run within the same play context.

**Expected output (when run against a single host):**
```
TASK [Print the fact from first playbook]
ok: [host] => {
    "msg": "First playbook - example_variable: This is a test variable set in first playbook"
}

TASK [Print the fact set in first playbook]
ok: [host] => {
    "msg": "Second playbook - example_variable: This is a test variable set in first playbook"
}
```

**Why it works:**
- All included tasks execute within the same play
- The `set_fact` creates a host-bound fact available throughout the play

### Scenario 2 (import_playbook with set_fact)
**Result: ✅ SUCCESS**

Facts created with `set_fact` persist across imported playbooks because they are stored on the host itself.

**Expected output (when run against a single host):**
```
PLAY [First playbook - Set and print fact] *************************************

TASK [Set a fact in first playbook]
ok: [host]

TASK [Print the fact from first playbook]
ok: [host] => {
    "msg": "First playbook - example_variable: This is a test variable set in first playbook"
}

PLAY [Second playbook - Print fact from first playbook] ************************

TASK [Print the fact set in first playbook]
ok: [host] => {
    "msg": "Second playbook - example_variable: This is a test variable set in first playbook"
}
```

**Why it works:**
- `set_fact` creates **host-bound facts** that are stored on each host in Ansible's fact cache
- These facts persist across all plays in the same playbook execution
- Even though each imported playbook runs as a separate play, they target the same hosts which already have the fact stored

### Scenario 3 (import_playbook with play variables)
**Result: ❌ FAILURE**

Play variables do NOT persist across imported playbooks because they are play-scoped, not host-scoped.

**Expected output (when run against a single host):**
```
PLAY [First playbook - Set play variable and print it] *************************

TASK [Print the play variable from first playbook]
ok: [host] => {
    "msg": "First playbook - example_variable: This is a test variable set in first playbook as play var"
}

PLAY [Second playbook - Try to print play variable from first playbook] ********

TASK [Try to print the play variable set in first playbook]
fatal: [host]: FAILED! => {
    "msg": "The task includes an option with an undefined variable. The error was: 'example_variable' is undefined"
}
```

**Why it fails:**
- Play variables (defined with `vars:`) are **play-scoped**, not host-scoped
- Each imported playbook creates a new play with its own variable scope
- Variables from one play are not accessible in another play
- This is the expected behavior and demonstrates the fundamental difference between play variables and host facts

## Key Concepts Explained

### Host Facts vs Play Variables

Understanding the difference between host facts and play variables is crucial:

**Host Facts (created with `set_fact`):**
- Stored **on each host** in Ansible's fact cache for the entire playbook execution
- Persist across **all plays** in the same playbook run
- Available as long as you target the same host
- Work across `import_playbook` boundaries
- Example: `set_fact: my_var="value"`

**Play Variables (defined with `vars:`):**
- Scoped to a **single play** only
- Do NOT persist across play boundaries
- Not available in other plays, even in the same playbook execution
- Do NOT work across `import_playbook` boundaries
- Example: `vars: my_var="value"`

### include_tasks vs import_playbook

**`include_tasks`:**
- Includes task files into the current play
- All tasks share the same play context
- Both facts and play variables are accessible across included task files
- Tasks execute in a single play

**`import_playbook`:**
- Each imported playbook runs as a completely separate play
- Only host facts (from `set_fact`) persist across plays
- Play variables do NOT persist
- Each play has its own variable scope

### Summary Matrix

| Scenario | Method | Variable Type | Persists? | Reason |
|----------|--------|---------------|-----------|---------|
| 1 | `include_tasks` | `set_fact` | ✅ Yes | Same play context |
| 2 | `import_playbook` | `set_fact` | ✅ Yes | Host-bound facts persist |
| 3 | `import_playbook` | `vars:` | ❌ No | Play variables are play-scoped |

## Notes

- All scenarios target `all` hosts in the inventory
- You can use a simple inline inventory like `-i "localhost,"` for quick testing
- All playbooks use `gather_facts: false` for faster execution
- The test variable is named `example_variable`
- You can modify the `hosts:` directive in the playbooks to test against different host patterns
- These behaviors are consistent across ansible-playbook, ansible-navigator, and AAP
