# First question
Te ayudo a entender el flujo completo de las ofertas en la sección de oportunidades, usando la **portabilidad de nómina** como ejemplo. Aquí está el análisis detallado:

## 📋 Arquitectura del Sistema de Ofertas

### **1. Archivos de Configuración Principal**

#### **A. Definición de la Oferta**
_constants.js
```javascript
CAMPAIGNS = {
  ACCOUNTS: {
    OCPP: {
      code: 'payroll-portability',  // Código único de la oferta
      link: {
        page: 'ocPayrollPortability',  // Página destino
        params: {
          offerId: 'offerId'  // Parámetros que recibe
        }
      }
    }
  }
}
```

#### **B. Configuración de Analytics**
_ocAccess.js
```javascript
'payroll-portability': {
  globalApplication: 'payroll portability',
  name: 'portabilidad nomina',
  type: 'contratacion',
  productName: {
    '0557': 'cuenta nomina'  // Código de producto
  },
  isFunnel: true,
  hasOnClickStartAnalytic: true  // Activa analytics al hacer clic
}
```

### **2. Página Principal: contractProductSection.js**

contractProductSection.js

Este archivo es el **compositor de la página** que define:

**Componentes clave:**

1. **cells-oc-banner-list-spherica** (línea 238-269)
   - Renderiza la lista de ofertas
   - Recibe las ofertas parseadas mediante el canal `contract_product_section_offers_parsed`
   - Conecta los clics a las navegaciones correspondientes

2. **glomo-campaigns-dm-mx** (referenciado en _cross-components.js)
   - Data Manager que obtiene las ofertas del backend
   - Parsea y formatea las ofertas según configuración
   - Incluye customizaciones visuales (imágenes, descripciones)

3. **glomo-analytics-oc-banner-manager-mx** (línea 270-293)
   - Gestiona toda la analítica de las ofertas
   - Envía eventos cuando se muestran, hacen clic, etc.

### **3. Data Manager de Campañas**

_cross-components.js

**glomo-campaigns-dm-mx** configuración:
```javascript
{
  tag: 'glomo-campaigns-dm-mx',
  properties: {
    host: CONFIG.globalServices.host,
    campaignsApiVersion: CONFIG.globalServices.versions.campaigns,
    includeOffers: campaignsCode,  // Códigos de ofertas a incluir
    showReviewOffers: true,
    objectOffersFlags: offersFlagsData,
    customizableOffers: {
      // Personalizaciones por producto
      // Imágenes, descripciones, etc.
    }
  }
}
```

### **4. Flujo de Datos Completo**

```
┌─────────────────────────────────────────────────────────────┐
│ 1. INICIO - contractProductSection.js                       │
│    - Usuario navega a oportunidades                         │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│ 2. glomo-campaigns-dm-mx                                    │
│    - Llama API: /campaigns/v1                               │
│    - Filtra ofertas según includeOffers                     │
│    - Aplica customizableOffers                              │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│ 3. Parseo de Ofertas                                        │
│    - getCampaigns() desde _utils.js                         │
│    - Mapea códigos a configuraciones                        │
│    - Envía: contract_product_section_offers_parsed          │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│ 4. cells-oc-banner-list-spherica                            │
│    - Recibe array de ofertas                                │
│    - Renderiza cada banner                                  │
│    - Banner destacado (featured: [1])                       │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│ 5. Usuario hace clic en oferta                              │
│    - Dispara: cells-oc-banner-click-payroll-portability     │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ├──────────────────────────────────────┐
                      ▼                                      ▼
┌───────────────────────────────────┐  ┌──────────────────────────────┐
│ 6A. Analytics                     │  │ 6B. Navegación               │
│ - glomo-analytics-oc-banner-mgr   │  │ - glomo-navigation-handler   │
│ - Envía APP_ON_CLICK_START        │  │ - Navega a:                  │
│ - Tracking de conversión          │  │   ocPayrollPortability       │
└───────────────────────────────────┘  └──────────┬───────────────────┘
                                                   │
                                                   ▼
                                    ┌──────────────────────────────────┐
                                    │ 7. ocPayrollPortability.js       │
                                    │    - Carga flujo de portabilidad │
                                    │    - Recibe offerId como param   │
                                    └──────────────────────────────────┘
```

### **5. Archivos Clave para Configurar una Oferta**

Para configurar **Portabilidad de Nómina** desde cero, necesitas:

