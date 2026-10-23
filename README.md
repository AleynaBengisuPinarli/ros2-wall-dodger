# ROS 2 TurtleSim Otonom Duvar Kaçınma Projesi (`turtle_duvar_py`)

Bu depo, ROS 2 Humble ve Python altyapısı kullanılarak geliştirilmiş otonom bir simülasyon uygulamasıdır. `turtlesim` ortamındaki kaplumbağanın anlık konum verilerini takip ederek sınır çizgilerine çarpmasını önler ve güvenli bir şekilde manevra yapmasını sağlar.

---

## 📌 Temel Özellikler ve Mimari

- **Hibrit Düğüm Yapısı (`duvar_kacis_node`):** Tek bir ROS 2 düğümü içerisinde hem abone (`/turtle1/pose` üzerinden konum takibi) hem de yayıncı (`/turtle1/cmd_vel` üzerinden hareket komutları) mekanizmaları eş zamanlı çalışır.

- **Akıllı Sınır Algoritması:** Kaplumbağa belirlenen güvenli sınırların dışına çıktığında ve yönü duvara dönükse, doğrusal hızı kesilip açısal hızla iç kısımlara doğru döner; böylece köşelerde veya kenarlarda takılıp kalma (daire çizme) problemi ortadan kalkar.

- **Zamanlayıcı Destekli Loglama:** `create_timer` yardımıyla her 1 saniyede bir `get_logger().info()` fonksiyonu tetiklenerek anlık koordinatlar konsola aktarılır.

- **Dinamik Parametre Yönetimi (Bonus):** Hızlar ve çalışma sınırları kod içerisinden bağımsız olarak ROS 2 parametreleri aracılığıyla değiştirilebilir:

| Parametre Adı | Varsayılan Değer | Açıklama |
| :--- | :--- | :--- |
| `linear_speed` | `1.5` | Standart ileri hareket hızı |
| `angular_speed` | `2.0` | Kaçış anındaki dönüş hızı |
| `min_limit` | `1.0` | Alt koordinat eşiği (X ve Y ekseni) |
| `max_limit` | `10.0` | Üst koordinat eşiği (X ve Y ekseni) |

---

## 🖼️ Sistem Çıktıları ve Görseller

### 1. Simülasyon Ortamı
Aşağıdaki görselde kaplumbağanın simülasyon ortamındaki hareket yörüngesi görülmektedir. Kaplumbağa ekranın ortasından ileri doğru hareket etmeye başlamış, sağ kenarda x ≈ 10.0 sınırına ulaştığında dönerek yön değiştirmiştir. Aynı davranış diğer kenarlarda da tekrar ettiği için kaplumbağa duvara çarpmadan ekranın kenarlarını dolaşmakta, basit sınır kontrollerinde yaşanan köşede takılıp kalma veya daire çizme sorunlarını başarıyla aşmaktadır:

![turtlesim](turtlesim.jpeg)

### 2. Simülatör Başlangıç Ekranı
`ros2 run turtlesim turtlesim_node` komutuyla başlatılan simülatör düğümünün terminal çıktısıdır. Düğüm başlatıldığında kaplumbağa (`turtle1`) ekranın tam ortasında, yaklaşık $x = 5.54$, $y = 5.54$ konumunda ve $\theta = 0$ açısıyla otomatik olarak oluşturulur. Düğümümüz anlık konum verilerini `/turtle1/pose` topiği üzerinden dinlemeye bu aşamada başlar:

![turtlesim_node](turtlesim_node.jpeg)

### 3. Konsol Log Akışı
`duvar_kacis_node` düğümünün terminal çıktısıdır. Düğüm, zamanlayıcı mekanizması sayesinde her saniye `[INFO]` seviyesinde mevcut konumu konsola yazdırmaktadır. Loglar incelendiğinde kaplumbağanın $y \approx 5.54$ sabitken $x$ değerinin arttığı, sınır değerine ulaştığında açısal dönüş yaparak yön değiştirdiği net bir şekilde görülmektedir:

![node_log](node_log.jpeg)

### 4. Hız Komutları İzleme (`/turtle1/cmd_vel`)
Düğümün simülatöre gönderdiği hız komutlarının `ros2 topic echo` yardımıyla anlık olarak izlendiği terminal görüntüsüdür. Loglarda `linear.x` değerinin sabit olarak `1.5` birim tutulduğu, dönüş anlarında ise açısal hız komutlarının (`angular.z`) devreye girerek kaplumbağayı güvenli bölgeye yönlendirdiği doğrulanmaktadır:

![cmd_vel_echo](cmd_vel.jpeg)

### 5. Konum ve Pozisyon Verisi İzleme (`/turtle1/pose`)
Kaplumbağanın simülatörden yayınladığı anlık koordinatların ve hız vektörlerinin (`x`, `y`, `theta`, `linear_velocity`, `angular_velocity`) `ros2 topic echo /turtle1/pose` komutuyla konsola yansıyan ham veri akışıdır:

![pose_echo](pose.jpeg)

---

## ⚙️ Algoritma Çalışma Mantığı

1. Düğüm aktifleştiğinde `/turtle1/pose` topiğini dinlemeye başlar (`pose_callback`).
2. Gelen anlık `x` ve `y` değerleri, tanımlanan minimum ve maksimum sınır aralığıyla kıyaslanır.
3. Eğer konum sınır dışındaysa ve hareket vektörü duvara doğru yönelmişse, doğrusal hız sıfırlanarak (`linear.x = 0`) yalnızca açısal dönüş komutu (`angular.z`) uygulanır.
4. Sınır içerisindeyken normal doğrusal sürüş (`linear_speed`) devam ettirilir.
5. Üretilen `geometry_msgs/msg/Twist` mesajı `/turtle1/cmd_vel` üzerinden simülatöre iletilir.

---

## 🚀 Kurulum ve Çalıştırma

Projeyi kendi çalışma alanınızda (`ros2_ws/src`) çalıştırmak için aşağıdaki adımları izleyebilirsiniz:

1. **Depoyu klonlayın:**
   ```bash
   cd ~/ros2_ws/src
   git clone [https://github.com/AleynaBengisuPinarli/ros2-wall-dodger.git](https://github.com/AleynaBengisuPinarli/ros2-wall-dodger.git) turtle_duvar_py
