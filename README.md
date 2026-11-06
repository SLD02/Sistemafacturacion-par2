# Sistema de facturación y cajeros por turnos

Proyecto académico de escritorio creado con **C# / Windows Forms / Entity Framework Core 8 / SQL Server**. Compatible con **Visual Studio Community 2022** y Windows.

## Funcionalidades

- Primer inicio: configuración del servidor SQL y creación segura de una cuenta administrador.
- Administrador: crear cajeros y asignarles turno **Mañana, Tarde o Noche**; desactivar/reactivar y cambiar contraseñas.
- Administrador: alta y edición de **productos con stock** y **clientes**.
- Cajero: acceso con usuario y contraseña, venta contado o crédito, cantidades y validación de stock.
- Facturas: numeración única correlativa de **cinco dígitos desde 00001**, fecha/hora, cajero/turno, cliente, artículos, subtotal, ITBIS del 18%, total, pago y devuelta.
- En contado: valida dinero recibido y calcula el cambio; en crédito: crea cuenta por cobrar asociada a cliente.
- Impresión: vista previa y botón de imprimir. Para guardar PDF, seleccionar **Microsoft Print to PDF** en Windows.
- Cuentas por cobrar: consulta de saldos y registro de abonos parciales o completos.
- Reportes: **cuadre del día, ventas por cajero, inventario en stock, facturas contado/crédito y top 5 clientes**.
- Historial y reimpresión de facturas; cajeros solo ven las suyas.

## Abrir en Visual Studio Community 2022

1. Instala en **Visual Studio Installer** la carga de trabajo **Desarrollo de escritorio con .NET**, y asegúrate de tener el **SDK .NET 8**.
2. Instala o configura **SQL Server Express** o **SQL Server LocalDB**, con autenticación de Windows habilitada. El servicio de SQL Server debe estar iniciado.
3. Extrae el ZIP y abre `SistemaFacturacionWinForms.sln` en Visual Studio Community 2022.
4. Deja que Visual Studio restaure los paquetes NuGet (`Microsoft.EntityFrameworkCore.SqlServer`). Requiere internet durante la restauración inicial.
5. Pulsa **F5**. Si es el primer inicio, escribe el nombre de la instancia SQL, por ejemplo:
   - SQL Server Express: `.\SQLEXPRESS`
   - LocalDB: `(localdb)\MSSQLLocalDB`
   - Instancia predeterminada: `localhost`
6. El programa intentará crear la base `FacturacionTurnosDB` automáticamente (el usuario Windows debe tener permiso para crear bases de datos).
7. Crea el usuario **administrador** y una contraseña de al menos 8 caracteres. No existe contraseña predeterminada.
8. Entra, registra productos y clientes, crea un cajero, cierra sesión y entra con las credenciales de ese cajero para registrar ventas.

## Orden de pruebas para una entrega

1. Administrador: crea tres cajeros de los turnos mañana, tarde y noche.
2. Crea un producto `P001 / Cuaderno / RD$100 / stock 10` y otro `P002 / Lapicero / RD$25 / stock 20`.
3. Crea un cliente con nombre.
4. Entra como cajero, agrega **2 cuadernos**, elige contado, introduce **RD$500** y factura. Total base RD$200, ITBIS RD$36, a pagar RD$236, devuelta RD$264.
5. Registra otra factura a crédito con cliente; luego ve a **Cuentas por cobrar** y registra un abono.
6. Inicia sesión como administrador y prueba los cinco reportes y la reimpresión.

## Organización del código

- `Models/Entities.cs`: entidades y relaciones.
- `Data/AppDbContext.cs`: Entity Framework + SQL Server y estructura de la base.
- `Services/BillingService.cs`: ventas, numeración, stock, cobros y transacciones.
- `Services/Security.cs`: hash PBKDF2 con salt aleatoria por usuario (no se guardan claves en texto plano).
- `Services/InvoicePrinter.cs`: impresión con PrintDocument.
- `Services/DatabaseSettings.cs`: nombre de servidor y base, guardados en `%LOCALAPPDATA%\SistemaFacturacionTurnos\dbconfig.json` (no en GitHub).
- `Forms/`: formularios hechos **por código WinForms**, no con el diseñador de arrastrar/soltar.

