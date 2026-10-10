Actúa como desarrollador full-stack senior. Necesito que construyas desde cero una aplicación web completa, segura y lista para producción llamada "Portal de Colaboradores PET TIME", para publicarla en nuestro dominio web. Trabaja en español (interfaz, mensajes, código comentado y documentación).

## Forma de trabajo (Claude Code)
- Trabaja dentro de este repositorio. Este archivo es la especificación oficial del proyecto: si algo no está claro, pregúntame antes de suponer.
- Referencia visual aprobada: `docs/Propuesta_Portal_Colaboradores_PET_TIME_v1.0.pdf` (colores, tipografía, estructura de pantallas). Los módulos nuevos (cese, justificaciones, solicitudes, adelantos, amonestaciones) siguen el mismo estilo.
- Construye por etapas (ver el final). Al terminar cada etapa: ejecuta las pruebas, haz commit con un mensaje claro, haz push y detente para mostrarme el avance. No pases a la siguiente etapa sin mi confirmación.
- Las mejoras de la sección 8 llevan entre corchetes la etapa en la que se integran (ej. [E1]).
- Registra las decisiones importantes (stack, librerías, cambios al modelo de datos) en `docs/DECISIONES.md`.
- Nunca subas al repositorio contraseñas, el archivo `.env`, ni archivos de trabajadores.

## 0. Antes de escribir código
1. Pregúntame qué hosting/dominio tenemos (hosting compartido con cPanel, VPS, Vercel, etc.) y elige el stack según eso:
   - Hosting compartido con cPanel → Laravel 11 + MySQL (PHP 8.2+).
   - VPS o plataforma moderna → Next.js + PostgreSQL (por ejemplo, Supabase) o Laravel, lo que sea más simple de mantener.
2. Pregúntame qué correo usaremos para enviar notificaciones (SMTP del dominio, Gmail/Google Workspace o Microsoft 365).
3. Propón la estructura del proyecto y el modelo de base de datos, y espera mi confirmación antes de construir.
4. Al final entrega un README de despliegue paso a paso pensado para una persona que solo va a subirlo al dominio.

## 1. Contexto de la empresa
- PET TIME es una red de clínicas veterinarias y servicios para mascotas en Lima, Perú.
- Sedes: Chorrillos, Surco, Miraflores, San Borja y Jesús María (más "Oficina central"). Deben ser editables.
- Empresas pagadoras (editables, con RUC y razón social): Veterinarias Torres Arévalo E.I.R.L. y Veterinarias Pet Time E.I.R.L.
- Cada trabajador tiene una modalidad de contratación: PLANILLA o RECIBO POR HONORARIOS (RH). La modalidad define qué módulos ve.

## 2. Identidad visual
- Turquesa principal #027f7a · Fucsia para destacar #ee2a8a · Celeste de fondo #dceff8 · Gris para textos secundarios #6c6e6c.
- Tipografía de marca: MADE TOMMY (Regular, Medium, Bold, ExtraBold, Black). Dejar preparado para cargar el archivo de fuente; usar Nunito (Google Fonts) como respaldo.
- Espacio para el logo de PET TIME (te lo daré como archivo). El logo y los datos de la empresa deben aparecer también en los PDF que genere el sistema (formatos de adelanto, constancias).
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
- Los documentos de salud (descansos médicos, certificados) son datos sensibles: solo los ven el propio trabajador y los administradores con permiso de RR. HH.
- Registro de auditoría: quién subió, vio, descargó, aprobó, denegó, editó o eliminó cada cosa, con fecha, hora e IP.

### Roles
- SUPERADMIN (Director): todo. Carga pagos (constancias de abono) y boletas de planilla. Gestiona administradores y configuración. Descarga masiva.
- ADMIN (Vanessa): carga y mantiene la base de trabajadores (ingresos y ceses, modalidad PLANILLA/RH), carga los montos de RH a facturar, revisa y aprueba/observa los recibos por honorarios. Descarga masiva.
- TRABAJADOR: solo su información: perfil, boletas o recibos, pagos, solicitudes, justificaciones, amonestaciones y bandeja de notificaciones.
Los permisos de cada administrador deben poder ajustarse desde Configuración, por módulo y por acción (ver, cargar, aprobar/denegar, descargar). Por defecto, ambos administradores pueden gestionar solicitudes, justificaciones y amonestaciones.

