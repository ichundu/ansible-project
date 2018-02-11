Exercise 3: Ansible
===================

Create a simple ansible project consisting of a playbook and role(s) to configure a remote ubuntu server:

- Use ansible to install postgresql on target server
- Use ansible to ensure postgresql runs on server startup
- Use ansible to enable inbound postgresql network connections from the network range 10.241.0.0/16 (md5)
- Use ansible to ensure that SSH logins only work with ssh keypairs (no passwords)

Bonus tasks (optional):
- Use ansible to configure the firewall on the remote server to allow inbound connections on postgres port
- Use ansible to configure a schedulet backup (dump) of the postgresql server.

Prerequisits
============

The following steps are necessary on a fresh Ubuntu 16.04 minimal installation before running this playbook:

1. Install python2, by default ubuntu ships with python3, but python2 is required by ansible:

    ```bash
    sudo apt update && sudo apt install -y python
    ```

2. Because this playbook enables the UFW firewall on Ubuntu, ensure SSH access is granted before the UFW service is started by ansible to prevent being locked out of the system:

    ```bash
    sudo ufw allow OpenSSH
    ```

3. Because the `ssh` role included here disables SSH password authentication and allows authentication only via private/public keypair, ensure your public key is copied to the remote server in the file `/home/remote_user/.ssh/authorized_keys`. Also ensure the permissions on the `.ssh` directory are correct because OpenSSH is very sensitive about permissions:

    ```bash
    chmod 700 /home/remote_user/.ssh
    chmod 600 /home/remote_user/.ssh/authorized_keys
    ```

Solution
========

1. There are two roles included in this project:

    - `postgresql`, which:
        - Installs postgresql database
        - Starts postgresql service and enables it to run on boot
        - Allows inbound connections from network range 10.231.0.0/16 to the postgres listener (tcp 5432)
        - Configures the UFW firewall in Ubuntu to allow network connections from network 10.231.0.0/16 to the postgres port. When enabling the UFW firewall service in Ubuntu ensure it already allows SSH connections, it is not handeled by this playbook since we are asuming SSH is enabled by default.
        - Sets up a cron job for the postgres user to make daily backups (database dumps) of the postgresql database. The name of the database to backup is declared by the variable `postgresql_database_name`, which defaults to `postgresql`.

    - `ssh`, which ensures the following entries are set as follows in the SSH configuration file `/etc/ssh/sshd_config`. Ensure your public key is copied to the authorized_keys file at the remote host prior to running this playbook as this will disable password authentication.
        - "PasswordAuthentication no"
        - "PubkeyAuthentication yes"

2. How to run this playbook:

    By default we have configured this playbook to run against localhost, the same machine where ansible is installed. To run against a remote host edit the `inventory` file and add the host IP:

    ```ini
    remote_host     ansible_host=192.168.1.2
    ```

    Execute the playbook by running:

    ```bash
    ansible-laybook -i inventory -u *remote_user* site.yml
    ```

    where:

    - `"-i inventory"` specifies the hosts/inventory file.
    - `"-u remote_user"` is the username to be used to connect to the remote host.
    - `"site.yml"` is the ansible playbook to be executed.

