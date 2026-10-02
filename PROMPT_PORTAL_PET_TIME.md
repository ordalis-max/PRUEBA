Actúa como desarrollador full-stack senior. Necesito que construyas desde cero una aplicación web completa, segura y lista para producción llamada "Portal de Colaboradores PET TIME", para publicarla en nuestro dominio web. Trabaja en español (interfaz, mensajes, código comentado y documentación).

## Forma de trabajo (Claude Code)
- Trabaja dentro de este repositorio. Este archivo es la especificación oficial del proyecto: si algo no está claro, pregúntame antes de suponer.
- Construye por etapas (ver el final). Al terminar cada etapa: ejecuta las pruebas, haz commit con un mensaje claro, haz push y detente para mostrarme el avance. No pases a la siguiente etapa sin mi confirmación.
- Registra las decisiones importantes (stack, librerías, cambios al modelo de datos) en `docs/DECISIONES.md`.
- Nunca subas al repositorio contraseñas, el archivo `.env`, ni archivos de trabajadores.

## 0. Antes de escribir código
1. Pregúntame qué hosting/dominio tenemos (hosting compartido con cPanel, VPS, Vercel, etc.) y elige el stack según eso:
   - Hosting compartido con cPanel → Laravel 11 + MySQL (PHP 8.2+).
   - VPS o plataforma moderna → Next.js + PostgreSQL (por ejemplo, Supabase) o Laravel, lo que sea más simple de mantener.
2. Propón la estructura del proyecto y el modelo de base de datos, y espera mi confirmación antes de construir.
3. Al final entrega un README de despliegue paso a paso pensado para una persona que solo va a subirlo al dominio.

## 1. Contexto de la empresa
- PET TIME es una red de clínicas veterinarias y servicios para mascotas en Lima, Perú.
- Sedes: Chorrillos, Surco, Miraflores, San Borja y Jesús María (más "Oficina central"). Deben ser editables.
- Empresas pagadoras (editables, con RUC y razón social): Veterinarias Torres Arévalo E.I.R.L. y Veterinarias Pet Time E.I.R.L.
- Cada trabajador tiene una modalidad de contratación: PLANILLA o RECIBO POR HONORARIOS (RH). La modalidad define qué módulos ve.

## 2. Identidad visual
- Turquesa principal #027f7a · Fucsia para destacar #ee2a8a · Celeste de fondo #dceff8 · Gris para textos secundarios #6c6e6c.
- Tipografía de marca: MADE TOMMY (Regular, Medium, Bold, ExtraBold, Black). Dejar preparado para cargar el archivo de fuente; usar Nunito (Google Fonts) como respaldo.
- Espacio para el logo de PET TIME (te lo daré como archivo).
- Diseño limpio y amigable, con íconos de mascotas sutiles. Debe funcionar muy bien en celular, porque la mayoría de colaboradores entrará desde el teléfono.

## 3. Usuarios, acceso y seguridad (obligatorio)
- Usuario = número de documento (DNI, carné de extranjería o pasaporte). Clave inicial = el mismo número de documento.
- En el primer ingreso es obligatorio cambiar la contraseña (mínimo 8 caracteres).
- Contraseñas guardadas con hash (bcrypt/argon2), nunca en texto plano.
- Roles y permisos reales en el servidor, no solo ocultar botones. Si un trabajador escribe a mano una URL de administrador o pide un archivo de otra persona, el servidor debe responder "No autorizado" (403).
- Cada trabajador ve SOLO su propia información y sus propios archivos.
- Los archivos (PDF, imágenes) se guardan en almacenamiento privado, fuera de la carpeta pública, y se entregan solo mediante una ruta que verifica permisos (o enlaces firmados que expiran).
- Bloqueo temporal tras 5 intentos fallidos de inicio de sesión. HTTPS obligatorio. Cierre de sesión por inactividad.
- "Olvidé mi contraseña": la restablece un administrador (vuelve a ser el número de documento y obliga a cambiarla otra vez).
- Casilla de consentimiento de tratamiento de datos personales (Ley N.° 29733) que el trabajador acepta en su primer ingreso.
- Registro de auditoría: quién subió, vio, descargó, aprobó, editó o eliminó cada cosa, con fecha y hora.

### Roles
- SUPERADMIN (Director): todo. Carga pagos (constancias de abono) y boletas de planilla. Gestiona administradores y configuración. Descarga masiva.
- ADMIN (Vanessa): carga y mantiene la base de trabajadores (incluida la modalidad PLANILLA/RH), carga los montos de RH a facturar, revisa y aprueba/observa los recibos por honorarios. Descarga masiva.
- TRABAJADOR: solo su información: perfil, boletas o recibos, pagos, bandeja de notificaciones.
Los permisos de cada administrador deben poder ajustarse desde Configuración.

## 4. Módulos del ADMINISTRADOR