## 4. Módulos del ADMINISTRADOR

### 4.1 Panel de inicio (dashboard)
- Total de trabajadores activos y cesados, por sede y por modalidad (Planilla / RH). Ingresos y ceses del mes.
- Perfiles incompletos.
- Boletas del mes cargadas vs. trabajadores en planilla; boletas aún no vistas.
- RH del mes: notificados / recibo cargado / aprobados / observados / pendientes / pagados.
- Pagos registrados del mes y montos totales.
- Solicitudes pendientes (vacaciones, permisos, adelantos) y justificaciones de inasistencia pendientes.
- Amonestaciones sin descargo y con plazo por vencer.
- Alertas: documentos de identidad por vencer, recibos con monto distinto al solicitado.

### 4.2 Trabajadores: ingreso, datos y cese
- Crear, editar y restablecer contraseña.
- Campos mínimos: tipo y número de documento, apellidos y nombres, sede, cargo/área, modalidad (PLANILLA / RH), empresa pagadora, FECHA DE INGRESO (obligatoria), correo, celular, estado (ACTIVO / CESADO).
- Antigüedad calculada automáticamente desde la fecha de ingreso.
- DAR DE BAJA (cese): botón que pide fecha de cese (obligatoria), motivo (renuncia, término de contrato, despido, abandono, mutuo acuerdo, otro), observaciones y documento de sustento opcional (carta de renuncia, liquidación).
  - Al cesar, el trabajador pasa a CESADO y sale de las listas activas, cargas masivas y solicitudes de RH.
  - Conserva acceso de SOLO LECTURA durante un plazo configurable (por defecto 90 días) para descargar sus boletas, liquidación y constancias; luego su acceso se bloquea automáticamente. Su información nunca se borra.
  - Opción "Reingresar" para un excesado: registra la nueva fecha de ingreso y guarda el historial de periodos laborados (ingreso–cese).
- Carga masiva desde Excel (.xlsx) con vista previa: nuevos, actualizados, ceses, con errores (documento duplicado o vacío, modalidad inválida, fecha de ingreso faltante). Solo se guarda al confirmar. Debe incluir columnas FECHA_INGRESO, FECHA_CESE y MOTIVO_CESE.
- Plantilla Excel descargable desde la app.
- Al crear un trabajador se crea automáticamente su usuario (documento / documento).
- Ver la ficha completa del trabajador con pestañas: datos, documentos, boletas/recibos, pagos, solicitudes, justificaciones, amonestaciones e historial. Descargar ficha en PDF.
- Exportar la base de personal a Excel con filtros (incluye activos y cesados, fecha de ingreso, cese y motivo).

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
- Vista previa con validaciones (documento no existe, trabajador no es RH o está cesado, monto vacío) antes de confirmar.
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
- RH: el pago se compone de (1) Constancia SUNAT (pago de la retención de 4.ª categoría, cuando aplique), (2) Abono 1 (constancia + monto + fecha), (3) Abono 2 (constancia + monto + fecha). Cada pago queda vinculado a su recibo del periodo; al completarse, el recibo pasa a PAGADO y el trabajador recibe notificación.
- Cálculo de apoyo: monto bruto, retención (8 % si supera el tope vigente y no hay suspensión de 4.ª categoría; tasa y tope editables), neto a pagar, total abonado y saldo pendiente.
- Tipo de pago "Adelanto": se vincula a la solicitud de adelanto aprobada (ver 4.8).
- Historial de pagos con filtros y exportación a Excel.

### 4.7 Justificaciones de inasistencias (revisión)
- Bandeja con todas las justificaciones enviadas por los trabajadores, filtrable por sede, trabajador, tipo, fecha y estado.
- Ver el soporte adjunto, y responder: APROBADO (justificada), DENEGADO (no justificada) u OBSERVADO (pide más información), con comentario. Comentario obligatorio al denegar u observar.
- Cada cambio de estado notifica al trabajador en su bandeja.
- Exportar a Excel el registro de inasistencias por periodo (útil para el cierre de planilla).