## Aclaraciones importantes

- El enunciado dice «5 dígitos iniciando en 0001»; `0001` tiene 4 dígitos. Se implementó `00001`, `00002`, ... `99999`. Para usar 4 dígitos, cambia `D5` a `D4` en `BillingService.cs` y adapta el máximo y campo `Invoice.Number` antes de crear la base.
- El precio del producto se considera **antes de ITBIS**. Cada línea aplica 18%; el ITBIS se redondea a dos decimales por línea. Ajusta las reglas tributarias y el formato fiscal si deseas usarlo en un comercio real de República Dominicana.
- El turno queda asociado al cajero y registrado en cada factura, **sin bloqueo por horario** (así el profesor puede probar los tres turnos en cualquier momento).
- El reporte del **top 5** utiliza todas las ventas históricas que tengan cliente asociado. Las ventas de «Consumidor final» no se pueden atribuir a un cliente.
- El cuadre usa fecha local del equipo; diferencia ventas de contado, crédito y abonos de clientes. No implementa arqueo físico de caja ni apertura/cierre formal.
- Esta versión usa `EnsureCreated` para iniciar **una base nueva**. Si cambias las entidades más adelante, necesitas planificar migraciones de EF Core en lugar de depender de `EnsureCreated` para modificar tablas existentes.
- Al validar y guardar una venta se usa una transacción SQL serializable para evitar números duplicados y reducir el riesgo de sobreventa concurrente. Aun así, antes de uso real se deben implementar auditorías, pruebas integrales, respaldo de base, manejo robusto de concurrencia y requisitos fiscales locales (NCF/e-CF).
- No guardes copias de seguridad, datos de clientes reales ni cadenas de conexión con contraseña en un repositorio público.

## Subirlo a GitHub

En Visual Studio, con **esta solución abierta**, usa `Git → Crear repositorio de Git` en la **carpeta de esta solución**, asigna un nombre como `SistemaFacturacion-WinForms`, revisa la lista de archivos y confirma/publica. **No publiques tu carpeta completa C:\Users ni bin/obj.**

Este proyecto es una implementación académica de referencia; **debe compilarse y probarse en Windows con Visual Studio y SQL Server** antes de su entrega. El archivo ZIP se creó en un entorno sin SDK .NET ni SQL Server, por lo que no ha sido ejecutado aquí.


## Parche de diagnóstico (06/11/2026)

- La ventana de errores muestra la excepción completa y un botón **Copiar detalles**.
- Guarda el archivo `errores.log` en `%LOCALAPPDATA%\SistemaFacturacionWinForms`.
- El sistema confirma si la factura se guardó antes de abrir la vista previa; un error de impresión no debe provocar que se facture dos veces.
- Se intenta usar `Microsoft Print to PDF` si no hay impresora predeterminada válida.
- Si ninguna impresora está instalada, se muestra el motivo concreto.

Si una venta falla, revisa **Historial** antes de volver a facturar. Para aplicar este parche respalda el proyecto y abre la solución actualizada; **no borres la base de datos**.

## Corrección v1.2 – selección de cliente y validaciones de venta

- Se corrigió la `NullReferenceException` en `MainForm.Sales.cs`, `LoadSaleOptions()`, cuando
  la opción «Cliente no registrado (solo contado)» tiene un valor nulo.
- Se corrigió el mismo riesgo en la selección anterior del cliente y en `Sell()`.
- Pulsar «Facturar e imprimir» con la lista de productos vacía muestra ahora un aviso claro
  (se debe seleccionar producto y pulsar **Agregar** antes de facturar).
- Para ventas a crédito se exige seleccionar un cliente registrado; para ventas al contado,
  se indica si el monto recibido es menor que el total.
- Se mantienen el diagnóstico y las protecciones de impresión de la versión anterior.
- **No eliminar ni recrear la base de datos**; usar la misma instancia de SQL Server.
- Si ya se intentó una venta, **verificar Historial** antes de repetirla.

Este parche no pudo ser compilado en el entorno de preparación porque no cuenta con
.NET SDK ni SQL Server. Debe probarse en Windows/Visual Studio.
