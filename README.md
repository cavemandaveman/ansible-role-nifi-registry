# Ansible Role: NiFi Registry

An Ansible Role that installs [NiFi Registry](https://nifi.apache.org/registry.html) on Linux. By default, it installs in a way that makes upgrading painless.

## Requirements

Requires at least Java 8.

## Role Variables

See `defaults/main.yml` for all variables and how to specify them. For a deeper dive, the [NiFi Registry System Administrator’s Guide](https://nifi.apache.org/docs/nifi-registry-docs/html/administration-guide.html) is a great resource.

The following specifies where to install NiFi Registry, along with a home directory (which will be symbolically linked to the release). Also, a centralized config directory to store files that need not be changed (to avoid copying during upgrades).

```yaml
nifi_registry_config_dirs:
  install: /opt/nifi-registry/releases
  home: /opt/nifi-registry/releases/current
  external_config: /opt/nifi-registry/config_resources
```

By default, this is the directory structure that will be created:

```text
|--opt/
  |--nifi-registry/
    |--releases/
      |--current -> nifi-registry-0.4.0/
      |--nifi-registry-0.3.0/
      |--nifi-registry-0.4.0/
    |--config_resources/
      |--authorizations.xml
      |--database/
      |--extension_bundles/
      |--flow_storage/
      |--users.xml
```

Any key/value pair from a config file can be added to the following dicts. Dict names correspond to file names. The current config options for these files can be found [here](https://github.com/apache/nifi-registry/tree/master/nifi-registry-core/nifi-registry-resources/src/main/resources/conf).

```yaml
nifi_registry_properties:
bootstrap:
logback:
identity_providers:
authorizers:
providers:
```

## Dependencies

None.

## Example Playbooks

These assume you have `hash_behaviour=merge` [set in your config](https://docs.ansible.com/ansible/latest/reference_appendices/config.html#default-hash-behaviour). If not, please also include the default dict key/values from `defaults/main.yml`.

Basic single node NiFi Registry instance:

```yaml
- hosts: nifi_registry_servers
  become: yes
  roles:
    - role: cavemandaveman.nifi_registry
```

Secure single node NiFi Registry instance with LDAP:

```yaml
- hosts: nifi_registry_servers
  become: yes
  roles:
    - role: cavemandaveman.nifi_registry
      nifi_registry_properties:
        # HTTP properties must be unset for HTTPS to work
        nifi.registry.web.http.host: ""
        nifi.registry.web.http.port: ""
        nifi.registry.web.https.host: "{{ ansible_fqdn }}"
        nifi.registry.web.https.port: 9443
        nifi.registry.security.keystore: /path/to/keystore.jks
        nifi.registry.security.keystoreType: JKS
        nifi.registry.security.keystorePasswd: keystorePassword
        nifi.registry.security.keyPasswd: keyPassword
        nifi.registry.security.truststore: /path/to/truststore.jks
        nifi.registry.security.truststoreType: JKS
        nifi.registry.security.truststorePasswd: truststorePassword
      identity_providers:
        /loginIdentityProviders/provider/identifier: ldap-provider
        /loginIdentityProviders/provider/property[@name="Authentication Strategy"]: SIMPLE
        /loginIdentityProviders/provider/property[@name="Manager DN"]: cn=nifi-registry,ou=people,dc=example,dc=com
        /loginIdentityProviders/provider/property[@name="Manager Password"]: password
        /loginIdentityProviders/provider/property[@name="Url"]: ldap://hostname:port
        /loginIdentityProviders/provider/property[@name="User Search Base"]: OU=people,DC=example,DC=com
        /loginIdentityProviders/provider/property[@name="User Search Filter"]: sAMAccountName={0}
      authorizers:
        /authorizers/userGroupProvider/property[@name="Initial User Identity 1"]: cn=John Smith,ou=people,dc=example,dc=com
        /authorizers/accessPolicyProvider/property[@name="Initial Admin Identity"]: cn=John Smith,ou=people,dc=example,dc=com
```

## License

GPLv3

## Author Information

This role was created in 2018 by cavemandaveman.
