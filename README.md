# GitHub Actions to Free Disk Space on Ubuntu runners

A customizable GitHub Actions to free disk space on Linux GitHub Actions runners.

On a typical Ubuntu runner, with all options turned on (or not turned off rather), this can clear up to 31 GB of disk space in about 3 minutes (the longest period is calling `apt` to uninstall packages). This is useful when you need a lot of disk space to run computations.

Please don't hesitate to [submit issues](https://github.com/jlumbroso/free-disk-space/issues) to report problems or suggest new features (or sets of files to help remove).

Also, please ⭐️ the repo if you like this GitHub Actions! Thanks! 😊

## Example

```yaml
name: Free Disk Space (Ubuntu)
on: push

jobs:
  free-disk-space:
    runs-on: ubuntu-latest
    steps:

    - name: Free Disk Space (Ubuntu)
      uses: jlumbroso/free-disk-space@main
      with:
        # this might remove tools that are actually needed,
        # if set to "true" but frees about 6 GB
        tool-cache: false
        
        # all of these default to true, but feel free to set to
        # "false" if necessary for your workflow
        mandb:   true
        android: true
        dotnet: true
        haskell: true
        large-packages: true
        docker-images: true
        swap-storage: true
```
## Options

Most of the options are self-explanatory.

The option `tool-cache` removes all the pre-cached tools (Node, Go, Python, Ruby, ...) that are loaded in a runner's environment, [installed in the path specified by the `AGENT_TOOLSDIRECTORY` environment variable](https://github.com/actions/virtual-environments/blob/5a2cb18a48bce5da183486b95f5494e4fd0c0640/images/linux/scripts/installers/configure-environment.sh#L25-L29) (the same environment variable is used across Windows/macOS/Linux runners, see an example of its use on [the `setup-python` GitHub Action](https://github.com/actions/setup-python)). This option was [suggested](https://github.com/actions/virtual-environments/issues/2875#issuecomment-1163392159) by [@miketimofeev](https://github.com/miketimofeev).

## Acknowledgement

This GitHub Actions came around because I kept rewriting the same few lines of `rm -rf` code.

Here are a few sources of inspiration:
- https://github.community/t/bigger-github-hosted-runners-disk-space/17267/11
- https://github.com/apache/flink/blob/master/tools/azure-pipelines/free_disk_space.sh
- https://github.com/ShubhamTatvamasi/free-disk-space-action
- https://github.com/actions/virtual-environments/issues/2875#issuecomment-1163392159
- https://github.com/easimon/maximize-build-space/

## Typical Output

The amount of space storage saved by each option on an `ubuntu-20.04.4` runner is summarized here:
```
=> Android library: Saved 14GiB
=> .NET runtime: Saved 2.7GiB
=> Haskell runtime: Saved 0B
=> Large misc. packages: Saved 5.3GiB
=> Tool cache: Saved 5.9GiB
=> Swap storage: Saved 4.0GiB

Total: Saved 31GiB
```

