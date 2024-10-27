#   Trivial Debian preseed templater

To simplify Debian automated installation
with [preseed](https://wiki.debian.org/DebianInstaller/Preseed),
and to make it more manageable,
this is my personal, trivial, and definitely ugly, tiny tool.

It helps me to manage multiple preseed templates,
each belongs to a host group.
Hopefully, it can help you, too.

##  How-to

1.  Create a new template for your host group,

    ```shell
    ./preseeder -t homelab
    ```

2.  That will generate two template files:

    -   `tmpl/homelab/preseed.cfg.tmpl`
    -   `tmpl/homelab/latecmd.sh.tmpl`

    Edit them to match your needs.

3.  Extract all variables used in those template files,

    ```shell
    ./preseeder -e homelab
    ```

4.  That will generate a starter variable file `vars/homelab/var`.

    Copy or rename it to match your planned installation, _e.g._,

    ```shell
    mv vars/homelab/var vars/homelab/host-01
    ```

5.  Edit the variable file to match your infrastructure, _e.g._,

    ```shell
    DOMAIN="home.lab"
    HOSTNAME="host-01"
    FULLNAME="Debian Admin"
    GH_USERNAME="geekobiloba"
    KEYBOARD="us"
    LOCALE="id_ID.utf8"
    TIMEZONE="Asia/Jakarta"
    MIRROR_COUNTRY="Indonesia"
    PACKAGES="build-essential git curl jq htop vim ripgrep bat"
    FILESYSTEM="ext4"
    USERNAME="geekobiloba"
    PASSWD_HASH_USER='$6$RXZ9ccXz07zIlcWK$Q5HMTePavdX9L9HBQWrQtQLcaD6Qz8494CdCBsaPqtfJJVebrGspSYeAFx15X5RzBhdg0rGywEuiqgn4LQ.kW0'
    PASSWD_HASH_ROOT='$6$RXZ9ccXz07zIlcWK$Q5HMTePavdX9L9HBQWrQtQLcaD6Qz8494CdCBsaPqtfJJVebrGspSYeAFx15X5RzBhdg0rGywEuiqgn4LQ.kW0'
    NTP_SERVER="time.bmkg.go.id"
    LATECMD_URL="http://192.168.111.111:8000/homelab/host-01/latecmd.sh"
    ```

    You can create hashed passwords with `openssl passwd -6`.

6.  Optionally,
    copy and edit the variable file for other hosts under the same group,
    each file for a single host.

7.  Then, make preseed files for the whole host group.

    ```shell
    ./preseeder -p homelab
    ```

    Your final directory structure should resemble this,

    ```text
    .
    ├── preseed
    │   └── homelab
    │       ├── host-01
    │       │   ├── latecmd.sh
    │       │   └── preseed.cfg
    │       └── host-02
    │           ├── latecmd.sh
    │           └── preseed.cfg
    ├── preseeder
    ├── tmpl
    │   ├── default
    │   │   ├── latecmd.sh.tmpl
    │   │   └── preseed.cfg.tmpl
    │   └── homelab
    │       ├── latecmd.sh.tmpl
    │       └── preseed.cfg.tmpl
    └── vars
        └── homelab
            ├── host-01
            └── host-02
    ```

### Using the preseed

The straightforward way is to deliver them via HTTP.

1.  Serve the preseed directory with Python,

    ```shell
    python3 -m http.server -d preseed
    ```

2.  Run an ordinary Debian installer,
    choose **Advanced options**,
    then **Automated install**.

    Wait until a prompt
    titled **Download debconf preconfiguration file** appears,
    then enter `http://${WEB_SERVER_IP}:8000/homelab/host-01/preseed.cfg`.

    The installer will continue unattendedly to finish,
    unless there's an error, of course.

Other options
are listed [here](https://www.debian.org/releases/stable/i386/apbs02.en.html),
or using a [DHCP server](https://www.growse.com/2023/05/05/automated-debian-installs-with-preseeding-from-dhcp.html).

##  FAQs

-   _Why bother with preseed when there's Ansible?_

    Ansible runs post-installation,
    and it can't give answers to the installer.
    Besides, preseed is there with every Debian installer since like forever.

-   _Why not cloud-init?_

    Then I'll have to use the Debian cloud images,
    which give me only a few architectures.

-   _How do you use this mostly?_

    Automating Debian installation on niche archs,
    like ARMv7 or MIPSel,
    mostly on QEMU,
    to compile stuff for embedded devices,
    usually runs OpenWrt.

-   _Is there any caveats?_

    Yes.
    This tool doesn't check preseed files validity.
    You should know what you're doing when authoring them.

    Read the [official docs](https://www.debian.org/releases/stable/amd64/apb.en.html)
    and
    [partition recipe](https://salsa.debian.org/installer-team/debian-installer/-/blob/master/doc/devel/partman-auto-recipe.txt)
    to learn more.


