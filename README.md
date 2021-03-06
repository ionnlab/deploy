# Deconst Ansible Playbook

This is an Ansible playbook that deploys Deconst onto a cluster.

Clone this repository once for each deconst instance you wish to administer. The contents of `credentials.yml` customize and identify each deployment.

## Prerequisites

You'll need a recent Python 2.7, at least Ansible 1.9.0.1, and pyrax. You can use a virtualenv if you wish.

```bash
# Check your Python version
python -V

# Install virtualenv and virtualenvwrapper, if desired.
sudo pip install virtualenv virtualenvwrapper
source /usr/local/bin/virtualenvwrapper.sh

mkvirtualenv deconst-ansible

# Install Ansible and pyrax.
pip install -r requirements.txt
```

## Running

To deploy or update a cluster:

 1. Copy the example credentials file and fill in your credentials and customizations. Alternately, use a credentials file corresponding to an existing deployment you'd like to maintain.

    ```bash
    cp credentials.example.yml credentials.yml
    ${EDITOR} credentials.yml

    # Or:
    script/decrypt ~/cred-repo/credentials-staging.yml.enc
    ```

 2. Copy the SSH Private Key used for the deconst instante into `/keys`.
    - `cp instance-private-key keys/{instance-name}.private.key`
    - `chmod 600 keys/{instance-name}.private.key`

    The instance name is found in credentials.yml and is used to locate the SSH key used for communication automatically. We recommend storing the SSH private key in an encrypted document store.

 3. Run the playbook with the `deploy` script.

    ```bash
    script/deploy
    ```

### Parameters

Deconst guards against inconsistent `credentials.yml` files being run by multiple maintainers. If you intentionally make changes to the credentials file, you'll need to provide extra variables to `script/deploy`.

If you change the `deployment`, run with:

```bash
script/deploy -e 'new_deployment=true'
```

If you make any other local changes to a `credentials.yml` file, run with:

```bash
script/deploy -e 'credentials_update=true'
```

To only update the control repository's content map, layout map or templates:

```bash
script/deploy --tags control
```

To force a restart of selected services:

```bash
# Restart only presenters
script/deploy --tags restart -e 'presenter_restart=true'

# Other restart control variables:
# -e 'service_pod_restart=true'         Service pods (content services and presenter)
# -e 'logstash_forwarder_restart=true'  Logstash-forwarder
# -e 'logstash_restart=true'            Logstash
```

To force the generation of new TLS certificates:

```bash
script/deploy --extra-vars="gencerts=yes"
```

## Utilities

This repository contains a number of utilities to assist in basic ops work. Each script keys off of the credentials in `credentials.yml`, so it will use the correct Rackspace account and hosts.

 * `script/status` performs a `docker status` on each host. It's useful for quickly seeing if all expected services are up and running.
 * `script/logs <component>` tails the Docker container logs of each matching service across the cluster. The number of lines given can be controlled by setting `LOG_LINES`. For example: `LOG_LINES=50 script/logs presenter`.
 * `script/genkey <name>` reads the admin API key from your credentials file and issues a new API key with the provided name.
 * `script/ssh <hostpattern>` logs in to a uniquely identified host in the cluster.
 * `script/ips` lists the IP addresses of each host in the cluster.
 * `script/lb` audits and corrects load-balancer node membership on the cluster. Consult `--help` for details.
 * `script/reindex` asynchronously triggers a full content reindex in Elasticsearch.
