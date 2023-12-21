# astria_rollup

Öncelikle işletim sistemimizi güncelliyoruz.
```
sudo apt-get update  && sudo apt-get upgrade
```
Cargo yüklüyoruz.
```
apt install cargo
```
Rust Yüklüyoruz
```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source "$HOME/.cargo/env"
rustc –version
```
Git Yüklüyoruz
```
apt install git
```
Astria dev-cluster reposunu klonluyoruz.
```
git clone --branch dusk-2 https://github.com/astriaorg/dev-cluster.git
```
En son sürüm astria cli kuruyoruz
```
cargo install astria-cli --git=https://github.com/astriaorg/astria --tag=cli-v0.2.1 –locked