### 4.8 Solicitudes (vacaciones, permisos y adelantos) — revisión
- Bandeja única con todas las solicitudes, filtrable por tipo, sede, trabajador, fecha y estado.
- Estados: PENDIENTE → APROBADO / DENEGADO / OBSERVADO, con comentarios del administrador. Comentario obligatorio al denegar u observar. Hilo de comentarios entre trabajador y administrador dentro de cada solicitud.
- Vacaciones: ver fechas solicitadas, días, y el saldo de días del trabajador. El administrador puede aprobar fechas distintas a las pedidas (contrapropuesta). Calendario por sede para ver quién está de vacaciones o con permiso y evitar cruces.
- Permisos: ver fecha, horario u horas, motivo, si es con o sin goce de haber y sustento.
- Adelantos:
  - El administrador evalúa y define: monto aprobado (puede ser menor al solicitado), número de cuotas, monto por cuota y meses de descuento.
  - Al aprobar, el sistema genera automáticamente el FORMATO DE AUTORIZACIÓN DE DESCUENTO en PDF (ver 5.7) y lo envía al trabajador para su aceptación.
  - El adelanto solo pasa a "LISTO PARA PAGO" cuando el trabajador firma/acepta el formato. Luego se registra el pago (4.6) y queda en estado PAGADO.
  - Control de descuentos: cronograma de cuotas por trabajador (pendiente / descontado) y saldo pendiente; exportar a Excel los descuentos del mes para planilla.
  - Configurable: monto máximo de adelanto (por ejemplo, % de la remuneración) y no permitir un nuevo adelanto si hay saldo pendiente.

### 4.9 Amonestaciones y descargos
- El administrador registra una amonestación: trabajador, fecha, tipo (verbal registrada, escrita, suspensión, carta de preaviso u otro), motivo/hecho, archivo PDF de la amonestación (obligatorio) y plazo para el descargo (configurable; por defecto 6 días naturales).
- El trabajador recibe notificación en su bandeja y por correo. El sistema registra fecha y hora en que la abre (constancia de recepción).
- Seguimiento: SIN LEER → LEÍDA → DESCARGO PRESENTADO → CERRADA (con decisión final y comentario del administrador), o "PLAZO VENCIDO SIN DESCARGO".
- Cuando el trabajador presenta su descargo, el sistema envía automáticamente un CORREO ELECTRÓNICO al administrador de RR. HH. (dirección configurable, se pueden poner varias) con:
  - Asunto: "Descargo – [Apellidos y nombres] – DNI [número] – Amonestación del [fecha]".
  - Cuerpo con los datos del trabajador, sede, cargo, fecha de la amonestación, fecha y hora del descargo y el texto del descargo.
  - Adjuntos: el PDF de la amonestación, un PDF del descargo generado por el sistema y los archivos de sustento que subió el trabajador.
  - Copia al correo del trabajador como constancia.
- Historial de amonestaciones por trabajador visible en su ficha. Exportar a Excel.

### 4.10 Descargas masivas (SUPERADMIN y ADMIN)
- Descargar en .zip todos los PDF de recibos por honorarios, filtrando por periodo, empresa, sede y estado. Archivos renombrados: RUC_EMPRESA/PERIODO/DOCUMENTO_APELLIDOS_SERIE-NUMERO.pdf.
- Descargar la relación en Excel de los RH: documento, apellidos y nombres, RUC del trabajador, serie y número, fecha de emisión, periodo, monto bruto, retención, neto, empresa receptora (RUC y razón social), sede, cuenta/CCI, estado, fecha de carga, fecha de pago.
- Descarga masiva de boletas, constancias de pago, justificaciones y formatos de adelanto firmados, por periodo.

### 4.11 Comunicados
- Enviar comunicados a todos, por sede o por modalidad (llegan a la bandeja del trabajador), con confirmación de lectura.

