# Build cardano-address on aarch64

{% hint style="danger" %}
This is a work in progress and note taking. Here be dragons...
{% endhint %}

![](.gitbook/assets/image%20%283%29.png)

### Install zram

```bash
sudo apt-get install zram-config
sudo reboot
```

```bash
sudo apt install build-essential curl libffi-dev libffi8ubuntu1 libgmp-dev libgmp10 libncurses-dev libncurses5 libtinfo5 libpcre3-dev
```

```bash
cd $HOME/git
git clone https://github.com/input-output-hk/libsodium
cd libsodium
git checkout 66f017f1
./autogen.sh
./configure
make
sudo make install
```

```bash
wget https://github.com/llvm/llvm-project/releases/download/llvmorg-9.0.1/clang+llvm-9.0.1-aarch64-linux-gnu.tar.xz
tar -xvf clang+llvm-9.0.1-aarch64-linux-gnu.tar.xz
mv clang+llvm-9.0.1-aarch64-linux-gnu/bin/* /home/ada/.local/bin
```

### Install ghcup

```text
curl --proto '=https' --tlsv1.2 -sSf https://get-ghcup.haskell.org | sh
```

No need for the Haskel IDE. Type yes for .bashrc additions and source.

```bash
source $HOME/.bashrc
```

{% embed url="https://github.com/haskell/cabal/issues/6521\#issuecomment-774306372" %}

{% embed url="https://llvm.org/docs/HowToBuildOnARM.html" %}

```bash
ghcup install ghc 8.6.5
ghcup set ghc
```

```bash
git clone https://github.com/input-output-hk/cardano-addresses.git
cd cardano-addresses
cabal update
git fetch --all --tags
git checkout tags/3.2.0
echo -e "package cardano-crypto-praos\n  flags: -external-libsodium-vrf" > cabal.project.local
cabal build --ghc-options=-dynamic all
```

{% embed url="https://downloads.haskell.org/~ghcup/0.1.13/armv7-linux-ghcup-0.1.13" %}

{% embed url="https://github.com/llvm/llvm-project/releases/download/llvmorg-9.0.1/clang+llvm-9.0.1-aarch64-linux-gnu.tar.xz" %}

{% hint style="info" %}
[https://github.com/input-output-hk/cardano-addresses](https://github.com/input-output-hk/cardano-addresses)[https://gist.github.com/ilap/3fd57e39520c90f084d25b0ef2b96894](https://gist.github.com/ilap/3fd57e39520c90f084d25b0ef2b96894)

[https://gist.github.com/shroomist/2b261d3cc8784c0df743787e8ce12a01\#file-rpi-cardano-build-L28](https://gist.github.com/shroomist/2b261d3cc8784c0df743787e8ce12a01#file-rpi-cardano-build-L28)
{% endhint %}

