# Ansible Variables Tests

This repository contains test scenarios to verify variable/fact persistence behavior across different playbook inclusion methods in Ansible and Ansible Automation Platform (AAP).

## Purpose

The main goal is to test whether variables set using `set_fact` in one playbook remain accessible in subsequently included playbooks. This behavior may differ depending on:
- The inclusion method used (`include_tasks` vs `import_playbook`)
- The execution environment (ansible-playbook, ansible-navigator, AAP)

## Test Scenarios

### Scenario 1: Using `include_tasks`
- **Main playbook**: `scenario_1.yml`
- **Method**: Uses `include_tasks` to include task files
- **Structure**: Single play with multiple included task lists
- **Files**:
  - `scenario_1/first_playbook.yml` - Sets `example_variable` fact and prints it
  - `scenario_1/second_playbook.yml` - Attempts to print the `example_variable` from first playbook

### Scenario 2: Using `import_playbook`
- **Main playbook**: `scenario_2.yml`
- **Method**: Uses `import_playbook` to include complete playbooks
- **Structure**: Multiple plays, each from a separate playbook file
- **Files**:
  - `scenario_2/first_playbook.yml` - Sets `example_variable` fact and prints it
  - `scenario_2/second_playbook.yml` - Attempts to print the `example_variable` from first playbook

## Running Tests

### With ansible-playbook (Traditional CLI)

```bash
# Test Scenario 1
ansible-playbook scenario_1.yml

# Test Scenario 2
ansible-playbook scenario_2.yml
```

### With ansible-navigator (Execution Environments)

```bash
# Test Scenario 1
ansible-navigator run scenario_1.yml -m stdout

# Test Scenario 2
ansible-navigator run scenario_2.yml -m stdout

# Or with interactive mode
ansible-navigator run scenario_1.yml
```

### With Ansible Automation Platform (AAP)

1. **Create a new Project** in AAP pointing to this repository
2. **Create Job Templates** for each scenario:
   - **Job Template 1**:
     - Name: "Variables Test - Scenario 1"
     - Playbook: `scenario_1.yml`
     - Inventory: Select an inventory with localhost or any target host
   - **Job Template 2**:
     - Name: "Variables Test - Scenario 2"
     - Playbook: `scenario_2.yml`
     - Inventory: Select an inventory with localhost or any target host
3. **Launch the Job Templates** and review the output logs
4. **Compare the results** to see if variables persist across included playbooks

## Expected Behavior

### Scenario 1 (include_tasks)
Variables should persist because all tasks run within the same play context.

**Expected output:**
```
TASK [Print the fact from first playbook]
ok: [localhost] => {
    "msg": "First playbook - example_variable: This is a test variable set in first playbook"
}

TASK [Print the fact set in first playbook]
ok: [localhost] => {
    "msg": "Second playbook - example_variable: This is a test variable set in first playbook"
}
```

### Scenario 2 (import_playbook)
Variables may NOT persist as each imported playbook runs as a separate play with its own scope.

**Expected behavior:**
- First playbook sets and prints the variable successfully
- Second playbook may fail with an undefined variable error, depending on Ansible version and execution environment

## Notes

- All playbooks target `localhost` with `gather_facts: false` for quick testing
- The test variable is named `example_variable`
- Modify the `hosts:` directive in the playbooks to test against different inventories