### 4.12 Configuración
- Sedes, cargos, empresas pagadoras (RUC, razón social, representante), tipos de documento, tasa y tope de retención, fecha límite de RH.
- Motivos de cese, tipos de inasistencia, tipos de permiso, tipos de amonestación, plazo de descargo, plazo de acceso de solo lectura tras el cese.
- Reglas de vacaciones (días por año) y de adelantos (monto máximo, cuotas máximas).
- Texto editable del formato de autorización de descuento.
- Correos de RR. HH. que reciben descargos y alertas; configuración del correo de envío (SMTP).
- Permisos de administradores y textos de notificaciones.

## 5. Módulos del TRABAJADOR

### 5.1 Inicio / Bandeja de entrada
- Resumen personal y bandeja de notificaciones (nueva boleta, solicitud de RH con monto, recibo observado, pago realizado, respuesta a solicitudes o justificaciones, formato de adelanto por firmar, amonestación recibida, comunicados), con contador de no leídas.
- Accesos rápidos: "Nueva solicitud" y "Justificar inasistencia".
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
- Si es RH: número de RUC y constancia de suspensión de retenciones de 4.ª categoría (archivo y año).
- Talla de uniforme (opcional).
- Ve (sin poder editar) su fecha de ingreso, antigüedad, sede, cargo y modalidad.
- El administrador puede marcar datos como "verificados". Documento, modalidad, sede y fecha de ingreso solo los cambia el administrador.

### 5.3 Si es PLANILLA → "Mis boletas de pago"
- Lista por periodo y tipo, con fecha de emisión. Ver en pantalla y descargar el PDF.
- Botón "Confirmo la recepción" (queda como constancia de entrega).

### 5.4 Si es RECIBO POR HONORARIOS → "Mis recibos por honorarios"
- Ve muy destacada la solicitud del mes: monto a facturar, empresa (razón social y RUC) y periodo.
- Formulario de carga: fecha de emisión, periodo que cobra, serie y número (ej. E001-25), monto, su RUC y nombre, empresa a la que facturó (desplegable), ¿retención? / ¿suspensión de 4.ª categoría?, PDF del recibo (obligatorio), observaciones.
- Validaciones: RUC de 11 dígitos, monto igual al solicitado (si difiere, pedir justificación), número de recibo no repetido, solo PDF de máximo 5 MB.
- Historial mes a mes con estado (pendiente, en revisión, aprobado, observado con motivo, pagado).

### 5.5 Mis pagos
- PLANILLA: constancia de abono por periodo.
- RH: vista grande y clara por periodo con el recibo emitido, la constancia de SUNAT, el Abono 1 y el Abono 2 (montos, fechas y archivos para ver/descargar), total pagado y saldo.
- Adelantos: monto recibido, cronograma de cuotas, cuotas descontadas y saldo pendiente.

### 5.6 Justificar inasistencia
- Formulario: fecha o rango de fechas de la inasistencia (o tardanza / salida anticipada), tipo (descanso médico, cita médica, emergencia familiar, fallecimiento de familiar, trámite, otro), descripción y archivo de soporte (PDF o foto: certificado médico, constancia, etc.). Puede subir varios archivos.
- Historial con estado (PENDIENTE, APROBADO, DENEGADO, OBSERVADO) y comentarios del administrador. Si está OBSERVADO, puede completar o reenviar.

### 5.7 Mis solicitudes (vacaciones, permisos y adelantos)
- Nueva solicitud según tipo:
  - VACACIONES: fecha de inicio y fin, días calculados automáticamente, saldo disponible visible, comentario.
  - PERMISO: fecha, horario u horas, motivo, con o sin goce de haber, sustento opcional.
  - ADELANTO: monto solicitado, motivo, número de cuotas propuestas.
