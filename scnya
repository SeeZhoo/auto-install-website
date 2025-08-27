#!/bin/bash

# Pastikan skrip dijalankan sebagai root
if [ "$EUID" -ne 0 ]; then
  echo "Tolong jalankan skrip ini sebagai root."
  exit
fi

# Fungsi untuk membuat website
buat_website() {
    clear
    echo "--- OPSI 1: BUAT WEBSITE BARU ---"
    read -p "Masukkan nama domain (contoh: haii.seezhoo.my.id): " DOMAIN
    read -p "Masukkan URL repositori GitHub (contoh: https://github.com/user/repo.git): " GITHUB_REPO

    if [ -z "$DOMAIN" ] || [ -z "$GITHUB_REPO" ]; then
      echo "Nama domain atau URL repositori tidak boleh kosong."
      read -p "Tekan [Enter] untuk kembali ke menu..."
      return
    fi

    echo "Domain: $DOMAIN"
    echo "Repositori: $GITHUB_REPO"
    read -p "Yakin ingin melanjutkan? (y/n) " -n 1 -r
    echo
    if [[ ! $REPLY =~ ^[Yy]$ ]]; then
      echo "Proses dibatalkan."
      read -p "Tekan [Enter] untuk kembali ke menu..."
      return
    fi

    WEBSITE_PATH="/var/www/$DOMAIN"
    NGINX_CONF="/etc/nginx/sites-available/$DOMAIN"
    NGINX_SYMLINK="/etc/nginx/sites-enabled/$DOMAIN"

    echo "Memperbarui sistem..."
    apt update -y && apt upgrade -y

    echo "Menginstal paket yang diperlukan..."
    apt install -y nginx certbot python3-certbot-nginx git

    echo "Mengkloning repositori GitHub..."
    git clone "$GITHUB_REPO" "$WEBSITE_PATH"

    echo "Mengatur permissions..."
    chown -R www-data:www-data "$WEBSITE_PATH"
    chmod -R 755 "$WEBSITE_PATH"

    echo "Membuat konfigurasi Nginx..."
    cat <<EOF > "$NGINX_CONF"
server {
    listen 80;
    listen [::]:80;
    server_name $DOMAIN;
    root $WEBSITE_PATH;
    index index.html;

    location ~* /(assets|images|css|js)/$ {
        return 404;
    }

    location / {
        try_files \$uri \$uri/ =404;
    }
}
EOF

    echo "Mengaktifkan situs..."
    ln -s "$NGINX_CONF" "$NGINX_SYMLINK"

    echo "Menguji dan me-restart Nginx..."
    nginx -t
    if [ $? -eq 0 ]; then
        systemctl restart nginx
        echo "Nginx berhasil di-restart."
    else
        echo "Konfigurasi Nginx gagal."
        read -p "Tekan [Enter] untuk kembali ke menu..."
        return
    fi

    echo "Mendapatkan sertifikat SSL..."
    certbot --nginx --agree-tos --no-eff-email --redirect --hsts --staple-ocsp --email andi@seezhoo.my.id -d $DOMAIN

    echo "Selesai! Website di https://$DOMAIN sudah siap."
    read -p "Tekan [Enter] untuk kembali ke menu..."
}

# Fungsi untuk menghapus website
hapus_website() {
    clear
    echo "--- OPSI 2: HAPUS WEBSITE ---"
    read -p "Masukkan nama domain yang ingin dihapus: " DOMAIN

    if [ -z "$DOMAIN" ]; then
      echo "Nama domain tidak boleh kosong."
      read -p "Tekan [Enter] untuk kembali ke menu..."
      return
    fi

    read -p "Anda yakin ingin menghapus website $DOMAIN? (y/n) " -n 1 -r
    echo
    if [[ ! $REPLY =~ ^[Yy]$ ]]; then
      echo "Proses dibatalkan."
      read -p "Tekan [Enter] untuk kembali ke menu..."
      return
    fi

    NGINX_CONF="/etc/nginx/sites-available/$DOMAIN"
    NGINX_SYMLINK="/etc/nginx/sites-enabled/$DOMAIN"
    WEBSITE_PATH="/var/www/$DOMAIN"

    echo "Menghapus konfigurasi Nginx..."
    rm -f "$NGINX_SYMLINK"
    rm -f "$NGINX_CONF"

    echo "Menghapus sertifikat Certbot..."
    certbot delete --non-interactive --cert-name "$DOMAIN"

    echo "Menghapus folder website..."
    rm -rf "$WEBSITE_PATH"

    echo "Menguji dan me-restart Nginx..."
    nginx -t
    systemctl restart nginx

    echo "Selesai! Website $DOMAIN telah berhasil dihapus."
    read -p "Tekan [Enter] untuk kembali ke menu..."
}

# Fungsi untuk menampilkan menu utama
tampilkan_menu() {
    while true; do
        clear
        echo "--- PILIH OPSI ---"
        echo "1. Buat Website Baru"
        echo "2. Hapus Website"
        echo "3. Keluar"
        echo "-------------------"
        read -p "Pilihan kamu: " PILIHAN

        case "$PILIHAN" in
            1)
                buat_website
                ;;
            2)
                hapus_website
                ;;
            3)
                echo "Terima kasih, sampai jumpa!"
                exit 0
                ;;
            *)
                echo "Pilihan tidak valid."
                read -p "Tekan [Enter] untuk kembali ke menu..."
                ;;
        esac
    done
}

# Jalankan menu utama
tampilkan_menu
