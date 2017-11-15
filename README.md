# NSD

Install and configure NSD (dns) server.re.

# Requirements

- **ansible** version minimal is **2.3**
- role **hudecof.filter-plugins** adds some jinja filters
- for **RedHat** based distribution  **EPEL** repository is needed
 
# Role Variables

All default variables are defined in the **defaults/main.yml**.

**OS** specific variables are in **vars/os-<distribution>.yml** with `__` prefix. See **defaults** which could be overwritten.

## General

- `nsd_service_state` define service state, defaults to **started**
- `nsd_service_enabled` define service enable state, defaults to **yes**
- `nsd_backup` define wheather create config backup files, defaults to **False**
- `nsd_test` dfine, whearther the role is in testing mode, defaults to **False**

## OS spececific

- `nsd_packages` list of packages to install/remove, defaults to **__nsd_packages**
- `nsd_user` usename to run service under, defaults to **__nsd_user**
- `nsd_group` group name to run service under, defaults to **__nsd_group**
- `nsd_service` service name, defaults to **__nsd_service**
- `nsd_dir_etc` configuration directory, defaults to **__nsd_dir_etc**
- `nsd_dir_db` database directory, defaults to **__nsd_dir_db**
- `nsd_dir_run` pid file directory, defaults to **__nsd_dir_run**
- `nsd_dir_state` state directory, defaults to **__nsd_dir_state**
- `nsd_bin_control_setup` path to binary **nsd-control-setup**
- `nsd_bin_check_conf` path to binary **nsd-checkconf**


## Server Configuration

I will provide sample configuration. The dictionaty keys are configuration parameters for the NSD, so please see **nsd.conf(5)** maunal page.

```
nsd_conf_keys:
  test:
    algorithm: hmac-sha256
    secret: aaaaaabbbbbbccccccdddddd

nsd_conf_zones:
  example.com:
    include-pattern: test

nsd_conf_patterns:
  test:
    'zonefile': 'zones.d/%s'
    'provide-xfr': ['0.0.0.0/0 test', '::0/0 test']

nsd_conf_server:
  'username': "{{ nsd_user }}"
  'do-ip4': 'yes'
  'do-ip6': 'no'
  'hide-version': 'yes'
  'zonesdir': "{{ nsd_dir_etc }}"
  'database': "{{ nsd_dir_db }}/nsd.db"
  'pidfile': "{{ nsd_dir_run }}/nsd.pid"
  'xfrdfile': "{{ nsd_dir_state }}/xfrd.state"
  'verbosity': 0
  'ip-address': ['127.0.0.1']
```
To override onl part of the config you could use small trick with **combine** filter plugin.

```
nsd_conf_server_base:
  'username': "{{ nsd_user }}"
  'do-ip4': 'yes'
  'do-ip6': 'no'
  'hide-version': 'yes'
  'zonesdir': "{{ nsd_dir_etc }}"
  'database': "{{ nsd_dir_db }}/nsd.db"
  'pidfile': "{{ nsd_dir_run }}/nsd.pid"
  'xfrdfile': "{{ nsd_dir_state }}/xfrd.state"
  'verbosity': 0
  'ip-address': "{{ ansible_all_ipv4_addresses }} + {{ ['127.0.0.1'] }}"

nsd_conf_server_host: {}

nsd_conf_server: "{{ nsd_conf_server_base|combine(nsd_conf_server_host) }}"
```
# Dependencies

```
  roles:
    - role: hudecof.packages
      packages_centos_epel: True
      packages_ubuntu_multiverse: True
      packages_ubuntu_universe: True
    - role: hudecof.filter-plugins
```

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

```
- hosts: servers
  roles:
    - role: hudecof.packages
      packages_centos_epel: True
      packages_ubuntu_multiverse: True
      packages_ubuntu_universe: True
    - role: hudecof.filter-plugins
    - role: hudecof.nsd
      nsd_conf_keys:
        test:
          algorithm: hmac-sha256
          secret: aaaaaabbbbbbccccccdddddd
      nsd_conf_zones:
        example.com:
          include-pattern: test
      nsd_conf_patterns:
        test:
          'zonefile': 'zones.d/%s'
         'provide-xfr': ['0.0.0.0/0 test', '::0/0 test']
      nsd_conf_server:
        'username': "{{ nsd_user }}"
        'do-ip4': 'yes'
        'do-ip6': 'no'
        'hide-version': 'yes'
        'zonesdir': "{{ nsd_dir_etc }}"
        'database': "{{ nsd_dir_db }}/nsd.db"
        'pidfile': "{{ nsd_dir_run }}/nsd.pid"
        'xfrdfile': "{{ nsd_dir_state }}/xfrd.state"
        'verbosity': 0
        'ip-address': ['127.0.0.1']
```


License
-------

BSD

Author Information
------------------

Peter Hudec
CNC, a.s.