- Historial con estado (PENDIENTE, APROBADO, DENEGADO, OBSERVADO), comentarios del administrador y respuesta del trabajador en el mismo hilo. Puede cancelar una solicitud mientras esté PENDIENTE.
- Si un ADELANTO es aprobado, aparece el FORMATO DE AUTORIZACIÓN DE DESCUENTO, generado en PDF con logo y datos de la empresa, con este contenido (texto editable desde Configuración):
  "Yo, [APELLIDOS Y NOMBRES], identificado(a) con [TIPO DE DOCUMENTO] N.° [NÚMERO], trabajador(a) de [RAZÓN SOCIAL], RUC [RUC], en el cargo de [CARGO], sede [SEDE], declaro haber recibido / solicitar un adelanto por la suma de S/ [MONTO APROBADO] ([MONTO EN LETRAS]) y AUTORIZO expresamente a la empresa a efectuar los descuentos correspondientes de mis remuneraciones y/o beneficios sociales, de acuerdo con lo evaluado y aprobado por la empresa, en [N.° DE CUOTAS] cuota(s) de S/ [MONTO POR CUOTA], a partir de [MES/AÑO]. En caso de cese, autorizo que el saldo pendiente se descuente de mi liquidación de beneficios sociales."
  Incluye: tabla del cronograma de cuotas, fecha, firma del trabajador.
  - Aceptación: el trabajador marca "He leído y autorizo los descuentos", dibuja su firma en pantalla (en el celular con el dedo) y confirma. El sistema guarda el PDF firmado con fecha, hora e IP, y lo deja disponible para descarga del trabajador y del administrador.
  - Si el trabajador no acepta, el adelanto no avanza a pago.

### 5.8 Mis amonestaciones
- Ve las amonestaciones recibidas (PDF para ver/descargar), con fecha, motivo y plazo para el descargo con cuenta regresiva.
- Botón "Presentar descargo": texto del descargo (obligatorio) y archivos de sustento opcionales (PDF, fotos). Antes de enviar, confirma que el descargo no podrá editarse después.
- Al enviar: se genera el PDF del descargo, se envía el correo automático a RR. HH. con la amonestación y el descargo adjuntos (ver 4.9) y el trabajador recibe copia en su correo y una constancia en su bandeja.
- Ve el estado y la decisión final del administrador.

### 5.9 Mi cuenta
- Cambiar contraseña, ver el consentimiento de datos aceptado, cerrar sesión.

## 6. Requisitos técnicos
- Base de datos relacional normalizada (usuarios, roles/permisos, trabajadores, periodos laborales (ingreso/cese), sedes, empresas, documentos de identidad, formación, boletas, solicitudes RH, recibos RH, pagos y archivos, justificaciones, solicitudes (vacaciones/permisos/adelantos), cuotas de adelanto, formatos firmados, amonestaciones, descargos, notificaciones, comunicados, correos enviados, auditoría, configuración).
- Envío de correos con cola de reintentos y registro de cada correo enviado (destinatarios, fecha, estado). Si un envío falla, alertar al administrador en el panel.
- Generación de PDF en el servidor (formato de adelanto, descargo, ficha del trabajador).
- Lectura de Excel robusta: tolera mayúsculas, espacios y tildes en encabezados; conserva ceros a la izquierda en documentos (no convertir a número); acepta fechas en formato dd/mm/aaaa.
- Montos en soles con 2 decimales. Zona horaria America/Lima. Fechas dd/mm/aaaa.
- Tareas programadas: recordatorios de RH, aviso de plazos de descargo por vencer, bloqueo automático del acceso de cesados al vencer el plazo.
- Límites de tamaño y validación del tipo real de archivo (no solo la extensión).
- Paginación y buscadores en todas las tablas de administración.
- Seed inicial: crear SUPERADMIN y ADMIN (Vanessa) con credenciales en variables de entorno, sedes, empresas y catálogos.
- Pruebas automáticas de autorización: un trabajador no puede ver datos ni archivos de otro, ni entrar a rutas de administrador; un cesado fuera de plazo no puede ingresar.
- Documentar cómo respaldar la base de datos y la carpeta de archivos.