### 4.1 Panel de inicio (dashboard)
- Total de trabajadores activos, por sede y por modalidad (Planilla / RH).
- Perfiles incompletos.
- Boletas del mes cargadas vs. trabajadores en planilla; boletas aún no vistas por el trabajador.
- RH del mes: notificados / recibo cargado / aprobados / observados / pendientes / pagados.
- Pagos registrados del mes y montos totales.
- Alertas: documentos de identidad por vencer, recibos con monto distinto al solicitado.

### 4.2 Trabajadores (gestión de usuarios)
- Crear, editar, activar/desactivar trabajadores; restablecer contraseña.
- Campos mínimos: tipo de documento, número de documento, apellidos y nombres, sede, cargo/área, modalidad (PLANILLA / RH), empresa pagadora, fecha de ingreso, correo, celular, estado.
- Carga masiva desde Excel (.xlsx) con vista previa: nuevos, actualizados, con errores (documento duplicado o vacío, modalidad inválida). Solo se guarda al confirmar.
- Plantilla Excel descargable desde la app.
- Al crear un trabajador se crea automáticamente su usuario (documento / documento).
- Ver la ficha completa del trabajador y descargarla en PDF.
- Exportar la base de personal a Excel con filtros.

### 4.3 Carga de boletas de pago (PDF) — trabajadores en PLANILLA
- Subida individual o masiva (arrastrar muchos PDF a la vez o un .zip).
- Asignación automática por nombre de archivo: el sistema busca el número de documento dentro del nombre (ej. 45781236_2026-09.pdf, BOLETA 45781236 SET.pdf) y lo asigna a esa persona.
- Alternativa: subir un Excel de relación con columnas NOMBRE_ARCHIVO | DOCUMENTO | PERIODO.
- Vista previa antes de confirmar: asignados correctamente, sin coincidencia (asignación manual con buscador), documentos inexistentes en la base, posibles duplicados del mismo periodo.
- Seleccionar periodo (mes/año) y tipo (remuneración mensual, gratificación, CTS, vacaciones, liquidación, otro).
- Reemplazar o anular un documento (queda en el historial).
- Ver quién ya vio/descargó su boleta y quién no.
- Al cargar, el trabajador recibe una notificación en su bandeja.

### 4.4 Montos de Recibos por Honorarios (RH) — lo carga ADMIN (Vanessa)
- Carga desde Excel con columnas: DOCUMENTO | NOMBRE | MONTO | EMPRESA (RUC o razón social) | PERIODO | CONCEPTO (opcional). Plantilla descargable. También carga individual.
- Vista previa con validaciones (documento no existe, trabajador no es RH, monto vacío) antes de confirmar.
- Al confirmar, cada trabajador RH recibe en su bandeja de entrada: "Emite tu Recibo por Honorarios del periodo [MES AÑO] por S/ [MONTO] a nombre de [RAZÓN SOCIAL] – RUC [RUC]. Concepto sugerido: [CONCEPTO]."
- Fecha límite configurable (ej. día 5 del mes) y recordatorios automáticos a quienes aún no cargan.

### 4.5 Revisión de Recibos por Honorarios
- Bandeja con todos los RH cargados, filtrable por periodo, empresa, sede, estado y trabajador.
- Estados: PENDIENTE DE CARGA → CARGADO / EN REVISIÓN → APROBADO u OBSERVADO (comentario obligatorio) → PAGADO.
- Si se observa, el trabajador ve el motivo y puede volver a subir el recibo corregido.
- Alerta automática si el monto no coincide con el solicitado, si el RUC receptor no corresponde a la empresa indicada o si el número de recibo ya existe.

### 4.6 Pagos — lo carga SUPERADMIN
- Registrar pagos individual o masivamente, con la misma asignación automática por nombre de archivo o por Excel de relación (NOMBRE_ARCHIVO | DOCUMENTO | PERIODO | TIPO | MONTO | FECHA).
- Acepta imágenes (JPG, PNG) y PDF de constancias de transferencia.
- PLANILLA: constancia de abono del periodo.
- RH: el pago se compone de (1) Constancia SUNAT (pago de la retención de 4.a categoría, cuando aplique), (2) Abono 1 (constancia + monto + fecha), (3) Abono 2 (constancia + monto + fecha). Cada pago queda vinculado a su recibo del periodo; al completarse, el recibo pasa a PAGADO y el trabajador recibe notificación.
- Cálculo de apoyo: monto bruto, retención (8 % si supera el tope vigente y no hay suspensión de 4.a categoría; tasa y tope editables), neto a pagar, total abonado y saldo pendiente.
- Historial de pagos con filtros y exportación a Excel.

### 4.7 Descargas masivas (SUPERADMIN y ADMIN)
- Descargar en .zip todos los PDF de recibos por honorarios, filtrando por periodo, empresa, sede y estado. Archivos renombrados: RUC_EMPRESA/PERIODO/DOCUMENTO_APELLIDOS_SERIE-NUMERO.pdf.
- Descargar la relación en Excel de los RH: documento, apellidos y nombres, RUC del trabajador, serie y número, fecha de emisión, periodo, monto bruto, retención, neto, empresa receptora (RUC y razón social), sede, cuenta/CCI, estado, fecha de carga, fecha de pago.
- Opcional: descarga masiva de boletas y constancias de pago por periodo.

