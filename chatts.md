¡Exacto! En esta arquitectura se usan objetos de configuración para TODO. Te voy a explicar cómo funciona y cómo crear una nueva funcionalidad completa.

## **Objetos de configuración para todo**

### **1. Configuración de componentes:**
```javascript
const miComponenteDM = {
  zone: 'app__main',
  type: 'DM',
  tag: 'mi-componente-dm',
  properties: {
    // configuración
  }
};
```

### **2. Configuración de routing:**
```javascript
// En _web-routes.js
'miNuevaPagina': '/mi-nueva-pagina/:parametro'
```

### **3. Configuración de navegación:**
```javascript
cellsConnections: {
  out: {
    navigation_to_mi_pagina: {
      bind: 'navigate-to-mi-pagina',
      link: {
        page: 'miNuevaPagina',
        params: {
          parametro: 'valor'
        }
      }
    }
  }
}
```

## **Ejemplo completo: Crear nueva funcionalidad**

### **Paso 1: Crear la nueva página**

### **Paso 2: Agregar la ruta**

### **Paso 3: Agregar navegación desde el dashboard**

### **Paso 4: Crear una card en el dashboard que navegue**

## **Flujo completo de la nueva funcionalidad**

### **1. Flujo de navegación:**
```
Dashboard
    ↓ (click en mi-card-personalizada)
navigation_to_mi_pagina
    ↓ (Cells Framework)
/mi-nueva-pagina/desde-dashboard
    ↓ (routing)
miNuevaPagina.js
    ↓ (instancia componentes)
mi-componente-dm + mi-componente-ui
```

### **2. Flujo de datos:**
```
dashboard_accounts (datos del dashboard)
    ↓
mi-componente-dm (procesa datos)
    ↓
datos_procesados
    ↓
mi-componente-ui (muestra datos)
```

### **3. Estructura de archivos:**
```
glomo-mx/
├── app/
│   ├── composerMocksTpl/
│   │   ├── miNuevaPagina.js (← NUEVA PÁGINA)
│   │   └── dashboard/
│   │       └── _oldDashboard.js (← MODIFICADO)
│   └── config/mx/base/
│       └── _web-routes.js (← MODIFICADO)
└── package.json (← AGREGAR DEPENDENCIAS)
```

### **4. Dependencias necesarias:**
```json
{
  "@mi-components/mi-componente-dm": "1.0.0",
  "@mi-components/mi-componente-ui": "1.0.0",
  "@mi-components/mi-card-personalizada": "1.0.0"
}
```

## **Resumen de la arquitectura**

### **Todo se configura con objetos:**

1. **Componentes:** Objetos con `zone`, `type`, `tag`, `properties`
2. **Routing:** Objetos con rutas en `_web-routes.js`
3. **Navegación:** Objetos con `cellsConnections` y `link`
4. **Datos:** Objetos con `bind` que conectan componentes

### **Cells Framework hace el resto:**
- Lee configuraciones
- Crea instancias reales
- Maneja routing
- Conecta componentes
- Gestiona ciclo de vida

¿Te queda claro cómo funciona esta arquitectura basada en configuración? ¿Quieres que te ayude a implementar algo específico?


Te voy a explicar exactamente cómo se instancia `bbva-feature-product-detail-accounts-mx` y cómo recibe la propiedad `dashboardAccounts`.

## **Flujo de instanciación y asignación de propiedades**

### **1. Dónde se instancia el componente:**

El componente se instancia cuando navegas a la ruta `/account/:productId`. El flujo es:

```
Navegación a /account/123
    ↓
Cells Framework busca configuración de 'account'
    ↓
Carga app/composerMocksTpl/account.js
    ↓
Instancia bbva-feature-product-detail-accounts-mx
```

### **2. Configuración en account.js:**

Read file: app/composerMocksTpl/account.js
### **3. Cómo se asigna `dashboardAccounts`:**

El binding clave está en la línea 71:

```javascript
dashboard_accounts: {
  bind: 'dashboardAccounts'
}
```

**Esto significa:**
- Cuando se emite el evento `dashboard_accounts` en el sistema
- Se llama al método `dashboardAccounts` del componente `bbva-feature-product-detail-accounts-mx`
- Este método recibe los datos y los asigna a `this.dashboardAccounts`

### **4. Quién emite `dashboard_accounts`:**

El evento `dashboard_accounts` se emite desde el componente `glomo-financial-overview-dm-mx` en el dashboard:

Read file: app/composerMocksTpl/dashboard/_oldDashboard.js
### **5. Flujo completo de datos:**

```
1. glomo-financial-overview-dm-mx (en dashboard)
   ↓ (emite evento)
   dashboard_accounts_from_financial: { bind: 'accounts-fetched' }
   
2. glomo-financial-overview-dm-mx (en dashboard)
   ↓ (emite evento)
   dashboard_accounts: { bind: 'dashboard-cached-accounts' }
   
3. bbva-feature-product-detail-accounts-mx (en account.js)
   ↓ (recibe evento)
   dashboard_accounts: { bind: 'dashboardAccounts' }
   
4. bbva-feature-product-detail-accounts-mx (componente interno)
   ↓ (asigna propiedad)
   this.dashboardAccounts = datos
```

### **6. Dónde interceptar para remapear alias:**

**Opción A: En account.js (recomendado)**
```javascript
// Agregar antes del featureAccount
const aliasNormalizer = {
  zone: 'app__main',
  type: 'UI',
  tag: 'glomo-alias-normalizer',
  properties: {
    cellsConnections: {
      in: {
        dashboard_accounts: {
          bind: 'normalizeAlias'
        }
      },
      out: {
        dashboard_accounts_normalized: {
          bind: 'sendNormalizedAccounts'
        }
      }
    }
  }
};

// Modificar el featureAccount
const featureAccount = {
  // ...
  cellsConnections: {
    in: {
      dashboard_accounts_normalized: {  // ← Cambiar aquí
        bind: 'dashboardAccounts'
      }
    }
  }
};
```

**Opción B: En el dashboard**
```javascript
// En _oldDashboard.js, modificar el binding
dashboard_accounts: {
  bind: 'dashboard-cached-accounts-normalized'  // ← Agregar normalización aquí
}
```

### **7. Cuándo se ejecuta:**

1. **Al navegar** a `/account/:productId`
2. **Cells Framework** instancia `bbva-feature-product-detail-accounts-mx`
3. **Se establecen** los bindings de `cellsConnections`
4. **Cuando se emite** `dashboard_accounts`, se llama `this.dashboardAccounts(datos)`
5. **El componente** procesa los datos y actualiza su estado interno

¿Te queda claro el flujo? ¿Quieres que implemente el normalizador en `account.js` para que puedas probar el remapeo del alias?
