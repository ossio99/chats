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

# Third question

## 1️⃣ **productName: ¿Cuál es el código oficial?** 

Perfecto, ahora te muestro:

### ✅ **Respuesta 1: productName - Código Oficial**

```javascript
productName: {
  '0557': 'cuenta nomina'
  // ^^^^   ^^^^^^^^^^^^^^
  // CLAVE  VALOR
}
```

**La CLAVE numérica `'0557'` es el CÓDIGO OFICIAL del catálogo BBVA**
- **NO es arbitrario** 
- Es el **LDA (Línea De Actividad)** del producto
- Viene del **backend** en la respuesta de campaigns API
- Se usa para tracking, reportes y debe coincidir con el sistema de catálogos BBVA

**El VALOR `'cuenta nomina'` es SEMI-ARBITRARIO**
- Es descriptivo para reportes de analytics
- Debe ser consistente con Adobe Analytics
- Usado en `products.attributes.0.productName` en digitalData
- **Debe coincidir** con lo que espera el equipo de analytics

Puedes ver que el código `0557` se usa en otros lugares (dashboard/_operationals.js) confirmando que es un código oficial del sistema.

---

## 2️⃣ **_constants.js - ¿A qué nivel define la oferta?** 

### ✅ **Respuesta 2: _constants.js define a NIVEL DE SISTEMA COMPLETO**

_constants.js es el **MAPEO CENTRAL** que:

1. **Define qué ofertas existen en el sistema** (catálogo de ofertas)
2. **Mapea código → página destino** (enrutamiento)
3. **Se usa en MÚLTIPLES lugares**:

```javascript
// _constants.js
OCPP: {
  code: 'payroll-portability',     // ← Identificador único
  link: {
    page: 'ocPayrollPortability',  // ← A qué página navega
    params: { offerId: 'offerId' } // ← Qué parámetros necesita
  }
}

// Este objeto se consume en:
// 1. _utils.js → getCampaigns() → Extrae todas las ofertas
// 2. contractProductSection.js → getCampaignsChannels() → Genera canales automáticamente
// 3. _configOperativeUtils.js → Configura analytics
// 4. glomo-campaigns-dm-mx → includeOffers
```

**NO es solo para listado ni solo para page**, es la **CONFIGURACIÓN MAESTRA** del sistema de ofertas.

---

## 3️⃣ **includeOffers en glomo-campaigns-dm-mx** 

### ✅ **Respuesta 3: includeOffers se genera AUTOMÁTICAMENTE**

```javascript
// En _cross-components.js línea 4955:
const campaignsCode = Object.keys(utils.getCampaignsCode());

// Esto ejecuta:
getCampaignsCode() {
  const campaignAnalytics = getCampaignAnalytics(); // Lee _ocAccess.js
  return Object.keys(campaignAnalytics).reduce(
    (acc, code) => Object.assign(acc, campaignAnalytics[code].productName), {}
  );
}

// Resultado: includeOffers = ['0557', '0788', '0766', '0785', '0813', ...]
```

**¿Necesitas agregarlo manualmente?** ❌ **NO**

Se genera automáticamente desde _ocAccess.js:
```javascript
// Si tienes esto en _ocAccess.js:
'payroll-portability': {
  productName: { '0557': 'cuenta nomina' }  // ← Este '0557' se extrae
}

// Automáticamente se incluye en:
includeOffers: ['0557', '0788', '0766', ...]
```

**¿Qué pasa si NO se incluye?**
- El backend **SÍ devuelve** la oferta
- Pero `glomo-campaigns-dm-mx` la **FILTRA** y no la muestra
- **NO aparecerá** en la lista de oportunidades

---

## 4️⃣ **El Binding Automático - AQUÍ ESTÁ LA MAGIA** 🎩✨

Tienes razón, no lo ves explícitamente porque **se genera automáticamente**: 

### ✅ **Respuesta 4: EL BINDING ESTÁ EN LA LÍNEA 11** 🎯

```javascript
// contractProductSection.js línea 11:
const campaignsChannels = utils.getCampaignsChannels(PAGE_NAME);

// línea 32:
const outChannels = {
  ...campaignsChannels,  // ← AQUÍ SE EXPANDEN TODOS LOS BINDINGS AUTOMÁTICAMENTE
  // ... otros canales manuales ...
};

// línea 260:
cellsConnections: {
  out: outChannels  // ← Aquí se asignan
}
```