### 4.8 Comunicados
- Enviar comunicados a todos, por sede o por modalidad (llegan a la bandeja del trabajador), con confirmación de lectura.

### 4.9 Configuración
- Sedes, cargos, empresas pagadoras (RUC, razón social), tipos de documento, tasa y tope de retención, fecha límite de RH, permisos de administradores, textos de notificaciones.

## 5. Módulos del TRABAJADOR

### 5.1 Inicio / Bandeja de entrada
- Resumen personal y bandeja de notificaciones (nueva boleta, solicitud de RH con monto, recibo observado, pago realizado, comunicados), con contador de no leídas.
- Banner si el perfil está incompleto.

### 5.2 Mi ficha de personal (la llena el trabajador)
- Foto personal (con recorte).
- Documento de identidad: anverso y reverso (foto o PDF), fecha de vencimiento.
- Datos personales: nombres, apellidos, fecha de nacimiento, sexo, estado civil, nacionalidad, celular, correo, dirección y distrito.
- Contacto de emergencia: nombre, parentesco, celular.
- Escolaridad y formación: nivel (secundaria, técnico, universitario, posgrado), institución, carrera, grado/título, año, archivo de sustento; cursos y certificaciones.
- Colegiatura profesional (médicos veterinarios u otros): número, estado de habilitación, archivo.
- Datos bancarios: banco, número de cuenta, CCI, titular.
- Si es PLANILLA: sistema de pensiones (AFP/ONP) y AFP.
- Si es RH: número de RUC y constancia de suspensión de retenciones de 4.a categoría (archivo y año).
- Talla de uniforme (opcional).
- El administrador puede marcar datos como "verificados". Documento, modalidad y sede solo los cambia el administrador.

### 5.3 Si es PLANILLA → "Mis boletas de pago"
- Lista por periodo y tipo, con fecha de emisión. Ver en pantalla y descargar el PDF.
- Botón "Confirmo la recepción" (queda como constancia de entrega).

### 5.4 Si es RECIBO POR HONORARIOS → "Mis recibos por honorarios"
- Ve muy destacada la solicitud del mes: monto a facturar, empresa (razón social y RUC) y periodo.
- Formulario de carga: fecha de emisión, periodo que cobra, serie y número (ej. E001-25), monto, su RUC y nombre, empresa a la que facturó (desplegable), ¿retención? / ¿suspensión de 4.a categoría?, PDF del recibo (obligatorio), observaciones.
- Validaciones: RUC de 11 dígitos, monto igual al solicitado (si difiere, pedir justificación), número de recibo no repetido, solo PDF de máximo 5 MB.
- Historial mes a mes con estado (pendiente, en revisión, aprobado, observado con motivo, pagado).

### 5.5 Mis pagos
- PLANILLA: constancia de abono por periodo.
- RH: vista grande y clara por periodo con el recibo emitido, la constancia de SUNAT, el Abono 1 y el Abono 2 (montos, fechas y archivos para ver/descargar), total pagado y saldo.

### 5.6 Mi cuenta
- Cambiar contraseña, ver el consentimiento de datos aceptado, cerrar sesión.

## 6. Requisitos técnicos
- Base de datos relacional normalizada (usuarios, roles/permisos, trabajadores, sedes, empresas, documentos de identidad, formación, boletas, solicitudes RH, recibos RH, pagos y archivos, notificaciones, comunicados, auditoría, configuración).
- Lectura de Excel robusta: tolera mayúsculas, espacios y tildes en encabezados; conserva ceros a la izquierda en documentos (no convertir a número).
- Montos en soles con 2 decimales. Zona horaria America/Lima. Fechas dd/mm/aaaa.
- Límites de tamaño y validación del tipo real de archivo (no solo la extensión).
- Paginación y buscadores en todas las tablas de administración.
- Seed inicial: crear SUPERADMIN y ADMIN (Vanessa) con credenciales en variables de entorno, sedes y empresas.
- Pruebas automáticas de autorización: un trabajador no puede ver datos ni archivos de otro ni entrar a rutas de administrador.
- Documentar cómo respaldar la base de datos y la carpeta de archivos.

## 7. Entregables
1. Código completo del proyecto.
2. Plantillas Excel descargables: trabajadores, montos RH, relación de archivos de boletas, relación de archivos de pagos.
3. README de despliegue paso a paso para nuestro hosting (requisitos, variables de entorno, base de datos, subida de archivos, correo si se usa, creación del primer administrador, copias de seguridad).
4. Manual corto de uso: uno para administradores y uno para trabajadores.

## 8. Etapas
Construye por etapas y muéstrame avances (commit + push + pausa al final de cada una):
1) Login, roles y gestión de trabajadores con carga Excel
2) Ficha de personal del trabajador
3) Boletas (carga masiva con asignación automática)
4) Flujo RH completo (montos → notificación → carga → revisión)
5) Pagos (SUNAT + 2 abonos)
6) Descargas masivas, dashboard, comunicados y auditoría
7) Pruebas, README y manuales
