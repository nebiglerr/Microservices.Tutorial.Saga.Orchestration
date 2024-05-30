# Order Management Microservices

Order Management Microservices, sipariş yönetimi, ödeme işlemleri ve stok yönetimi için yazılmış bir .NET Core mikroservis uygulamasıdır. Bu proje, siparişlerin oluşturulması, güncellenmesi ve tamamlanması süreçlerini yönetir. Ayrıca, ödeme ve stok yönetimi süreçleri de entegre edilmiştir.

## Özellikler

- Sipariş oluşturma ve yönetimi
- Ödeme işlemleri
- Stok yönetimi ve stok rezervasyonu
- Saga desenine dayalı sipariş iş akışı yönetimi
- RabbitMQ ile asenkron mesajlaşma

## Teknolojiler

- .NET 8 SDK
- Entity Framework Core
- MassTransit
- RabbitMQ
- MSSQL Server
- MongoDB
- Docker (opsiyonel)

## Kurulum

### Gereksinimler

Bu projeyi çalıştırmak için aşağıdaki araçlar gereklidir:
- .NET 8 SDK
- MSSQL Server
- RabbitMQ
- MongoDB

### Adımlar

1. Depoyu klonlayın:

    ```bash
    git clone https://github.com/kullaniciadi/order-management-microservices.git
    cd order-management-microservices
    ```

2. Gerekli bağımlılıkları yükleyin:

    ```bash
    dotnet restore
    ```

3. MSSQL Server, RabbitMQ ve MongoDB bağlantı ayarlarını `appsettings.json` dosyasında yapılandırın:

    ```json
    {
      "ConnectionStrings": {
        "MSSQLServer": "Server=your_server;Database=your_database;User Id=your_user;Password=your_password;"
      },
      "RabbitMQ": "rabbitmq_connection_string",
      "MongoDB": "mongodb_connection_string"
    }
    ```

4. Veritabanı migrasyonlarını uygulayın:

    ```bash
    dotnet ef database update -p Order.API
    ```

5. Uygulamayı çalıştırın:

    ```bash
    dotnet run --project Order.API
    dotnet run --project Payment.API
    dotnet run --project Stock.API
    dotnet run --project SagaStateMachine.Service
    ```

Opsiyonel olarak Docker kullanarak çalıştırabilirsiniz:

```bash
docker-compose up
```

Kullanım
Sipariş Oluşturma
POST isteği ile /create-order endpointine sipariş bilgilerini gönderin:

```json
curl -X POST "https://localhost:5001/create-order" -H "Content-Type: application/json" -d '{
  "BuyerId": 1,
  "OrderItems": [
    {
      "ProductId": 1,
      "Count": 2,
      "Price": 10.5
    },
    {
      "ProductId": 2,
      "Count": 1,
      "Price": 20.0
    }
  ]
}'
```
**Ödeme İşlemleri**
Ödeme işlemleri Payment.API mikroservisi tarafından yönetilir ve PaymentStartedEvent tetiklenerek başlatılır. Ödeme başarılı olursa PaymentCompletedEvent, başarısız olursa PaymentFailedEvent tetiklenir.

**Stok Yönetimi**
Stok yönetimi Stock.API mikroservisi tarafından gerçekleştirilir. OrderCreatedEvent tetiklendiğinde stok kontrol edilir ve yeterli stok varsa StockReservedEvent, yoksa StockNotReservedEvent tetiklenir.

**Mimari**
Proje, mikroservis mimarisi ile tasarlanmıştır ve her bir servis kendi veritabanına sahiptir. Saga desenine dayalı olarak sipariş iş akışları yönetilir. RabbitMQ kullanılarak servisler arasında asenkron iletişim sağlanır.

**Proje Yapısı**
Order.API: Sipariş işlemlerinin yönetildiği mikroservis.

OrderDbContext: Veritabanı bağlamı ve DbSet tanımları içerir.
OrderCompletedEventConsumer, OrderFailedEventConsumer: Sipariş durumlarını yönetir.
Modeller, ViewModeller ve Enumlar sipariş verilerini temsil eder.
Program.cs: Uygulamanın başlangıç noktası.
Payment.API: Ödeme işlemlerinin yönetildiği mikroservis.

PaymentStartedEventConsumer: Ödeme işlemlerini yönetir.
Ödeme tamamlandığında veya başarısız olduğunda ilgili olayları tetikler.
Program.cs: Uygulamanın başlangıç noktası.
Stock.API: Stok yönetiminin yapıldığı mikroservis.

MongoDbService: MongoDB ile etkileşim sağlar.
OrderCreatedEventConsumer, StockRollbackMessageConsumer: Stok durumlarını yönetir.
Program.cs: Uygulamanın başlangıç noktası.
SagaStateMachine.Service: Saga durum makinelerini ve ilgili bileşenleri içerir.

OrderStateMachine, OrderStateInstance, OrderStateDbContext, OrderStateMap: Sipariş durumu makinelerini ve ilişkili veritabanı yapılarını içerir.
Program.cs: Uygulamanın başlangıç noktası.
Shared: Ortak mesajlar ve ayarlar.

Messages: Ortak mesaj sınıfları.
OrderEvents, PaymentEvents, StockEvents: Sipariş, ödeme ve stok olaylarını tanımlar.
RabbitMQSettings: RabbitMQ ayarlarını içerir.