**Paso a paso de cómo funciona:**

```javascript
// 1. Se llama a getCampaignsChannels('contractProductSection')
const campaignsChannels = utils.getCampaignsChannels('contractProductSection');

// 2. Esto ejecuta (en _utils.js línea 1042):
const getCampaignsChannels = (PAGENAME) => {
  const campaigns = getCampaigns();  // Trae TODAS las ofertas de _constants.js
  const channelName = (code) => 
    `${snakeCase(PAGENAME)}_oc_banner_${code}_clicked`.replace(/-/g, '_');
  const eventName = (code) => 
    `cells-oc-banner-click-${code}`.replace(/_/g, '-');
  
  let channelOut = {};
  
  // 3. PARA CADA oferta genera el binding:
  Object.keys(campaigns).forEach((campaign) => {
    channelOut[channelName(campaigns[campaign].code)] = {
      bind: eventName(campaigns[campaign].code),
      link: campaigns[campaign].link
    };
  });
  
  return channelOut;
};

// 4. RESULTADO (objeto generado automáticamente):
{
  contract_product_section_oc_banner_payroll_portability_clicked: {
    bind: 'cells-oc-banner-click-payroll-portability',
    link: {
      page: 'ocPayrollPortability',
      params: { offerId: 'offerId' }
    }
  },
  contract_product_section_oc_banner_debit_card_request_clicked: {
    bind: 'cells-oc-banner-click-debit-card-request',
    link: { page: 'requestDebitCard', params: { offerId: 'offerId' } }
  },
  // ... y así para TODAS las ofertas en _constants.js
}

// 5. Se expande con spread operator en línea 32:
const outChannels = {
  ...campaignsChannels,  // ← Aquí se inyectan TODOS los bindings
  // ofertas adicionales manuales...
};
```

---

## 📊 **RESUMEN VISUAL**

```
┌─────────────────────────────────────────────────────────────────┐
│ 1. _constants.js                                                │
│    OCPP: { code: 'payroll-portability', page: '...' }          │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│ 2. _utils.js → getCampaigns()                                   │
│    Lee _constants.js y retorna todas las ofertas               │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│ 3. _utils.js → getCampaignsChannels('contractProductSection')  │
│    GENERA AUTOMÁTICAMENTE:                                      │
│    {                                                            │
│      contract_product_section_oc_banner_payroll_portability... │
│         bind: 'cells-oc-banner-click-payroll-portability'      │
│         link: { page: 'ocPayrollPortability' }                 │
│    }                                                            │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│ 4. contractProductSection.js línea 11                           │
│    const campaignsChannels = utils.getCampaignsChannels(...)   │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│ 5. contractProductSection.js línea 32                           │
│    const outChannels = { ...campaignsChannels, ... }           │
│    ← AQUÍ SE EXPANDEN TODOS LOS BINDINGS                       │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│ 6. contractProductSection.js línea 260                          │
│    cellsConnections: { out: outChannels }                      │
│    ← Se asignan a cells-oc-banner-list-spherica                │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🎯 **CONCLUSIONES**

1. **`'0557'` es el código oficial** (LDA), `'cuenta nomina'` es semi-arbitrario pero debe coincidir con analytics
2. **_constants.js define a nivel de sistema completo** - es el catálogo maestro de ofertas
3. **`includeOffers` se genera automáticamente** desde los `productName` en _ocAccess.js
4. **Los bindings NO están escritos explícitamente** porque se generan con `getCampaignsChannels()` y se expanden con `...campaignsChannels`

**Por eso NUNCA ves escrito:**
```javascript
contract_product_section_oc_banner_payroll_portability_clicked: {
  bind: 'cells-oc-banner-click-payroll-portability'
}
```

**Porque se genera automáticamente** a partir de la definición en _constants.js ✨

# Fourth question

## 1. ¿Los archivos de analíticas son exclusivos para analíticas?

**Respuesta: SÍ**, son exclusivos para analíticas.

- Si borras `_ocPayrollPortabilityN2.js` o su configuración en `_configOperativeUtils.js`, **TODO lo demás seguirá funcionando**
- El renderizado de ofertas, navegación, bindings, etc., **NO dependen** de los archivos de analíticas
- La analítica es un sistema **paralelo y desacoplado** del flujo funcional

**Dependencias funcionales vs analíticas:**
- ✅ **Funcional** (necesario): `_constants.js`, _cross-components.js, _operationals.js
- 📊 **Analítico** (opcional): `_ocAccess.js`, `_ocPayrollPortabilityN2.js`, `_configOperativeUtils.js`

---

## 2. ¿Cómo ubicar componentes en proceedings.js?

Veo que **proceedings.js** sí existe. Del código:

**Componente principal:**
```javascript
tag: 'bbva-feature-proceedings'
tagSpherica: 'bbva-feature-proceedings-spherica'
familyPath: '@glomo-web-components/bbva-feature-proceedings'
```

**Parámetros recibidos desde la oferta:**
```javascript
// En _constants.js configuraste:
link: { 
  page: 'proceedings', 
  params: { offerId: 'offerId' } 
}