#### ✅ **Archivo 1: _constants.js**
Define el código y navegación
```javascript
// app/composerMocksTpl/common/helpers/_constants.js
OCPP: {
  code: 'payroll-portability',
  link: {
    page: 'ocPayrollPortability',
    params: { offerId: 'offerId' }
  }
}
```

#### ✅ **Archivo 2: _ocAccess.js (Analytics)**
Configuración de analítica
```javascript
// app/composerMocksTpl/analytics/_ocAccess.js
'payroll-portability': {
  globalApplication: 'payroll portability',
  name: 'portabilidad nomina',
  type: 'contratacion',
  productName: { '0557': 'cuenta nomina' },
  isFunnel: true,
  hasOnClickStartAnalytic: true
}
```

#### ✅ **Archivo 3: _ocPayrollPortability.js (Analytics específicos)**
_ocPayrollPortability.js - Eventos específicos del flujo

#### ✅ **Archivo 4: _configOperativeUtils.js**
Mapea analytics a ofertas (ya configurado automáticamente)

#### ✅ **Archivo 5: contractProductSection.js**
Añadir canal de navegación:
```javascript
outChannels = {
  contract_product_section_oc_banner_payroll_portability_clicked: {
    bind: 'cells-oc-banner-click-payroll-portability',
    link: {
      page: 'ocPayrollPortability',
      params: { offerId: 'offerId' }
    }
  }
}
```

#### ✅ **Archivo 6: _cross-components.js**
Customizar visualización en `glomo-campaigns-dm-mx`:
```javascript
customizableOffers: {
  ACCOUNTS: {
    '0557': {
      description: 'Texto personalizado',
      smallImg: {
        src: './images/ruta-imagen.svg'
      }
    }
  }
}
```

#### ✅ **Archivo 7: ocPayrollPortability.js**
La página/flujo destino de la oferta

### **6. Componentes Web Involucrados**

| Componente | Ubicación | Función |
|------------|-----------|---------|
| **cells-oc-banner-list-spherica** | cells-oc-banner-list-spherica | Lista las ofertas |
| **cells-oc-banner-spherica** | cells-oc-banner-spherica | Banner individual |
| **glomo-campaigns-dm** | glomo-campaigns-dm | Obtiene datos del backend |
| **glomo-analytics-oc-banner-manager** | glomo-analytics-oc-banner-manager | Analytics |

### **7. Canales de Comunicación**

**Entrada (IN):**
- `contract_product_section_offers_parsed` → Recibe ofertas parseadas
- `contract_product_section_reset_page` → Resetea lista

**Salida (OUT):**
- `cells-oc-banner-click-[offer-code]` → Click en oferta
- `analytics-oc-banner-clicked-[offer-code]` → Evento analítico

### **8. API Backend**

El DM `glomo-campaigns-dm-mx` consume:
```
GET {host}/campaigns/{version}/offers
Headers: 
  - tsec token
  - Authorization
```

### **9. Feature Flags**

Las ofertas pueden controlarse con feature flags:
```javascript
featureFlag: {
  key: 'glomo.analytics.ocPayrollPortability',
  defaultValue: true
}
```

### **10. Checklist para Nueva Oferta**

1. ✅ Añadir en _constants.js (código, link, params)
2. ✅ Configurar analytics en _ocAccess.js
3. ✅ Crear archivo analytics específico `_oc[NombreOferta].js`
4. ✅ Añadir canal en contractProductSection.js
5. ✅ Customizar visual en _cross-components.js
6. ✅ Crear página destino `[nombreOferta].js`
7. ✅ Añadir imágenes en images
8. ✅ Configurar textos en locales
9. ✅ Feature flag (opcional)
10. ✅ Testing en e2e

# Second question 
Excelente pregunta. Los valores **NO son arbitrarios**, siguen **convenciones estrictas** y **patrones específicos** para que todo el sistema funcione correctamente. Te explico cada tipo de valor:

## 🎯 **1. CÓDIGOS DE PRODUCTO (productName)**

**Origen:** Backend / Catálogos BBVA
```javascript
productName: {
  '0557': 'cuenta nomina'  // ← Código oficial del producto
}
```

**Reglas:**
- Son **códigos oficiales** del catálogo de productos BBVA
- Deben coincidir con el código que devuelve el backend en `/campaigns/v1`
- Se usan para tracking y reportes
- **NO son inventables** 