The log output of a typical example (with all options set to `true`) looks like this:
```
================================================================================
BEFORE CLEAN-UP:

$ df -h /

Filesystem      Size  Used Avail Use% Mounted on
/dev/root        84G   53G   31G  64% /
$ df -a /

Filesystem     1K-blocks     Used Available Use% Mounted on
/dev/root       87218124 55405432  31796308  64% /
$ df -a

Filesystem     1K-blocks     Used Available Use% Mounted on
/dev/root       87218124 55405432  31796308  64% /
devtmpfs         3552992        0   3552992   0% /dev
sysfs                  0        0         0    - /sys
proc                   0        0         0    - /proc
securityfs             0        0         0    - /sys/kernel/security
tmpfs            3556564        4   3556560   1% /dev/shm
devpts                 0        0         0    - /dev/pts
tmpfs             711316     1068    710248   1% /run
tmpfs               5120        0      5120   0% /run/lock
tmpfs            3556564        0   3556564   0% /sys/fs/cgroup
cgroup2                0        0         0    - /sys/fs/cgroup/unified
cgroup                 0        0         0    - /sys/fs/cgroup/systemd
pstore                 0        0         0    - /sys/fs/pstore
none                   0        0         0    - /sys/fs/bpf
cgroup                 0        0         0    - /sys/fs/cgroup/cpuset
cgroup                 0        0         0    - /sys/fs/cgroup/pids
cgroup                 0        0         0    - /sys/fs/cgroup/devices
cgroup                 0        0         0    - /sys/fs/cgroup/hugetlb
cgroup                 0        0         0    - /sys/fs/cgroup/misc
cgroup                 0        0         0    - /sys/fs/cgroup/net_cls,net_prio
cgroup                 0        0         0    - /sys/fs/cgroup/blkio
cgroup                 0        0         0    - /sys/fs/cgroup/cpu,cpuacct
cgroup                 0        0         0    - /sys/fs/cgroup/perf_event
cgroup                 0        0         0    - /sys/fs/cgroup/memory
cgroup                 0        0         0    - /sys/fs/cgroup/rdma
cgroup                 0        0         0    - /sys/fs/cgroup/freezer
systemd-1              -        -         -    - /proc/sys/fs/binfmt_misc
hugetlbfs              0        0         0    - /dev/hugepages
mqueue                 0        0         0    - /dev/mqueue
debugfs                0        0         0    - /sys/kernel/debug
tracefs                0        0         0    - /sys/kernel/tracing
fusectl                0        0         0    - /sys/fs/fuse/connections
configfs               0        0         0    - /sys/kernel/config
/dev/loop0         63488    63488         0 100% /snap/core20/1518
/dev/loop1         69504    69504         0 100% /snap/lxd/22753
/dev/loop2         48128    48128         0 100% /snap/snapd/16010
/dev/sda15        106858     5321    101537   5% /boot/efi
binfmt_misc            0        0         0    - /proc/sys/fs/binfmt_misc
/dev/sdb1       14382088  4235296   9396508  32% /mnt
tmpfs             711316     1068    710248   1% /run/snapd/ns
nsfs                   0        0         0    - /run/snapd/ns/lxd.mnt
================================================================================


********************************************************************************
=> Android library: Saved 14GiB
********************************************************************************


********************************************************************************
=> .NET runtime: Saved 2.7GiB
********************************************************************************


********************************************************************************
=> Haskell runtime: Saved 0B
********************************************************************************

Reading package lists...
Building dependency tree...
Reading state information...
Package 'dotnet-nightly' is not installed, so not removed
Package 'dotnet-hostfxr-2.1' is not installed, so not removed
Package 'dotnet-runtime-2.1' is not installed, so not removed
Package 'dotnet-runtime-deps-2.1' is not installed, so not removed
Package 'dotnet-sdk-2.1' is not installed, so not removed
The following packages will be REMOVED:
  aspnetcore-runtime-3.1 aspnetcore-runtime-5.0 aspnetcore-runtime-6.0
  aspnetcore-targeting-pack-3.1 aspnetcore-targeting-pack-5.0
  aspnetcore-targeting-pack-6.0 dotnet-apphost-pack-3.1
  dotnet-apphost-pack-5.0 dotnet-apphost-pack-6.0 dotnet-host
  dotnet-hostfxr-3.1 dotnet-hostfxr-5.0 dotnet-hostfxr-6.0 dotnet-runtime-3.1
  dotnet-runtime-5.0 dotnet-runtime-6.0 dotnet-runtime-deps-3.1
  dotnet-runtime-deps-5.0 dotnet-runtime-deps-6.0 dotnet-sdk-3.1
  dotnet-sdk-5.0 dotnet-sdk-6.0 dotnet-targeting-pack-3.1
  dotnet-targeting-pack-5.0 dotnet-targeting-pack-6.0
0 upgraded, 0 newly installed, 25 to remove and 14 not upgraded.
After this operation, 1183 MB disk space will be freed.
(Reading database ... 
(Reading database ... 5%
(Reading database ... 10%
(Reading database ... 15%
(Reading database ... 20%
(Reading database ... 25%
(Reading database ... 30%
(Reading database ... 35%
(Reading database ... 40%
(Reading database ... 45%
(Reading database ... 50%
(Reading database ... 55%
(Reading database ... 60%
(Reading database ... 65%
(Reading database ... 70%
(Reading database ... 75%
(Reading database ... 80%
(Reading database ... 85%
(Reading database ... 90%
(Reading database ... 95%
(Reading database ... 100%
(Reading database ... 235534 files and directories currently installed.)
Removing dotnet-sdk-3.1 (3.1.420-1) ...
Removing aspnetcore-runtime-3.1 (3.1.26-1) ...
Removing dotnet-sdk-5.0 (5.0.408-1) ...
Removing aspnetcore-runtime-5.0 (5.0.17-1) ...
Removing dotnet-sdk-6.0 (6.0.301-1) ...
Removing aspnetcore-runtime-6.0 (6.0.6-1) ...
Removing aspnetcore-targeting-pack-3.1 (3.1.10-1) ...
Removing aspnetcore-targeting-pack-5.0 (5.0.0-1) ...
Removing aspnetcore-targeting-pack-6.0 (6.0.6-1) ...
Removing dotnet-apphost-pack-3.1 (3.1.26-1) ...
Removing dotnet-apphost-pack-5.0 (5.0.17-1) ...
Removing dotnet-apphost-pack-6.0 (6.0.6-1) ...
Removing dotnet-runtime-6.0 (6.0.6-1) ...
Removing dotnet-hostfxr-6.0 (6.0.6-1) ...
Removing dotnet-runtime-5.0 (5.0.17-1) ...
Removing dotnet-hostfxr-5.0 (5.0.17-1) ...
Removing dotnet-runtime-3.1 (3.1.26-1) ...
Removing dotnet-hostfxr-3.1 (3.1.26-1) ...
Removing dotnet-runtime-deps-3.1 (3.1.26-1) ...
Removing dotnet-runtime-deps-5.0 (5.0.17-1) ...
Removing dotnet-runtime-deps-6.0 (6.0.6-1) ...
Removing dotnet-targeting-pack-3.1 (3.1.0-1) ...
Removing dotnet-targeting-pack-5.0 (5.0.0-1) ...
Removing dotnet-targeting-pack-6.0 (6.0.6-1) ...
Removing dotnet-host (6.0.6-1) ...
Processing triggers for man-db (2.9.1-1) ...
Reading package lists...
Building dependency tree...
Reading state information...
Package 'llvm-13' is not installed, so not removed
Package 'llvm-10-doc' is not installed, so not removed
Package 'llvm-10-examples' is not installed, so not removed
Package 'llvm-6.0' is not installed, so not removed
Package 'llvm-6.0-dev' is not installed, so not removed
Package 'llvm-6.0-examples' is not installed, so not removed
Package 'llvm-6.0-runtime' is not installed, so not removed
Package 'llvm-6.0-tools' is not installed, so not removed
Package 'llvm-7' is not installed, so not removed
Package 'llvm-7-dev' is not installed, so not removed
Package 'llvm-7-examples' is not installed, so not removed
Package 'llvm-7-runtime' is not installed, so not removed
Package 'llvm-7-tools' is not installed, so not removed
Package 'llvm-8' is not installed, so not removed
Package 'llvm-8-dev' is not installed, so not removed
Package 'llvm-8-doc' is not installed, so not removed
Package 'llvm-8-examples' is not installed, so not removed
Package 'llvm-8-runtime' is not installed, so not removed
Package 'llvm-8-tools' is not installed, so not removed
Package 'llvm-9' is not installed, so not removed
Package 'llvm-9-dev' is not installed, so not removed
Package 'llvm-9-doc' is not installed, so not removed
Package 'llvm-9-examples' is not installed, so not removed
Package 'llvm-9-runtime' is not installed, so not removed
Package 'llvm-9-tools' is not installed, so not removed
Package 'llvm-dev' is not installed, so not removed
Package 'llvm-runtime' is not installed, so not removed
Package 'llvm-spirv' is not installed, so not removed
Package 'llvm-11-doc' is not installed, so not removed
Package 'llvm-11-examples' is not installed, so not removed
Package 'llvm-12-doc' is not installed, so not removed
Package 'llvm-12-examples' is not installed, so not removed
The following packages will be REMOVED:
  clang-12 clang-tidy-12 clang-tools-12 lldb-11 llvm-10 llvm-10-dev
  llvm-10-runtime llvm-10-tools llvm-11 llvm-11-dev llvm-11-runtime
  llvm-11-tools llvm-12 llvm-12-dev llvm-12-linker-tools llvm-12-runtime
  llvm-12-tools
0 upgraded, 0 newly installed, 17 to remove and 14 not upgraded.
After this operation, 804 MB disk space will be freed.
(Reading database ... 
(Reading database ... 5%
(Reading database ... 10%
(Reading database ... 15%
(Reading database ... 20%
(Reading database ... 25%
(Reading database ... 30%
(Reading database ... 35%
(Reading database ... 40%
(Reading database ... 45%
(Reading database ... 50%
(Reading database ... 55%
(Reading database ... 60%
(Reading database ... 65%
(Reading database ... 70%
(Reading database ... 75%
(Reading database ... 80%
(Reading database ... 85%
(Reading database ... 90%
(Reading database ... 95%
(Reading database ... 100%
(Reading database ... 225791 files and directories currently installed.)
Removing clang-tidy-12 (1:12.0.0-3ubuntu1~20.04.5) ...
Removing clang-tools-12 (1:12.0.0-3ubuntu1~20.04.5) ...
Removing clang-12 (1:12.0.0-3ubuntu1~20.04.5) ...
Removing lldb-11 (1:11.0.0-2~ubuntu20.04.1) ...
Removing llvm-10-dev (1:10.0.0-4ubuntu1) ...
Removing llvm-10 (1:10.0.0-4ubuntu1) ...
Removing llvm-10-runtime (1:10.0.0-4ubuntu1) ...
Removing llvm-10-tools (1:10.0.0-4ubuntu1) ...
Removing llvm-11-dev (1:11.0.0-2~ubuntu20.04.1) ...
Removing llvm-11 (1:11.0.0-2~ubuntu20.04.1) ...
Removing llvm-11-runtime (1:11.0.0-2~ubuntu20.04.1) ...
Removing llvm-11-tools (1:11.0.0-2~ubuntu20.04.1) ...
Removing llvm-12-dev (1:12.0.0-3ubuntu1~20.04.5) ...
Removing llvm-12 (1:12.0.0-3ubuntu1~20.04.5) ...
Removing llvm-12-linker-tools (1:12.0.0-3ubuntu1~20.04.5) ...
Removing llvm-12-runtime (1:12.0.0-3ubuntu1~20.04.5) ...
Removing llvm-12-tools (1:12.0.0-3ubuntu1~20.04.5) ...
Processing triggers for man-db (2.9.1-1) ...
Processing triggers for libc-bin (2.31-0ubuntu9.9) ...
Reading package lists...
Building dependency tree...
Reading state information...
Package 'phpgacl' is not installed, so not removed
Package 'php-crypt-gpg' is not installed, so not removed
Package 'libapache2-mod-php7.2' is not installed, so not removed
Package 'libapache2-mod-php7.3' is not installed, so not removed
Package 'php-smbclient' is not installed, so not removed
Package 'php5.6-common' is not installed, so not removed
Package 'php5.6-json' is not installed, so not removed
Package 'php7.0-common' is not installed, so not removed
Package 'php-pear-frontend-gtk' is not installed, so not removed
Package 'php-pear-frontend-web' is not installed, so not removed
Package 'php7.0-curl' is not installed, so not removed
Package 'php7.2-sodium' is not installed, so not removed
Package 'php5-cli' is not installed, so not removed
Package 'php5-curl' is not installed, so not removed
Package 'php-mcrypt' is not installed, so not removed
Package 'php5-mysql' is not installed, so not removed
Package 'php5-pgsql' is not installed, so not removed
Package 'php5-sqlite' is not installed, so not removed
Package 'php5' is not installed, so not removed
Package 'php5-ldap' is not installed, so not removed
Package 'php-apc' is not installed, so not removed
Package 'php-suhosin' is not installed, so not removed
Package 'php5-gd' is not installed, so not removed
Package 'php5-imagick' is not installed, so not removed
Package 'php5-json' is not installed, so not removed
Package 'libapache2-mod-php5' is not installed, so not removed
Package 'php5-fpm' is not installed, so not removed
Package 'php5-mcrypt' is not installed, so not removed
Package 'libapache2-mod-suphp' is not installed, so not removed
Package 'libgv-php5' is not installed, so not removed
Package 'libow-php5' is not installed, so not removed
Package 'phpphylotree' is not installed, so not removed
Package 'php-xcache' is not installed, so not removed
Package 'php-console-color2' is not installed, so not removed
Package 'php-alcaeus-mongo-php-adapter' is not installed, so not removed
Package 'php-doctrine-phpcr-odm' is not installed, so not removed
Package 'php-doctrine-mongodb-odm' is not installed, so not removed
Package 'php-ml-json-ld' is not installed, so not removed
Package 'php-com-dotnet' is not installed, so not removed
Package 'php-rar' is not installed, so not removed
Package 'php-cordoval-hamcrest-php' is not installed, so not removed
Package 'php-davedevelopment-hamcrest-php' is not installed, so not removed
Package 'php-kodova-hamcrest-php' is not installed, so not removed
Package 'php-fzaninotto-faker' is not installed, so not removed
Package 'php-illuminate-console' is not installed, so not removed
Package 'php-illuminate-events' is not installed, so not removed
Package 'php-illuminate-filesystem' is not installed, so not removed
Package 'php-illuminate-pagination' is not installed, so not removed
Package 'php-tightenco-collect' is not installed, so not removed
Package 'php-moontoast-math' is not installed, so not removed
Package 'php-ramsey-uuid' is not installed, so not removed
Package 'php-vlucas-phpdotenv' is not installed, so not removed
Package 'php-scrutinizer-ocular' is not installed, so not removed
Package 'php-sqlite' is not installed, so not removed
Package 'php-mdb2-driver-fbsql' is not installed, so not removed
Package 'php-mdb2-driver-ibase' is not installed, so not removed
Package 'php-mdb2-driver-mssql' is not installed, so not removed
Package 'php-mdb2-driver-mysqli' is not installed, so not removed
Package 'php-mdb2-driver-oci8' is not installed, so not removed
Package 'php-mdb2-driver-odbc' is not installed, so not removed
Package 'php-mdb2-driver-querysim' is not installed, so not removed
Package 'php-mdb2-driver-sqlite' is not installed, so not removed
Package 'php-mdb2-driver-sqlsrv' is not installed, so not removed
Package 'php-barnabywalters-mf-cleaner' is not installed, so not removed
Package 'php-graylog2-gelf-php' is not installed, so not removed
Package 'php-sentry' is not installed, so not removed
Package 'php-doctrine-couchdb' is not installed, so not removed
Package 'php-ruflin-elastica' is not installed, so not removed
Package 'php-mongo' is not installed, so not removed
Package 'php-aws-sdk' is not installed, so not removed
Package 'php-rollbar' is not installed, so not removed
Package 'php-console' is not installed, so not removed
Package 'php-wfio' is not installed, so not removed
Package 'php-dbase' is not installed, so not removed
Package 'php-libsodium' is not installed, so not removed
Package 'php-ocramius-generated-hydrator' is not installed, so not removed
Package 'php-zend-xmlrpc' is not installed, so not removed
Package 'php-zend-json' is not installed, so not removed
Package 'php-zend-soap' is not installed, so not removed
Package 'php-reactivex-rxphp' is not installed, so not removed
Package 'php-event' is not installed, so not removed
Package 'php-egulias-email-validator' is not installed, so not removed
Package 'php-zendframework-zend-validator' is not installed, so not removed
Package 'php-friendsofphp-php-cs-fixer' is not installed, so not removed
Package 'php-crypt-blowfish' is not installed, so not removed
Package 'php-compat' is not installed, so not removed
Package 'php-cache' is not installed, so not removed
Package 'php-xml-serializer' is not installed, so not removed
Package 'libssh2-php' is not installed, so not removed
Package 'php-enqueue-messenger-adapter' is not installed, so not removed
Package 'php-symfony-class-loader' is not installed, so not removed
Package 'php-numbers-words' is not installed, so not removed
Package 'php7.0-thrift' is not installed, so not removed
Package 'php7.2-thrift' is not installed, so not removed
Package 'php-net-idna' is not installed, so not removed
Package 'php-vimeo-psalm' is not installed, so not removed
Package 'php-container-interop' is not installed, so not removed
Package 'php-recode' is not installed, so not removed
Package 'php-gd2' is not installed, so not removed
Package 'php-pragmarx-google2fa' is not installed, so not removed
Package 'php-bacon-qr-code' is not installed, so not removed
Package 'php-samyoul-u2f-php-server' is not installed, so not removed
Package 'php5-xsl' is not installed, so not removed
Package 'php-uopz' is not installed, so not removed
Package 'php-mkopinsky-zxcvbn-php' is not installed, so not removed
Package 'php5-mysqlnd' is not installed, so not removed
Package 'libapache2-mod-php8.0' is not installed, so not removed
Package 'libapache2-mod-php8.1' is not installed, so not removed
Package 'libapache2-mod-php' is not installed, so not removed
Package 'php' is not installed, so not removed
Package 'php-all-dev' is not installed, so not removed
Package 'php-cgi' is not installed, so not removed
Package 'php-cli' is not installed, so not removed
Package 'php-curl' is not installed, so not removed
Package 'php-dev' is not installed, so not removed
Package 'php-gd' is not installed, so not removed
Package 'php-gmp' is not installed, so not removed
Package 'php-ldap' is not installed, so not removed
Package 'php-mysql' is not installed, so not removed
Package 'php-odbc' is not installed, so not removed
Package 'php-pgsql' is not installed, so not removed
Package 'php-pspell' is not installed, so not removed
Package 'php-snmp' is not installed, so not removed
Package 'php-sqlite3' is not installed, so not removed
Package 'php-tidy' is not installed, so not removed
Package 'php-xml' is not installed, so not removed
Package 'php-xmlrpc' is not installed, so not removed
Package 'pkg-php-tools' is not installed, so not removed
Package 'cakephp' is not installed, so not removed
Package 'cakephp-scripts' is not installed, so not removed
Package 'dh-php' is not installed, so not removed
Package 'elpa-php-mode' is not installed, so not removed
Package 'golang-github-phpdave11-gofpdi-dev' is not installed, so not removed
Package 'gosa-plugin-phpgw' is not installed, so not removed
Package 'gosa-plugin-phpgw-schema' is not installed, so not removed
Package 'gosa-plugin-phpscheduleit' is not installed, so not removed
Package 'gosa-plugin-phpscheduleit-schema' is not installed, so not removed
Package 'kdevelop-php' is not installed, so not removed
Package 'kdevelop-php-l10n' is not installed, so not removed
Package 'libgv-php7' is not installed, so not removed
Package 'libhtml-wikiconverter-phpwiki-perl' is not installed, so not removed
Package 'libmarkdown-php' is not installed, so not removed
Package 'libnusoap-php' is not installed, so not removed
Package 'libow-php7' is not installed, so not removed
Package 'libownet-php' is not installed, so not removed
Package 'libphp-adodb' is not installed, so not removed
Package 'libphp-embed' is not installed, so not removed
Package 'libphp-jabber' is not installed, so not removed
Package 'libphp-jpgraph' is not installed, so not removed
Package 'libphp-jpgraph-examples' is not installed, so not removed
Package 'libphp-magpierss' is not installed, so not removed
Package 'libphp-phpmailer' is not installed, so not removed
Package 'libphp-predis' is not installed, so not removed
Package 'libphp-serialization-perl' is not installed, so not removed
Package 'libphp-simplepie' is not installed, so not removed
Package 'libphp-snoopy' is not installed, so not removed
Package 'libphp-swiftmailer' is not installed, so not removed
Package 'libsparkline-php' is not installed, so not removed
Package 'mlmmj-php-web' is not installed, so not removed
Package 'mlmmj-php-web-admin' is not installed, so not removed
Package 'php-amqp' is not installed, so not removed
Package 'php-amqplib' is not installed, so not removed
Package 'php-apcu' is not installed, so not removed
Package 'php-apcu-bc' is not installed, so not removed
Package 'php-ast' is not installed, so not removed
Package 'php-auth-sasl' is not installed, so not removed
Package 'php-bcmath' is not installed, so not removed
Package 'php-bz2' is not installed, so not removed
Package 'php-cache-integration-tests' is not installed, so not removed
Package 'php-cache-lite' is not installed, so not removed
Package 'php-cache-tag-interop' is not installed, so not removed
Package 'php-cas' is not installed, so not removed
Package 'php-cboden-ratchet' is not installed, so not removed
Package 'php-cocur-slugify' is not installed, so not removed
Package 'php-codecoverage' is not installed, so not removed
Package 'php-codesniffer' is not installed, so not removed
Package 'php-composer-ca-bundle' is not installed, so not removed
Package 'php-composer-semver' is not installed, so not removed
Package 'php-composer-spdx-licenses' is not installed, so not removed
Package 'php-composer-xdebug-handler' is not installed, so not removed
Package 'php-console-commandline' is not installed, so not removed
Package 'php-console-table' is not installed, so not removed
Package 'php-constant-time' is not installed, so not removed
Package 'php-date' is not installed, so not removed
Package 'php-db' is not installed, so not removed
Package 'php-db-dataobject' is not installed, so not removed
Package 'php-deepcopy' is not installed, so not removed
Package 'php-defuse-php-encryption' is not installed, so not removed
Package 'php-dflydev-fig-cookies' is not installed, so not removed
Package 'php-directory-scanner' is not installed, so not removed
Package 'php-doctrine-annotations' is not installed, so not removed
Package 'php-doctrine-bundle' is not installed, so not removed
Package 'php-doctrine-cache' is not installed, so not removed
Package 'php-doctrine-collections' is not installed, so not removed
Package 'php-doctrine-common' is not installed, so not removed
Package 'php-doctrine-data-fixtures' is not installed, so not removed
Package 'php-doctrine-dbal' is not installed, so not removed
Package 'php-doctrine-event-manager' is not installed, so not removed
Package 'php-doctrine-inflector' is not installed, so not removed
Package 'php-doctrine-instantiator' is not installed, so not removed
Package 'php-doctrine-lexer' is not installed, so not removed
Package 'php-doctrine-orm' is not installed, so not removed
Package 'php-doctrine-persistence' is not installed, so not removed
Package 'php-doctrine-reflection' is not installed, so not removed
Package 'php-dompdf' is not installed, so not removed
Package 'php-ds' is not installed, so not removed
Package 'php-easyrdf' is not installed, so not removed
Package 'php-email-validator' is not installed, so not removed
Package 'php-embed' is not installed, so not removed
Package 'php-enchant' is not installed, so not removed
Package 'php-evenement' is not installed, so not removed
Package 'php-excimer' is not installed, so not removed
Package 'php-fabiang-sasl' is not installed, so not removed
Package 'php-fdomdocument' is not installed, so not removed
Package 'php-fig-link-util' is not installed, so not removed
Package 'php-file-iterator' is not installed, so not removed
Package 'php-finder-facade' is not installed, so not removed
Package 'php-finder-facade-doc' is not installed, so not removed
Package 'php-font-lib' is not installed, so not removed
Package 'php-fpdf' is not installed, so not removed
Package 'php-fpm' is not installed, so not removed
Package 'php-fxsl' is not installed, so not removed
Package 'php-gearman' is not installed, so not removed
Package 'php-geoip' is not installed, so not removed
Package 'php-geos' is not installed, so not removed
Package 'php-geshi' is not installed, so not removed
Package 'php-getid3' is not installed, so not removed
Package 'php-gmagick' is not installed, so not removed
Package 'php-gnupg' is not installed, so not removed
Package 'php-google-recaptcha' is not installed, so not removed
Package 'php-guestfs' is not installed, so not removed
Package 'php-guzzlehttp-promises' is not installed, so not removed
Package 'php-guzzlehttp-psr7' is not installed, so not removed
Package 'php-hamcrest' is not installed, so not removed
Package 'php-htmlawed' is not installed, so not removed
Package 'php-htmlpurifier' is not installed, so not removed
Package 'php-http' is not installed, so not removed
Package 'php-http-httplug' is not installed, so not removed
Package 'php-http-message-factory' is not installed, so not removed
Package 'php-http-promise' is not installed, so not removed
Package 'php-http-psr7-integration-tests' is not installed, so not removed
Package 'php-http-request' is not installed, so not removed
Package 'php-http-request2' is not installed, so not removed
Package 'php-http-webdav-server' is not installed, so not removed
Package 'php-httpful' is not installed, so not removed
Package 'php-icinga' is not installed, so not removed
Package 'php-igbinary' is not installed, so not removed
Package 'php-illuminate-container' is not installed, so not removed
Package 'php-illuminate-contracts' is not installed, so not removed
Package 'php-illuminate-database' is not installed, so not removed
Package 'php-illuminate-support' is not installed, so not removed
Package 'php-image-text' is not installed, so not removed
Package 'php-imagick' is not installed, so not removed
Package 'php-imap' is not installed, so not removed
Package 'php-interbase' is not installed, so not removed
Package 'php-intl' is not installed, so not removed
Package 'php-invoker' is not installed, so not removed
Package 'php-json' is not installed, so not removed
Package 'php-json-schema' is not installed, so not removed
Package 'php-klogger' is not installed, so not removed
Package 'php-league-commonmark' is not installed, so not removed
Package 'php-league-html-to-markdown' is not installed, so not removed
Package 'php-letodms-core' is not installed, so not removed
Package 'php-libvirt-php' is not installed, so not removed
Package 'php-log' is not installed, so not removed
Package 'php-lorenzo-pinky' is not installed, so not removed
Package 'php-lua' is not installed, so not removed
Package 'php-luasandbox' is not installed, so not removed
Package 'php-mail' is not installed, so not removed
Package 'php-mail-mime' is not installed, so not removed
Package 'php-mailparse' is not installed, so not removed
Package 'php-mapi' is not installed, so not removed
Package 'php-mapscript' is not installed, so not removed
Package 'php-mapscript-ng' is not installed, so not removed
Package 'php-markdown' is not installed, so not removed
Package 'php-masterminds-html5' is not installed, so not removed
Package 'php-mbstring' is not installed, so not removed
Package 'php-mdb2' is not installed, so not removed
Package 'php-mdb2-driver-mysql' is not installed, so not removed
Package 'php-mdb2-driver-pgsql' is not installed, so not removed
Package 'php-memcache' is not installed, so not removed
Package 'php-memcached' is not installed, so not removed
Package 'php-mf2' is not installed, so not removed
Package 'php-mikey179-vfsstream' is not installed, so not removed
Package 'php-mime-type' is not installed, so not removed
Package 'php-mockery' is not installed, so not removed
Package 'php-mockery-doc' is not installed, so not removed
Package 'php-mongodb' is not installed, so not removed
Package 'php-monolog' is not installed, so not removed
Package 'php-msgpack' is not installed, so not removed
Package 'php-nesbot-carbon' is not installed, so not removed
Package 'php-net-dime' is not installed, so not removed
Package 'php-net-dns2' is not installed, so not removed
Package 'php-net-ftp' is not installed, so not removed
Package 'php-net-idna2' is not installed, so not removed
Package 'php-net-imap' is not installed, so not removed
Package 'php-net-ipv6' is not installed, so not removed
Package 'php-net-ldap2' is not installed, so not removed
Package 'php-net-ldap3' is not installed, so not removed
Package 'php-net-nntp' is not installed, so not removed
Package 'php-net-publicsuffix' is not installed, so not removed
Package 'php-net-sieve' is not installed, so not removed
Package 'php-net-smtp' is not installed, so not removed
Package 'php-net-socket' is not installed, so not removed
Package 'php-net-url' is not installed, so not removed
Package 'php-net-url2' is not installed, so not removed
Package 'php-net-whois' is not installed, so not removed
Package 'php-netscape-bookmark-parser' is not installed, so not removed
Package 'php-nikic-fast-route' is not installed, so not removed
Package 'php-nrk-predis' is not installed, so not removed
Package 'php-nyholm-psr7' is not installed, so not removed
Package 'php-oauth' is not installed, so not removed
Package 'php-parsedown' is not installed, so not removed
Package 'php-parser' is not installed, so not removed
Package 'php-patchwork-utf8' is not installed, so not removed
Package 'php-pclzip' is not installed, so not removed
Package 'php-pcov' is not installed, so not removed
Package 'php-pecl-http' is not installed, so not removed
Package 'php-pecl-http-dev' is not installed, so not removed
Package 'php-phar-io-manifest' is not installed, so not removed
Package 'php-phar-io-version' is not installed, so not removed
Package 'php-phpdbg' is not installed, so not removed
Package 'php-phpdocumentor-reflection-common' is not installed, so not removed
Package 'php-phpdocumentor-reflection-docblock' is not installed, so not removed
Package 'php-phpdocumentor-type-resolver' is not installed, so not removed
Package 'php-phpmyadmin-motranslator' is not installed, so not removed
Package 'php-phpmyadmin-shapefile' is not installed, so not removed
Package 'php-phpmyadmin-sql-parser' is not installed, so not removed
Package 'php-phpseclib' is not installed, so not removed
Package 'php-phpspec-prophecy' is not installed, so not removed
Package 'php-pimple' is not installed, so not removed
Package 'php-pinba' is not installed, so not removed
Package 'php-propro' is not installed, so not removed
Package 'php-propro-dev' is not installed, so not removed
Package 'php-proxy-manager' is not installed, so not removed
Package 'php-ps' is not installed, so not removed
Package 'php-psr' is not installed, so not removed
Package 'php-psr-cache' is not installed, so not removed
Package 'php-psr-container' is not installed, so not removed
Package 'php-psr-event-dispatcher' is not installed, so not removed
Package 'php-psr-http-client' is not installed, so not removed
Package 'php-psr-http-factory' is not installed, so not removed
Package 'php-psr-http-message' is not installed, so not removed
Package 'php-psr-link' is not installed, so not removed
Package 'php-psr-log' is not installed, so not removed
Package 'php-psr-simple-cache' is not installed, so not removed
Package 'php-pubsubhubbub-publisher' is not installed, so not removed
Package 'php-radius' is not installed, so not removed
Package 'php-raintpl' is not installed, so not removed
Package 'php-random-compat' is not installed, so not removed
Package 'php-raphf' is not installed, so not removed
Package 'php-raphf-dev' is not installed, so not removed
Package 'php-ratchet-pawl' is not installed, so not removed
Package 'php-ratchet-rfc6455' is not installed, so not removed
Package 'php-react-cache' is not installed, so not removed
Package 'php-react-child-process' is not installed, so not removed
Package 'php-react-dns' is not installed, so not removed
Package 'php-react-event-loop' is not installed, so not removed
Package 'php-react-http' is not installed, so not removed
Package 'php-react-promise' is not installed, so not removed
Package 'php-react-promise-stream' is not installed, so not removed
Package 'php-react-promise-timer' is not installed, so not removed
Package 'php-react-socket' is not installed, so not removed
Package 'php-react-stream' is not installed, so not removed
Package 'php-readline' is not installed, so not removed
Package 'php-redis' is not installed, so not removed
Package 'php-remctl' is not installed, so not removed
Package 'php-respect-validation' is not installed, so not removed
Package 'php-robmorgan-phinx' is not installed, so not removed
Package 'php-rrd' is not installed, so not removed
Package 'php-sabre-vobject' is not installed, so not removed
Package 'php-sass' is not installed, so not removed
Package 'php-seclib' is not installed, so not removed
Package 'php-services-json' is not installed, so not removed
Package 'php-services-weather' is not installed, so not removed
Package 'php-shellcommand' is not installed, so not removed
Package 'php-soap' is not installed, so not removed
Package 'php-solr' is not installed, so not removed
Package 'php-sql-formatter' is not installed, so not removed
Package 'php-ssh2' is not installed, so not removed
Package 'php-stomp' is not installed, so not removed
Package 'php-swiftmailer' is not installed, so not removed
Package 'php-sybase' is not installed, so not removed
Package 'php-symfony' is not installed, so not removed
Package 'php-symfony-amazon-mailer' is not installed, so not removed
Package 'php-symfony-asset' is not installed, so not removed
Package 'php-symfony-browser-kit' is not installed, so not removed
Package 'php-symfony-cache' is not installed, so not removed
Package 'php-symfony-cache-contracts' is not installed, so not removed
Package 'php-symfony-config' is not installed, so not removed
Package 'php-symfony-console' is not installed, so not removed
Package 'php-symfony-contracts' is not installed, so not removed
Package 'php-symfony-css-selector' is not installed, so not removed
Package 'php-symfony-debug' is not installed, so not removed
Package 'php-symfony-debug-bundle' is not installed, so not removed
Package 'php-symfony-dependency-injection' is not installed, so not removed
Package 'php-symfony-doctrine-bridge' is not installed, so not removed
Package 'php-symfony-dom-crawler' is not installed, so not removed
Package 'php-symfony-dotenv' is not installed, so not removed
Package 'php-symfony-event-dispatcher' is not installed, so not removed
Package 'php-symfony-event-dispatcher-contracts' is not installed, so not removed
Package 'php-symfony-expression-language' is not installed, so not removed
Package 'php-symfony-filesystem' is not installed, so not removed
Package 'php-symfony-finder' is not installed, so not removed
Package 'php-symfony-form' is not installed, so not removed
Package 'php-symfony-framework-bundle' is not installed, so not removed
Package 'php-symfony-google-mailer' is not installed, so not removed
Package 'php-symfony-http-client' is not installed, so not removed
Package 'php-symfony-http-client-contracts' is not installed, so not removed
Package 'php-symfony-http-foundation' is not installed, so not removed
Package 'php-symfony-http-kernel' is not installed, so not removed
Package 'php-symfony-inflector' is not installed, so not removed
Package 'php-symfony-intl' is not installed, so not removed
Package 'php-symfony-ldap' is not installed, so not removed
Package 'php-symfony-lock' is not installed, so not removed
Package 'php-symfony-mailchimp-mailer' is not installed, so not removed
Package 'php-symfony-mailer' is not installed, so not removed
Package 'php-symfony-mailgun-mailer' is not installed, so not removed
Package 'php-symfony-messenger' is not installed, so not removed
Package 'php-symfony-mime' is not installed, so not removed
Package 'php-symfony-monolog-bridge' is not installed, so not removed
Package 'php-symfony-options-resolver' is not installed, so not removed
Package 'php-symfony-phpunit-bridge' is not installed, so not removed
Package 'php-symfony-postmark-mailer' is not installed, so not removed
Package 'php-symfony-process' is not installed, so not removed
Package 'php-symfony-property-access' is not installed, so not removed
Package 'php-symfony-property-info' is not installed, so not removed
Package 'php-symfony-proxy-manager-bridge' is not installed, so not removed
Package 'php-symfony-routing' is not installed, so not removed
Package 'php-symfony-security' is not installed, so not removed
Package 'php-symfony-security-acl' is not installed, so not removed
Package 'php-symfony-security-bundle' is not installed, so not removed
Package 'php-symfony-security-core' is not installed, so not removed
Package 'php-symfony-security-csrf' is not installed, so not removed
Package 'php-symfony-security-guard' is not installed, so not removed
Package 'php-symfony-security-http' is not installed, so not removed
Package 'php-symfony-sendgrid-mailer' is not installed, so not removed
Package 'php-symfony-serializer' is not installed, so not removed
Package 'php-symfony-service-contracts' is not installed, so not removed
Package 'php-symfony-stopwatch' is not installed, so not removed
Package 'php-symfony-templating' is not installed, so not removed
Package 'php-symfony-translation' is not installed, so not removed
Package 'php-symfony-translation-contracts' is not installed, so not removed
Package 'php-symfony-twig-bridge' is not installed, so not removed
Package 'php-symfony-twig-bundle' is not installed, so not removed
Package 'php-symfony-validator' is not installed, so not removed
Package 'php-symfony-var-dumper' is not installed, so not removed
Package 'php-symfony-var-exporter' is not installed, so not removed
Package 'php-symfony-web-link' is not installed, so not removed
Package 'php-symfony-web-profiler-bundle' is not installed, so not removed
Package 'php-symfony-web-server-bundle' is not installed, so not removed
Package 'php-symfony-workflow' is not installed, so not removed
Package 'php-symfony-yaml' is not installed, so not removed
Package 'php-tcpdf' is not installed, so not removed
Package 'php-text-captcha' is not installed, so not removed
Package 'php-text-figlet' is not installed, so not removed
Package 'php-text-languagedetect' is not installed, so not removed
Package 'php-text-password' is not installed, so not removed
Package 'php-text-template' is not installed, so not removed
Package 'php-text-wiki' is not installed, so not removed
Package 'php-thrift' is not installed, so not removed
Package 'php-tideways' is not installed, so not removed
Package 'php-tijsverkoyen-css-to-inline-styles' is not installed, so not removed
Package 'php-timer' is not installed, so not removed
Package 'php-token-stream' is not installed, so not removed
Package 'php-tokenizer' is not installed, so not removed
Package 'php-twig' is not installed, so not removed
Package 'php-twig-cssinliner-extra' is not installed, so not removed
Package 'php-twig-doc' is not installed, so not removed
Package 'php-twig-extensions' is not installed, so not removed
Package 'php-twig-extra-bundle' is not installed, so not removed
Package 'php-twig-html-extra' is not installed, so not removed
Package 'php-twig-inky-extra' is not installed, so not removed
Package 'php-twig-intl-extra' is not installed, so not removed
Package 'php-twig-markdown-extra' is not installed, so not removed
Package 'php-uploadprogress' is not installed, so not removed
Package 'php-uuid' is not installed, so not removed
Package 'php-validate' is not installed, so not removed
Package 'php-webmozart-assert' is not installed, so not removed
Package 'php-wikidiff2' is not installed, so not removed
Package 'php-wmerrors' is not installed, so not removed
Package 'php-xajax' is not installed, so not removed
Package 'php-xdebug' is not installed, so not removed
Package 'php-xml-htmlsax3' is not installed, so not removed
Package 'php-xml-rpc2' is not installed, so not removed
Package 'php-xml-svg' is not installed, so not removed
Package 'php-yac' is not installed, so not removed
Package 'php-yaml' is not installed, so not removed
Package 'php-zend-code' is not installed, so not removed
Package 'php-zend-eventmanager' is not installed, so not removed
Package 'php-zend-stdlib' is not installed, so not removed
Package 'php-zeroc-ice' is not installed, so not removed
Package 'php-zeta-base' is not installed, so not removed
Package 'php-zeta-console-tools' is not installed, so not removed
Package 'php-zeta-unit-test' is not installed, so not removed
Package 'php-zip' is not installed, so not removed
Package 'php-zmq' is not installed, so not removed
Package 'phpab' is not installed, so not removed
Package 'phpcpd' is not installed, so not removed
Package 'phpdox' is not installed, so not removed
Package 'phpldapadmin' is not installed, so not removed
Package 'phpliteadmin' is not installed, so not removed
Package 'phpliteadmin-themes' is not installed, so not removed
Package 'phploc' is not installed, so not removed
Package 'phpmd' is not installed, so not removed
Package 'phpmyadmin' is not installed, so not removed
Package 'phppgadmin' is not installed, so not removed
Package 'phpqrcode' is not installed, so not removed
Package 'phpsysinfo' is not installed, so not removed
Package 'phpunit' is not installed, so not removed
Package 'phpunit-code-unit-reverse-lookup' is not installed, so not removed
Package 'phpunit-comparator' is not installed, so not removed
Package 'phpunit-diff' is not installed, so not removed
Package 'phpunit-environment' is not installed, so not removed
Package 'phpunit-exporter' is not installed, so not removed
Package 'phpunit-git' is not installed, so not removed
Package 'phpunit-global-state' is not installed, so not removed
Package 'phpunit-object-enumerator' is not installed, so not removed
Package 'phpunit-object-reflector' is not installed, so not removed
Package 'phpunit-recursion-context' is not installed, so not removed
Package 'phpunit-resource-operations' is not installed, so not removed
Package 'phpunit-type' is not installed, so not removed
Package 'phpunit-version' is not installed, so not removed
Package 'phpwebcounter' is not installed, so not removed
Package 'phpwebcounter-extra' is not installed, so not removed
Package 'python3-phply' is not installed, so not removed
Package 'python3-phpserialize' is not installed, so not removed
Package 'simplesamlphp' is not installed, so not removed
Package 'slbackup-php' is not installed, so not removed
Package 'uphpmvault' is not installed, so not removed
Package 'uwsgi-plugin-php' is not installed, so not removed
Package 'weechat-php' is not installed, so not removed
Package 'zabbix-frontend-php' is not installed, so not removed
Package 'php-mythtv' is not installed, so not removed
Package 'libapache2-mod-php7.4' is not installed, so not removed
Package 'libawl-php' is not installed, so not removed
Package 'libphp7.4-embed' is not installed, so not removed
The following packages will be REMOVED:
  php-common php-pear php7.4 php7.4-amqp php7.4-apcu php7.4-bcmath php7.4-bz2
  php7.4-cgi php7.4-cli php7.4-common php7.4-curl php7.4-dba php7.4-dev
  php7.4-enchant php7.4-fpm php7.4-gd php7.4-gmp php7.4-igbinary
  php7.4-imagick php7.4-imap php7.4-interbase php7.4-intl php7.4-json
  php7.4-ldap php7.4-mbstring php7.4-memcache php7.4-memcached php7.4-mongodb
  php7.4-msgpack php7.4-mysql php7.4-odbc php7.4-opcache php7.4-pcov
  php7.4-pgsql php7.4-phpdbg php7.4-pspell php7.4-readline php7.4-redis
  php7.4-snmp php7.4-soap php7.4-sqlite3 php7.4-sybase php7.4-tidy
  php7.4-xdebug php7.4-xml php7.4-xmlrpc php7.4-xsl php7.4-yaml php7.4-zip
  php7.4-zmq php8.0 php8.0-amqp php8.0-apcu php8.0-bcmath php8.0-bz2
  php8.0-cgi php8.0-cli php8.0-common php8.0-curl php8.0-dba php8.0-dev
  php8.0-enchant php8.0-fpm php8.0-gd php8.0-gmp php8.0-igbinary
  php8.0-imagick php8.0-imap php8.0-interbase php8.0-intl php8.0-ldap
  php8.0-mbstring php8.0-memcache php8.0-memcached php8.0-mongodb
  php8.0-msgpack php8.0-mysql php8.0-odbc php8.0-opcache php8.0-pcov
  php8.0-pgsql php8.0-phpdbg php8.0-pspell php8.0-readline php8.0-redis
  php8.0-snmp php8.0-soap php8.0-sqlite3 php8.0-sybase php8.0-tidy
  php8.0-xdebug php8.0-xml php8.0-xsl php8.0-yaml php8.0-zip php8.0-zmq php8.1
  php8.1-amqp php8.1-apcu php8.1-bcmath php8.1-bz2 php8.1-cgi php8.1-cli
  php8.1-common php8.1-curl php8.1-dba php8.1-dev php8.1-enchant php8.1-fpm
  php8.1-gd php8.1-gmp php8.1-igbinary php8.1-imagick php8.1-imap
  php8.1-interbase php8.1-intl php8.1-ldap php8.1-mbstring php8.1-memcache
  php8.1-memcached php8.1-mongodb php8.1-msgpack php8.1-mysql php8.1-odbc
  php8.1-opcache php8.1-pcov php8.1-pgsql php8.1-phpdbg php8.1-pspell
  php8.1-readline php8.1-redis php8.1-snmp php8.1-soap php8.1-sqlite3
  php8.1-sybase php8.1-tidy php8.1-xdebug php8.1-xml php8.1-xsl php8.1-yaml
  php8.1-zip php8.1-zmq
0 upgraded, 0 newly installed, 142 to remove and 14 not upgraded.
After this operation, 158 MB disk space will be freed.
(Reading database ... 
(Reading database ... 5%
(Reading database ... 10%
(Reading database ... 15%
(Reading database ... 20%
(Reading database ... 25%
(Reading database ... 30%
(Reading database ... 35%
(Reading database ... 40%
(Reading database ... 45%
(Reading database ... 50%
(Reading database ... 55%
(Reading database ... 60%
(Reading database ... 65%
(Reading database ... 70%
(Reading database ... 75%
(Reading database ... 80%
(Reading database ... 85%
(Reading database ... 90%
(Reading database ... 95%
(Reading database ... 100%
(Reading database ... 218380 files and directories currently installed.)
Removing php7.4-bcmath (1:7.4.30-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.1-zmq (1.1.3-23+ubuntu20.04.1+deb.sury.org+10) ...
Removing php-pear (1:1.10.13+submodules+notgz+2022032202-2+ubuntu20.04.1+deb.sury.org+1) ...
Removing php7.4 (1:7.4.30-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php7.4-amqp (1.11.0-4+ubuntu20.04.1+deb.sury.org+10) ...
Removing php7.4-apcu (5.1.21+4.0.11-7+ubuntu20.04.1+deb.sury.org+10) ...
Removing php7.4-bz2 (1:7.4.30-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php7.4-zmq (1.1.3-23+ubuntu20.04.1+deb.sury.org+10) ...
Removing php7.4-yaml (2.2.2+2.1.0+2.0.4+1.3.2-5+ubuntu20.04.1+deb.sury.org+10) ...
Removing php7.4-cgi (1:7.4.30-1+ubuntu20.04.1+deb.sury.org+1) ...
apache2_invoke php7.4-cgi prerm: No action required
Removing php7.4-dev (1:7.4.30-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php7.4-xdebug (3.1.2+2.9.8+2.8.1+2.5.5-4+ubuntu20.04.1+deb.sury.org+10) ...
Removing php7.4-phpdbg (1:7.4.30-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php7.4-redis (5.3.6+4.3.0-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php7.4-zip (1:7.4.30-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php7.4-xsl (1:7.4.30-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php7.4-curl (1:7.4.30-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php7.4-dba (1:7.4.30-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php7.4-enchant (1:7.4.30-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php7.4-pcov (1.0.11-4+ubuntu20.04.1+deb.sury.org+10) ...
Removing php7.4-fpm (1:7.4.30-1+ubuntu20.04.1+deb.sury.org+1) ...
apache2_invoke php7.4-fpm prerm: No action required
Removing php7.4-gd (1:7.4.30-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php7.4-gmp (1:7.4.30-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php7.4-memcached (3.2.0+2.2.0-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php7.4-igbinary (3.2.6+2.0.8-6+ubuntu20.04.1+deb.sury.org+10) ...
Removing php7.4-imagick (3.6.0-4+ubuntu20.04.1+deb.sury.org+10) ...
Removing php7.4-imap (1:7.4.30-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php7.4-interbase (1:7.4.30-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php7.4-intl (1:7.4.30-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php7.4-ldap (1:7.4.30-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php7.4-mbstring (1:7.4.30-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php7.4-memcache (8.0+4.0.5.2+3.0.9~20170802.e702b5f9+-7+ubuntu20.04.1+deb.sury.org+10) ...
Removing php7.4-mongodb (1.13.0+1.9.2+1.7.5-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php7.4-msgpack (2.2.0~rc1+2.1.2+0.5.7-6+ubuntu20.04.1+deb.sury.org+10) ...
Removing php7.4-mysql (1:7.4.30-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php7.4-odbc (1:7.4.30-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php7.4-pgsql (1:7.4.30-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php7.4-pspell (1:7.4.30-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php7.4-snmp (1:7.4.30-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php7.4-soap (1:7.4.30-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php7.4-sqlite3 (1:7.4.30-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php7.4-sybase (1:7.4.30-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php7.4-tidy (1:7.4.30-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php7.4-xml (1:7.4.30-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php7.4-xmlrpc (1:7.4.30-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.0 (1:8.0.20-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.0-amqp (1.11.0-4+ubuntu20.04.1+deb.sury.org+10) ...
Removing php8.0-apcu (5.1.21+4.0.11-7+ubuntu20.04.1+deb.sury.org+10) ...
Removing php8.0-bcmath (1:8.0.20-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.0-bz2 (1:8.0.20-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.0-zmq (1.1.3-23+ubuntu20.04.1+deb.sury.org+10) ...
Removing php8.0-yaml (2.2.2+2.1.0+2.0.4+1.3.2-5+ubuntu20.04.1+deb.sury.org+10) ...
Removing php8.0-cgi (1:8.0.20-1+ubuntu20.04.1+deb.sury.org+1) ...
apache2_invoke php8.0-cgi prerm: No action required
Removing php8.0-dev (1:8.0.20-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.0-xdebug (3.1.2+2.9.8+2.8.1+2.5.5-4+ubuntu20.04.1+deb.sury.org+10) ...
Removing php8.0-phpdbg (1:8.0.20-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.0-redis (5.3.6+4.3.0-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.0-zip (1:8.0.20-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.0-xsl (1:8.0.20-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.0-curl (1:8.0.20-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.0-dba (1:8.0.20-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.0-enchant (1:8.0.20-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.0-pcov (1.0.11-4+ubuntu20.04.1+deb.sury.org+10) ...
Removing php8.0-fpm (1:8.0.20-1+ubuntu20.04.1+deb.sury.org+1) ...
apache2_invoke php8.0-fpm prerm: No action required
Removing php8.0-gd (1:8.0.20-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.0-gmp (1:8.0.20-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.0-memcached (3.2.0+2.2.0-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.0-igbinary (3.2.6+2.0.8-6+ubuntu20.04.1+deb.sury.org+10) ...
Removing php8.0-imagick (3.6.0-4+ubuntu20.04.1+deb.sury.org+10) ...
Removing php8.0-imap (1:8.0.20-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.0-interbase (1:8.0.20-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.0-intl (1:8.0.20-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.0-ldap (1:8.0.20-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.0-mbstring (1:8.0.20-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.0-memcache (8.0+4.0.5.2+3.0.9~20170802.e702b5f9+-7+ubuntu20.04.1+deb.sury.org+10) ...
Removing php8.0-mongodb (1.13.0+1.9.2+1.7.5-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.0-msgpack (2.2.0~rc1+2.1.2+0.5.7-6+ubuntu20.04.1+deb.sury.org+10) ...
Removing php8.0-mysql (1:8.0.20-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.0-odbc (1:8.0.20-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.0-pgsql (1:8.0.20-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.0-pspell (1:8.0.20-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.0-snmp (1:8.0.20-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.0-soap (1:8.0.20-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.0-sqlite3 (1:8.0.20-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.0-sybase (1:8.0.20-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.0-tidy (1:8.0.20-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.0-xml (1:8.0.20-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.1 (8.1.7-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.1-amqp (1.11.0-4+ubuntu20.04.1+deb.sury.org+10) ...
Removing php8.1-apcu (5.1.21+4.0.11-7+ubuntu20.04.1+deb.sury.org+10) ...
Removing php8.1-bcmath (8.1.7-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.1-bz2 (8.1.7-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.1-yaml (2.2.2+2.1.0+2.0.4+1.3.2-5+ubuntu20.04.1+deb.sury.org+10) ...
Removing php8.1-cgi (8.1.7-1+ubuntu20.04.1+deb.sury.org+1) ...
apache2_invoke php8.1-cgi prerm: No action required
Removing php8.1-dev (8.1.7-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.1-xdebug (3.1.2+2.9.8+2.8.1+2.5.5-4+ubuntu20.04.1+deb.sury.org+10) ...
Removing php8.1-phpdbg (8.1.7-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.1-redis (5.3.6+4.3.0-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.1-zip (8.1.7-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.1-xsl (8.1.7-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.1-curl (8.1.7-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.1-dba (8.1.7-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.1-enchant (8.1.7-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.1-pcov (1.0.11-4+ubuntu20.04.1+deb.sury.org+10) ...
Removing php8.1-fpm (8.1.7-1+ubuntu20.04.1+deb.sury.org+1) ...
apache2_invoke php8.1-fpm prerm: No action required
Removing php8.1-gd (8.1.7-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.1-gmp (8.1.7-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.1-memcached (3.2.0+2.2.0-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.1-igbinary (3.2.6+2.0.8-6+ubuntu20.04.1+deb.sury.org+10) ...
Removing php8.1-imagick (3.6.0-4+ubuntu20.04.1+deb.sury.org+10) ...
Removing php8.1-imap (8.1.7-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.1-interbase (8.1.7-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.1-intl (8.1.7-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.1-ldap (8.1.7-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.1-mbstring (8.1.7-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.1-memcache (8.0+4.0.5.2+3.0.9~20170802.e702b5f9+-7+ubuntu20.04.1+deb.sury.org+10) ...
Removing php8.1-mongodb (1.13.0+1.9.2+1.7.5-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.1-msgpack (2.2.0~rc1+2.1.2+0.5.7-6+ubuntu20.04.1+deb.sury.org+10) ...
Removing php8.1-mysql (8.1.7-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.1-odbc (8.1.7-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.1-pgsql (8.1.7-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.1-pspell (8.1.7-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.1-snmp (8.1.7-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.1-soap (8.1.7-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.1-sqlite3 (8.1.7-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.1-sybase (8.1.7-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.1-tidy (8.1.7-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.1-xml (8.1.7-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php7.4-cli (1:7.4.30-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php7.4-json (1:7.4.30-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php7.4-opcache (1:7.4.30-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php7.4-readline (1:7.4.30-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.0-cli (1:8.0.20-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.0-opcache (1:8.0.20-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.0-readline (1:8.0.20-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.1-cli (8.1.7-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.1-opcache (8.1.7-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.1-readline (8.1.7-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php7.4-common (1:7.4.30-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.0-common (1:8.0.20-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php8.1-common (8.1.7-1+ubuntu20.04.1+deb.sury.org+1) ...
Removing php-common (2:92+ubuntu20.04.1+deb.sury.org+2) ...
Warning: Stopping phpsessionclean.service, but it can still be activated by:
  phpsessionclean.timer
Processing triggers for man-db (2.9.1-1) ...
Reading package lists...
Building dependency tree...
Reading state information...
Package 'mongodb-org-tools-unstable' is not installed, so not removed
Package 'mongodb-enterprise-tools' is not installed, so not removed
Package 'mongodb-enterprise-tools-unstable' is not installed, so not removed
Package 'mongodb-10gen' is not installed, so not removed
Package 'mongodb-10gen-enterprise' is not installed, so not removed
Package 'mongodb-10gen-unstable' is not installed, so not removed
Package 'mongodb-10gen-unstable-enterprise' is not installed, so not removed
Package 'mongodb-10gen-unstable-enterprise-mongos' is not installed, so not removed
Package 'mongodb-10gen-unstable-enterprise-server' is not installed, so not removed
Package 'mongodb-10gen-unstable-enterprise-shell' is not installed, so not removed
Package 'mongodb-10gen-unstable-enterprise-tools' is not installed, so not removed
Package 'mongodb-10gen-unstable-mongos' is not installed, so not removed
Package 'mongodb-10gen-unstable-server' is not installed, so not removed
Package 'mongodb-10gen-unstable-shell' is not installed, so not removed
Package 'mongodb-10gen-unstable-tools' is not installed, so not removed
Package 'mongodb-enterprise' is not installed, so not removed
Package 'mongodb-enterprise-mongos' is not installed, so not removed
Package 'mongodb-enterprise-server' is not installed, so not removed
Package 'mongodb-enterprise-shell' is not installed, so not removed
Package 'mongodb-enterprise-unstable' is not installed, so not removed
Package 'mongodb-enterprise-unstable-mongos' is not installed, so not removed
Package 'mongodb-enterprise-unstable-server' is not installed, so not removed
Package 'mongodb-enterprise-unstable-shell' is not installed, so not removed
Package 'mongodb-enterprise-unstable-tools' is not installed, so not removed
Package 'mongodb-nightly' is not installed, so not removed
Package 'mongodb-org-unstable' is not installed, so not removed
Package 'mongodb-org-unstable-mongos' is not installed, so not removed
Package 'mongodb-org-unstable-server' is not installed, so not removed
Package 'mongodb-org-unstable-shell' is not installed, so not removed
Package 'mongodb-org-unstable-tools' is not installed, so not removed
Package 'mongodb-stable' is not installed, so not removed
Package 'mongodb-enterprise-database-tools-extra' is not installed, so not removed
Package 'mongodb-enterprise-unstable-database-tools-extra' is not installed, so not removed
Package 'mongodb-org-unstable-database-tools-extra' is not installed, so not removed
Package 'mongodb-dev' is not installed, so not removed
Package 'mongodb-clients' is not installed, so not removed
Package 'mongodb-server' is not installed, so not removed
Package 'mongodb-server-core' is not installed, so not removed
The following packages will be REMOVED:
  mongodb-database-tools mongodb-mongosh mongodb-org mongodb-org-database
  mongodb-org-database-tools-extra mongodb-org-mongos mongodb-org-server
  mongodb-org-shell mongodb-org-tools
0 upgraded, 0 newly installed, 9 to remove and 14 not upgraded.
After this operation, 461 MB disk space will be freed.
(Reading database ... 
(Reading database ... 5%
(Reading database ... 10%
(Reading database ... 15%
(Reading database ... 20%
(Reading database ... 25%
(Reading database ... 30%
(Reading database ... 35%
(Reading database ... 40%
(Reading database ... 45%
(Reading database ... 50%
(Reading database ... 55%
(Reading database ... 60%
(Reading database ... 65%
(Reading database ... 70%
(Reading database ... 75%
(Reading database ... 80%
(Reading database ... 85%
(Reading database ... 90%
(Reading database ... 95%
(Reading database ... 100%
(Reading database ... 215804 files and directories currently installed.)
Removing mongodb-org (5.0.9) ...
Removing mongodb-org-tools (5.0.9) ...
Removing mongodb-database-tools (100.5.3) ...
Removing mongodb-mongosh (1.5.0) ...
Removing mongodb-org-database (5.0.9) ...
Removing mongodb-org-database-tools-extra (5.0.9) ...
Removing mongodb-org-mongos (5.0.9) ...
Removing mongodb-org-server (5.0.9) ...
Removing mongodb-org-shell (5.0.9) ...
Processing triggers for man-db (2.9.1-1) ...
Reading package lists...
Building dependency tree...
Reading state information...
Package 'mysql-client-5.7' is not installed, so not removed
Package 'mysql-client-core-5.7' is not installed, so not removed
Package 'mysql-server-5.5' is not installed, so not removed
Package 'mysql-server-5.7' is not installed, so not removed
Package 'mysql-server-core-5.7' is not installed, so not removed
Package 'mysql-client-core-5.5' is not installed, so not removed
Package 'mysql-client-core-5.6' is not installed, so not removed
Package 'mysql-client-5.5' is not installed, so not removed
Package 'mysql-client-5.6' is not installed, so not removed
Package 'mysql-server-core-5.5' is not installed, so not removed
Package 'mysql-server-core-5.6' is not installed, so not removed
Package 'mysql-server-5.6' is not installed, so not removed
Package 'mysql-testsuite-5.5' is not installed, so not removed
Package 'mysql-testsuite-5.6' is not installed, so not removed
Package 'mysql-testsuite-5.7' is not installed, so not removed
Package 'mysql-sandbox' is not installed, so not removed
Package 'mysql-router' is not installed, so not removed
Package 'mysql-source-8.0' is not installed, so not removed
Package 'mysql-testsuite' is not installed, so not removed
Package 'mysql-testsuite-8.0' is not installed, so not removed
The following packages will be REMOVED:
  libmysqlclient-dev libmysqlclient21 libsnmp35 mysql-client mysql-client-8.0
  mysql-client-core-8.0 mysql-common mysql-server mysql-server-8.0
  mysql-server-core-8.0 snmp sphinxsearch
0 upgraded, 0 newly installed, 12 to remove and 14 not upgraded.
After this operation, 245 MB disk space will be freed.
(Reading database ... 
(Reading database ... 5%
(Reading database ... 10%
(Reading database ... 15%
(Reading database ... 20%
(Reading database ... 25%
(Reading database ... 30%
(Reading database ... 35%
(Reading database ... 40%
(Reading database ... 45%
(Reading database ... 50%
(Reading database ... 55%
(Reading database ... 60%
(Reading database ... 65%
(Reading database ... 70%
(Reading database ... 75%
(Reading database ... 80%
(Reading database ... 85%
(Reading database ... 90%
(Reading database ... 95%
(Reading database ... 100%
(Reading database ... 215746 files and directories currently installed.)
Removing libmysqlclient-dev (8.0.29-0ubuntu0.20.04.3) ...
Removing snmp (5.8+dfsg-2ubuntu2.3) ...
Removing libsnmp35:amd64 (5.8+dfsg-2ubuntu2.3) ...
Removing sphinxsearch (2.2.11-2ubuntu2) ...
Removing libmysqlclient21:amd64 (8.0.29-0ubuntu0.20.04.3) ...
Removing mysql-client (8.0.29-0ubuntu0.20.04.3) ...
Removing mysql-server (8.0.29-0ubuntu0.20.04.3) ...
Removing mysql-server-8.0 (8.0.29-0ubuntu0.20.04.3) ...
update-alternatives: using /etc/mysql/my.cnf.fallback to provide /etc/mysql/my.cnf (my.cnf) in auto mode
Removing mysql-client-8.0 (8.0.29-0ubuntu0.20.04.3) ...
Removing mysql-client-core-8.0 (8.0.29-0ubuntu0.20.04.3) ...
Removing mysql-common (5.8+1.0.5ubuntu2) ...
Removing mysql-server-core-8.0 (8.0.29-0ubuntu0.20.04.3) ...
Processing triggers for man-db (2.9.1-1) ...
Processing triggers for libc-bin (2.31-0ubuntu9.9) ...
Reading package lists...
Building dependency tree...
Reading state information...
The following packages will be REMOVED:
  azure-cli esl-erlang firefox google-chrome-stable google-cloud-sdk hhvm
  libgl1 libgl1-mesa-dri libglx-mesa0 libglx0 libwxgtk3.0-gtk3-0v5
  mono-complete mono-devel mono-roslyn mono-xsp4 mono-xsp4-base monodoc-http
  monodoc-manual powershell x11-utils xvfb
0 upgraded, 0 newly installed, 21 to remove and 14 not upgraded.
After this operation, 3168 MB disk space will be freed.
(Reading database ... 
(Reading database ... 5%
(Reading database ... 10%
(Reading database ... 15%
(Reading database ... 20%
(Reading database ... 25%
(Reading database ... 30%
(Reading database ... 35%
(Reading database ... 40%
(Reading database ... 45%
(Reading database ... 50%
(Reading database ... 55%
(Reading database ... 60%
(Reading database ... 65%
(Reading database ... 70%
(Reading database ... 75%
(Reading database ... 80%
(Reading database ... 85%
(Reading database ... 90%
(Reading database ... 95%
(Reading database ... 100%
(Reading database ... 215378 files and directories currently installed.)
Removing azure-cli (2.37.0-1~focal) ...
Removing esl-erlang (1:25.0.1-1) ...
Removing firefox (101.0.1+build1-0ubuntu0.20.04.1) ...
Removing google-chrome-stable (102.0.5005.115-1) ...
Removing google-cloud-sdk (369.0.0-0) ...
Removing hhvm (4.162.0-1~focal) ...
Stopping hhvm (via systemctl): hhvm.service.
Removing xvfb (2:1.20.13-1ubuntu1~20.04.2) ...
Removing libwxgtk3.0-gtk3-0v5:amd64 (3.0.4+dfsg-15build1) ...
Removing mono-complete (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing powershell (7.2.4-1.deb) ...
Removing x11-utils (7.7+5) ...
Removing libgl1:amd64 (1.3.2-1~ubuntu0.20.04.2) ...
Removing libglx0:amd64 (1.3.2-1~ubuntu0.20.04.2) ...
Removing libglx-mesa0:amd64 (21.2.6-0ubuntu0.1~20.04.2) ...
Removing libgl1-mesa-dri:amd64 (21.2.6-0ubuntu0.1~20.04.2) ...
Removing monodoc-http (4.2-3xamarin5+ubuntu2004b1) ...
Restarting mono-xsp4 (via systemctl): mono-xsp4.service.
Removing monodoc-manual (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing mono-xsp4 (4.7.1-0xamarin3+ubuntu2004b1) ...
Removing mono-xsp4-base (4.7.1-0xamarin3+ubuntu2004b1) ...
Removing mono-roslyn (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing mono-devel (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Processing triggers for hicolor-icon-theme (0.17-2) ...
Processing triggers for libc-bin (2.31-0ubuntu9.9) ...
Processing triggers for man-db (2.9.1-1) ...
Processing triggers for mime-support (3.64ubuntu1) ...
Reading package lists...
Building dependency tree...
Reading state information...
The following packages will be REMOVED:
  dictionaries-common emacsen-common firebird3.0-common firebird3.0-common-doc
  freetds-common hunspell-en-us libaspell15 libboost-context1.71.0
  libboost-filesystem1.71.0 libboost-program-options1.71.0
  libboost-thread1.71.0 libc-client2007e libcgi-fast-perl libcgi-pm-perl
  libclang-common-12-dev libclang1-12 libdbusmenu-glib4 libdbusmenu-gtk3-4
  libdrm-amdgpu1 libdrm-intel1 libdrm-nouveau2 libdrm-radeon1 libdwarf1
  libenchant-2-2 libevent-core-2.1-7 libevent-pthreads-2.1-7 libfbclient2
  libfcgi-perl libfontenc1 libgflags2.2 libglapi-mesa libglvnd0
  libgoogle-glog0v5 libhtml-template-perl libhunspell-1.7-0 libjemalloc2
  libjs-xmlextras liblldb-10 liblldb-11 liblldb-12 libmcrypt4 libmecab2
  libmemcached11 libmono-2.0-1 libmono-2.0-dev libmono-cairo4.0-cil
  libmono-cecil-private-cil libmono-cil-dev libmono-codecontracts4.0-cil
  libmono-compilerservices-symbolwriter4.0-cil libmono-cscompmgd0.0-cil
  libmono-csharp4.0c-cil libmono-custommarshalers4.0-cil
  libmono-data-tds4.0-cil libmono-db2-1.0-cil libmono-debugger-soft4.0a-cil
  libmono-http4.0-cil libmono-management4.0-cil
  libmono-messaging-rabbitmq4.0-cil libmono-microsoft-build-engine4.0-cil
  libmono-microsoft-build-tasks-v4.0-4.0-cil libmono-microsoft-build4.0-cil
  libmono-microsoft-visualc10.0-cil
  libmono-microsoft-web-infrastructure1.0-cil libmono-oracle4.0-cil
  libmono-parallel4.0-cil libmono-peapi4.0a-cil libmono-profiler
  libmono-rabbitmq4.0-cil libmono-relaxng4.0-cil libmono-sharpzip4.84-cil
  libmono-simd4.0-cil libmono-smdiagnostics0.0-cil
  libmono-system-data-datasetextensions4.0-cil
  libmono-system-data-entity4.0-cil libmono-system-data-linq4.0-cil
  libmono-system-data-services4.0-cil libmono-system-deployment4.0-cil
  libmono-system-drawing-design4.0-cil libmono-system-dynamic4.0-cil
  libmono-system-io-compression-filesystem4.0-cil
  libmono-system-json-microsoft4.0-cil libmono-system-json4.0-cil
  libmono-system-ldap-protocols4.0-cil libmono-system-management4.0-cil
  libmono-system-net-http-formatting4.0-cil
  libmono-system-net-http-webrequest4.0-cil libmono-system-net4.0-cil
  libmono-system-numerics-vectors4.0-cil libmono-system-reactive-core2.2-cil
  libmono-system-reactive-debugger2.2-cil
  libmono-system-reactive-experimental2.2-cil
  libmono-system-reactive-interfaces2.2-cil
  libmono-system-reactive-linq2.2-cil
  libmono-system-reactive-observable-aliases0.0-cil
  libmono-system-reactive-platformservices2.2-cil
  libmono-system-reactive-providers2.2-cil
  libmono-system-reactive-runtime-remoting2.2-cil
  libmono-system-reactive-windows-forms2.2-cil
  libmono-system-reactive-windows-threading2.2-cil
  libmono-system-reflection-context4.0-cil
  libmono-system-runtime-caching4.0-cil
  libmono-system-runtime-durableinstancing4.0-cil
  libmono-system-runtime4.0-cil libmono-system-servicemodel-discovery4.0-cil
  libmono-system-servicemodel-routing4.0-cil
  libmono-system-servicemodel-web4.0-cil libmono-system-serviceprocess4.0-cil
  libmono-system-threading-tasks-dataflow4.0-cil
  libmono-system-web-abstractions4.0-cil libmono-system-web-dynamicdata4.0-cil
  libmono-system-web-extensions-design4.0-cil
  libmono-system-web-extensions4.0-cil libmono-system-web-http-selfhost4.0-cil
  libmono-system-web-http-webhost4.0-cil libmono-system-web-http4.0-cil
  libmono-system-web-mobile4.0-cil libmono-system-web-mvc3.0-cil
  libmono-system-web-razor2.0-cil libmono-system-web-regularexpressions4.0-cil
  libmono-system-web-routing4.0-cil
  libmono-system-web-webpages-deployment2.0-cil
  libmono-system-web-webpages-razor2.0-cil libmono-system-web-webpages2.0-cil
  libmono-system-windows-forms-datavisualization4.0a-cil
  libmono-system-windows4.0-cil libmono-system-workflow-activities4.0-cil
  libmono-system-workflow-componentmodel4.0-cil
  libmono-system-workflow-runtime4.0-cil
  libmono-system-xml-serialization4.0-cil libmono-tasklets4.0-cil
  libmono-webmatrix-data4.0-cil libmono-xbuild-tasks4.0-cil libmonoboehm-2.0-1
  libmonosgen-2.0-1 libmonosgen-2.0-dev libncurses5 libnorm1 libnotify4
  libpciaccess0 libpcre2-posix2 libpfm4 libpgm-5.2-0 libqdbm14 librabbitmq4
  libre2-5 libsctp1 libsnappy1v5 libsnmp-base libsybdb5 libtbb2 libtidy5deb1
  libtinfo-dev libtinfo5 libtommath1 libu2f-udev libvpx6 libvulkan1
  libwxbase3.0-0v5 libx11-xcb1 libxcb-dri2-0 libxcb-dri3-0 libxcb-glx0
  libxcb-present0 libxcb-shape0 libxcb-sync1 libxcb-xfixes0 libxfont2
  libxmlrpc-epi0 libxshmfence1 libxv1 libxxf86dga1 libz3-4 libz3-dev libzip4
  libzmq5 mecab-ipadic mecab-ipadic-utf8 mecab-utils mlock mono-4.0-service
  mono-csharp-shell mono-mcs mono-utils mono-xbuild monodoc-base msbuild
  msbuild-libhostfxr msbuild-sdkresolver netstandard-targeting-pack-2.1
  python3-crcmod python3-lldb-11 python3-pygments referenceassemblies-pcl
  shtool x11-xkb-utils xserver-common xul-ext-ubufox
0 upgraded, 0 newly installed, 198 to remove and 14 not upgraded.
After this operation, 433 MB disk space will be freed.
(Reading database ... 
(Reading database ... 5%
(Reading database ... 10%
(Reading database ... 15%
(Reading database ... 20%
(Reading database ... 25%
(Reading database ... 30%
(Reading database ... 35%
(Reading database ... 40%
(Reading database ... 45%
(Reading database ... 50%
(Reading database ... 55%
(Reading database ... 60%
(Reading database ... 65%
(Reading database ... 70%
(Reading database ... 75%
(Reading database ... 80%
(Reading database ... 85%
(Reading database ... 90%
(Reading database ... 95%
(Reading database ... 100%
(Reading database ... 125456 files and directories currently installed.)
Removing libenchant-2-2:amd64 (2.2.8-1ubuntu0.20.04.1) ...
Removing hunspell-en-us (1:2018.04.16-1) ...
Removing dictionaries-common (1.28.1) ...
Removing 'diversion of /usr/share/dict/words to /usr/share/dict/words.pre-dictionaries-common by dictionaries-common'
Removing emacsen-common (3.0.4) ...
Removing libfbclient2:amd64 (3.0.5.33220.ds4-1build2) ...
Removing firebird3.0-common (3.0.5.33220.ds4-1build2) ...
Removing firebird3.0-common-doc (3.0.5.33220.ds4-1build2) ...
Removing libsybdb5:amd64 (1.1.6-1.1) ...
Removing freetds-common (1.1.6-1.1) ...
Removing libaspell15:amd64 (0.60.8-1ubuntu0.1) ...
Removing libboost-context1.71.0:amd64 (1.71.0-6ubuntu6) ...
Removing libboost-filesystem1.71.0:amd64 (1.71.0-6ubuntu6) ...
Removing libboost-program-options1.71.0:amd64 (1.71.0-6ubuntu6) ...
Removing libboost-thread1.71.0:amd64 (1.71.0-6ubuntu6) ...
Removing libc-client2007e (8:2007f~dfsg-7) ...
Removing libcgi-fast-perl (1:2.15-1) ...
Removing libhtml-template-perl (2.97-1) ...
Removing libcgi-pm-perl (4.46-1) ...
Removing libclang-common-12-dev (1:12.0.0-3ubuntu1~20.04.5) ...
Removing libclang1-12 (1:12.0.0-3ubuntu1~20.04.5) ...
Removing libdbusmenu-gtk3-4:amd64 (16.04.1+18.10.20180917-0ubuntu6) ...
Removing libdbusmenu-glib4:amd64 (16.04.1+18.10.20180917-0ubuntu6) ...
Removing libdrm-amdgpu1:amd64 (2.4.107-8ubuntu1~20.04.2) ...
Removing libdrm-intel1:amd64 (2.4.107-8ubuntu1~20.04.2) ...
Removing libdrm-nouveau2:amd64 (2.4.107-8ubuntu1~20.04.2) ...
Removing libdrm-radeon1:amd64 (2.4.107-8ubuntu1~20.04.2) ...
Removing libdwarf1:amd64 (20200114-1) ...
Removing libevent-pthreads-2.1-7:amd64 (2.1.11-stable-1) ...
Removing libevent-core-2.1-7:amd64 (2.1.11-stable-1) ...
Removing libfcgi-perl (0.79-1) ...
Removing libxfont2:amd64 (1:2.0.3-1) ...
Removing libfontenc1:amd64 (1:1.1.4-0ubuntu1) ...
Removing libgoogle-glog0v5 (0.4.0-1build1) ...
Removing libgflags2.2 (2.2.2-1build1) ...
Removing libglapi-mesa:amd64 (21.2.6-0ubuntu0.1~20.04.2) ...
Removing libglvnd0:amd64 (1.3.2-1~ubuntu0.20.04.2) ...
Removing libhunspell-1.7-0:amd64 (1.7.0-2build2) ...
Removing libjemalloc2:amd64 (5.2.1-1ubuntu1) ...
Removing libjs-xmlextras (20060529-1) ...
Removing liblldb-10 (1:10.0.0-4ubuntu1) ...
Removing python3-lldb-11 (1:11.0.0-2~ubuntu20.04.1) ...
Removing liblldb-11 (1:11.0.0-2~ubuntu20.04.1) ...
Removing liblldb-12 (1:12.0.0-3ubuntu1~20.04.5) ...
Removing libmcrypt4 (2.5.8-3.4) ...
Removing mecab-ipadic-utf8 (2.7.0-20070801+main-2.1) ...
update-alternatives: using /var/lib/mecab/dic/ipadic to provide /var/lib/mecab/dic/debian (mecab-dictionary) in auto mode
Removing mecab-ipadic (2.7.0-20070801+main-2.1) ...
Removing mecab-utils (0.996-10build1) ...
Removing libmecab2:amd64 (0.996-10build1) ...
Removing libmemcached11:amd64 (1.0.18-4.2ubuntu2) ...
Removing libmono-2.0-1 (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-2.0-dev (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-cil-dev (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-cairo4.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-codecontracts4.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing monodoc-base (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-compilerservices-symbolwriter4.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-cscompmgd0.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing mono-csharp-shell (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-csharp4.0c-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-custommarshalers4.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-data-tds4.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-db2-1.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-debugger-soft4.0a-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-http4.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-management4.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-messaging-rabbitmq4.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing referenceassemblies-pcl (2014.04.14-1xamarin7+ubuntu2004b1) ...
Removing mono-xbuild (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-microsoft-build-tasks-v4.0-4.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-microsoft-build-engine4.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-microsoft-build4.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-microsoft-visualc10.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-system-web-http-webhost4.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-system-web-mvc3.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing msbuild (1:16.10.1+xamarinxplat.2021.05.26.14.00-0xamarin2+ubuntu2004b1) ...
Removing libmono-oracle4.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-parallel4.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-peapi4.0a-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-profiler (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-rabbitmq4.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-relaxng4.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-sharpzip4.84-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-simd4.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-smdiagnostics0.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-system-data-datasetextensions4.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-system-data-entity4.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-system-web-dynamicdata4.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-system-web-http-selfhost4.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-system-web-http4.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-system-data-services4.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-system-deployment4.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-system-drawing-design4.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-system-dynamic4.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-system-io-compression-filesystem4.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-system-json-microsoft4.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-system-json4.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-system-ldap-protocols4.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-system-management4.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-system-net-http-formatting4.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-system-net-http-webrequest4.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-system-net4.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-system-numerics-vectors4.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-system-reactive-experimental2.2-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-system-reactive-windows-threading2.2-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-system-reactive-debugger2.2-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-system-reactive-windows-forms2.2-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-system-reactive-observable-aliases0.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-system-reactive-providers2.2-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-system-reactive-platformservices2.2-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-system-reactive-linq2.2-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-system-reactive-runtime-remoting2.2-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-system-reflection-context4.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-system-runtime-caching4.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-system-runtime-durableinstancing4.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-system-runtime4.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-system-servicemodel-discovery4.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-system-servicemodel-routing4.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-system-servicemodel-web4.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing mono-4.0-service (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-system-serviceprocess4.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-system-threading-tasks-dataflow4.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-system-web-abstractions4.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-system-web-extensions-design4.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-system-web-extensions4.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-system-web-mobile4.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-system-web-webpages-razor2.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-system-web-regularexpressions4.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-system-web-routing4.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-system-windows-forms-datavisualization4.0a-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-system-windows4.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-system-workflow-activities4.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-system-workflow-componentmodel4.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-system-workflow-runtime4.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-system-xml-serialization4.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-tasklets4.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-webmatrix-data4.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-xbuild-tasks4.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing mono-utils (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmonoboehm-2.0-1 (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmonosgen-2.0-dev (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmonosgen-2.0-1 (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libncurses5:amd64 (6.2-0ubuntu2) ...
Removing libzmq5:amd64 (4.3.2-2ubuntu1) ...
Removing libnorm1:amd64 (1.5.8+dfsg2-2build1) ...
Removing libnotify4:amd64 (0.7.9-1ubuntu2) ...
Removing libpciaccess0:amd64 (0.16-0ubuntu1) ...
Removing libpcre2-posix2:amd64 (10.34-7) ...
Removing libpfm4:amd64 (4.10.1+git20-g7700f49-2) ...
Removing libpgm-5.2-0:amd64 (5.2.122~dfsg-3ubuntu1) ...
Removing libqdbm14 (1.8.78-9build3) ...
Removing librabbitmq4:amd64 (0.10.0-1) ...
Removing libre2-5:amd64 (20200101+dfsg-1build1) ...
Removing libsctp1:amd64 (1.0.18+dfsg-1) ...
Removing libsnappy1v5:amd64 (1.1.8-1build1) ...
Removing libsnmp-base (5.8+dfsg-2ubuntu2.3) ...
Removing libtbb2:amd64 (2020.1-2) ...
Removing libtidy5deb1:amd64 (2:5.6.0-11) ...
Removing libtinfo-dev:amd64 (6.2-0ubuntu2) ...
Removing libtinfo5:amd64 (6.2-0ubuntu2) ...
Removing libtommath1:amd64 (1.2.0-3) ...
Removing libu2f-udev (1.1.10-1) ...
Removing libvpx6:amd64 (1.8.2-1build1) ...
Removing libvulkan1:amd64 (1.2.131.2-1) ...
Removing libwxbase3.0-0v5:amd64 (3.0.4+dfsg-15build1) ...
Removing libx11-xcb1:amd64 (2:1.6.9-2ubuntu1.2) ...
Removing libxcb-dri2-0:amd64 (1.14-2) ...
Removing libxcb-dri3-0:amd64 (1.14-2) ...
Removing libxcb-glx0:amd64 (1.14-2) ...
Removing libxcb-present0:amd64 (1.14-2) ...
Removing libxcb-shape0:amd64 (1.14-2) ...
Removing libxcb-sync1:amd64 (1.14-2) ...
Removing libxcb-xfixes0:amd64 (1.14-2) ...
Removing libxmlrpc-epi0:amd64 (0.54.2-1.2) ...
Removing libxshmfence1:amd64 (1.3-1) ...
Removing libxv1:amd64 (2:1.0.11-1) ...
Removing libxxf86dga1:amd64 (2:1.1.5-0ubuntu1) ...
Removing libz3-dev:amd64 (4.8.7-4build1) ...
Removing libz3-4:amd64 (4.8.7-4build1) ...
Removing libzip4:amd64 (1.7.3-1+ubuntu20.04.1+deb.sury.org+2) ...
Removing mlock (8:2007f~dfsg-7) ...
Removing mono-mcs (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing netstandard-targeting-pack-2.1 (2.1.0-1) ...
Removing python3-crcmod (1.7+dfsg-2build2) ...
Removing python3-pygments (2.3.1+dfsg-1ubuntu2.2) ...
Removing shtool (2.0.8-10) ...
Removing xserver-common (2:1.20.13-1ubuntu1~20.04.2) ...
Removing x11-xkb-utils (7.7+5) ...
Removing xul-ext-ubufox (3.4-0ubuntu1.17.10.1) ...
Removing libmono-cecil-private-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-system-web-webpages2.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-system-data-linq4.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-system-reactive-core2.2-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-system-reactive-interfaces2.2-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-system-web-razor2.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-system-web-webpages-deployment2.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing libmono-microsoft-web-infrastructure1.0-cil (6.12.0.182-0xamarin1+ubuntu2004b1) ...
Removing msbuild-libhostfxr (3.0.0.2019.04.16.02.13-0xamarin4+ubuntu2004b1) ...
Removing msbuild-sdkresolver (1:16.10.1+xamarinxplat.2021.05.26.14.00-0xamarin2+ubuntu2004b1) ...
Processing triggers for man-db (2.9.1-1) ...
Processing triggers for postgresql-common (241.pgdg20.04+1) ...
Building PostgreSQL dictionaries from installed myspell/hunspell packages...
Removing obsolete dictionary files:
  /var/cache/postgresql/dicts/en_us.affix
  /var/cache/postgresql/dicts/en_us.dict
  /usr/share/postgresql//14/tsearch_data/en_us.affix
  /usr/share/postgresql//14/tsearch_data/en_us.dict
Processing triggers for libc-bin (2.31-0ubuntu9.9) ...

********************************************************************************
=> Large misc. packages: Saved 5.3GiB
********************************************************************************


********************************************************************************
=> Tool cache: Saved 5.9GiB
********************************************************************************

              total        used        free      shared  buff/cache   available
Mem:          6.8Gi       482Mi       5.0Gi       1.0Mi       1.3Gi       6.0Gi
Swap:            0B          0B          0B

********************************************************************************
=> Swap storage: Saved 4.0GiB
********************************************************************************


================================================================================
AFTER CLEAN-UP:

$ df -h /

Filesystem      Size  Used Avail Use% Mounted on
/dev/root        84G   26G   58G  31% /
$ df -a /

Filesystem     1K-blocks     Used Available Use% Mounted on
/dev/root       87218124 26471028  60730712  31% /
$ df -a

Filesystem     1K-blocks     Used Available Use% Mounted on
/dev/root       87218124 26471028  60730712  31% /
devtmpfs         3552992        0   3552992   0% /dev
sysfs                  0        0         0    - /sys
proc                   0        0         0    - /proc
securityfs             0        0         0    - /sys/kernel/security
tmpfs            3556564        4   3556560   1% /dev/shm
devpts                 0        0         0    - /dev/pts
tmpfs             711316     1060    710256   1% /run
tmpfs               5120        0      5120   0% /run/lock
tmpfs            3556564        0   3556564   0% /sys/fs/cgroup
cgroup2                0        0         0    - /sys/fs/cgroup/unified
cgroup                 0        0         0    - /sys/fs/cgroup/systemd
pstore                 0        0         0    - /sys/fs/pstore
none                   0        0         0    - /sys/fs/bpf
cgroup                 0        0         0    - /sys/fs/cgroup/cpuset
cgroup                 0        0         0    - /sys/fs/cgroup/pids
cgroup                 0        0         0    - /sys/fs/cgroup/devices
cgroup                 0        0         0    - /sys/fs/cgroup/hugetlb
cgroup                 0        0         0    - /sys/fs/cgroup/misc
cgroup                 0        0         0    - /sys/fs/cgroup/net_cls,net_prio
cgroup                 0        0         0    - /sys/fs/cgroup/blkio
cgroup                 0        0         0    - /sys/fs/cgroup/cpu,cpuacct
cgroup                 0        0         0    - /sys/fs/cgroup/perf_event
cgroup                 0        0         0    - /sys/fs/cgroup/memory
cgroup                 0        0         0    - /sys/fs/cgroup/rdma
cgroup                 0        0         0    - /sys/fs/cgroup/freezer
systemd-1              -        -         -    - /proc/sys/fs/binfmt_misc
hugetlbfs              0        0         0    - /dev/hugepages
mqueue                 0        0         0    - /dev/mqueue
debugfs                0        0         0    - /sys/kernel/debug
tracefs                0        0         0    - /sys/kernel/tracing
fusectl                0        0         0    - /sys/fs/fuse/connections
configfs               0        0         0    - /sys/kernel/config
/dev/loop0         63488    63488         0 100% /snap/core20/1518
/dev/loop1         69504    69504         0 100% /snap/lxd/22753
/dev/loop2         48128    48128         0 100% /snap/snapd/16010
/dev/sda15        106858     5321    101537   5% /boot/efi
binfmt_misc            0        0         0    - /proc/sys/fs/binfmt_misc
/dev/sdb1       14382088    40988  13590816   1% /mnt
tmpfs             711316     1060    710256   1% /run/snapd/ns
nsfs                   0        0         0    - /run/snapd/ns/lxd.mnt
================================================================================


/dev/root:

********************************************************************************
=> Saved 27GiB
********************************************************************************

overall:

********************************************************************************
=> Saved 31GiB
********************************************************************************
```
