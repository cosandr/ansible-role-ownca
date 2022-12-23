# ansible-role-ownca

![GitHub](https://img.shields.io/github/license/cosandr/ansible-role-ownca) ![GitHub last commit](https://img.shields.io/github/last-commit/cosandr/ansible-role-ownca) ![GitHub issues](https://img.shields.io/github/issues-raw/cosandr/ansible-role-ownca)

Ansible role for managing certificates and certificate authorities. The private keys can be encrypted using Ansible Vault.

## Requirements

Ansible 2.12 or higher, `cryptography` Python library on localhost.

## Variables

| Name           | Default Value | Description                        |
| -------------- | ------------- | -----------------------------------|
| `ownca_vault_password_file` | "" | Path to Ansible Vault password file/script, if not provided, private keys are left unencrypted |
| `ownca_ca_list` | [] | List of certificate authorities to configure, see example playbook |
| `ownca_cert_list` | [] | List of (self-signed) certificates, see example playbook |
| `ownca_delegate_to` | localhost | Only localhost is supported at the moment due to the use of `lookup('file')` |

## Dependencies

None.

## Example Playbook

```yaml
---

- hosts: all
  # Tasks run on localhost, you probably don't need root for them
  become: false
  # Facts aren't required
  gather_facts: false
  vars:
    ownca_vault_password_file: "/tmp/get-vault-pass.sh"
    # Should only run this once, but this role is indempotent
    ownca_ca_list:
      - cn: "Example CA"
        passphrase: "dontusemelikethis"
        publickey_path: "/tmp/example.ca.crt"
        privatekey_path: "/tmp/example.ca.key"

    ownca_cert_list:
      - privatekey_path: "/tmp/{{ inventory_hostname }}.key"
        publickey_path: "/tmp/{{ inventory_hostname }}.crt"
        passphrase: "changemetoo"
        subject_alt_name:
          - "DNS:example.com"
          - "IP:10.1.2.3"
        ca_passphrase: "{{ ownca_ca_list[0].passphrase }}"
        ca_publickey_path: "{{ ownca_ca_list[0].publickey_path }}"
        ca_privatekey_path: "{{ ownca_ca_list[0].privatekey_path }}"
    roles:
    - role: ansible-role-ownca
```

## Author

[Andrei Costescu](https://github.com/cosandr/)