// proceedings.js recibe parámetros a través de cellsConnections:
params: {
  productId: 'productId',
  feedbackOperation: 'feedbackOperation'
}
```

**Canales de entrada relevantes para tu oferta:**
```javascript
navigate_to_pivot_proceedings: {
  bind: 'setPropertiesInit'  // Inicializa propiedades
},
navigation_from_deep_link_to_operations: {
  bind: 'initPropertiesFromDeeplink'  // Para deep links
},
upgrade_customer_hybrid_send_data_to_proceedings: {
  bind: 'setPropertiesInit'
}
```

**Canales de salida para continuar el flujo:**
```javascript
send_from_proceedings_to_portability: {
  bind: 'bbva-feature-proceedings-porta-n2-continue-button-link',
  link: { page: 'payrollPortability' }  // ← Navega a portabilidad
}
```

---

## 3. ¿_ocAccess.js es catálogo maestro o archivo de analíticas?

**Respuesta: Es AMBOS** (híbrido con funcionalidad dual):

```javascript
// _utils.js línea 1035-1043
const getCampaignsCode = () => {
  const campaignAnalytics = getCampaignAnalytics(); // ← Lee _ocAccess.js
  return Object.keys(campaignAnalytics).reduce(
    (acc, code) => Object.assign(acc, campaignAnalytics[code].productName), {});
};

// _cross-components.js línea 5180
includeOffers: campaignsCode,  // ← Auto-generado desde _ocAccess.js
```

**Doble función:**
1. **📊 Analítica**: Define eventos de Adobe Analytics
2. **⚙️ Funcional**: El campo `productName` se usa para generar `includeOffers` en `glomo-campaigns-dm-mx`

Por eso es **obligatorio** para funcionalidad, no solo analítica.

---

## 4. ¿Por qué la nueva oferta aparece arriba en la lista?

**Respuesta: NO agregamos configuración explícita de ordenamiento**

El orden viene del **servicio API** en el campo `priority`. Del código de `glomo-campaigns-dm-mx`:

```javascript
// _cross-components.js línea 5199
customizableOffers: {
  LOANS: {
    '0835': {
      description: 'Trae tu nómina',
      // NO hay campo "priority" aquí
    }
  }
}
```

**El orden lo controla:**
1. El **API** `/campaigns/v1` devuelve ofertas con campo `priority`
2. El componente `cells-oc-banner-list-spherica` ordena por `priority`
3. Si tiene `priority: 0`, aparece primero

**Para verificar:** Revisa el JSON del API que compartiste, debe tener un campo `priority` bajo.

---

## 5. ¿Qué alcance tienen las configuraciones en _operationals.js?

**Respuesta: Es para el DASHBOARD (smart spaces) y feature flags**

```javascript
// _operationals.js línea 341-348
const homeNBA = [
  createDictionaryEntry('sales.0835', ['glomo.contractProductSection.ocPayrollPortabilityN2']),
  // ...
];

