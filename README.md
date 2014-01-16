## A Flexible Staging Server

### The Problem

We need a way to hack around with Rails apps and their databases in Docker
containers. A full staging environment can be slow and inflexible.

### The Solution

Run an instance on Amazon Web Services. Just authorize Ansible with your secrets
and use it to run the playbook in this repo. Soon after, an EC2 instance will be
ready to receive your code and run it in lightweight Docker containers.

### Getting Started

Log in to the management console: http://console.aws.amazon.com/. Create a new
keypair by uploading your public key (~/.ssh/id_rsa.pub) and giving it a
memorable name. I called mine *mbp* for MacBook Pro.

If you choose to generate a new keypair instead, you can download it to the
~/.ssh folder. You may need to set permissions using a command along the lines
of `chmod 0600 ~/.ssh/keypair.pem`. For convenience, add this key to your SSH
agent using a command such as `ssh-add ~/.ssh/keypair.pem`.

You might also need to generate an AWS access token (and secret). You can do
this in the IAM section of the management console:
http://console.aws.amazon.com/iam

You will need an AWS token and secret in your ~/.boto file for
[boto](https://github.com/boto/boto) (the open-source library that Ansible
uses). See the boto wiki for more information:
http://docs.pythonboto.org/en/latest/boto_config_tut.html.

### Usage

Firstly, using the CloudFormation blueprint, Ansible provisions a single Linux
instance (with all the security groups and access keys and stuff) and configures
it to act as a git+docker server.

Secondly, push code which has a Dockerfile, wait as it gets built and assigned a
TCP port number. Then access the container at http://<instance-hostname>:$PORT/.

```bash
    cd src && ansible-playbook setup.yml -e keypair=YOUR_KEYPAIRS_ID
    git remote add staging root@<hostname>:repo.git
    git push --force staging <branch>
    open http://<hostname>:<port>/index.html
```

### Notes

Requires: Ansible ~v1.4.
License: MIT.
