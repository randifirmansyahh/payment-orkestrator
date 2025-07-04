# Payment Orchestrator (NestJS)

## Setup
- Copy `.env.example` to `.env` and configure DB URL
- Install dependencies
```bash
npm install
```

## Install & Generate Prisma Client
```bash
npm install prisma @prisma/client
npx prisma generate
```

## Start Dev
```bash
npm run start:dev
```

## Folders Structure
```
📦 root
├── prisma/
│   └── schema.prisma
│
├── src/
│   ├── app.module.ts
│   ├── main.ts
│   │
│   ├── payin/
│   │   ├── dto/
│   │   │   └── create-payin.dto.ts
│   │   ├── controller/
│   │   │   ├── payin.controller.ts
│   │   │   └── webhook.controller.ts
│   │   ├── service/
│   │   │   ├── payin.service.ts
│   │   │   ├── gateway-resolver.service.ts
│   │   │   └── webhook-handler.service.ts
│   │   ├── adapter/
│   │   │   ├── finmo/
│   │   │   │   ├── finmo.adapter.ts
│   │   │   │   ├── payload-builder/
│   │   │   │   │   ├── va.builder.ts
│   │   │   │   │   └── qris.builder.ts
│   │   │   │   └── normalizer/
│   │   │   │       ├── va.normalizer.ts
│   │   │   │       └── qris.normalizer.ts
│   │   │   └── localpayment/
│   │   │       └── localpayment.adapter.ts
│   │   ├── interface/
│   │   │   ├── i-payin-payload-builder.ts
│   │   │   └── i-payin-response-normalizer.ts
│   │   └── payin.module.ts
│   │
│   ├── payout/
│   │   ├── dto/
│   │   │   └── create-payout.dto.ts
│   │   ├── controller/
│   │   │   └── payout.controller.ts
│   │   ├── service/
│   │   │   └── payout.service.ts
│   │   ├── adapter/
│   │   │   ├── finmo/
│   │   │   │   ├── finmo-payout.adapter.ts
│   │   │   │   ├── payload-builder/
│   │   │   │   │   └── va.builder.ts
│   │   │   │   └── normalizer/
│   │   │   │       └── va.normalizer.ts
│   │   ├── interface/
│   │   │   ├── i-payout-payload-builder.ts
│   │   │   └── i-payout-response-normalizer.ts
│   │   └── payout.module.ts
│   │
│   ├── merchant/
│   │   ├── middleware/
│   │   │   └── api-key.middleware.ts
│   │   ├── service/
│   │   │   └── merchant-config.service.ts
│   │   └── merchant.module.ts
│   │
│   ├── shared/
│   │   ├── db/
│   │   │   ├── prisma.service.ts
│   │   │   └── db-generic-sync-writer.service.ts
│   │   ├── interface/
│   │   │   └── gateway.interface.ts
│   │   ├── constant.ts
│   │   └── shared.module.ts
│   │
│   └── job/
│       ├── retry-job.service.ts
│       └── job.module.ts
│   
├── .env.example
├── .gitignore
├── folder-structure.ini
├── package-lock.json
├── package.json
├── README.md
└── tsconfig.json
```

## ⚙️ Prisma Multi-Database

### Koneksi:
- `central` client → default Prisma Client.
- `merchant` client → dibuat dinamis berdasarkan `merchant_id` dari `central` DB.

---
## 🧩 Menambahkan Flow Baru di Payment Orchestrator

Dokumen ini menjelaskan langkah-langkah untuk membuat flow baru (seperti `payin`, `payout`, `refund`, dll) dan integrasi dengan metode pembayaran baru (seperti `QRIS`, `VA`, dll).

## ✅ Langkah-langkah
### Menggunakan make command:
#### 1. generate template
```
make generate-flow
```
#### 2. isi Flow name (e.g. payin, payout, etc)
```
refund
```
#### 3. isi Payment Provider atau Gateway (comma-separated, e.g. finmo,localpayment)
```
finmo, localpayment, kpay, octopay
```
#### 4. isi Payment Methods (comma-separated, e.g. QRIS,VA)
```
QRIS, VA, TRASNFERBANK
```
#### 5. tambah gateway, method & currency jika belum tersedia
```
src\shared\constant.ts 
```
example : 
```
export const FINMO_GATEWAY_NAME: GATEWAY_NAME = 'FINMO';
export const SUPPORTED_REFUND_METHODS = ['VA', 'QRIS'];
export const SUPPORTED_REFUND_CURRENCIES = ['IDR', 'PHP', 'USD'];
```
#### 6. adjust normalize (mapping response psp to our standard pattern response)
- adjust normalize(raw any) function di ```src\{flow}\adapter\{gateway}\normalizer\{method}.normalizer.ts```
- dan sesuaikan modelnya di ```src\{flow}\dto\create-{flow}.dto.ts```

#### 7. adjust build (mapping payload internal ke aturan mapping gateway)
- adjust build(dto: Create{flow}Dto, merchant: merchant_users) function di ```src\{flow}\adapter\{gateway}\payload-builder\{method}.builder.ts```

#### 8. map the builder, normalizer and external request API to gateway
- mapping bulder dan normalizer yang sebelumnya dibuat, dan buat fungsi untuk request API ke gateway di ```src\{flow}\adapter\{gateway}\{gateway}.adapter.ts```

### 9. (Optional) handle webhook controller
- ```src\{flow}\controller\webhook-{flow}.controller.ts```

### 10. adjust property and method to mapping merchant request, mapping response, normalizer, and mapping to our table
- ```src\{flow}\dto\create-{flow}.dto.ts```

