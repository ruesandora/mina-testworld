# mina-testworld


> Bu rehber Blok producerler için hazırlanmıştır. Bir hata ile karşılaşırsanız, lütfen Discord'daki #testworld-2 kanalı aracılığıyla iletişime geçin.

```console
# İlk olarak, platformunuz için rampup Debian Deposunu kurun ve güncelleyin.
sudo rm /etc/apt/sources.list.d/mina*.list

# CODENAME'i makineniz için uygun kod adı ile değiştirin ve çalıştırın
# Ubuntu 20 - focal
# Debian 11 - bullseye
# Debian 10 - buster

echo "deb [trusted=yes] http://packages.o1test.net/ CODENAME rampup" | sudo tee /etc/apt/sources.list.d/mina-rampup.list
sudo apt-get update

# Ardından, ihtiyacınız olan paketi veya paketleri yükleyin:
sudo apt-get install -y mina-berkeley=2.0.0rampup5-55b7818
```

```console
# Ufw kullanıyorsanız, şu izinlere izin verin
sudo ufw enable
sudo ufw allow 22
sudo ufw allow 8301
sudo ufw allow 3089
```

```console
# nodu kurduktan sonra anahtarlarınızı node'a eklemeniz gerekir.
# Lütfen tüm Ağ Performansı nodelarınızda aynı anahtar çiftini kullanın.
# Anahtar çiftlerinizi kimseyle paylaşmayın.
# Anahtar çifti size testnet üzerinde bir hisse tahsis edilmiş bir hesaba erişim sağlar.
# Bu pay ile test ağında blok üretiminize başlayabilirsiniz.

# Anahtarlarınızı ve şifrenizi içeren ekteki .zip dosyasını indirin.
# Sunucunuzda:

# xxx.zip dosyasını çıkarın
# Anahtar dosyalarını saklamak için sisteminizde bir klasör oluşturun. Biz ~/keys klasörünü öneriyoruz.
mkdir ~/keys

# Doğru izinleri ayarlayın.
chmod 700 ~/keys

# Özel anahtarınız için ~/keys içinde bir dosya oluşturun ve çıkarılan xxx.zip dosyasının içeriğini bu dosyaya ekleyin.
nano ~/keys/my-wallet

# community-yyy-key içeriğini ekle
# Dosyayı kaydet

# Doğru izinleri ayarlayın.
chmod 600 ~/keys/my-wallet

# Açık anahtarınız için bir dosya oluşturun ve çıkardığınız community-yyy.pub dosyasının içeriğini ekleyin.
nano ~/keys/my-wallet.pub 

# community-yyy.pub içeriğini ekle
# Dosyayı kaydet

# Yeni cüzdanınızın parolası, çıkardığınız community-yyy-password.txt dosyasında bulunabilir. 

```


```console
# Sonraki adım yeni libp2p anahtar çifti oluşturmaktır.
mina libp2p generate-keypair -privkey-path /root/keys/libkey

# Bir parola belirleyin. Ve bu şifre sizin olacak MINA_LIBP2P_PASS
# MINA_PRIVKEY_PASS ve UPTIME_PRIVKEY_PASS aynıdır ve
# ayıkladığınız community-yyy-password.txt dosyasında bulunabilir.

# Aşağıdaki "PASS" değişkenlerini (çift tırnakları silmeyin) yukarıdaki gibi değiştirin. 
RAYON_NUM_THREADS:6
UPTIME_PRIVKEY_PASS="PASS"
MINA_LIBP2P_PASS="PASS"
MINA_PRIVKEY_PASS="PASS"

# mina.env dosyasını oluşturun ve değişkenleri içine yapıştırın. Bu değişkenleri servis dosyasında belirterek de kullanabilirsiniz. 
# Ancak bunları tek bir yerde kullandığınızdan emin olun.
nano ~/.mina-env
```


```console
# Şimdi bir servis dosyası oluşturacağız. Bu şekilde mina çöktüğünde kendini yeniden başlatacak.
sudo nano /etc/systemd/system/mina.service

# ExecStart komutunu dikkatlice okuyun ve değişkenlerini kendi ayarlarınıza ve yollarınıza göre değiştirin.
# YOURIP'yi değiştirin.

[Unit]
Description=Mina Protocol
After=network.target

[Service]
User=root
EnvironmentFile=/root/.mina-env
ExecStart=/usr/local/bin/mina daemon --log-json --log-snark-work-gossip true --internal-tracing --insecure-rest-server --log-level Debug --file-log-level Debug --config-directory /root/.mina-config/ --external-ip YOURIP --itn-keys  f1F38+W3zLcc45fGZcAf9gsZ7o9Rh3ckqZQw6yOJiS4=,6GmWmMYv5oPwQd2xr6YArmU1YXYCAxQAxKH7aYnBdrk=,ZJDkF9EZlhcAU1jyvP3m9GbkhfYa0yPV+UdAqSamr1Q=,NW2Vis7S5G1B9g2l9cKh3shy9qkI1lvhid38763vZDU=,Cg/8l+JleVH8yNwXkoLawbfLHD93Do4KbttyBS7m9hQ= --itn-graphql-port 3089 --uptime-submitter-key  /root/keys/my-wallet --uptime-url https://block-producers-uptime-itn.minaprotocol.tools/v1/submit --metrics-port 10001 --enable-peer-exchange  true --libp2p-keypair /root/keys/libkey --log-precomputed-blocks true --max-connections 200 --peer-list-url  https://storage.googleapis.com/seed-lists/testworld-2-0_seeds.txt --generate-genesis-proof  true --block-producer-key /root/keys/my-wallet --node-status-url https://nodestats-itn.minaprotocol.tools/submit/stats  --node-error-url https://nodestats-itn.minaprotocol.tools/submit/stats  --file-log-rotations 500
Restart=always
RestartSec=3

[Install]
WantedBy=multi-user.target

```

```console
# Daemon'u yeniden yükleyin ve mina.service dosyasını etkinleştirin. Başlatın ve günlükleri gözlemleyin.

sudo systemctl daemon-reload
sudo systemctl enable mina
sudo systemctl start mina
sudo journalctl -u mina -n 1000 -f

# Birkaç dakika sonra düğüm çalışmaya başlayacaktır.
# ile son durumu alın;

mina client status
```
