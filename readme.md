# Packer
This repo contains information and examples of Packer templates for creating images of both Linux and Windows.

## What is Packer ?
Packer is an open source tool for creating identical machine images for multiple platforms from a single source configuration. Packer is lightweight, runs on every major operating system, and is highly performant, creating machine images for multiple platforms in parallel. Packer does not replace configuration management like Chef. In fact, when building images, Packer is able to use tools like Chef to install software onto the image.

A machine image is a single static unit that contains a pre-configured operating system and installed software which is used to quickly create new running machines. Machine image formats change for each platform. Some examples include AMIs for EC2, VMDK/VMX files for VMware, OVF exports for VirtualBox, etc.

## Packer Template
The configuration file used to define what image we want built and how is called a template in Packer terminology. The format of a template is simple JSON. JSON struck the best balance between human-editable and machine-editable, allowing both hand-made templates as well as machine generated templates to easily be made.

## Chef 
The Chef Client Packer provisioner installs and configures software on machines built by Packer using chef-client. Packer configures a Chef client to talk to a remote Chef Server to provision the machine.

### Chef Client Provisioner
The provisioner will even install Chef onto your machine if it isn't already installed, using the official Chef installers provided by Chef.

The example below is fully functional. It will install Chef onto the remote machine and run Chef client.

```
{
  "type": "chef-client",
  "server_url": "https://mychefserver.com/"
}
```

### Chef Client Local Mode
The following example shows how to run the chef-client provisioner in local mode, while passing a run_list using a variable.

#### Local environment variables
```
# Machine's Chef directory
export PACKER_CHEF_DIR=/var/chef-packer
# Comma separated run_list
export PACKER_CHEF_RUN_LIST="recipe[apt],recipe[nginx]"
```
#### Packer variables

Set the necessary Packer variables using environment variables or provide a var file.
```
"variables": {
  "chef_dir": "{{env `PACKER_CHEF_DIR`}}",
  "chef_run_list": "{{env `PACKER_CHEF_RUN_LIST`}}",
  "chef_client_config_tpl": "{{env `PACKER_CHEF_CLIENT_CONFIG_TPL`}}",
  "packer_chef_bootstrap_dir": "{{env `PACKER_CHEF_BOOTSTRAP_DIR`}}" ,
  "packer_uid": "{{env `PACKER_UID`}}",
  "packer_gid": "{{env `PACKER_GID`}}"
}
```
#### Setup the chef-client provisioner

Make sure we have the correct directories and permissions for the chef-client provisioner. You will need to bootstrap the Chef run by providing the necessary cookbooks using Berkshelf or some other means.
```
{
  "type": "file",
  "source": "{{user `packer_chef_bootstrap_dir`}}",
  "destination": "/tmp/bootstrap"
},
{
  "type": "shell",
  "inline": [
    "sudo mkdir -p {{user `chef_dir`}}",
    "sudo mkdir -p /tmp/packer-chef-client",
    "sudo chown {{user `packer_uid`}}.{{user `packer_gid`}} /tmp/packer-chef-client",
    "sudo sh /tmp/bootstrap/bootstrap.sh"
  ]
},
{
  "type": "chef-client",
  "server_url": "http://localhost:8889",
  "config_template": "{{user `chef_client_config_tpl`}}/client.rb.tpl",
  "skip_clean_node": true,
  "skip_clean_client": true,
  "run_list": "{{user `chef_run_list`}}"
}
```

### Chef Configuration Reference
The reference of available configuration options is listed below. No configuration is actually required.

```chef_environment``` (string) - The name of the chef_environment sent to the Chef server. By default this is empty and will not use an environment.

```config_template``` (string) - Path to a template that will be used for the Chef configuration file. By default Packer only sets configuration it needs to match the settings set in the provisioner configuration. If you need to set configurations that the Packer provisioner doesn't support, then you should use a custom configuration template. See the dedicated "Chef Configuration" section below for more details.

```encrypted_data_bag_secret_path``` (string) - The path to the file containing the secret for encrypted data bags. By default, this is empty, so no secret will be available.

```execute_command``` (string) - The command used to execute Chef. This has various configuration template variables available. See below for more information.

```guest_os_type``` (string) - The target guest OS type, either "unix" or "windows". Setting this to "windows" will cause the provisioner to use Windows friendly paths and commands. By default, this is "unix".

```install_command``` (string) - The command used to install Chef. This has various configuration template variables available. See below for more information.

```json``` (object) - An arbitrary mapping of JSON that will be available as node attributes while running Chef.

```knife_command``` (string) - The command used to run Knife during node clean-up. This has various configuration template variables available. See below for more information.

```node_name``` (string) - The name of the node to register with the Chef Server. This is optional and by default is packer-{{uuid}}.

```policy_group``` (string) - The name of a policy group that exists on the Chef server. policy_name must also be specified.

```policy_name``` (string) - The name of a policy, as identified by the name setting in a Policyfile.rb file. policy_group must also be specified.

```prevent_sudo``` (boolean) - By default, the configured commands that are executed to install and run Chef are executed with sudo. If this is true, then the sudo will be omitted. This has no effect when guest_os_type is windows.

```run_list``` (array of strings) - The run list for Chef. By default this is empty, and will use the run list sent down by the Chef Server.

```server_url``` (string) - The URL to the Chef server. This is required.

```skip_clean_client``` (boolean) - If true, Packer won't remove the client from the Chef server after it is done running. By default, this is false.

```skip_clean_node``` (boolean) - If true, Packer won't remove the node from the Chef server after it is done running. By default, this is false.

```skip_clean_staging_directory``` (boolean) - If true, Packer won't remove the Chef staging directory from the machine after it is done running. By default, this is false.

```skip_install``` (boolean) - If true, Chef will not automatically be installed on the machine using the Chef omnibus installers.

```ssl_verify_mode``` (string) - Set to "verify_none" to skip validation of SSL certificates. If not set, this defaults to "verify_peer" which validates all SSL certifications.

```trusted_certs_dir``` (string) - This is a directory that contains additional SSL certificates to trust. Any certificates in this directory will be added to whatever CA bundle ruby is using. Use this to add self-signed certs for your Chef Server or local HTTP file servers.

```staging_directory``` (string) - This is the directory where all the configuration of Chef by Packer will be placed. By default this is "/tmp/packer-chef-client" when guest_os_type unix and "$env:TEMP/packer-chef-client" when windows. This directory doesn't need to exist but must have proper permissions so that the user that Packer uses is able to create directories and write into this folder. By default the provisioner will create and chmod 0777 this directory.

```client_key``` (string) - Path to client key. If not set, this defaults to a file named client.pem in staging_directory.

```validation_client_name``` (string) - Name of the validation client. If not set, this won't be set in the configuration and the default that Chef uses will be used.

```validation_key_path``` (string) - Path to the validation key for communicating with the Chef Server. This will be uploaded to the remote machine. If this is NOT set, then it is your responsibility via other means (shell provisioner, etc.) to get a validation key to where Chef expects it.