## 🔐 **REGLAS DE NOMENCLATURA Y CONVENCIONES**

### **1. CÓDIGO DE OFERTA (`code`)**

**Patrón:** `kebab-case` (palabras-separadas-por-guiones)

```javascript
code: 'payroll-portability'  // ✅ CORRECTO
code: 'payrollPortability'   // ❌ INCORRECTO
code: 'payroll_portability'  // ❌ INCORRECTO
```

**Reglas:**
- Debe ser **único** en todo el sistema
- Se usa para generar automáticamente:
  - Nombres de canales
  - Event bindings
  - Identificadores de analytics

### **2. NOMBRE DE PÁGINA (`page`)**

**Patrón:** `camelCase`

```javascript
page: 'ocPayrollPortability'  // ✅ CORRECTO
page: 'oc-payroll-portability'  // ❌ INCORRECTO
```

**Reglas:**
- Debe coincidir **exactamente** con el nombre del archivo `.js` en composerMocksTpl
- Debe existir el archivo: ocPayrollPortability.js

### **3. BINDINGS - Convención Automática** ⚙️

Los bindings se **generan automáticamente** siguiendo estas reglas: 

```javascript
// ENTRADA (lo que defines):
PAGENAME = 'contractProductSection'
code = 'payroll-portability'

// PROCESO AUTOMÁTICO:
// 1. Convierte pageName a snake_case
snakeCase('contractProductSection') 
→ 'contract_product_section'

// 2. Genera nombre de canal (OUT)
channelName = `contract_product_section_oc_banner_payroll_portability_clicked`
          //  ^^^^^^^^^^^^^^^^^^^^^^^ + _oc_banner_ + ^^^^^^^^^^^^^^^^^^^ + _clicked
          //     página en snake_case              código (- → _)

// 3. Genera event binding (componente escucha esto)
eventName = `cells-oc-banner-click-payroll-portability`
         // cells-oc-banner-click- + código

// RESULTADO:
{
  contract_product_section_oc_banner_payroll_portability_clicked: {
    bind: 'cells-oc-banner-click-payroll-portability',
    link: { page: 'ocPayrollPortability', params: { offerId: 'offerId' } }
  }
}
```

### **4. PARÁMETROS - Convenciones**

```javascript
params: {
  offerId: 'offerId',     // ✅ Valor literal 'offerId' - se reemplaza dinámicamente
  productId: 'productId'  // ✅ Valor literal 'productId'
}
```

**Regla:**
- Los valores son **literales string** que actúan como **placeholders**
- El componente `cells-oc-banner-list-spherica` los reemplaza con valores reales
- **NO** inventar nombres, usar: `offerId`, `productId`, `contractId`

### **5. ANALYTICS - Valores NO Arbitrarios**

```javascript
'payroll-portability': {
  globalApplication: 'payroll portability',  // ← Debe coincidir con Adobe Analytics
  name: 'portabilidad nomina',              // ← Texto descriptivo para reportes
  type: 'contratacion',                     // ← Tipos permitidos: contratacion, operativa
  productName: {
    '0557': 'cuenta nomina'  // ← Código oficial del catálogo BBVA
  }
}
```

**Valores de `type` permitidos:**
- `'contratacion'` - Para contratación de productos
- `'operativa'` - Para operaciones/gestiones
- `'consulta'` - Para consultas

### **6. FEATURE FLAGS - Convención**

```javascript
featureFlag: {
  key: 'glomo.contractProductSection.ocPayrollPortability',
  //    ^^^^^ ^^^^^^^^^^^^^^^^^^^^^^^^^^ ^^^^^^^^^^^^^^^^^^^
  //    app   página                      nombre oferta
  defaultValue: true
}
```

**Patrón:** `glomo.{pagina}.{nombreOferta}`

### **7. IMÁGENES - Convención de Rutas**

```javascript
smallImg: {
  src: './images/offers/payroll/portability.svg'
  //   ./images/ + carpeta categoria + nombre descriptivo.svg
}
```

**Categorías comunes:**
- `offers/` - Ofertas generales
- `oneClicks/` - One clicks
- `spherica/` - Imágenes Spherica

### **8. LOCALES - Convención de Nombres**

```javascript
titleName: 'payroll-portability-header-title'
//         ^^^^^^^^^^^^^^^^^^^^^-^^^^^^-^^^^^
//         código de oferta     tipo   elemento

description: 'payroll-portability-banner-description'
```

