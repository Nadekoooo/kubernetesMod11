# Refleksi Hello Minikube

## 1. Perbandingan Log Aplikasi sebelum & sesudah expose Service
- **Sebelum expose Service**: Log hanya menampilkan  
`Started HTTP server on port 8080`
`Started UDP server on port 8081`
tanpa ada entri baru meski saya cek berulang kali.  
- **Setelah expose & proxy berjalan**: Saat membuka URL via `minikube service`, setiap permintaan HTTP menghasilkan baris log baru, misalnya:  
`GET /`
`GET /`
- Jumlah entri “GET” bertambah setiap kali saya refresh halaman, membuktikan Service berhasil meneruskan traffic eksternal ke pod.  

## 2. Tujuan opsi `-n` dan alasan resource tidak muncul
- Flag `-n` (`--namespace`) menentukan namespace yang dipakai `kubectl` untuk mencari resource.  
- Default tanpa `-n` mengacu ke namespace `default`, tempat Deployment `hello-node` berada.  
- Saat menjalankan `kubectl get pods -n kube-system`, hanya pod di namespace `kube-system` (metrics-server, CoreDNS, dll.) yang muncul.  
- Karena `hello-node` ada di namespace `default`, ia tidak tampil dalam daftar `kube-system`; untuk melihatnya kembali, gunakan `kubectl get pods` tanpa `-n` atau `kubectl get pods -n default`.  
