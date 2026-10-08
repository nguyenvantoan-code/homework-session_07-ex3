# Bai 3: Thiet lap Co so du lieu va Tu cau hinh dich vu Systemd cho Spring Boot

## Muc tieu

- Khoi tao co so du lieu MySQL va cau hinh phan quyen nguoi dung bao mat.
- Tu thiet ke va viet tep tin cau hinh dich vu Systemd hoan chinh tu dau.
- Thuc thi ung dung duoi quyen nguoi dung gioi han (non-root).

---

## Buoc 1: Khoi tao MySQL Database va User

Dang nhap vao MySQL voi quyen root:

```bash
sudo mysql -u root -p
```

Thuc thi cac lenh SQL sau:

```sql
-- Tao co so du lieu
CREATE DATABASE springboot_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- Tao user spring-admin voi mat khau bao mat
CREATE USER 'spring-admin'@'localhost' IDENTIFIED BY 'SpringSecure@123';

-- Cap toan quyen tren springboot_db cho spring-admin
GRANT ALL PRIVILEGES ON springboot_db.* TO 'spring-admin'@'localhost';

-- Ap dung thay doi phan quyen
FLUSH PRIVILEGES;

-- Kiem tra phan quyen (tuy chon)
SHOW GRANTS FOR 'spring-admin'@'localhost';
```

---

## Buoc 2: Tao User He Thong spring-runner

Tao nguoi dung Linux khong co shell login (bao mat hon):

```bash
sudo useradd --system --no-create-home --shell /usr/sbin/nologin spring-runner
```

Xac nhan user da duoc tao:

```bash
id spring-runner
# Ket qua mong doi:
# uid=999(spring-runner) gid=999(spring-runner) groups=999(spring-runner)
```

---

## Buoc 3: Chuan bi Thu muc Ung dung

```bash
# Tao thu muc chua ung dung
sudo mkdir -p /opt/spring-app

# Sao chep file JAR vao thu muc
sudo cp app.jar /opt/spring-app/app.jar

# Cap quyen so huu cho spring-runner
sudo chown -R spring-runner:spring-runner /opt/spring-app

# Dat quyen thuc thi
sudo chmod 750 /opt/spring-app
sudo chmod 640 /opt/spring-app/app.jar
```

---

## Buoc 4: Cai dat Tep tin Dich vu Systemd

Sao chep tep tin service vao thu muc systemd:

```bash
sudo cp spring-app.service /etc/systemd/system/spring-app.service
```

Kiem tra noi dung tep tin:

```bash
cat /etc/systemd/system/spring-app.service
```

Nap lai daemon de systemd nhan biet dich vu moi:

```bash
sudo systemctl daemon-reload
```

---

## Buoc 5: Khoi dong va Kich hoat Dich vu

```bash
# Khoi dong dich vu
sudo systemctl start spring-app.service

# Cau hinh tu dong khoi dong cung he thong
sudo systemctl enable spring-app.service
```

---

## Ket qua Kiem tra

### Kiem tra trang thai dich vu

Lenh:
```bash
sudo systemctl status spring-app.service
```

Ket qua mong doi:
```
* spring-app.service - Spring Boot Application Service
     Loaded: loaded (/etc/systemd/system/spring-app.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2026-10-08 19:00:00 UTC; 1min ago
   Main PID: 1234 (java)
      Tasks: 30 (limit: 4915)
     Memory: 256.0M
        CPU: 5.123s
     CGroup: /system.slice/spring-app.service
             └─1234 /usr/bin/java -jar /opt/spring-app/app.jar --server.port=8082

Oct 08 19:00:00 hostname spring-app[1234]: Started Application in 5.123 seconds
```

> Dich vu hien thi trang thai **active (running)**.

---

### Kiem tra cong lang nghe 8082

Lenh:
```bash
ss -tlnp | grep 8082
```

Ket qua mong doi:
```
LISTEN  0  100  0.0.0.0:8082  0.0.0.0:*  users:(("java",pid=1234,fd=10))
```

> Cong 8082 dang duoc lang nghe boi tien trinh Java chay duoi quyen **spring-runner**.

---

## Giai thich Tep tin spring-app.service

| Tham so | Gia tri | Y nghia |
|---------|---------|---------|
| After | 
etwork.target mysql.service | Khoi dong sau khi mang va MySQL san sang |
| User | spring-runner | Chay duoi user gioi han, khong phai root |
| ExecStart | /usr/bin/java -jar /opt/spring-app/app.jar | Lenh khoi chay ung dung |
| Restart | on-failure | Chi restart khi crash bat ngo |
| RestartSec | 10 | Cho 10 giay truoc khi restart |
| NoNewPrivileges | 	rue | Ngan ung dung leo thang quyen |
| PrivateTmp | 	rue | Thu muc /tmp rieng, bao mat hon |
| WantedBy | multi-user.target | Tu dong khoi dong khi he thong boot |

---

## Xem Log Ung dung

```bash
# Xem log theo thoi gian thuc
sudo journalctl -u spring-app.service -f

# Xem 100 dong log gan nhat
sudo journalctl -u spring-app.service -n 100
```
