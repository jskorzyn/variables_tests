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
- **Target hosts**: Uses `hosts: all` to test against all hosts in the inventory
- **Files**:
  - `scenario_1/first_playbook.yml` - Sets `example_variable` fact and prints it
  - `scenario_1/second_playbook.yml` - Attempts to print the `example_variable` from first playbook

### Scenario 2: Using `import_playbook`
- **Main playbook**: `scenario_2.yml`
- **Method**: Uses `import_playbook` to include complete playbooks
- **Structure**: Multiple plays, each from a separate playbook file
- **Target hosts**: Uses `hosts: all` to test against all hosts in the inventory
- **Files**:
  - `scenario_2/first_playbook.yml` - Sets `example_variable` fact and prints it
  - `scenario_2/second_playbook.yml` - Attempts to print the `example_variable` from first playbook

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

# Or with interactive mode
ansible-navigator run scenario_1.yml -i "localhost,"
ansible-navigator run scenario_2.yml -i "localhost,"
```

### With Ansible Automation Platform (AAP)

1. **Create a new Project** in AAP pointing to this repository
2. **Create Job Templates** for each scenario:
   - **Job Template 1**:
     - Name: "Variables Test - Scenario 1"
     - Playbook: `scenario_1.yml`
     - Inventory: Select an inventory with target hosts (playbook targets all hosts)
   - **Job Template 2**:
     - Name: "Variables Test - Scenario 2"
     - Playbook: `scenario_2.yml`
     - Inventory: Select an inventory with target hosts (playbook targets all hosts)
3. **Launch the Job Templates** and review the output logs
4. **Compare the results** to see if variables persist across included playbooks

## Expected Behavior

### Scenario 1 (include_tasks)
Variables should persist because all tasks run within the same play context.

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

### Scenario 2 (import_playbook)
Variables may NOT persist as each imported playbook runs as a separate play with its own scope.

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
fatal: [host]: FAILED! => {
    "msg": "The task includes an option with an undefined variable. The error was: 'example_variable' is undefined"
}
```

**Expected behavior:**
- First playbook sets and prints the variable successfully
- Second playbook typically fails with an undefined variable error because facts don't persist across separate plays in imported playbooks
- Behavior may vary depending on Ansible version and execution environment (especially in AAP)

## Notes

- Both scenarios target `all` hosts in the inventory
- You can use a simple inline inventory like `-i "localhost,"` for quick testing
- All playbooks use `gather_facts: false` for faster execution
- The test variable is named `example_variable`
- You can modify the `hosts:` directive in the playbooks to test against different host patterns