// _config.js línea 206
{
  id: 'home_nba',  // ← Smart Space en dashboard
  collections: [{
    modelCollection: 'nba',
    elements: []  // ← Aquí se renderizarían ofertas NBA
  }]
}
```

**Tres funciones:**

1. **Dashboard Smart Spaces**: El array `homeNBA` define qué ofertas pueden aparecer en el espacio "NBA" del dashboard
   - `home_nba`: Next Best Action (ofertas personalizadas)
   - `home_nbo`: Next Best Offer
   - `home_maincard`: Tarjetas principales

2. **Feature Flag Mapping**: 
   ```javascript
   '0835': {
     flag: 'glomo.contractProductSection.ocPayrollPortabilityN2',
     lda: '0835'
   }
   ```
   Mapea product ID → feature flag para habilitar/deshabilitar

3. **Diccionario de productos**: Consolida todas las ofertas del sistema con sus flags correspondientes

**NO es solo para el dashboard**, también se usa para:
- Filtrar ofertas según feature flags activos
- Validar si una oferta está habilitada antes de mostrarla en **cualquier** página

---

## Resumen:

| Archivo | Propósito | ¿Es obligatorio? |
|---------|-----------|------------------|
| `_constants.js` | Catálogo maestro - Define ofertas | ✅ Sí |
| `_ocAccess.js` | Analíticas + `includeOffers` | ✅ Sí (dual) |
| _cross-components.js | Customización visual | ✅ Sí |
| _operationals.js | Dashboard + feature flags | ✅ Sí |
| `_ocPayrollPortabilityN2.js` | Eventos analíticos específicos | ❌ No (solo analytics) |
| `_configOperativeUtils.js` | Registro de módulos analíticos | ❌ No (solo analytics) |

---

# Fifth question - _operationals.js operation

## 📊 Alcance de _operationals.js - Aclarando la Confusión

### **`offersFlagsData` en _operationals.js**

```javascript
// dashboard/_operationals.js línea 528
const offersFlagsData = {
  '0835': {
    flag: 'glomo.contractProductSection.ocPayrollPortabilityN2', 
    lda: '0835'
  },
  // ... más ofertas
};
```

**Este objeto tiene DOBLE alcance:**

#### **1. Para el Dashboard (NBA/NBO Smart Spaces):**
```javascript
// dashboard/_operationals.js línea 341-348
const homeNBA = [
  createDictionaryEntry('sales.0835', ['glomo.contractProductSection.ocPayrollPortabilityN2']),
];

// Esto controla QUÉ ofertas pueden aparecer en el dashboard
```

#### **2. Para glomo-campaigns-dm-mx (GLOBAL):**
```javascript
// _cross-components.js línea 5180
objectOffersFlags: offersFlagsData,  // ← Se pasa al DM

// El componente glomo-campaigns-dm-mx usa esto para FILTRAR ofertas
// en TODAS las páginas: dashboard, contractProductSection, etc.
```

### **Flujo de Filtrado en glomo-campaigns-dm-mx:**

```javascript
// components/glomo-campaigns-dm/glomo-campaigns-dm-mx.js línea 392
_filterFlagsOffers(offers) {
  return offers.filter(offer => {
    const productId = offer.config.product.productId;
    
    // 1. Busca en objectOffersFlags (viene de _operationals.js):
    const offerFlag = this.objectOffersFlags[productId];
    
    if (!offerFlag) {
      return true; // Si no está en la lista, se permite
    }
    
    // 2. Verifica si el feature flag está activo:
    const flagConfig = this.featureFlagsCollection[offerFlag.flag];
    
    // 3. Solo permite la oferta si flag.value === true:
    return flagConfig && flagConfig.value === true;
  });
}
```

**Conclusión:** SÍ, la configuración en _operationals.js afecta **contractProductSection** y cualquier otra página que muestre ofertas.

---

Excelente pregunta. Voy a explicarte **exactamente** cómo funciona el flujo completo, porque es crucial entender que hay **DOS sistemas independientes pero relacionados**:

---

## 🎯 Flujo Completo: Feature Flags + Ofertas

### **1. Feature Flags: Se cargan desde un SERVICIO**

```javascript
// Las feature flags se recuperan de un servicio REST:
// GET /app-settings/v1 o similar