---

## 🧩 Manual Flow
### 1. Jalankan Migrasi Database
```bash
npx prisma migrate dev --name init
````

### 2. Bersihkan dan Install Ulang Prisma Client

```bash
rm -rf node_modules/@prisma/client
npm install prisma @prisma/client
```

### 3. Buat Model Baru di Prisma

Edit file `prisma/schema.prisma` dan tambahkan model baru sesuai kebutuhan flow baru (contoh: `Refund`, `Payout`).

Lalu jalankan:

```bash
npx prisma generate
```

---

## 🛠 Struktur File Flow Baru

Misal flow-nya adalah `payout`, dan akan mendukung `VA` dari gateway `xendit`.

### 4. Tambah Flow Baru

```bash
src/payout/
```

### 5. Update Constant

Edit file berikut:

```ts
src/shared/constant.ts
```

Tambahkan method atau currency baru jika diperlukan:

```ts
export const SUPPORTED_PAYIN_METHODS = ['VA', 'QRIS'];
export const SUPPORTED_CURRENCIES = ['IDR', 'PHP'];
```

---

### 6. Buat DTO

```ts
src/payout/dto/create-payout.dto.ts
```

### 7. Buat Interface Payload Builder

```ts
src/payout/interface/i-payout-payload-builder.ts
```

### 8. Buat Interface Response Normalizer

```ts
src/payout/interface/i-payout-payload-normalize.ts
```

---

### 9. Buat Normalizer

```ts
src/payout/adapter/xendit/normalizer/va.normalizer.ts
```

### 10. Buat Payload Builder

```ts
src/payout/adapter/xendit/payload-builder/va.builder.ts
```

---

### 11. Buat Interface Global (jika belum)

```ts
src/shared/gateway.interface.ts
```

---

### 12. Register Builder & Normalizer ke Adapter

Edit:

```ts
src/payout/adapter/xendit/payout.adapter.ts
```

Tambahkan ke dalam mapping:

```ts
getPayloadBuilder(...) { ... }
normalizeResponse(...) { ... }
```

---

### 13. Buat Gateway Resolver Service

```ts
src/payout/service/gateway-resolver.service.ts
```

### 14. Buat Main Flow Service

```ts
src/payout/service/payout.service.ts
```

---

### 15. (Opsional) Buat Webhook Handler Service

Jika butuh menerima callback:

```ts
src/payout/service/webhook-handler.service.ts
```

---

### 16. Buat Controller

```ts
src/payout/controller/payout.controller.ts
```

### 17. (Opsional) Buat Webhook Controller

```ts
src/payout/controller/webhook.controller.ts
```

---

### 18. Buat Module

```ts
src/payout/payout.module.ts
```

### 19. Daftarkan Controller di App Module

```ts
src/app.module.ts
```

```ts
import { PayoutModule } from './payout/payout.module';

@Module({
  imports: [
    PayoutModule,
    // ...
  ]
})
```

---

### 20. Register Prefix API di Merchant Module

```ts
src/merchant/merchant.module.ts
```

---

### ✅ SELESAI!

Sekarang kamu bisa menjalankan aplikasi dan mengakses endpoint baru:

```bash
npm run start:dev
```

## 🧩 Menambahkan Currency dan Payment Method Baru

Sistem ini mendukung kombinasi dinamis antara `currency` dan `payment_method` menggunakan Strategy Pattern. Untuk menambahkan kombinasi baru (misalnya `USD` + `CARD`), ikuti langkah berikut:

### 1. Tambahkan currency baru ke Constant (jika belum ada)

File: `shared/constants/payment.ts`

```ts
export const SUPPORTED_CURRENCIES = ['IDR', 'PHP', 'USD'];
export const SUPPORTED_PAYIN_METHODS = ['VA', 'QRIS', 'CARD'];
```

### 2. Tambahkan konfigurasi di database
```sql
INSERT INTO payment_configs (merchant_id, currency, method, gateway, active)
VALUES ('M123', 'USD', 'CARD', 'YOUR_GATEWAY', true);
```

### 3. Buat PayloadBuilder baru
Lokasi: payment/gateway/`your_gateway`/builders/`your_gateway`-card.builder.ts
```ts
@Injectable()
export class YourGatewayNameCardPayloadBuilder implements IYourGatewayPayloadBuilder {
  supports(currency: string, method: string): boolean {
    return currency === 'USD' && method === 'CARD';
  }

  build(input: NormalizedPaymentInput): YourGatewayPayload {
    return {
      // Sesuai dokumentasi YourGateway untuk pembayaran kartu
    };
  }
}
```
### 4. Buat ResponseNormalizer baru
Lokasi: payment/gateway/{your_gateway}/normalizers/{your_gateway}-card.normalizer.ts
```ts
@Injectable()
export class YourGatewayCardResponseNormalizer implements IPaymentResponseNormalizer {
  supports(gateway: GatewayName, method: string): boolean {
    return gateway === 'YOUR_GATEWAY' && method === 'CARD';
  }

  normalize(raw: any): NormalizedPaymentResponse {
    return {
      status: raw.status,
      reference: raw.transaction_id,
      // Format sesuai standar sistem
    };
  }
}
```
### 5. Update Gateway Resolver
File: shared/gateway-resolver.service.ts
```ts
const mapping = {
  'IDR:VA': 'MIDTRANS',
  'PHP:QRIS': 'LOCALPAYMENT',
  'USD:CARD': 'YOUR_GATEWAY', // Tambahkan di sini
};
```
