# Packer
This repo contains information and examples of Packer templates for creating images of both Linux and Windows.

## What is Packer ?
Packer is an open source tool for creating identical machine images for multiple platforms from a single source configuration. Packer is lightweight, runs on every major operating system, and is highly performant, creating machine images for multiple platforms in parallel. Packer does not replace configuration management like Chef. In fact, when building images, Packer is able to use tools like Chef to install software onto the image.

A machine image is a single static unit that contains a pre-configured operating system and installed software which is used to quickly create new running machines. Machine image formats change for each platform. Some examples include AMIs for EC2, VMDK/VMX files for VMware, OVF exports for VirtualBox, etc.

## Packer Template
The configuration file used to define what image we want built and how is called a template in Packer terminology. The format of a template is simple JSON. JSON struck the best balance between human-editable and machine-editable, allowing both hand-made templates as well as machine generated templates to easily be made.

## Chef Client Provisioner
The Chef Client Packer provisioner installs and configures software on machines built by Packer using chef-client. Packer configures a Chef client to talk to a remote Chef Server to provision the machine.

The provisioner will even install Chef onto your machine if it isn't already installed, using the official Chef installers provided by Chef.

The example below is fully functional. It will install Chef onto the remote machine and run Chef client.

```
{
  "type": "chef-client",
  "server_url": "https://mychefserver.com/"
}
```