// Respuesta típica:
{
  "featureFlags": {
    "glomo.contractProductSection.ocPayrollPortabilityN2": true,
    "glomo.contractProductSection.ocPayrollPortability": true,
    "glomo.offers.dynamicLifeInsuranceMxn": false,
    // ... más banderas
  }
}
```

**Este servicio es gestionado por:**
- `glomo-loading-manager` o similar
- Se carga al iniciar sesión
- Se dispara el canal `global_feature_flag_collection_updated`

### **2. Ofertas: Se cargan desde OTRO servicio**

```javascript
// Las ofertas se recuperan de:
// GET /campaigns/v1/offers

// Respuesta típica:
{
  "data": {
    "offers": [
      {
        "id": "offer-id-123",
        "product": {
          "id": "0835",  // ← Este es el LDA code
          "type": "LOANS"
        },
        "name": "Trae tu nómina",
        "priority": 0
        // ...
      }
    ]
  }
}
```

---

## 🔗 Conexión: _operationals.js es el MAPEO

_operationals.js **NO crea las banderas**, solo define **la relación** entre:
- **Product ID** (0835) ↔ **Feature Flag** (glomo.contractProductSection.ocPayrollPortabilityN2)

```javascript
// dashboard/_operationals.js
const offersFlagsData = {
  '0835': {
    flag: 'glomo.contractProductSection.ocPayrollPortabilityN2',  // ← Nombre de la bandera
    lda: '0835'  // ← ID del producto
  }
};
```

Este objeto se pasa a `glomo-campaigns-dm-mx`:

```javascript
// _cross-components.js
objectOffersFlags: offersFlagsData,  // ← Se inyecta al componente
```
# Sixth question

**Tu override SÍ incluye la oferta 0835, pero tiene dos problemas estructurales:** 

Veo que se revirtió mi inyección del mock. Perfecto, porque tu override es la solución correcta. El problema es que **tu payload tiene dos errores**:

## Problemas en tu override de la oferta 0835:

```javascript
{
  "id": "pLgwPVbPyLfmRYY_zTCf3F_KE01Jl9q5",
  "product": {
    "id": "0835",
    "productType": {
      "id": "CARDS"  // ❌ PROBLEMA 1: Debería ser "ACCOUNTS"
    }
  },
  // ❌ PROBLEMA 2: Falta completamente el campo "offerType"
  "image": { ... }
}
```

**Por eso no funcionaba originalmente:**

1. **`productType.id: "CARDS"`** → El filtro `_filterOffersByPage()` solo muestra ofertas de tipo `ACCOUNTS` en `contractProductSection`, por eso la oferta no aparecía

2. **Falta `offerType`** → El método `_parseBannerType()` no puede extraer el tipo, entonces defaultea a "coming-soon"

## Solución: Corrige tu override así:

```json
{
  "id": "pLgwPVbPyLfmRYY_zTCf3F_KE01Jl9q5",
  "name": "Trae tu nómina",
  "description": "Tus beneficios ya estan aquí, no los dejes escapar.",
  "priority": 1000000,
  "isSimulationAllowed": false,
  "isImproved": false,
  "isShownInDashboard": false,
  "period": {
    "startDate": "2025-06-01",
    "endDate": "2026-12-31"
  },
  "product": {
    "id": "0835",
    "contractId": "jHImr_oth7bCpPPq2pTWkuKjvDPbnws-zvVxnym9chw",
    "productType": {
      "id": "ACCOUNTS"
    }
  },
  "offerType": {
    "id": "payroll-portability-n2"
  },
  "origin": {
    "id": "CRM",
    "description": "NEW_OFFER_APX"
  },
  "image": {
    "id": "X_-_TNSBSJWYNGOSAxKcCOuP9PhBV6mW",
    "name": "avatar_CAMPAIGN0835-OF",
    "url": "http://archivingbbva.axtel.mx:8082/v2/documents/logo?key=avatar_CAMPAIGN0835-OF"
  },
  "isRead": false,
  "links": []
}
```

**Cambios necesarios:**
- `productType.id`: `"CARDS"` → `"ACCOUNTS"`
- Agregar campo `"offerType": { "id": "payroll-portability-n2" }`

Con esto, **solo necesitarías los 6 archivos iniciales** y el hardcoded del feature flag. La inyección manual del mock ya no sería necesaria.
