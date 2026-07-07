# 05-kode

Source code implementasi — **Tahap 2** (API Gateway) dan **Tahap 3** (skrip pengujian k6).

## Struktur yang direncanakan

```
05-kode/
├── gateway/              # API Gateway (Go + Echo)
│   ├── cmd/gateway/
│   ├── internal/
│   ├── migrations/       # migration SQL (signing_keys, rate_limit_counters)
│   ├── scripts/          # skrip seed (generate RSA keypair, insert signing_keys)
│   ├── docker-compose.yml
│   └── .env.example
└── k6/                   # skrip pengujian k6
    ├── legitimate.js
    ├── attack.js
    └── mixed.js
```

## Acuan

- Rencana implementasi Gateway: [../09-docs/tahap-2-implementasi-gateway.md](../09-docs/tahap-2-implementasi-gateway.md)
- Rencana skrip k6: [../09-docs/tahap-3-pengujian-k6.md](../09-docs/tahap-3-pengujian-k6.md)
