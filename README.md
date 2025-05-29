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
- Saya mengamati bahwa jumlah baris log “GET” terus bertambah sesuai jumlah permintaan yang dikirim, yang menegaskan bahwa Service dan tunnel Minikube berhasil memforward HTTP request dari luar cluster ke container. Dengan demikian, expose Service tidak hanya membuat pod “hidup”, tetapi juga mengaktifkan endpoint HTTP sehingga trafik nyata tercatat di log aplikasi. Proses ini menunjukkan perbedaan nyata antara pod yang berjalan dalam isolasi (tanpa Service) dan pod yang disajikan ke dunia luar melalui LoadBalancer.

## 2. Tujuan opsi `-n` dan alasan resource tidak muncul
- Flag -n atau --namespace pada perintah kubectl get berguna untuk menentukan namespace spesifik dalam cluster di mana resource dicari. Namespace di Kubernetes berfungsi sebagai virtual cluster—memisahkan objek (pod, service, configmap, dsb.) ke dalam domain logis yang terisolasi, memungkinkan manajemen RBAC, quota resource, dan isolasi nama. Pada tutorial, pertama kali saya menjalankan kubectl get pods tanpa opsi, sehingga konteks default (default namespace) digunakan, menampilkan hello-node beserta podnya. Kemudian saya mengecek namespace kube-system dengan kubectl get pods -n kube-system, dan hanya melihat core component seperti coredns dan metrics-server. Karena hello-node memang dideploy di default, ia tidak muncul di hasil -n kube-system. Untuk mengakses kembali pod aplikasi, saya bisa menjalankan kubectl get pods tanpa -n atau kubectl get pods -n default. Penggunaan namespace ini penting untuk membedakan resource antar tim atau lingkungan (misal development vs production) di satu cluster yang sama.
