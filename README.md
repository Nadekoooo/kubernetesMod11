# Refleksi Hello Minikube

## 1. Log Aplikasi sebelum vs. setelah expose Service
- Sebelum expose Service, log pod hanya menampilkan pesan startup HTTP/UDP dan tidak bertambah meski dicek berkali-kali.  
- Setelah menjalankan `kubectl expose deployment hello-node --type=LoadBalancer --port=8080` dan membuka aplikasi via `minikube service`, setiap request HTTP menimbulkan entri log baru (misal `HTTP GET /netexec`).  
- Jumlah entri log bertambah sesuai frekuensi refresh halaman, membuktikan Service berhasil meneruskan traffic eksternal ke pod.  

## 2. Tujuan opsi `-n` dan alasan resource tidak muncul
- Opsi `-n` (`--namespace`) menentukan namespace di mana `kubectl` akan mencari sumber daya.  
- Tanpa `-n`, `kubectl` otomatis menggunakan namespace `default`, tempat `hello-node` dideploy.  
- Saat menggunakan `kubectl get pods -n kube-system`, hanya resource di namespace `kube-system` (metrics-server, CoreDNS, dsb.) yang ditampilkan.  
- Karena `hello-node` ada di `default`, ia tidak muncul dalam daftar `kube-system`; untuk melihatnya kembali, cukup jalankan `kubectl get pods` tanpa `-n`, atau `kubectl get pods -n default`.  
