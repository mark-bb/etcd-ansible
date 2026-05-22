# ETCD [un]installing

With ssl support


## Notes
- The user specified in the `ansible_user` variable in the file below must have `sudo` with the key in the `ansible_ssh_private_key_file` variable on the hosts in the `inv` file
```
$ cat group_vars/all/ansible.yml
ansible_connection: ssh
ansible_user: myuser
ansible_become: yes
ansible_ssh_private_key_file: ~/.ssh/id_rsa
```
- The only playbook at the moment is `etcd.yml`
- Set the `etcd_with_ssl` variable to `no` if you don't use SSL
```
grep '^etcd_with_ssl:' roles/etcd/defaults/main.yml
```
- Distribution (with its original name) & certs must be placed or linked to the `roles/etcd/files/` directory.
Its current content is:
```
$ tree roles/etcd/files
roles/etcd/files
├── certs
│   ├── fqdn-01.internal.crt -> /etc/security/certs/fqdn-01.internal.crt
│   ├── fqdn-01.internal.key -> /etc/security/certs/fqdn-01.internal.key
│   ├── fqdn-02.internal.crt -> /etc/security/certs/fqdn-02.internal.crt
│   ├── fqdn-02.internal.key -> /etc/security/certs/fqdn-02.internal.key
│   ├── fqdn-03.internal.crt -> /etc/security/certs/fqdn-03.internal.crt
│   ├── fqdn-03.internal.key -> /etc/security/certs/fqdn-03.internal.key
│   └── truststore.pem -> /etc/security/certs/certs/truststore.pem
└── distrib
    └── etcd-v3.6.5-linux-amd64.tar.gz -> /distrib/etcd/etcd-v3.6.5-linux-amd64.tar.gz

```
- Certificates generation
  - Node cert & key files in the PEM format must have the `<inventory_hostname>.{crt,key}` name
  - `truststore.pem` may contain a single root CA cert or root CA & intermediate CA certs

## Commands examples
- Installing / reconfiguring
```
ansible-playbook etcd.yml
```
- Stopping
```
ansible-playbook etcd.yml -e 'play_action=stop'
```
- Starting
```
ansible-playbook etcd.yml -e 'play_action=start'
```
- Update non-SSL -> SSL
```
ansible-playbook etcd.yml -e 'play_action=update2ssl'
```
- Update SSL -> non-SSL
```
ansible-playbook etcd.yml -e 'play_action=update2non-ssl'
```
- Uninstalling
```
ansible-playbook etcd.yml -e 'play_action=uninstall'
```

## Useful commands

- Set some env variables
```
ENDPOINTS="$(echo $(tail -n +2 inv | sed 's/$/:2379/') | tr ' ' ',')"
# export ETCDCTL_API=3  # seems that it's not needed anymore with newer versions

# Correct the cert paths or if you don't use certs
ETCDCTL=$(cat <<EOF
/opt/etcd/etcdctl --endpoints=${ENDPOINTS?} --write-out=table \
--cacert ${PWD}/roles/etcd/files/certs/truststore.pem \
--cert   /etc/security/certs/$(hostname -f).crt \
--key    /etc/security/certs/$(hostname -f).key
EOF
)
```

- Checking...
```
${ETCDCTL?} member list
${ETCDCTL?} endpoint status --cluster
${ETCDCTL?} endpoint health
```
