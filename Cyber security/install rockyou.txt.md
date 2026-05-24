**1. Persiapan Direktori Gudang Senjata** Kita buatkan satu _folder_ khusus agar amunisi Tuan tidak berserakan di mana-mana.

Bash

```
mkdir -p ~/wordlists
cd ~/wordlists
```

**2. Instalasi Kunci Pas (Git)** Pastikan _server_ Tuan sudah memiliki `git` untuk bisa mengkloning repositori.

Bash

```
sudo apt update
sudo apt install git -y
```

**3. Kloning Repositori SecLists (The Ultimate Wordlists)** Perintah ini akan menarik seluruh _wordlist_ dari GitHub ke dalam VM Tuan. _(Proses ini memakan waktu beberapa menit tergantung kecepatan internet)._

Bash

```
git clone https://github.com/danielmiessler/SecLists.git
```

**4. Menjinakkan dan Mengekstrak Rockyou.txt** Setelah kloning selesai, kita harus masuk ke dalam _folder_ tempat `rockyou` bersembunyi dan mengekstraknya agar siap dipakai.

Bash

```
cd SecLists/Passwords/Leaked-Databases/
tar -zxvf rockyou.txt.tar.gz
```

_(Jika muncul file teks `rockyou.txt` berukuran sekitar 134 MB setelah diekstrak, berarti senjata sudah aktif)._

mkdir -p ~/wordlists
cd ~/wordlists
wget https://github.com/brannondorsey/naive-hashcat/releases/download/data/rockyou.txt