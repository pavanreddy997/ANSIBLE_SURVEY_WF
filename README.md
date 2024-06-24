
# Automated Workflow Management with Ansible and AWX

## Overview

This project automates the process of collecting and utilizing survey data within two interconnected workflows using Ansible and AWX (Ansible Tower). The first workflow gathers essential information via survey questions, which is then passed to the second workflow. The second workflow includes an additional survey question that requires user input before completion. An automated email is sent to notify the responsible party to provide the necessary input.

## Features

- **Survey Data Collection:** Gathers essential information via survey questions.
- **Data Passing:** Seamlessly passes data from the first workflow to the second.
- **Interim Survey Question:** Includes an additional survey question in the second workflow that requires user input.
- **Email Notification:** Notifies stakeholders to provide required input.
- **Workflow Execution:** Completes the process using combined data from both workflows once input is provided.

## Prerequisites

- Ansible
- AWX or Ansible Tower
- Access to the required systems and permissions to run playbooks and workflows

## Usage

1. Clone the Repository:

   ```sh
   git clone https://github.com/your-username/ansible-awx-automation.git
   cd ansible-awx-automation
   ```

2. Configure Variables:

   Update the variables in the playbook to match your environment. Specifically, modify the following:

   ```yaml
   vars:
     email_to: "dummy@example.com"
     email_subject: "Action Required: Provide Datastore Input"
     tower_host: "example.com"
     tower_username: "dummy-username"
     tower_password: "dummy-password"
     next_workflow_id: 12345
   ```

3. Run the Playbook:

   Execute the playbook to start the workflow:

   ```sh
   ansible-playbook playbook.yml
   ```

## Playbook Details

### playbook.yml

```yaml
---
- hosts: all
  become: false
  gather_facts: false

  vars:
    email_to: "dummy@example.com"
    email_subject: "Action Required: Provide Datastore Input"
    tower_host: "example.com"
    tower_username: "dummy-username"
    tower_password: "dummy-password"
    next_workflow_id: 12345

  tasks:

    - name: Collect initial survey data
      set_fact:
        initial_data:
          input_VM: "{{ input_VM_name}}"
          input_ip: "{{ input_ip }}"

    - name: Convert initial data to JSON
      set_fact:
        extra_vars_json: "{{ initial_data | to_json }}"

    - name: Print Initial Survey Data
      debug:
        msg: "{{ initial_data }}"

    - name: Convert initial data to JSON string
      set_fact:
        extra_vars_json: "{{ initial_data | to_nice_json }}"

    - name: Get Tower API token
      uri:
        url: "{{ tower_host }}/api/v2/tokens/"
        method: POST
        user: "{{ tower_username }}"
        password: "{{ tower_password }}"
        force_basic_auth: yes
        status_code: 201
        headers:
          Content-Type: "application/json"
        body: "{}"
        validate_certs: no
      register: auth_token_response
      delegate_to: localhost

    - name: Extract Tower API token
      set_fact:
        tower_token: "{{ auth_token_response.json.token }}"

    - name: Update the workflow job template with extra vars
      uri:
        url: "{{ tower_host }}/api/v2/workflow_job_templates/{{ next_workflow_id }}/"
        method: PATCH
        headers:
          Authorization: "Bearer {{ tower_token }}"
          Content-Type: "application/json"
        validate_certs: no
        status_code: 200
        body: "{{ {'extra_vars': extra_vars_json} | to_nice_json }}"
      delegate_to: localhost

    - name: Email Report with Initial Survey Data and Instructions
      mail:
        host: '127.0.0.1'
        from: 'ansible@example.com'
        to: '{{ email_to }}'
        subject: '{{ email_subject }}'
        body: |
          The initial data has been gathered and the job template has been updated. Please provide the required datastore input at the following link when you are ready to launch the job:

          Job Link: {{ tower_host }}/#/templates/workflow_job_template/{{ next_workflow_id }}

          IP: {{ initial_data.input_ip }}

          The job template variables have been set. Please provide the datastore input and then launch the job.
      delegate_to: localhost
      run_once: true
```

## License

This project is licensed under the MIT License. See the LICENSE file for details.

## Contributing

Contributions are welcome! Please open an issue or submit a pull request.

## Contact

For any questions or issues, please contact me.
