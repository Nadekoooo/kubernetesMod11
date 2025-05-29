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


# Refleksi Rolling Update & Manifest Kubernetes

## 1. Perbedaan Rolling Update dan Recreate  
- **Rolling Update** menggantikan Pod lama dengan Pod baru secara bertahap: Kubernetes akan membuat Pod baru, menunggu hingga siap, lalu mematikan satu Pod lama, dan seterusnya hingga semua replika menggunakan versi baru. Ini memungkinkan **zero-downtime** untuk aplikasi stateless.  
- **Recreate** mematikan **seluruh** Pod lama terlebih dahulu, kemudian baru membuat Pod baru setelah semuanya mati. Strategi ini menimbulkan **downtime singkat** namun menjamin tidak ada dua versi aplikasi berjalan bersamaan—berguna jika kita perlu migrasi skema database atau memastikan state tunggal.

## 2. Deploy dengan Recreate & Pengamatan  
- Saya membuat file `deployment-recreate.yaml` dengan menambahkan:
  ```yaml
  strategy:
    type: Recreate

Kemudian saya menjalankan

```
kubectl delete deployment spring-petclinic-rest
kubectl apply -f deployment-recreate.yaml
kubectl set image deployment/spring-petclinic-rest \
  spring-petclinic-rest=docker.io/springcommunity/spring-petclinic-rest:3.2.1

```

Saat memantau kubectl get pods --watch, terlihat Pod lama berstatus Terminating dan selesai shutdown sebelum Pod baru Pending → Running dibuat. Ini membuktikan strategi Recreate bekerja sesuai harapan.


## 3. Manifest untuk Recreate Strategy
menggunakan `deployment-recreate.yaml`

## 4. Manfaat Menggunakan Manifest

Manifests Kubernetes bersifat deklaratif dan membuat konfigurasi dapat diterapkan ulang kapan saja hanya dengan menjalankan `kubectl apply -f`. Dengan menyimpan semua pengaturan di file YAML, tim dapat menggunakan sistem version control seperti Git untuk melacak perubahan, melakukan audit, dan rollback dengan mudah. Perintah `kubectl apply` bersifat idempotent, sehingga menerapkannya berkali-kali tidak akan menciptakan resource duplikat. Format deklaratif ini juga memudahkan kolaborasi antar anggota tim, karena setiap orang bekerja dari satu sumber kebenaran yang sama. Integrasi manifest ke dalam pipeline CI/CD memungkinkan otomatisasi penuh proses build dan deploy. Selain itu, menggunakan manifest membantu memastikan bahwa lingkungan staging dan production memiliki konfigurasi yang identik. Dengan begitu, masalah “it works on my machine” dapat diminimalkan.  
