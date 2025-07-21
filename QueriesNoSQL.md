# Consultas NoSQL para Sistema Bancario en MongoDB

A continuación se presentan 5 enunciados de consultas basados en las colecciones NoSQL del sistema bancario, junto con las soluciones utilizando operaciones avanzadas de MongoDB.

## 1. Análisis de Saldos por Tipo de Cuenta

**Enunciado:** El departamento financiero necesita un informe que muestre el saldo total, promedio, máximo y mínimo por cada tipo de cuenta (ahorro y corriente) para evaluar la distribución de fondos en el banco.

**Consulta MongoDB:**
```javascript
db.Clientes.aggregate([
  { 
    $unwind: "$cuentas" 
  },
  { 
    $group: {
      _id: "$cuentas.tipo_cuenta", 
      saldo_total: { $sum: "$cuentas.saldo" }, 
      saldo_promedio: { $avg: "$cuentas.saldo" }, 
      saldo_maximo: { $max: "$cuentas.saldo" }, 
      saldo_minimo: { $min: "$cuentas.saldo" }  
    }
  },
  { 
    $project: {
      tipo_cuenta: "$_id", 
      _id: 0, 
      saldo_total: 1,
      saldo_promedio: 1,
      saldo_maximo: 1,
      saldo_minimo: 1
    }
  }
]);
```

## 2. Patrones de Transacciones por Cliente

**Enunciado:** El equipo de análisis de comportamiento necesita identificar los patrones de transacciones de cada cliente, mostrando la cantidad y el monto total de transacciones por tipo (depósito, retiro, transferencia) para cada cliente.

**Consulta MongoDB:**
```javascript
db.Transacciones.aggregate([
  {
    $group: {
      _id: {
        cliente: "$cliente_ref", 
        tipo_transaccion: "$tipo_transaccion" 
      },
      cantidad_transacciones: { $sum: 1 }, 
      monto_total: { $sum: "$monto" } 
    }
  },
  {
    $group: {
      _id: "$_id.cliente", 
      transacciones_por_tipo: {
        $push: {
          tipo_transaccion: "$_id.tipo_transaccion",
          cantidad_transacciones: "$cantidad_transacciones",
          monto_total: "$monto_total"
        }
      }
    }
  },
  {
    $project: {
      cliente: "$_id", 
      _id: 0, 
      transacciones_por_tipo: 1
    }
  }
]);
```

## 3. Clientes con Múltiples Tarjetas de Crédito

**Enunciado:** El departamento de riesgo crediticio necesita identificar a los clientes que poseen más de una tarjeta de crédito, mostrando sus datos personales, cantidad de tarjetas y el detalle de cada una.

**Consulta MongoDB:**
```javascript
db.Clientes.aggregate([
  {
    $unwind: "$cuentas" 
  },
  {
    $unwind: "$cuentas.tarjetas" 
  },
  {
    $match: { "cuentas.tarjetas.tipo_tarjeta": "credito" } 
  },
  {
    $group: {
      _id: "$_id", 
      nombre: { $first: "$nombre" }, 
      cedula: { $first: "$cedula" }, 
      correo: { $first: "$correo" }, 
      direccion: { $first: "$direccion" }, 
      cantidad_tarjetas: { $sum: 1 }, 
      detalle_tarjetas: { $push: "$cuentas.tarjetas" } 
    }
  },
  {
    $match: { cantidad_tarjetas: { $gt: 1 } } 
  },
  {
    $project: {
      _id: 0, 
      nombre: 1,
      cedula: 1,
      correo: 1,
      direccion: 1,
      cantidad_tarjetas: 1,
      detalle_tarjetas: 1
    }
  }
]);
```

## 4. Análisis de Medios de Pago más Utilizados

**Enunciado:** El departamento de marketing necesita conocer cuáles son los medios de pago más utilizados para depósitos, agrupados por mes, para orientar sus campañas promocionales.

**Consulta MongoDB:**
```javascript
db.Transacciones.aggregate([
  {
    $match: { tipo_transaccion: "deposito" } 
  },
  {
    $group: {
      _id: {
        mes: { $month: { $dateFromString: { dateString: "$fecha" } } }, 
        medio_pago: "$detalles_deposito.medio_pago" 
      },
      cantidad: { $sum: 1 } 
    }
  },
  {
    $group: {
      _id: "$_id.mes", 
      medios_pago: {
        $push: {
          medio_pago: "$_id.medio_pago",
          cantidad: "$cantidad"
        }
      }
    }
  },
  {
    $project: {
      mes: "$_id", 
      _id: 0, 
      medios_pago: 1
    }
  },
  {
    $sort: { mes: 1 } 
  }
]);
```

## 5. Detección de Cuentas con Transacciones Sospechosas

**Enunciado:** El departamento de seguridad necesita identificar cuentas con patrones de transacciones sospechosas, definidas como aquellas que tienen más de 3 retiros en un mismo día con un monto total superior a 1,000,000 COP.

**Consulta MongoDB:**
```javascript
db.Transacciones.aggregate([
  {
    $match: { tipo_transaccion: "retiro" } 
  },
  {
    $group: {
      _id: {
        num_cuenta: "$num_cuenta", 
        dia: { $dateToString: { format: "%Y-%m-%d", date: { $dateFromString: { dateString: "$fecha" } } } } 
      },
      cantidad_retiros: { $sum: 1 }, 
      monto_total: { $sum: "$monto" } 
    }
  },
  {
    $match: {
      cantidad_retiros: { $gt: 3 }, 
      monto_total: { $gt: 1000000 } 
    }
  },
  {
    $project: {
      _id: 0, 
      num_cuenta: "$_id.num_cuenta", 
      dia: "$_id.dia", 
      cantidad_retiros: 1,
      monto_total: 1
    }
  }
]);
```