**Patrón:** `{codigo-oferta}-{tipo}-{elemento}`

Archivo: es-MX.json

---

## 📊 **TABLA DE CONVERSIÓN AUTOMÁTICA**

| Concepto | Entrada | Procesamiento | Salida |
|----------|---------|---------------|---------|
| **Page Name** | `ocPayrollPortability` | `snakeCase()` | `oc_payroll_portability` |
| **Código** | `payroll-portability` | `.replace(/-/g, '_')` | `payroll_portability` |
| **Channel Name** | page + code | `{page}_oc_banner_{code}_clicked` | `oc_payroll_portability_oc_banner_payroll_portability_clicked` |
| **Event Bind** | code | `cells-oc-banner-click-{code}` | `cells-oc-banner-click-payroll-portability` |
| **Analytics Event** | code | `analytics-oc-banner-clicked-{code}` | `analytics-oc-banner-clicked-payroll-portability` |

---

## ✅ **CHECKLIST DE VALIDACIÓN**

Para que una nueva oferta funcione correctamente:

### **A. Nombres y Códigos**
- [ ] `code` en **kebab-case** y único
- [ ] `page` en **camelCase** y existe el archivo `.js`
- [ ] `productName` código coincide con catálogo backend

### **B. Archivos Requeridos**
- [ ] Definición en _constants.js
- [ ] Analytics en _ocAccess.js
- [ ] Archivo página `{nombre}.js` existe
- [ ] Imágenes en carpeta correcta
- [ ] Textos en `locales-app/es-MX.json`

### **C. Bindings (Automáticos)**
- [ ] El sistema genera: `{page}_oc_banner_{code}_clicked`
- [ ] El sistema genera: `cells-oc-banner-click-{code}`
- [ ] No necesitas crearlos manualmente

### **D. Parámetros**
- [ ] Usar solo: `offerId`, `productId`, `contractId`
- [ ] Valores como strings literales

---

## 🎯 **EJEMPLO COMPLETO: Nueva Oferta**

Supongamos que quieres crear **"Préstamo Personal Express"**

### **1. En _constants.js**
```javascript
LOANS: {
  PLE: {  // ← Identificador interno (puede ser lo que quieras)
    code: 'personal-loan-express',  // ← DEBE ser kebab-case
    link: {
      page: 'personalLoanExpress',  // ← DEBE existir archivo .js
      params: {
        offerId: 'offerId'  // ← Literal string
      }
    }
  }
}
```

### **2. En _ocAccess.js**
```javascript
CAMPAIGNS_ANALYTICS: {
  LOANS: {
    'personal-loan-express': {  // ← MISMO código
      globalApplication: 'personal loan express',  // ← Para Adobe
      name: 'prestamo personal express',
      type: 'contratacion',
      productName: {
        '0740': 'prestamos personales'  // ← Código oficial
      },
      isFunnel: true,
      hasOnClickStartAnalytic: true
    }
  }
}
```

### **3. Bindings Generados Automáticamente**

El sistema creará:
```javascript
// Canal de salida (OUT):
contract_product_section_oc_banner_personal_loan_express_clicked

// Event binding (componente escucha):
cells-oc-banner-click-personal-loan-express

// Analytics:
analytics-oc-banner-clicked-personal-loan-express
```

### **4. Crear archivo**
`app/composerMocksTpl/personalLoanExpress.js`

### **5. Feature Flag (opcional)**
```javascript
'glomo.contractProductSection.personalLoanExpress'
```

---

## ⚠️ **ERRORES COMUNES**

| Error | Incorrecto | Correcto |
|-------|-----------|----------|
| Código con camelCase | `personalLoanExpress` | `personal-loan-express` |
| Página con kebab-case | `personal-loan-express` | `personalLoanExpress` |
| Código producto inventado | `'XXXX': 'prestamos'` | `'0740': 'prestamos'` |
| Parámetro con valor real | `offerId: '12345'` | `offerId: 'offerId'` |
| Binding manual incorrecto | `bind: 'click-offer'` | Se genera automáticamente |

---

**Resumen:** Los valores NO son arbitrarios. Siguen **patrones estrictos** y muchos se **generan automáticamente**. Solo necesitas definir correctamente:
1. Código en `kebab-case`
2. Página en `camelCase` (que exista)
3. Código de producto oficial
4. El resto se construye por convención
