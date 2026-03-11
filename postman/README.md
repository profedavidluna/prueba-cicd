# 🪙 CoinDash API — Postman & Pruebas de Carga

Colección completa de Postman con pruebas de endpoints, variables de entorno y pruebas de carga para la **CoinDash API**.

---

## 📁 Estructura

```
postman/
├── CoinDash_API.postman_collection.json    # Colección principal
├── environments/
│   ├── CoinDash_DEV.postman_environment.json      # Entorno desarrollo local
│   ├── CoinDash_QA.postman_environment.json       # Entorno QA
│   ├── CoinDash_STAGING.postman_environment.json  # Entorno Staging
│   └── CoinDash_PROD.postman_environment.json     # Entorno Producción
└── load-tests/
    └── artillery.yml                              # Pruebas de carga con Artillery
```

---

## 🚀 Importar en Postman

### 1. Importar la Colección

1. Abre Postman
2. Haz clic en **Import**
3. Selecciona `CoinDash_API.postman_collection.json`
4. La colección aparecerá en el panel izquierdo

### 2. Importar un Entorno

1. En Postman ve a **Environments** → **Import**
2. Selecciona el archivo de entorno correspondiente a tu stage:
   - `CoinDash_DEV.postman_environment.json` — desarrollo local
   - `CoinDash_QA.postman_environment.json` — QA
   - `CoinDash_STAGING.postman_environment.json` — Staging
   - `CoinDash_PROD.postman_environment.json` — Producción
3. Activa el entorno usando el selector en la esquina superior derecha

### 3. Ejecutar la Colección completa

1. Haz clic en **...** junto al nombre de la colección
2. Selecciona **Run collection**
3. Elige el orden de ejecución (recomendado: mantener el orden original)
4. Haz clic en **Run CoinDash API**

---

## 📋 Endpoints cubiertos

| Módulo      | Endpoint                            | Método | Casos de prueba |
|-------------|-------------------------------------|--------|-----------------|
| Health      | `/health`                           | GET    | Respuesta 200, estructura, tiempo |
| Products    | `/api/products`                     | GET    | Lista completa, filtro por categoría |
| Products    | `/api/products/:id`                 | GET    | Exitoso, no encontrado |
| Products    | `/api/products`                     | POST   | Creación exitosa, validación |
| Users       | `/api/users/:id`                    | GET    | Exitoso, no encontrado, sin email |
| Users       | `/api/users`                        | POST   | Creación, email duplicado, validación |
| Orders      | `/api/orders`                       | GET    | Lista completa |
| Orders      | `/api/orders/:id`                   | GET    | Exitoso, no encontrado |
| Orders      | `/api/orders`                       | POST   | Creación exitosa, errores |
| Payments    | `/api/payments/balance/:userId`     | GET    | Exitoso, no encontrado |
| Payments    | `/api/payments/send`                | POST   | Exitoso, saldo insuficiente, mismo usuario, monto inválido, campos faltantes |

**Total: 20 requests con 70+ aserciones**

---

## 🌍 Variables de Entorno

Cada entorno define las siguientes variables:

| Variable           | DEV                      | QA                            | STAGING                          | PROD                           |
|--------------------|--------------------------|-------------------------------|----------------------------------|--------------------------------|
| `base_url`         | `http://localhost:3000`  | `http://coindash-qa.internal:3000` | `https://coindash-staging.example.com` | `https://api.coindash.example.com` |
| `environment`      | `DEV`                    | `QA`                          | `STAGING`                        | `PROD`                         |
| `user_id`          | `user-001`               | `user-001`                    | `user-001`                       | *(configurar manualmente)*     |
| `user_id_2`        | `user-002`               | `user-002`                    | `user-002`                       | *(configurar manualmente)*     |
| `product_id`       | `prod-001`               | `prod-001`                    | `prod-001`                       | *(configurar manualmente)*     |
| `timeout_ms`       | `5000`                   | `3000`                        | `2000`                           | `1500`                         |
| `order_id`         | *(dinámico)*             | *(dinámico)*                  | *(dinámico)*                     | *(dinámico)*                   |
| `created_user_id`  | *(dinámico)*             | *(dinámico)*                  | *(dinámico)*                     | *(dinámico)*                   |
| `created_product_id` | *(dinámico)*           | *(dinámico)*                  | *(dinámico)*                     | *(dinámico)*                   |

> **⚠️ PROD:** Las variables de usuario y producto están vacías por seguridad. Configúralas manualmente con IDs reales de producción antes de ejecutar pruebas.

---

## ⚡ Ejecutar con Newman (CLI)

[Newman](https://github.com/postmanlabs/newman) permite ejecutar la colección desde la línea de comandos, ideal para pipelines CI/CD.

### Instalación

```bash
npm install -g newman
npm install -g newman-reporter-htmlextra  # Para reportes HTML
```

### Ejecución por entorno

```bash
# DEV (local)
newman run CoinDash_API.postman_collection.json \
  -e environments/CoinDash_DEV.postman_environment.json

# QA
newman run CoinDash_API.postman_collection.json \
  -e environments/CoinDash_QA.postman_environment.json

# STAGING
newman run CoinDash_API.postman_collection.json \
  -e environments/CoinDash_STAGING.postman_environment.json

# Con reporte HTML
newman run CoinDash_API.postman_collection.json \
  -e environments/CoinDash_DEV.postman_environment.json \
  --reporters htmlextra \
  --reporter-htmlextra-export newman-report.html
```

### Integración en GitHub Actions

```yaml
- name: Run Postman Tests
  run: |
    npm install -g newman
    newman run postman/CoinDash_API.postman_collection.json \
      -e postman/environments/CoinDash_QA.postman_environment.json \
      --reporters cli,junit \
      --reporter-junit-export newman-results.xml
```

---

## 🔥 Pruebas de Carga con Artillery

### Instalación

```bash
npm install -g artillery
```

### Ejecución

```bash
# Prueba básica en DEV (localhost)
artillery run postman/load-tests/artillery.yml

# Contra QA
artillery run postman/load-tests/artillery.yml -e qa

# Contra Staging
artillery run postman/load-tests/artillery.yml -e staging

# Guardar resultados en JSON
artillery run postman/load-tests/artillery.yml --output results.json

# Generar reporte HTML desde los resultados
artillery report results.json
```

### Escenarios de carga definidos

| Escenario                    | Peso | Descripción |
|------------------------------|------|-------------|
| Navegación de catálogo       | 60%  | Health + listado + filtrado + detalle de productos |
| Consulta de usuarios/saldos  | 20%  | GET usuario + GET balance |
| Consulta de órdenes          | 15%  | GET listado de órdenes |
| Flujo de creación completo   |  5%  | POST usuario + POST orden |

### Fases de carga

| Fase            | Duración | Usuarios/seg |
|-----------------|----------|--------------|
| Calentamiento   | 30 seg   | 1 → 10       |
| Carga sostenida | 60 seg   | 10           |
| Pico de carga   | 30 seg   | 10 → 30      |
| Enfriamiento    | 20 seg   | 30 → 1       |

### Umbrales de aceptación

- **P95** tiempo de respuesta < 500ms
- **P99** tiempo de respuesta < 1000ms
- Tasa de errores 5xx < 1%

---

## 🔗 Referencias

- [Postman Documentation](https://learning.postman.com/docs/)
- [Newman CLI](https://github.com/postmanlabs/newman)
- [Artillery Docs](https://www.artillery.io/docs)
- [CoinDash API README](../coindash-api/README.md)
