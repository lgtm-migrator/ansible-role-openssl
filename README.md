[![Version on Galaxy](https://img.shields.io/badge/available%20on%20ansible%20galaxy-jonaspammer.openssl-brightgreen)](https://galaxy.ansible.com/jonaspammer/openssl) [![Testing CI](https://github.com/JonasPammer/ansible-role-openssl/actions/workflows/ci.yml/badge.svg)](https://github.com/JonasPammer/ansible-role-openssl/actions/workflows/ci.yml)

An Ansible role for generating OpenSSL/x509 Certificate Files (privatekey, csr, certificate, pkcs12).

This role is a wrapper for all related community.crypto modules. It…​

- prepares the system by installing packages

- defines and uses system-specific default openssl directories (see `/vars`)

- ensures said directories exist with correct permissions

- provides ability of shared defaults (e.g. `item.default_backup`)

- makes yourself DRY (i.e. by doing `x509_certificate.privatekey_passphrase: "{{ item.privatekey_passphrase …​`)

- offers the ability to install the certificate to the Systems CA-trust store ([sidebar_title](#openssl__system_ca_store_file)) if enabled (see [variablelist_title](#install_ca_to_system)).

# 🔎 Metadata

Below you can find information on…

- the role’s required Ansible version

- the role’s supported platforms

- the role’s [role dependencies](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html#role-dependencies)

**[meta/main.yml](meta/main.yml)**

    ---
    galaxy_info:
      role_name: "openssl"
      description: "An ansible role for generating OpenSSL Certificate Files."
      standalone: true

      author: "jonaspammer"
      license: "MIT"

      min_ansible_version: "2.11"
      platforms:
        - name: EL # (Enterprise Linux)
          versions:
            - "7" # actively tested: centos7
            - "8" # actively tested: rockylinux8, centos8
        - name: Fedora
          versions:
            - "35"
        - name: Debian
          versions:
            - buster # debian10 (actively tested)
            - bullseye # debian11 (actively tested)
        - name: Ubuntu
          versions:
            - xenial # ubuntu1604 (actively tested)
            - bionic # ubuntu1804 (actively tested)
            - focal # ubuntu2004 (actively tested)

      galaxy_tags:
        - networking
        - system
        - web
        #    - certbot
        #    - letsencrypt
        - encryption
        - certificates
        - ssl
        - https

    dependencies: []

# 📌 Requirements

The Ansible User needs to be able to `become`.

The [`community.crypto`-collection](https://galaxy.ansible.com/community/crypto) must be installed on the Ansible controller.

An up-to-date pip (i.e. &gt;21) must be installed on the host. Needed for installation of [ `cryptography>3` through pip](https://pypi.org/project/cryptography/).

A Rust compiler must be installed on the host. Needed for installation of [cryptography&gt;3 through pip](https://pypi.org/project/cryptography/), if `openssl_cryptography_build_rust` is true.

# 📜 Role Variables

## Role Variables to configure the installation of Cryptography

The installation of cryptography is often a pain in the ass. I have yet to find a single exception to this rule. It has cost me feelingly, or even really, dozens of brain-tearing hours.

The below values, and their defaults seem to do the job.

    openssl_cryptography_prefer_binary: true

Defines if `--prefer-binary` is passed to the `pip install` command.

    openssl_cryptography_build_rust: false

Defines if the environment-variable `CRYPTOGRAPHY_DONT_BUILD_RUST` should be set to `1` (true) or `0` (false) while issuing the `pip install` command.

## OpenSSL

    openssl_items: []

List of ssl key/csr/crt/p12’s to generate. An entry may have the following values:

This role gives you control over nearly all variables, but also fills-in variables on its own (e.g. relative ones like `privatekey_path`).

filename (Required)
Name used for every file.

<!-- -->

install*ca_to_system
\_Boolean*. Defaults to `false`.
Whether to execute steps to integrate the self signed certificate to the the System’s CA Trust Store ([sidebar_title](#openssl__system_ca_store_file)).

(Debian/Ubuntu ONLY) Messed up / Fucked up? Try setting `install_ca_to_system_fresh` to `true`.

install*ca_to_system_fresh
(Debian/Ubuntu ONLY) \_Boolean*. Defaults to value of `default_force` (if existent) or `false`.
Invokes `update-ca-certificates` with `--fresh` to remove symlinks in /etc/ssl/certs directory (_before completely resourcing the directory using the configured CA directories and settings_). This thus purges dead / potentially wrong or conflicting certificates.

The below values can be used to set the default parameters of every module where applicable. See the page of any below module for documentation.

default_group; default_owner; default_backup; default_selevel; default_serole; default_setype; default_seuser; default_state; default_force

Parameters used to generate OpenSSL private key ('.key') using [openssl_privatekey](https://docs.ansible.com/ansible/2.9/modules/openssl_privatekey_module.html).

`path`, `mode` and `unsafe_writes` can not be supplied.

_privatekey_backup_; privatekey*cipher; privatekey_curve; privatekey_group; privatekey_owner; privatekey_passphrase; \_privatekey_selevel*; _privatekey_serole_; _privatekey_setype_; _privatekey_seuser_; _privatekey_size_; _privatekey_state_; _privatekey_type_

Parameters used to generate OpenSSL Certificate Signing Request with appropriate subject information ('.csr') using [community.crypto.openssl_csr](https://docs.ansible.com/ansible/2.9/modules/openssl_csr_module.html).

`mode`, `path`, `privatekey_passphrase`, `privatekey_path` and `unsafe_writes` can not be supplied.

This role forces you to supply at-least `csr_common_name`.

csr*authority_cert_issuer; csr_authority_cert_serial_number; csr_authority_key_identifier; \_csr_backup*; csr*basic_constraints; csr_basic_constraints_critical; \_csr_common_name (Required)*; _csr_country_name_; csr*create_subject_key_identifier; csr_digest; \_csr_email_address*; csr*extended_key_usage; csr_extended_key_usage_critical; csr_force; csr_group; csr_key_usage; csr_key_usage_critical; csr_locality_name; csr_ocsp_must_staple; csr_ocsp_must_staple_critical; \_csr_organization_name*; csr*organizational_unit_name; csr_owner; \_csr_selevel*; _csr_serole_; _csr_setype_; _csr_seuser_; _csr_state_; _csr_state_or_province_name_; csr_subject; csr_subject_alt_name; csr_subject_alt_name_critical; csr_subject_key_identifier; csr_use_common_name_for_san

Parameters used to generate Self Signed OpenSSL certificate ('.crt') using [community.crypto.openssl_certificate](https://docs.ansible.com/ansible/2.9/modules/openssl_certificate_module.html).

`csr_path`, `mode`, `path`, `privatekey_passphrase` and `privatekey_path` can not be supplied. (`privatekey_passphrase` can be supplied using `csr_privatekey_passphrase`)

This role sets `provider` to “selfsigned” by default.

x509cert*acme_accountkey_path; x509cert_acme_chain; x509cert_acme_challenge_path; x509cert_attributes; \_x509cert_backup*; x509cert_entrust_api_client_cert_key_path; x509cert_entrust_api_client_cert_path; x509cert_entrust_api_key; x509cert_entrust_api_specification_path; x509cert_entrust_cert_type; x509cert_entrust_not_after; x509cert_entrust_requester_email; x509cert_entrust_requester_name; x509cert_entrust_requester_phone; x509cert_extended_key_usage; x509cert_extended_key_usage_strict; x509cert_force; x509cert_group; x509cert_has_expired; x509cert_invalid_at; x509cert_issuer; x509cert_issuer_strict; x509cert_key_usage; x509cert_key_usage_strict; x509cert_not_after; x509cert_not_before; x509cert_ownca_create_authority_key_identifier; x509cert_ownca_create_subject_key_identifier; x509cert_ownca_digest; x509cert_ownca_not_after; x509cert_ownca_not_before; x509cert_ownca_path; x509cert_ownca_privatekey_passphrase; x509cert_ownca_privatekey_path; x509cert_ownca_version; x509cert_owner; x509cert_provider
Defaults to `selfsigned` in this role.

x509cert*select_crypto_backend; \_x509cert_selevel*; _x509cert_serole_; _x509cert_setype_; _x509cert_seuser_; x509cert*selfsigned_create_subject_key_identifier; x509cert_selfsigned_digest; \_x509cert_selfsigned_not_after*; _x509cert_selfsigned_not_before_; x509cert*selfsigned_version; x509cert_signature_algorithms; \_x509cert_state*; x509cert_subject; x509cert_subject_alt_name; x509cert_subject_alt_name_strict; x509cert_subject_strict; x509cert_valid_at; x509cert_valid_in; x509cert_version

Parameters used to generate Self Signed OpenSSL certificate ('.crt') [community.crypto.openssl_pkcs12](https://docs.ansible.com/ansible/latest/collections/community/crypto/openssl_pkcs12_module.html).

`action`, `certificate_path`, `friendly_name`, `mode`, `path`, `privatekey_passphrase`, `privatekey_path`, `return_content` and `unsafe_writes` can not be supplied. (`privatekey_passphrase` can be supplied using `csr_privatekey_passphrase`)

pkcs12_attributes; pkcs12_backup; pkcs12_force; pkcs12_group; pkcs12_iter_size; pkcs12_maciter_size; pkcs12_other_certificates; pkcs12_other_certificates_parse_all; pkcs12_owner; pkcs12_passphrase; pkcs12_select_crypto_backend; pkcs12_selevel; pkcs12_serole; pkcs12_setype; pkcs12_seuser; pkcs12_state

## Directory Configuration

    openssl_directory: ~

If given, this sets the default for the `openssl_[key|crt|csr]_directory`.

    openssl_key_directory: [OS-specific by default, see /vars directory]

This directory stores sensitive objects (`key`, `p12` and `pkcs12`).

    openssl_crt_directory: [OS-specific by default, see /vars directory]

This directory stores public, non-persistent objects (`csr`).

    openssl_csr_directory: [OS-specific by default, see /vars directory]

This directory stores public, persistent objects (`crt`).

# 📜 Facts/Variables defined by this role

Each variable listed in this section is dynamically defined when executing this role (and can only be overwritten using `ansible.builtin.set_facts`) _and_ is meant to be used not just internally.

Location of the System’s trusted CA Certificate Store/Bundle File. As per <https://serverfault.com/a/722646>

# 🏷️ Tags

Tasks are tagged with the following [tags](https://docs.ansible.com/ansible/latest/user_guide/playbooks_tags.html#adding-tags-to-roles):

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th style="text-align: left;">Tag</th>
<th style="text-align: left;">Purpose</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td colspan="2" style="text-align: left;"><p>This role does not have officially documented tags yet.</p></td>
</tr>
</tbody>
</table>

You can use Ansible to skip tasks, or only run certain tasks by using these tags. By default, all tasks are run when no tags are specified.

# 👫 Dependencies

# 📚 Example Playbook Usages

This role is part of [ many compatible purpose-specific roles of mine](https://github.com/JonasPammer/ansible-roles).

The machine needs to be prepared. In CI, this is done in `molecule/resources/prepare.yml` which sources its soft dependencies from `requirements.yml`:

**[molecule/resources/prepare.yml](molecule/resources/prepare.yml)**

    ---
    - name: prepare
      hosts: all
      become: true
      gather_facts: false

      roles:
        - name: jonaspammer.bootstrap
        - name: jonaspammer.pip
        #    - name: jonaspammer.core_dependencies

The following diagram is a compilation of the "soft dependencies" of this role as well as the recursive tree of their soft dependencies.

![requirements.yml dependency graph of jonaspammer.openssl](https://raw.githubusercontent.com/JonasPammer/ansible-roles/master/graphs/dependencies_openssl.svg)

    roles:
      - jonaspammer.openssl

    vars:
      - filename: common_jonaspammer_at
        csr_common_name: common.jonaspammer.at
        csr_country_name: "AT"
        csr_state_or_province_name: "Vorarlberg"
        csr_locality_name: "Bregenz"
        csr_email_address: "opensource@jonaspammer.at"

        privatekey_backup: true
        install_ca_to_system: true

- The following play:

      roles:
        - jonaspammer.openssl

      vars:
        openssl_items:
          - filename: jonaspammer_at
            csr_common_name: jonaspammer.at

  creates the following files (on Debian):

      root@instance-py3-ansible-5-ubuntu1604:/# ls -l /etc/ssl/private/ # openssl_key_directory
      total 16
      -rw------- 1 root root 3243 Jun 23 09:37 jonaspammer_at.key
      -rw-r----- 1 root root 5031 Jun 23 09:37 jonaspammer_at.keycrt
      -rw-r----- 1 root root 4024 Jun 23 09:37 jonaspammer_at.p12

      root@instance-py3-ansible-5-ubuntu1604:/# ls -l /etc/ssl/misc # openssl_csr_directory
      total 4
      -rw-r--r-- 1 root root 1651 Jun 23 09:37 jonaspammer_at.csr

      root@instance-py3-ansible-5-ubuntu1604:/# ls -l /etc/ssl/certs # openssl_crt_directory
      total 580
      lrwxrwxrwx 1 root root     49 Sep 19  2021 01419da9.0 -> Microsoft_ECC_Root_Certificate_Authority_2017.pem
      lrwxrwxrwx 1 root root     45 Sep 19  2021 02265526.0 -> Entrust_Root_Certification_Authority_-_G2.pem
      …
      -rw-r--r-- 1 root root   1789 Jun 23 09:37 jonaspammer_at.crt
      …

      # Below commands as seen in https://www.xolphin.com/support/OpenSSL/Frequently_used_OpenSSL_Commands

      root@instance-py3-ansible-5-ubuntu1604:/etc/ssl# openssl x509 -text -noout -in certs/jonaspammer_at.crt
      Certificate:
          Data:
              Version: 3 (0x2)
              Serial Number:
                  0a:03:4d:1e:db:b5:48:d2:13:d8:1c:d5:28:46:41:1d:7c:1c:f1:7f
          Signature Algorithm: sha256WithRSAEncryption
              Issuer: CN=jonaspammer.at
              Validity
                  Not Before: Jun 23 09:37:25 2022 GMT
                  Not After : Jun 20 09:37:25 2032 GMT
              Subject: CN=jonaspammer.at
              Subject Public Key Info:
                  Public Key Algorithm: rsaEncryption
                      Public-Key: (4096 bit)
                      Modulus:
                          00:cf:62:15:5b:cb:3e:3a:7c:3a:9b:5e:5e:47:37:
                          ………
                      Exponent: 65537 (0x10001)
              X509v3 extensions:
                  X509v3 Subject Alternative Name:
                      DNS:jonaspammer.at
                  X509v3 Subject Key Identifier:
                      6C:5E:12:CE:E1:98:4C:F1:B4:74:4E:AB:C4:1E:93:15:69:79:72:3B
          Signature Algorithm: sha256WithRSAEncryption
               21:f6:11:83:83:15:01:62:e2:78:8e:78:44:cd:0e:8c:01:00:
               ………

      root@instance-py3-ansible-5-ubuntu1604:/etc/ssl# openssl req -text -noout -verify -in misc/jonaspammer_at.csr
      verify OK
      Certificate Request:
          Data:
              Version: 0 (0x0)
              Subject: CN=jonaspammer.at
              Subject Public Key Info:
                  Public Key Algorithm: rsaEncryption
                      Public-Key: (4096 bit)
                      Modulus:
                          00:cf:62:15:5b:cb:3e:3a:7c:3a:9b:5e:5e:47:37:
                          ………
                      Exponent: 65537 (0x10001)
              Attributes:
              Requested Extensions:
                  X509v3 Subject Alternative Name:
                      DNS:jonaspammer.at
          Signature Algorithm: sha256WithRSAEncryption
               81:09:f3:cf:55:3c:ef:2f:6c:b7:5e:cd:64:d0:66:f5:1d:d4:
               ………

      root@instance-py3-ansible-5-ubuntu1604:/etc/ssl# openssl rsa -noout -in private/jonaspammer_at.key -check
      RSA key ok

      root@instance-py3-ansible-5-ubuntu1604:/etc/ssl# openssl pkcs12 -noout -info -in private/jonaspammer_at.p12
      Enter Import Password:
      MAC Iteration 1
      MAC verified OK
      PKCS7 Data
      Certificate bag
      PKCS7 Data
      Key bag

      # Checking the MD5 hash of the public key to check if it is equal to what is in the CSR or private key.
      root@instance-py3-ansible-5-ubuntu1604:/etc/ssl# openssl x509 -noout -modulus -in certs/jonaspammer_at.crt | openssl md5
      -in private/jonaspammer_at.key | openssl md5
      opens(stdin)= da1f0a7e379330443660f098bfb64043
      root@instance-py3-ansible-5-ubuntu1604:/etc/ssl# openssl rsa -noout -modulus -in private/jonaspammer_at.key | openssl md5
      sl req -noout -modulus -in misc/jonaspammer_at.csr(stdin)= da1f0a7e379330443660f098bfb64043
      root@instance-py3-ansible-5-ubuntu1604:/etc/ssl# openssl req -noout -modulus -in misc/jonaspammer_at.csr | openssl md5
      (stdin)= da1f0a7e379330443660f098bfb64043

# 📝 Development

[![Conventional Commits](https://img.shields.io/badge/Conventional%20Commits-1.0.0-yellow.svg)](https://conventionalcommits.org) [![pre-commit.ci status](https://results.pre-commit.ci/badge/github/JonasPammer/ansible-role-openssl/master.svg)](https://results.pre-commit.ci/latest/github/JonasPammer/ansible-role-openssl/master)

## 📌 Development Machine Dependencies

- Python 3.8 or greater

- Docker

## 📌 Development Dependencies

Development Dependencies are defined in a [pip requirements file](https://pip.pypa.io/en/stable/user_guide/#requirements-files) named `requirements-dev.txt`. Example Installation Instructions for Linux are shown below:

    # "optional": create a python virtualenv and activate it for the current shell session
    $ python3 -m venv venv
    $ source venv/bin/activate

    $ python3 -m pip install -r requirements-dev.txt

## ℹ️ Ansible Role Development Guidelines

Please take a look at my [ Ansible Role Development Guidelines](https://github.com/JonasPammer/cookiecutter-ansible-role/blob/master/ROLE_DEVELOPMENT_GUIDELINES.adoc).

If interested, I’ve also written down some [ General Ansible Role Development (Best) Practices](https://github.com/JonasPammer/cookiecutter-ansible-role/blob/master/ROLE_DEVELOPMENT_TIPS.adoc).

## 🔢 Versioning

Versions are defined using [Tags](https://git-scm.com/book/en/v2/Git-Basics-Tagging), which in turn are [recognized and used](https://galaxy.ansible.com/docs/contributing/version.html) by Ansible Galaxy.

**Versions must not start with `v`.**

When a new tag is pushed, [ a GitHub CI workflow](https://github.com/JonasPammer/ansible-role-openssl/actions/workflows/release-to-galaxy.yml) (![Release CI](https://github.com/JonasPammer/ansible-role-openssl/actions/workflows/release-to-galaxy.yml/badge.svg)) takes care of importing the role to my Ansible Galaxy Account.

## 🧪 Testing

Automatic Tests are run on each Contribution using GitHub Workflows.

The Tests primarily resolve around running [Molecule](https://molecule.readthedocs.io/en/latest/) on a varying set of linux distributions and using various ansible versions, as detailed in [JonasPammer/ansible-roles](https://github.com/JonasPammer/ansible-roles).

The molecule test also includes a step which lints all ansible playbooks using [`ansible-lint`](https://github.com/ansible/ansible-lint#readme) to check for best practices and behaviour that could potentially be improved.

To run the tests, simply run `tox` on the command line. You can pass an optional environment variable to define the distribution of the Docker container that will be spun up by molecule:

    $ MOLECULE_DISTRO=centos7 tox

For a list of possible values fed to `MOLECULE_DISTRO`, take a look at the matrix defined in [.github/workflows/ci.yml](.github/workflows/ci.yml).

### 🐛 Debugging a Molecule Container

1.  Run your molecule tests with the option `MOLECULE_DESTROY=never`, e.g.:

        $ MOLECULE_DESTROY=never MOLECULE_DISTRO=ubuntu1604 tox -e py3-ansible-5
        ...
          TASK [ansible-role-pip : (redacted).] ************************
          failed: [instance-py3-ansible-5] => changed=false
        ...
         ___________________________________ summary ____________________________________
          pre-commit: commands succeeded
        ERROR:   py3-ansible-5: commands failed

2.  Find out the name of the molecule-provisioned docker container:

        $ docker ps
        30e9b8d59cdf   geerlingguy/docker-debian10-ansible:latest   "/lib/systemd/systemd"   8 minutes ago   Up 8 minutes                                                                                                    instance-py3-ansible-5

3.  Get into a bash Shell of the container, and do your debugging:

        $ docker exec -it 30e9b8d59cdf /bin/bash

        root@instance-py3-ansible-2:/#
        root@instance-py3-ansible-2:/# python3 --version
        Python 3.8.10
        root@instance-py3-ansible-2:/# ...

    If the failure you try to debug is part of `verify.yml` step and not the actual `converge.yml`, you may want to know that the output of ansible’s modules (`vars`), hosts (`hostvars`) and environment variables have been stored into files on both the provisioner and inside the docker machine under: \* `/var/tmp/vars.yml` \* `/var/tmp/hostvars.yml` \* `/var/tmp/environment.yml` `grep`, `cat` or transfer these as you wish!

    You may also want to know that the files mentioned in the admonition above are attached to the **GitHub CI Artifacts** of a given Workflow run.
    This allows one to check the difference between runs and thus help in debugging what caused the bit-rot or failure in general. image::https://user-images.githubusercontent.com/32995541/178442403-e15264ca-433a-4bc7-95db-cfadb573db3c.png\[\]

4.  After you finished your debugging, exit it and destroy the container:

        root@instance-py3-ansible-2:/# exit

        $ docker stop 30e9b8d59cdf

        $ docker container rm 30e9b8d59cdf
        or
        $ docker container prune

## 🧃 TIP: Containerized Ideal Development Environment

This Project offers a definition for a "1-Click Containerized Development Environment".

This Container even allow one to run docker containers inside of them (Docker-In-Docker, dind), allowing for molecule execution.

To use it:

1.  Ensure you fullfill the [ the System requirements of Visual Studio Code Development Containers](https://code.visualstudio.com/docs/remote/containers#_system-requirements), optionally following the _Installation_-Section of the linked page section.
    This includes: Installing Docker, Installing Visual Studio Code itself, and Installing the necessary Extension.

2.  Clone the project to your machine

3.  Open the folder of the repo in Visual Studio Code (_File - Open Folder…_).

4.  If you get a prompt at the lower right corner informing you about the presence of the devcontainer definition, you can press the accompanying button to enter it. **Otherwise,** you can also execute the Visual Studio Command `Remote-Containers: Open Folder in Container` yourself (_View - Command Palette_ → _type in the mentioned command_).

I recommend using `Remote-Containers: Rebuild Without Cache and Reopen in Container` once here and there as the devcontainer feature does have some problems recognizing changes made to its definition properly some times.

You may need to configure your host system to enable the container to use your SSH Keys.

The procedure is described [ in the official devcontainer docs under "Sharing Git credentials with your container"](https://code.visualstudio.com/docs/remote/containers#_sharing-git-credentials-with-your-container).

## 🍪 CookieCutter

This Project shall be kept in sync with [the CookieCutter it was originally templated from](https://github.com/JonasPammer/cookiecutter-ansible-role) using [cruft](https://github.com/cruft/cruft) (if possible) or manual alteration (if needed) to the best extend possible.

> ![Official Example Usage of `cruft update`](https://raw.githubusercontent.com/cruft/cruft/master/art/example_update.gif)

### 🕗 Changelog

When a new tag is pushed, an appropriate GitHub Release will be created by the Repository Maintainer to provide a proper human change log with a title and description.

## ℹ️ General Linting and Styling Conventions

General Linting and Styling Conventions are [**automatically** held up to Standards](https://stackoverflow.blog/2020/07/20/linters-arent-in-your-way-theyre-on-your-side/) by various [`pre-commit`](https://pre-commit.com/) hooks, at least to some extend.

Automatic Execution of pre-commit is done on each Contribution using [`pre-commit.ci`](https://pre-commit.ci/)[\*](#note_pre-commit-ci). Pull Requests even automatically get fixed by the same tool, at least by hooks that automatically alter files.

Not to confuse: Although some pre-commit hooks may be able to warn you about script-analyzed flaws in syntax or even code to some extend (for which reason pre-commit’s hooks are **part of** the test suite), pre-commit itself does not run any real Test Suites. For Information on Testing, see [🧪 Testing](#testing).

Nevertheless, I recommend you to integrate pre-commit into your local development workflow yourself.

This can be done by cd’ing into the directory of your cloned project and running `pre-commit install`. Doing so will make git run pre-commit checks on every commit you make, aborting the commit themselves if a hook alarm’ed.

You can also, for example, execute pre-commit’s hooks at any time by running `pre-commit run --all-files`.

# 💪 Contributing

![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=flat-square) [![Open in Visual Studio Code](https://img.shields.io/static/v1?logo=visualstudiocode&label=&message=Open%20in%20Visual%20Studio%20Code&labelColor=2c2c32&color=007acc&logoColor=007acc)](https://open.vscode.dev/JonasPammer/ansible-role-openssl)

The following sections are generic in nature and are used to help new contributors. The actual "Development Documentation" of this project is found under [📝 Development](#development).

## 🤝 Preamble

First off, thank you for considering contributing to this Project.

Following these guidelines helps to communicate that you respect the time of the developers managing and developing this open source project. In return, they should reciprocate that respect in addressing your issue, assessing changes, and helping you finalize your pull requests.

## 🍪 CookieCutter

This Project owns many of its files to [the CookieCutter it was originally templated from](https://github.com/JonasPammer/cookiecutter-ansible-role).

Please check if the edit you have in mind is actually applicable to the template and if so make an appropriate change there instead. Your change may also be applicable partly to the template as well as partly to something specific to this project, in which case you would be creating multiple PRs.

## 💬 Conventional Commits

A casual contributor does not have to worry about following [_the spec_](https://github.com/JonasPammer/JonasPammer/blob/master/demystifying/conventional_commits.adoc) [_by definition_](https://www.conventionalcommits.org/en/v1.0.0/), as pull requests are being squash merged into one commit in the project. Only core contributors, i.e. those with rights to push to this project’s branches, must follow it (e.g. to allow for automatic version determination and changelog generation to work).

## 🚀 Getting Started

Contributions are made to this repo via Issues and Pull Requests (PRs). A few general guidelines that cover both:

- Search for existing Issues and PRs before creating your own.

- If you’ve never contributed before, see [ the first timer’s guide on Auth0’s blog](https://auth0.com/blog/a-first-timers-guide-to-an-open-source-project/) for resources and tips on how to get started.

### Issues

Issues should be used to report problems, request a new feature, or to discuss potential changes **before** a PR is created. When you [ create a new Issue](https://github.com/JonasPammer/ansible-role-openssl/issues/new), a template will be loaded that will guide you through collecting and providing the information we need to investigate.

If you find an Issue that addresses the problem you’re having, please add your own reproduction information to the existing issue **rather than creating a new one**. Adding a [reaction](https://github.blog/2016-03-10-add-reactions-to-pull-requests-issues-and-comments/) can also help be indicating to our maintainers that a particular problem is affecting more than just the reporter.

### Pull Requests

PRs to this Project are always welcome and can be a quick way to get your fix or improvement slated for the next release. [In general](https://blog.ploeh.dk/2015/01/15/10-tips-for-better-pull-requests/), PRs should:

- Only fix/add the functionality in question **OR** address wide-spread whitespace/style issues, not both.

- Add unit or integration tests for fixed or changed functionality (if a test suite already exists).

- **Address a single concern**

- **Include documentation** in the repo

- Be accompanied by a complete Pull Request template (loaded automatically when a PR is created).

For changes that address core functionality or would require breaking changes (e.g. a major release), it’s best to open an Issue to discuss your proposal first.

In general, we follow the "fork-and-pull" Git workflow

1.  Fork the repository to your own Github account

2.  Clone the project to your machine

3.  Create a branch locally with a succinct but descriptive name

4.  Commit changes to the branch

5.  Following any formatting and testing guidelines specific to this repo

6.  Push changes to your fork

7.  Open a PR in our repository and follow the PR template so that we can efficiently review the changes.

# 🗒 Changelog

Please refer to the [Release Page of this Repository](https://github.com/JonasPammer/ansible-role-openssl/releases) for a human changelog of the corresponding [Tags (Versions) of this Project](https://github.com/JonasPammer/ansible-role-openssl/tags).

Note that this Project adheres to Semantic Versioning. Please report any accidental breaking changes of a minor version update.

# ⚖️ License

**[LICENSE](LICENSE)**

    MIT License

    Copyright (c) 2022, Jonas Pammer

    Permission is hereby granted, free of charge, to any person obtaining a copy
    of this software and associated documentation files (the "Software"), to deal
    in the Software without restriction, including without limitation the rights
    to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
    copies of the Software, and to permit persons to whom the Software is
    furnished to do so, subject to the following conditions:

    The above copyright notice and this permission notice shall be included in all
    copies or substantial portions of the Software.

    THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
    IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
    FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
    AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
    LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
    OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
    SOFTWARE.
