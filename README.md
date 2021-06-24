# deploy-truenas

deploy-truenas.py is a Python script to deploy TLS certificates to a FreeNAS/TrueNAS (Core) server using the FreeNAS/TrueNAS API.  This should ensure that the certificate data is properly stored in the configuration database, and that all appropriate services use this certificate.  Its original intent was to be called from a Let's Encrypt client like [acme.sh](https://github.com/Neilpang/acme.sh) after the certificate is issued, so that the entire process of issuance (or renewal) and deployment can be automated.  However, it can be used with certificates from any source, whether a different ACME-based certificate authority or otherwise.

# Installation
This script can run on any machine running Python 3 that has network access to your FreeNAS/TrueNAS server, but in most cases it's best to run it directly on the FreeNAS/TrueNAS box.  Change to a convenient directory and run `git clone https://github.com/danb35/deploy-truenas`.

# Usage

The relevant configuration takes place in the `deploy_config` file.  You can create this file either by copying `depoy_config.example` from this repository, or directly using your preferred text editor.  Its format is as follows:

```
[deploy]
password = YourReallySecureRootPassword
cert_fqdn = foo.bar.baz
connect_host = baz.bar.foo
verify = false
privkey_path = /some/other/path
fullchain_path = /some/other/other/path
protocol = https://
port = 443
ftp_enabled = false
webdav_enabled = false
cert_base_name = letsencrypt
```

Everything but `password` (or `api_key`) is optional, and the defaults are documented in `depoy_config.example`.

On TrueNAS (Core) 12.0 and up you should use API key authentication instead of password authentication.
[Generate a new API token in the UI](https://www.truenas.com/docs/hub/additional-topics/api/#creating-api-keys) first, then add it as `api_key` to the config, which replaces the `password` field:
```
api_key = 1-DXcZ19sZoZFdGATIidJ8vMP6dxk3nHWz3XX876oxS7FospAGMQjkOft0h4itJDSP
```

Once you've prepared `deploy_config`, you can run `deploy_truenas.py`.  The intended use is that it would be called by your ACME client after issuing a certificate.  With acme.sh, for example, you'd add `--reloadcmd "/path/to/deploy_truenas.py"` to your command.

There is an optional paramter, `-c` or `--config`, that lets you specify the path to your configuration file. By default the script will try to use `deploy_config` in the script working directoy:

```
/path/to/deploy_truenas.py --config /somewhere/else/deploy_config
```

# OPNsense
To create an automation in the OPNsense GUI, a custom [configd](https://docs.opnsense.org/development/backend/configd.html)
action will have to be created.  Create a new configd action in /usr/local/opnsense/service/conf/actions.d/
with something similar:

```
[deploy]
command:/usr/local/deploy-truenas/deploy_truenas.py
parameters:
type:script
message:Deploying TLS certificates to TrueNAS
description:Deploy certificates to TrueNAS
```

Note: The description is required for the action to show up in the web UI, ownership needs to be root:wheel
and configd and the web UI need to be restarted in order for the new service to show up:
`service configd restart && /usr/local/etc/rc.restart_webgui`
