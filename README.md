# mssql-server-audit-configuration
Microsoft sql server configuration for logs 

### Gereksinimler 

- MS SQL Server kurulu bir makine
- Microsoft Sql Server Management Studio
- Server'da Owner yetkisine sahip bir kullanıcı

### Giriş

SQL server audit işlemi için 2 farklı loglama biçimi sunmaktadır. Bunlar Windows Event Log ve Binary File formatlarıdır. Windows event log formatında kayıt yapmak mantıklı gözükmektedir. Çünkü birçok log yönetim çözümü bu formatı desteklemektedir. Ancak güvenlik ve performans açısından daha çok önerilen yöndem binary file formatı ile kayıt etmekdir. Dosyaya yazmak daha hızlı olduğu için performans sebebiyle Windows tarafından da bu yöntem önerilir. Bu kayıtların ele geçirilmiş yönetici hesaplarından gelebilecek olası saldırılar gibi çeşitli sebeplerle dosyalar halinde ve farklı bir server üzerinde tutulması en güvenli ve hızlı yol olacaktır.

### Audit Konfigürasyonu

-  Audit konfigürasyonu yapmak için SQL Server Management Studio açılır.
- <b>Object Explorer</b> üzerinde <b> Security -> Audits </b> yolu izlenir.
- <b>Audits</b> nodu üzerine sağ tıklanıp <b>New Audit</b> seçeneği seçildiğinde karşımıza <b>Create Audit</b> penceresi çıkar.

  ![create_audit](https://user-images.githubusercontent.com/28602326/110800957-cb6f4a00-828d-11eb-9075-055901e93257.png) ![new_audit_screen](https://user-images.githubusercontent.com/28602326/110801024-e04bdd80-828d-11eb-8607-3f096eb81daf.png)


- <b>Audit name</b> kısmına Audit objesinin ismi yazılır. <b>Destination</b> kısmında File, Security log,  Application log seçenekleri seçilebilir. File seçeneğini seçerek dosyanın kayıt edilmesi gerektiği yeri <b>File Path</b> bölümünde gösterebiliriz. Security ve Application log seçenekleri ile ise logların windows event log formatında olması sağlanabilir.

#### Server Bazlı Audit Kurallarını Belirleme
- Yeni Audit oluşturma işleminden sonra hangi olayların loglanacağını belirlemek için Object Explorer üzerinde <b>Server Audit Specification</b> nodu üzerine sağ tıklanıp <b>New Server Audit Specification</b> seçeneği seçilir. <b>Audit Action Type</b> içerisinden loglanmasını istediğimiz olaylar seçilir.

![Server_Audit_Specification](https://user-images.githubusercontent.com/28602326/110805478-2acf5900-8292-11eb-861c-5924ac0b619b.png)

- Burada bir önceki bölümde oluşturulan Audit objesine verilen isim seçilmelidir. Güvenlik ihlallerinin tespiti açısından mümkün olduğunca fazla bilgi olması işimize gelir ancak performans ve bazı diğer sebeplerden dolayı bu çoğu zaman mümkün olmamaktadır. Güvenlik açısından bakıldığında aşağıdaki olayların işaretlenmesi genel olarak önerilmektedir:
 ###### FAILED_LOGIN_GROUP 
 ###### SUCCESSFUL_LOGIN_GROUP 
 ###### DATABASE_OBJECT_CHANGE_GROUP
 ###### DATABASE_PRINCIPAL_CHANGE_GROUP 
 ###### SCHEMA_OBJECT_CHANGE_GROUP 
 ###### SERVER_PRINCIPAL_CHANGE_GROUP 
 ###### LOGIN_CHANGE_PASSWORD_GROUP 
 ###### SERVER_STATE_CHANGE_GROUP 
- Bu şekilde kayıt ettikten sonra Audit Object ve Audit Specifications nodelarına sağ tıklanarak <b>Enable</b> seçeneğine tıklanarak aktif hale getirilmelidir. Artık belirttiğimiz olayların logları tutulacaktır.  

#### Database Bazlı Audit Kurallarını Belirleme

- Loglama işlemleri veritabanı bazında da gerçekleştirilebilir. Bu şekilde istenilen tabloların loglarını açıp/kapatabiliriz.
- <b>Object Explorer</b> üzerinde <b> Databases -> [Veritabanınız] -> Security </b> yolu izlenir.
- <b>Database Audit Specification</b> nodu üzerine sağ tıklanıp <b>New Database Audit Specification</b> seçeneği seçilir.

![DB_Audit_Spec](https://user-images.githubusercontent.com/28602326/110807662-52bfbc00-8294-11eb-94b8-b2b2feced092.png)

- Burada belirtilecek Audit Action tipleri tüm veritabanını gözetleyecek şekilde aramak istenirse <b>Object Class</b> kısmı Database , <b>Object Name</b> kısmı
loglamak istenilen veritabanı seçilir. Tüm kullanıcıların izlenebilmesi için <b>Principal Name</b> public seçilmelidir.
- Audit action tablo bazında loglamak istenirse <b>Object Class</b> kısmı Object seçilir <b>Object Name</b> ' e loglamak istenilen tablo seçilir.

### Logları İnceleme

Oluşturulan Logları incelemek için :
- <b>Object Explorer</b> üzerinde <b> Security -> Audits </b> yolu izlenir.
- <b>Audits</b> nodu üzerine sağ tıklanıp <b>View Audit Logs</b> seçeneği seçilerek loglar incelenebilir, <b>csv</b> , <b>txt</b> ve <b>log</b> formatlarında export edilebilir. 
- Oluşturulan logların zamanı ile sistem zamanı arasında 3-5 saat arası fark olabilir. Bunun sebebi logların <b>UTC</b> saatine göre oluşturulmasıdır. İstenirse sistem saatine göre log raporu oluşturulabilir. Bunun için Sql komut satırına :

```sql
SELECT DATEADD(MINUTE, DATEDIFF(MINUTE, GETUTCDATE(), CURRENT_TIMESTAMP), event_time) AS event_time_afterconvert
    ,getdate() 'Current_system_time'
    ,*
FROM fn_get_audit_file('C:\PATH\*', DEFAULT, DEFAULT)
```
yazmak yeterlidir.
