# Ansible Role: Bitbucket Environment Variables Management

This Ansible role enables idempotent management of environment variables associated with Bitbucket deployments. It fetches existing variables, deletes them if necessary, and creates or updates new variables for each specified environment.

---

## Features

- Fetches existing variables for each environment in Bitbucket.
- Deletes existing variables based on their keys.
- Creates or updates environment variables for each specified environment.
- Complete management of environments (e.g., dev, preprod, production).

---

## Requirements

- A Bitbucket account with permissions to manage deployment variables.
- An API key or application password configured for authentication.
- Ansible version 2.9 or higher.

---

## Role Variables

### General Variables

| Variable              | Description                                            | Required | Example                          |
|-----------------------|--------------------------------------------------------|----------|----------------------------------|
| `bitbucket_workspace` | The name of the Bitbucket workspace.                   | Yes      | `your-workspace`                    |
| `bitbucket_repo`      | The name of the Bitbucket repository.                  | Yes      | `your-repo-name`                |
| `bitbucket_auth`      | Authentication header in the format `Basic base64`.    | Yes      | `Basic dXNlcm5hbWU6YXBpdG9rZW4=`|

### Environment Variables Definition

| Variable               | Description                                           | Required | Example                          |
|------------------------|-------------------------------------------------------|----------|----------------------------------|
| `environment_variables`| List of environments with associated variables.       | Yes      | See the example below.           |

#### Example Environment Definition

```yaml
environment_variables:
  - environment: "dev"
    uuid: "544ea064-504f-4a9b-86f2-17afa25b8e7c"
    variables:
      - key: "SSH_HOST"
        value: "dev.server.com"
        secured: false
      - key: "SSH_PORT"
        value: "22"
  - environment: "preprod"
    uuid: "3dd0ec9f-198b-4a85-8824-cb19e4398c24"
    variables:
      - key: "SSH_HOST"
        value: "preprod.server.com"
        secured: false
      - key: "SSH_PORT"
        value: "22"
  - environment: "production"
    uuid: "9404cfa3-25cc-48ba-8e7a-cd7f1c35e562"
    variables:
      - key: "SSH_HOST"
        value: "prod.server.com"
        secured: false
      - key: "SSH_PORT"
        value: "22"
```

---

## Usage

Include this role in an Ansible playbook to manage Bitbucket environment variables.

#### Example Playbook

```yaml
---
- hosts: localhost
  roles:
    - role: bitbucket_env_vars
      vars:
        bitbucket_workspace: "{{ lookup('env', 'BITBUCKET_WORKSPACE') }}"
        bitbucket_repo: "{{ lookup('env', 'BITBUCKET_REPO') }}"
        bitbucket_auth: "Basic {{ lookup('env', 'BITBUCKET_USER_PASSWORD') | b64encode }}"
        environment_variables:
          - environment: "dev"
            uuid: "544ea064-504f-4a9b-86f2-17afa25b8e7c"
            variables:
              - key: "SSH_HOST"
                value: "dev.server.com"
                secured: false
              - key: "SSH_PORT"
                value: "22"
```

---

## Expected Results

After running the role:

- Existing environment variables will be updated or deleted if no longer needed.
- New variables will be added.
- The final state of variables will match the configuration defined in `environment_variables`.

---

## How to Retrieve Environment UUIDs on Bitbucket Using the Browser Inspector

### Step 1: Access the Deployment Environment Page

1. Log in to Bitbucket.
2. Navigate to your target repository (e.g., `your-workspace/your-repo-name`).
3. Go to the following URL:
   ```
   https://bitbucket.org/{workspace}/{repo_slug}/admin/pipelines/deployment-settings
   ```
   Example:
   ```
   https://bitbucket.org/your-workspace/your-repo-name/admin/pipelines/deployment-settings
   ```

---

### Step 2: Open the Browser Inspector

1. Open the browser inspector:
   - Chrome/Edge/Firefox: Press `Ctrl+Shift+I` or right-click on the page → **Inspect**.
2. Click on the **Network** tab.

---

### Step 3: Filter Requests to Find UUIDs

1. In the **Network** tab:
   - Reload the Bitbucket page using `F5`.
   - In the search bar of the network inspector, type:
     ```
     deployments_config/environments/
     ```
2. Expand environments one by one on the page. Each expansion triggers a new request visible in the inspector.

---

### Step 4: Identify UUIDs in Requests

1. For each environment, you will see a request like this:
   ```
   /!api/internal/repositories/your-workspace/your-repo-name/deployments_config/environments/{uuid}/variables/
   ```
2. The UUID for the environment is located between `{}`.  
   Example:
   ```
   {544ea064-504f-4a9b-86f2-17afa25b8e7c}
   ```

---

### Step 5: Store Environment UUIDs

To avoid repeating these steps, document the UUIDs with their corresponding environment names.  
Example:

```yaml
environment_variables:
  - environment: "dev"
    uuid: "{544ea064-504f-4a9b-86f2-17afa25b8e7c}"
    variables:
      - key: "SSH_HOST"
        value: "dev.server.com"
        secured: false
      - key: "SSH_PORT"
        value: "22"
  - environment: "preprod"
    uuid: "{3dd0ec9f-198b-4a85-8824-cb19e4398c24}"
    variables:
      - key: "SSH_HOST"
        value: "preprod.server.com"
        secured: false
      - key: "SSH_PORT"
        value: "22"
  - environment: "production"
    uuid: "{9404cfa3-25cc-48ba-8e7a-cd7f1c35e562}"
    variables:
      - key: "SSH_HOST"
        value: "prod.server.com"
        secured: false
      - key: "SSH_PORT"
        value: "22"
```

---

## License

MIT

---

## Author

- **Karim Baidi** - Linux-man.fr