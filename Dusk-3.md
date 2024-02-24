# astria_rollup (Dusk-3)
<img src="https://github.com/hakandemirdev/astria_rollup/blob/960ad0b8f4f39da15ed1ea5d2ae3653538699dcf/dusk3logo.jpeg" width="auto">

Öncelikle işletim sistemimizi güncelliyoruz.Ubuntu 22.04.3 LTS üzerine kurdum.
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
git clone --branch dusk-3 https://github.com/astriaorg/dev-cluster.git
```
En son sürüm astria cli kuruyoruz
```
cargo install astria-cli --git=https://github.com/astriaorg/astria --tag=cli-v0.3.1 --locked
```
Docker Kurulumu
```
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
$(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo docker run hello-world
```
Kubectl Kurulumu
```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
kubectl version --client
```
Helm Kurulumu
```
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
sudo apt-get install apt-transport-https --yes
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```
Kind Kurulumu
```
[ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```
Just Kurulumu
```
wget -qO - 'https://proget.makedeb.org/debian-feeds/prebuilt-mpr.pub' | gpg --dearmor | sudo tee /usr/share/keyrings/prebuilt-mpr-archive-keyring.gpg 1> /dev/null
echo "deb [arch=all,$(dpkg --print-architecture) signed-by=/usr/share/keyrings/prebuilt-mpr-archive-keyring.gpg] https://proget.makedeb.org prebuilt-mpr $(lsb_release -cs)" | sudo tee /etc/apt/sources.list.d/prebuilt-mpr.list
sudo apt update
sudo apt install just
```
Foundry Cast ve Forge kurulumu
```
cargo install --git https://github.com/foundry-rs/foundry --profile local --locked forge cast chisel anvil
```
Yerel Ortamın Hazırlanması
```
apt install screen
screen -S rollup
cd dev-cluster
just create-cluster
just deploy-ingress-controller
```
Rollup Genesis Hesabı Oluşturuyoruz
(Bu komuttan sonra adres ve private keyimizi not alıyoruz.)
```
cast w new
export ROLLUP_FAUCET_PRIV_KEY=privatekeyimiziburayayazıyoruz.
export ROLLUP_GENESIS_ACCOUNTS=genesisadresiburayayazıyoruz:100000000000000000000
```
Yapılandırmaları ayarlıyoruz
(network id chainlist.org adresinde bulunmayan bir id olacak)
```
export ROLLUP_NAME=rollupismimiziburayayazıyoruz
export ROLLUP_NETWORK_ID=networkidmiziburayayazıyoruz
astria-cli rollup config create
export ROLLUP_CONF_FILE=$ROLLUP_NAME-rollup-conf.yaml
cat $ROLLUP_CONF_FILE
```
Yeni Bir Sequencer Hesabı Oluşturalım
(Bu komut ile private key, public key ve adresimiz yaratılacak, bunları not alıyoruz)
```
astria-cli sequencer account create
export SEQUENCER_PRIV_KEY=burayasequencerprivatekeyiyazıyoruz
export SEQUENCER_ACCOUNT_ADDRESS=burayasequenceradresimiziyazıyoruz
```
Faucetten token istiyoruz. Faucet' e sequencer adresimizi girmemiz gerekiyor.
[Faucet](https://faucet.sequencer.dusk-3.devnet.astria.org)

Token gelip gelmediğini kontrol ediyoruz.
```
astria-cli sequencer account balance $SEQUENCER_ACCOUNT_ADDRESS
```
Rollup Node deploy ediyoruz.
```
astria-cli rollup deployment create \
  --config $ROLLUP_CONF_FILE \
  --faucet-private-key $ROLLUP_FAUCET_PRIV_KEY \
  --sequencer-private-key $SEQUENCER_PRIV_KEY
```
Rollup'ımızın çaılıp çalışmadığını kontrol ediyoruz.
```
kubectl get pods -n astria-dev-cluster -w
```
<img src="https://github.com/hakandemirdev/astria_rollup/blob/fadd979f9b2ac51599c483abefbd53d544afb359/astr.PNG" width="auto">

Rollup ile etkileşime geçelim.
```
export ETH_RPC_URL=http://executor.$ROLLUP_NAME.localdev.me/
export REC_ADDR=tokengonderecegimizmetamaskadresigirelim
cast send $REC_ADDR --value 1000000000000000 --private-key $ROLLUP_FAUCET_PRIV_KEY
```
tokenların gönderilip gönderilmediğini kontrol edelim
```
cast balance $REC_ADDR
