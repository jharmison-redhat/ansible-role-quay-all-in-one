Red Hat Quay All-in-One
=========

Configures a RHEL host with Red Hat Quay, using a container-based installation with podman.

Requirements
------------

Python dependencies:

- jmespath
    - (for the `json_query` filter)
- cryptography
    - (for advanced X.509 certificate work)

Role Variables
--------------

    redhat_username: <some username>
    redhat_password: <some password>

_Undefaulted variables for logging in to registry.redhat.io, required for the official Red Hat Quay images._

    registry_admin:
      username: quay
      password: password
      email: quay@example.com

_The username, password, and email of the admin user to be created for the registry._

    registry_hostname: registry.example.com

_The public hostname of the registry (important for certificates!)_

    password_dir: '{{ playbook_dir }}/.passwords'

_The directory in which to save generated passwords._

    registry_s3_region: us-east-2

_The AWS S3 region to use for object storage. Optional if overriding registry_storage_details below._

    registry_s3_bucket: quay

_The name of the existing S3 bucket to use for object storage. Optional if overriding registry_storage_details below._

    registry_storage_type: S3Storage
    registry_storage_details:
      host: s3.{{ registry_s3_region }}.amazonaws.com
      s3_bucket: '{{ registry_s3_bucket }}'
    registry_storage_path: /datastorage/registry

_The storage configuration for the instance. For more information on how this might impact the storage configuration of your Quay instance, please use the config validator (or at least browse [the code](https://github.com/quay/config-tool/blob/redhat-3.6/pkg/lib/fieldgroups/distributedstorage/distributedstorage.go) for the appropriate version to understand the constraints). Note that this configuration assumes an instanceprofile that enables the instance to access the bucket!_

    deploy_clair: true

_Whether or not to deploy the Clair image scanner._

    do_redhat_login: true

_Whether to log in to registry.redhat.io, as required for official Red Hat Quay release images._

    cert_style: letsencrypt

_Which cert style to use for the registry, options from:_

- _letsencrypt_
- _byo_
- _selfsigned_

    registry_certificate: ""
    registry_certificate_key: ""

_When using BYO cert style, the PEM-encoded certificate and private key respectively._

    quay_version: "3.6.2"

_The version of Red Hat Quay to deploy - note, only tested on the 3.6 release._

    quay_image: registry.redhat.io/quay/quay-rhel8:v{{ quay_version }}
    clair_image: registry.redhat.io/quay/clair-rhel8:v{{ quay_version }}
    redis_image: registry.redhat.io/rhel8/redis-5:1
    postgres_image: registry.redhat.io/rhel8/postgresql-10:1

_The container images to use. Note that registry.redhat.io requires login._

Dependencies
------------

See [meta/requirements.yml](meta/requirements.yml).

- ansible.posix - Needed to set sysctls for unprivileged ports
- containers.podman - Needed to manage container instances and logins
- community.crypto - Needed to generate x509 certificates and requests for LetsEncrypt

Example Playbook
----------------

    - hosts: registry
      roles:
         - role: jharmison_redhat/redhat_quay
           vars:
             redhat_username: jharmison
             redhat_password: some super secure password
             registry_admin:
               username: james
               password: a different super secure password
               email: jharmison@redhat.com
             registry_hostname: quay.jharmison.net
             registry_s3_region: us-east-1
             registry_s3_bucket: registry

License
-------

BSD-2-Clause

TODO
----

- enable disconnected execution with (e.g. containers.podman.podman_load)
- provide up to 4 different ways to do TLS
  - ACME/LetsEncrypt (implemented now, default)
  - self-signed (implemented now)
  - BYO cert (implemented now)
  - dogtag aka ipa-getcert
- modularize backend to enable local storage instead of s3 if you want it, without understanding quay_config
- add disconnected clair loading (current recommendation is turn it off, but should be able to load the db as [documented](https://access.redhat.com/documentation/en-us/red_hat_quay/3.6/html/manage_red_hat_quay/clair-intro2#clair-disconnected)
