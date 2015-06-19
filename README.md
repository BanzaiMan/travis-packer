# travis-packer

Files you need to create Vagrant boxes for use with travis-cookbooks

In order to develop for [`travis-cookbooks`](https://github.com/travis-ci/travis-cookbooks),
a local development is useful.
While it is possible to use the local machine for this, a virtual machine is
desirable.

This repository contains a useful script to make it easier to provision a VM for
`travis-cookbooks` development.

## Requirements

1. [Packer](https://packer.io/)
2. [Vagrant](https://www.vagrantup.com/)
3. At least one of [VirtualBox](https://www.virtualbox.org/),
[VMWare Fusion](http://www.vmware.com/products/fusion) (for OS X) or [VMWare Workstation](http://www.vmware.com/products/workstation) (for Windows and Linux),
or [Parallels Desktop](http://www.parallels.com/products/desktop/) (for OS X).

## Usage

1. Clone this, [`bento`](https://github.com/opscode/bento) and `travis-cookbooks` in the same directory.
1. Change directory to `travis-packer`.
1. Install Active Support:

  `bundle install`

1. Run `bundle exec ./generate`. This prints to STDOUT the JSON document that is suitable for use by Packer. Save it in `../bento`:

  `bundle exec ./generate > ../bento/ubuntu-12.04-amd64-travis.json`

1. Move to `../bento`.
1. It seems that in the first packer build, the `system_info` cookbook
   appears to hang.
   It should be removed from the JSON template.

  `vi ubuntu-12.04-amd64-travis.json`

1. Build the base "standard" image. This could take a while, depending on the resources available to the VM.

  `packer build -parallel=false [-only=vmware-iso] ubuntu-12.04-amd64-travis.json`

1. Add the resulting Vagrant box as `travis-precise`:

  `vagrant box add --name travis-precise ../builds/vmware/travis_ubuntu-12.04_chef-latest.box`

1. In the `travis-cookbooks` directory, run the rest of language-specific cookbooks:

  `vagrant up ruby-precise`

## Hints

### Build times

The generated packer template allocates 2 GB of memory and 2 CPUs to the VM while provisioning
the base Vagrant box.
If your host machine has enough resources, you can allocate more.
The resource allocation is defined near `vboxmanage`, `vmx_data` and `prlctl`.

Unless you have a giant host machine to build the VMs, `-parallel=false` is strongly advised.

You can also pass `-only` flag to limit boxes to build.

### Building Trusty boxes

By default, `./generate` reads definitions given by Bento's Ubuntu 12.04 on x86_64 (i.e., amd64) template.
It has also been tested for use with Ubuntu 14.04. You can generate this with:

```
./generate 14.04
```

You can then develop cookbooks for Travis CI's next generation images.

While theoretically possible, other operating systems and architectures (notably x86) are not supported.
(There is no plan to support them on Travis CI.)

### Cookbooks layout

The [worker image cookbooks in `travis-cookbooks`](https://github.com/travis-ci/travis-cookbooks/tree/master/ci_environment)
are divided into two categories (though there is a small overlap).

1. Ones that run on the `standard` image (which `packer build` command executes)
2. Ones that run on each language image (which `vagrant up LANGUAGE-precise` command executes)

You can look at https://github.com/travis-ci/travis-cookbooks/tree/master/vm_templates which cookbooks runs where.

