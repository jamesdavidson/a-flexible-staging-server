## A Flexible Staging Server

### The Problem

Spinning up new EC2 instances is too cumbersome for rapid experimentation.

### The Solution

A remote git repository which, upon receiving your code, builds and runs it in a
lightweight Docker container.

### The Implementation

This repository has a CloudFormation template, Ansible playbook and Ruby post
receive hook for setting up such a service.

Disclaimer: the code in this repository, if used correctly, will add charges to
your monthly bill from Amazon! That said however, the resources are neatly
bundled up as a single CloudFormation and can be easily cleaned up (you just
have to remember to do it).

### Usage

Firstly, check the config options in src/parameters.yaml. Then, use Ansible to
provision the CloudFormation (a single Linux instance, with all the security
groups and access keys and stuff) and configure it to act as a git+docker
server.

```bash
cd src && ansible-playbook setup.yaml
```

If the playbook runs successfully then you should see an IP address in the
output stream. Use this to set up a git remote in your project's repository.
Ensure that your project has a Dockerfile in its root directory that exposes its
functionality on port 80. Git push to the remote repository and wait as it gets
built and assigned a TCP port. You can then access the container at
http://<instance-ip>:port/.

```bash
git remote add staging ec2-user@<instance-ip>:repo.git
git push --force staging <branch>
open http://<instance-ip>:port/
```

Your Dockerfile could look like this:
```
FROM nginx
COPY html_assets /usr/share/nginx/html
```

### Installation

Just install Python2 using your package manager, then install Pip using
easy_install, then install Ansible and Boto using Pip.

```bash
{port|apt-get|brew|yum|pacman} install python2
easy_install pip
pip install ansible boto
```

Log in to the management console: http://console.aws.amazon.com/. Create a new
keypair by uploading your public key (probably ~/.ssh/id_rsa.pub) and giving it
a memorable name. I called mine _mbp_ for MacBookPro.

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

### Notes

Requires: Ansible ~v1.4.
License: MIT.
