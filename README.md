# Customize Fedora Kernel

In general I followed the instructions here: https://docs.fedoraproject.org/en-US/quick-docs/kernel/build-custom-kernel/

# Get the Dependencies
```
sudo dnf install fedpkg
fedpkg clone -a kernel
cd kernel
sudo dnf builddep kernel.spec
```

# ccache

```
sudo dnf install ccache
```

# Secure boot

add the user doing the build to `/etc/pesign/users`

run the authorize user script

```
sudo /usr/libexec/pesign/pesign-authorize
```

Create a new Machine Owner Key (MOK) to import to UEFI:

```
openssl req -new -x509 -newkey rsa:2048 -keyout "key.pem" \
        -outform DER -out "cert.der" -nodes -days 36500 \
        -subj "/CN=<your name>/"
```

Import the new certificate into your UEFI database:

```
mokutil --import "cert.der"
```

Create a PKCS #12 key file:

```
openssl pkcs12 -export -out key.p12 -inkey key.pem -in cert.der
```

You can then import the certificate and key into the nss database:

```
certutil -A -i cert.der -n "<MOK certificate nickname>" -d /etc/pki/pesign/ -t "Pu,Pu,Pu"
pk12util -i key.p12 -d /etc/pki/pesign
```

Once the certificate and key are imported into your nss database, you can build the kernel with the selected key by adding `%define pe_signing_cert <MOK certificate nickname>` to the `kernel.spec` file or calling `rpmbuild` directly with the `--define "pe_signing_cert <MOK certificate nickname>"` flag.

# Building a Kernel from the Fedora dist-git

* `git checkout origin/f36`
* `kernel.spec` change `# define buildid .local` to `%define buidlid .<your_custom_id_here>`
* customize
* Build the RPMs: `fedpkg local`
* Install: `sudo dnf install --nogpgcheck ./x86_64/kernel-...$version.rpm`