## 7. Entregables
1. Código completo del proyecto.
2. Plantillas Excel descargables: trabajadores (con fecha de ingreso y cese), montos RH, relación de archivos de boletas, relación de archivos de pagos.
3. README de despliegue paso a paso para nuestro hosting (requisitos, variables de entorno, base de datos, configuración de correo, tareas programadas, subida de archivos, creación del primer administrador, copias de seguridad).
4. Manual corto de uso: uno para administradores y uno para trabajadores.

## 8. Mejoras adicionales (integrar en la etapa indicada entre corchetes)

### 8.1 Seguridad y firma
- [E1] Primer ingreso más seguro: el número de documento es fácil de conocer, así que en el primer ingreso pide además un dato de verificación que carga el administrador (por ejemplo, la fecha de nacimiento). Las claves iniciales que no se usen expiran a los N días (configurable).
- [E1] Al restablecer una contraseña, genera una clave temporal aleatoria válida por 24 horas en lugar de volver al número de documento. Opcional: autoservicio con código enviado al correo verificado del trabajador.
- [E1] Doble factor (código por correo o app autenticadora) obligatorio para SUPERADMIN y ADMIN.
- [E1] Al cesar a alguien o cambiarle el rol, cierra todas sus sesiones activas.
- [E1] Versiona el texto del consentimiento de datos: si cambia, se pide aceptarlo de nuevo. Registra las solicitudes de derechos ARCO (acceso, rectificación, cancelación y oposición) y su respuesta.
- [E2] Cifra en la base de datos los campos sensibles (cuenta bancaria, CCI, datos de salud) y elimina los metadatos (EXIF y ubicación GPS) de las fotos subidas.
- [E6] Respaldo de la firma del formato de adelanto: antes de firmar se envía un código de un solo uso al correo del trabajador; al firmar se guarda la huella (hash SHA-256) del PDF y se agrega una hoja final de evidencias (fecha, hora, IP, dispositivo y código usado). El PDF firmado no se puede modificar.
- [E8] Opción "Ver como trabajador" para soporte: solo lectura y registrada en la auditoría.

### 8.2 Reglas laborales (validar con el asesor laboral o legal)
- [E1] Módulos por modalidad configurables. Por defecto, vacaciones, permisos y amonestaciones solo para PLANILLA: aplicarlos al personal por RH (locación de servicios) puede interpretarse como una relación laboral encubierta.
- [E6] Adelantos para RH: tratarlos como "adelanto de honorarios", con su propio texto de formato (se descuenta de recibos futuros, no de remuneraciones ni beneficios sociales).
- [E3] Boletas electrónicas: para que la entrega por el portal reemplace a la boleta física, la normativa pide firma digital del empleador y constancia de recepción del trabajador (D.S. 009-2011-TR). Indica si el PDF cargado viene firmado digitalmente y alerta si las boletas del periodo no se cargaron a tiempo (plazo configurable; la norma fija el tercer día hábil siguiente al pago).
- [E1] Al cesar: genera el certificado de trabajo en PDF (planilla) o una constancia de prestación de servicios (RH), cancela las solicitudes pendientes y avisa si hay saldo de adelanto para descontarlo en la liquidación.
- [E6] Descansos médicos: contador de días acumulados en el año por trabajador, con alerta al llegar a 20 (desde el día 21 corresponde subsidio de EsSalud y se necesita CITT).
- [E6] Vacaciones: saldo por período vacacional (por año de servicio desde la fecha de ingreso) e importación desde Excel del saldo inicial de cada trabajador al arrancar.

### 8.3 Gestión de personal
- [E1] Contratos: tipo, fecha de inicio y fin, y archivo. Alerta 30 días antes del vencimiento para decidir la renovación.
- [E1] Movimientos de personal con fecha efectiva (cambio de sede, cargo, modalidad o empresa pagadora), visibles en el historial.
- [E1] Checklist de ingreso y de cese: alta y baja en T-Registro, contrato firmado, entrega y devolución de uniforme o equipos.
- [E2] Vencimientos además del documento de identidad: habilitación de la colegiatura, contrato y constancia de suspensión de 4.ª categoría (anual, con aviso en enero).
- [E2] Verificar el correo y el celular del trabajador con un código, para asegurar que los avisos lleguen.
- [E3] Documentos de la empresa con acuse de recibo: Reglamento Interno de Trabajo, RISST y políticas. Cada trabajador confirma la recepción y queda la constancia.
- [E3] Otros documentos personales que carga el administrador: certificados anuales de rentas y retenciones (4.ª y 5.ª categoría), contrato y liquidación.
- [E6] Constancia de trabajo vigente: el trabajador la descarga en PDF con logo, datos de la empresa y firma configurada.
- [E6] Rol opcional "Jefe de sede": da el visto bueno a vacaciones y permisos de su sede antes de RR. HH. y ve el calendario de su sede.

