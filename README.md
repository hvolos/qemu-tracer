# Building

sudo apt-get update
sudo apt-get install -y python3-venv python3-pip ninja-build
sudo apt-get install -y libglib2.0-dev flex bison capstone-tool libcapstone-dev

./configure --enable-capstone --enable-slirp

make -j 20

# Ubuntu Image

sudo apt-get install -y cloud-image-utils

wget https://cloud-images.ubuntu.com/noble/current/noble-server-cloudimg-amd64.img

wget https://cloud-images.ubuntu.com/minimal/releases/noble/release/ubuntu-24.04-minimal-cloudimg-amd64.img

wget https://cloud-images.ubuntu.com/minimal/releases/bionic/release-20230602/ubuntu-18.04-minimal-cloudimg-amd64.img

cat > metadata.yaml <<EOF
instance-id: iid-local01
local-hostname: cloudimg
EOF

cat >user-data <<EOF
#cloud-config
password: asdfqwer
chpasswd: { expire: False }
ssh_pwauth: True
EOF


cloud-localds seed.img user-data.yaml metadata.yaml

# Running 

## with HMP monitor

./build/qemu-system-x86_64 \
-m 2G \
-monitor telnet:127.0.0.1:1234,server,nowait \
-drive if=virtio,format=qcow2,file=ubuntu-18.04-minimal-cloudimg-amd64.img \
-drive if=virtio,format=raw,file=seed.img \
-serial telnet:localhost:4321,server,nowait \
-plugin ./build/contrib/plugins/libexeclog.so -d plugin \
-device virtio-net-pci,netdev=net0 \
-netdev user,id=net0,hostfwd=tcp::2222-:22

## with QMP monitor

./build/qemu-system-x86_64 \
-m 2G \
-chardev socket,id=qmp,port=4444,host=localhost,server=on \
-mon chardev=qmp,mode=control,pretty=on

# References

- [Qemu Monitor Interface
](https://balamuruhans.github.io/2019/01/16/qemu-monitor-interface.html)
- [Launching Ubuntu Cloud Images with QEMU](https://powersj.io/posts/ubuntu-qemu-cli/)
- [Boot an Ubuntu Cloud Image with QEMU](https://levelup.gitconnected.com/boot-an-ubuntu-cloud-image-with-qemu-c42c77cf92cc)
