## A Flexible Staging Server

### The Problem

Spinning up new EC2 instances is just to slow and cumbersome.

### The Solution

A remote git repository which, upon receiving your code, builds and runs it in a
lightweight Docker container.

### The Implementation

This repository has a CloudFormation template and Ansible playbook for setting
up such a service. Just install Python and Ansible using your favourite package
manager, supply your Amazon Web Services credentials and then you're off and
racing.

Disclaimer: the code in this repository, if used correctly, will add charges to
your monthly bill from Amazon! That said however, the resources are neatly
bundled up as a single CloudFormation and can be easily cleaned up (you just
have to remember to do it).

### Getting Started

Log in to the management console: http://console.aws.amazon.com/. Create a new
keypair by uploading your public key (~/.ssh/id_rsa.pub) and giving it a
memorable name. I called mine *mbp* for MacBook Pro.

If you choose to generate a new keypair instead, you can download it to your
~/.ssh folder. You may need to set restrictive permissions using a command along
the lines of `chmod 0600 ~/.ssh/keypair.pem`. For convenience, also add this key
to your SSH agent using a command such as `ssh-add ~/.ssh/keypair.pem`.

You will need an AWS token and secret in your ~/.boto file for py-boto, the
open-source library that Ansible uses behind the scenes. See the boto wiki for
more information: http://docs.pythonboto.org/en/latest/boto_config_tut.html.

If you have misplaced your credentials, you may need to generate new ones. You
can do this in the IAM section of the management console:
http://console.aws.amazon.com/iam.

### Usage

Use Ansible to provision the CloudFormation blueprint (a single Linux instance,
with all the security groups and access keys and stuff) and configure it to act
as a git+docker server.

```bash
cd src && ansible-playbook setup.yaml
```

If the playbook runs successfully then you should see an IP address in the
output stream labelled as PublicIP.  Use this to set up a git remote in your
project's repository and then push some code which has a Dockerfile. Wait as it
gets built and assigned a random TCP port. You can then access the container at
http://<instance-ip>:<port>/.

```bash
git remote add staging root@<hostname>:repo.git
git push --force staging <branch>
open http://<hostname>:<port>/index.html
```

### Notes

Requires: Ansible ~v1.4.
License: MIT.
