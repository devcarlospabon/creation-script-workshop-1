# Consultas SQL para Base de Datos Bancaria

## Enunciado 1: Clientes con múltiples cuentas y sus saldos totales

**Necesidad:** El banco necesita identificar a los clientes que tienen más de una cuenta, mostrar cuántas cuentas tiene cada uno y el saldo total acumulado en todas sus cuentas, ordenados por saldo total de mayor a menor.

**Consulta SQL:**
```sql
select c.id_cliente ,c.nombre ,c.cedula , count (c2.id_cliente) as cantidad_cuentas, sum(c2.saldo)as saldo_total from cliente c
join cuenta c2 on c.id_cliente =c2.id_cliente
group by c.id_cliente ,c.nombre,c.cedula
having count (c2.id_cliente)>1
order by sum(c2.saldo) desc;
```

## Enunciado 2: Comparativa entre depósitos y retiros por cliente

**Necesidad:** El departamento de análisis financiero necesita comparar los montos totales de depósitos y retiros realizados por cada cliente, para identificar patrones de comportamiento financiero.

**Consulta SQL:**
```sql
select c1.nombre,c1.cedula, ob1.total as total_deposito , ob2.total as total_retiro from cliente c1 
left join (select c.id_cliente ,SUM(t.monto) as total from cliente c 
join cuenta c2 on c.id_cliente =c2.id_cliente
join transaccion t on c2.num_cuenta =t.num_cuenta
where t.tipo_transaccion ='deposito'
group by c.id_cliente) ob1 on c1.id_cliente =ob1.id_cliente
left join (select c.id_cliente ,SUM(t.monto) as total from cliente c 
join cuenta c2 on c.id_cliente =c2.id_cliente
join transaccion t on c2.num_cuenta =t.num_cuenta
where t.tipo_transaccion ='retiro'
group by c.id_cliente) ob2 on c1.id_cliente =ob2.id_cliente
where ob1.total is not null or ob2.total is not null;
```

## Enunciado 3: Cuentas sin tarjetas asociadas

**Necesidad:** El departamento de tarjetas necesita identificar todas las cuentas que no tienen tarjetas asociadas para ofrecer productos específicos a estos clientes.

**Consulta SQL:**
```sql
select c2.nombre as cliente ,c.num_cuenta from cuenta c 
join cliente c2 on c2.id_cliente = c.id_cliente
where c.num_cuenta not in (select distinct t.num_cuenta from tarjeta t);
```

## Enunciado 4: Análisis de saldos promedio por tipo de cuenta y comportamiento transaccional

**Necesidad:** La gerencia necesita un análisis comparativo del saldo promedio entre cuentas de ahorro y corriente, pero solo considerando aquellas cuentas que han tenido al menos una transacción en los últimos 30 días.

**Consulta SQL:**
```sql
select (select AVG(c2.saldo) as promedio FROM cuenta c2 
join transaccion t on c2.num_cuenta =t.num_cuenta
where c2.tipo_cuenta  ='ahorro' 
and t.fecha between (now() - INTERVAL '30 days') AND now() 
) as promedio_ahorro,
(select AVG(c2.saldo) as promedio FROM cuenta c2 
join transaccion t on c2.num_cuenta =t.num_cuenta
where c2.tipo_cuenta  ='corriente' 
and t.fecha between (now() - INTERVAL '30 days') AND now()
) as promedio_corriente;
```

## Enunciado 5: Clientes con transferencias pero sin retiros en cajeros

**Necesidad:** El equipo de marketing digital necesita identificar a los clientes que utilizan transferencias pero no realizan retiros por cajeros automáticos, para dirigir campañas de banca digital.

**Consulta SQL:**
```sql
select distinct c.id_cliente ,c.nombre ,c.cedula from cliente c 
where c.id_cliente in (select c.id_cliente from cliente c 
join cuenta c2 on c.id_cliente =c2.id_cliente
join transaccion t on c2.num_cuenta =t.num_cuenta 
where t.tipo_transaccion ='transferencia' 
except select c.id_cliente from cliente c 
join cuenta c2 on c.id_cliente =c2.id_cliente
join transaccion t on c2.num_cuenta =t.num_cuenta 
where t.descripcion ='Retiro en cajero') ;
```
