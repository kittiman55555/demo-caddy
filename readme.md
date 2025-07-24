
---

# 🚀 Workshop Reverse Proxy Setup with Caddy

ระบบนี้ใช้ **Caddy** ในการทำ Reverse Proxy ให้กับ service ชื่อ `web` ซึ่งรันอยู่ใน container บน port `3000`
พร้อม domain สาธารณะผ่าน `duckdns.org`

## 🧾 โครงสร้าง

* **Caddy** ทำหน้าที่เป็น reverse proxy และรับ traffic ผ่าน HTTPS (Auto TLS)
* **Web** คือ container ที่ให้บริการเว็บจริงๆ บน port 3000
* ใช้ Docker Compose และใช้ network เดียวกัน (`web`) แบบ external

---

## 📁 File ที่สำคัญ

### `Caddyfile`

```caddyfile
demoworkshop.duckdns.org {
	reverse_proxy web:3000
}
```

### `compose.yaml` (Caddy)

```yaml
services:
  caddy:
    image: caddy:latest
    container_name: caddy_container
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile
      - caddy_data:/data
      - caddy_config:/config
    networks:
      - web

networks:
  web:
    external: true

volumes:
  caddy_data:
  caddy_config:
```

### `compose.yaml` (Web Service)

```yaml
services:
  web:
    container_name: web
    image: kittiman555555/workshop:latest
    platform: linux/amd64
    ports:
      - "3000:3000"
    networks:
      - web

networks:
  web:
    external: true
```

---

## ✅ วิธีใช้งาน

### 1. สร้าง external network ชื่อ `web` (ถ้ายังไม่มี)

```bash
docker network create web
```

### 2. รัน Web Service

```bash
docker compose -f compose.web.yaml up -d
```

> เปลี่ยนชื่อไฟล์หากไฟล์ของคุณไม่ได้ชื่อ `compose.web.yaml`

### 3. รัน Caddy

```bash
docker compose -f compose.caddy.yaml up -d
```

> อย่าลืมแก้ domain ใน `Caddyfile` ให้ตรงกับโดเมนจริงบน DuckDNS ที่คุณตั้งไว้

---

## 🌐 ทดลองใช้งาน

เมื่อทุกอย่างทำงานถูกต้อง ให้เปิดเว็บ:

👉 [https://demoworkshop.duckdns.org](https://demoworkshop.duckdns.org)

---

## 🛠 Debug & Tips

* ตรวจสอบ log ของ Caddy:

```bash
docker logs -f caddy_container
```

* เช็กว่า container ทั้งสองอยู่ใน network เดียวกัน:

```bash
docker network inspect web
```

---

## 📌 หมายเหตุ

* Caddy จะออก SSL cert ให้อัตโนมัติเมื่อโดเมนชี้มายังเครื่องนี้จริง
* DuckDNS ต้องตั้งค่าให้ domain ชี้ IP มายังเครื่องที่รัน Caddy ด้วย

---