### 8.4 Recibos por honorarios
- [E4] Lee automáticamente del PDF del recibo electrónico (generado por SUNAT) la serie, el número, los RUC del emisor y receptor, la fecha y el monto, para prellenar el formulario y compararlo con lo ingresado.
- [E4] Valida el RUC con su dígito verificador (módulo 11) y que el del emisor empiece con 10; valida el DNI de 8 dígitos.
- [E4] Opcional: consultar la validez del comprobante en SUNAT (servicio de validez de comprobantes; requiere credenciales de la empresa).

### 8.5 Archivos y cargas
- [E2] En el celular: tomar la foto directamente con la cámara, comprimirla antes de subir y unir varias fotos en un solo PDF (las fotos de certificados suelen pasar los 5 MB).
- [E3] Carga histórica de meses anteriores sin enviar notificaciones, y opción de anular un lote completo si se cargó con el periodo o el tipo equivocado.
- [E7] Si los adjuntos de un correo superan el límite del proveedor, envía un enlace seguro con vencimiento en lugar del adjunto.
- [E7] Amonestaciones: permite registrar la entrega física (cargo firmado escaneado) por si el trabajador no abre el portal.
- [E8] Descargas masivas y exportaciones grandes en segundo plano, con aviso cuando el archivo está listo (evita cortes por tiempo de espera en hosting compartido).

### 8.6 Avisos
- [E7] Incluye en el README la configuración de SPF, DKIM y DMARC del dominio para que los correos no lleguen a spam.
- [E8] Resumen diario por correo a los administradores con lo pendiente: solicitudes, justificaciones, descargos, recibos por revisar y correos fallidos.
- [E8] Aplicación instalable en el celular (PWA) con notificaciones push. A futuro, opcional: avisos por WhatsApp mediante WhatsApp Business API o un proveedor como respond.io.

### 8.7 Operación y puesta en marcha
- [E1] Usa la versión de Laravel (o del stack elegido) con soporte de seguridad vigente que admita el PHP del hosting.
- [E8] Indicadores de rotación (ingresos y ceses) y de ausentismo por sede y por mes.
- [E9] Respaldos automáticos diarios de la base de datos y los archivos, cifrados, con copia fuera del hosting, retención (por ejemplo, 30 diarios y 12 mensuales) y una prueba de restauración documentada. Guarda la clave de la aplicación (APP_KEY) por separado: sin ella no se pueden leer los datos cifrados.
- [E9] Entorno de pruebas separado del de producción, con datos ficticios.
- [E9] Monitoreo: registro de errores con aviso por correo, alerta de espacio en disco (los archivos crecen cada mes) y página de mantenimiento.
- [E9] Plan de arranque: carga inicial (trabajadores, saldos de vacaciones, adelantos en curso con sus cuotas pendientes y boletas históricas), piloto en una sede y capacitación breve.

## Etapas
Construye por etapas y muéstrame avances:
1) Login, roles y gestión de trabajadores (ingreso, cese, reingreso) con carga Excel
2) Ficha de personal del trabajador
3) Boletas (carga masiva con asignación automática)
4) Flujo RH completo (montos → notificación → carga → revisión)
5) Pagos (SUNAT + 2 abonos)
6) Justificaciones de inasistencia y solicitudes (vacaciones, permisos, adelantos con formato firmado y cuotas)
7) Amonestaciones y descargos con envío de correo a RR. HH.
8) Descargas masivas, dashboard, comunicados y auditoría
9) Pruebas, README y manuales
