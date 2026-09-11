# GymOps — Casos de Uso por Rol

Derivado directamente de `entity-relationships.md` (v5). Cada caso de uso nombra las
entidades que lee o escribe, para que este documento se mantenga trazable al modelo de
datos en lugar de convertirse en una lista de funcionalidades aspiracionales.

Una misma persona real (`PERSON`) puede tener varios roles a la vez — por ejemplo, un
socio adulto que paga su propia cuota y que además es entrenador, o un propietario que
también entrena. Los roles de abajo son *capacidades*, no puestos de trabajo: asigna
tantos como correspondan a una persona concreta.

**Regla universal, sin excepciones:** nadie, en ningún rol, puede forzar una doble
reserva. Los conflictos en `SPACE_BOOKING`, `EQUIPMENT_BOOKING` y `STAFF_BOOKING` son
rechazados por la propia base de datos. Las únicas salidas son cancelar la
`CLASS_SESSION` o reasignar/eliminar la reserva en conflicto — no existe una opción de
"forzar" para administradores.

**Marcado `[POR DECIDIR]`:** puntos que dependen del reparto de permisos entre
entrenador y administrador que el cliente aún no ha decidido. Se listan bajo el rol que
parece más probable según las entidades involucradas, no como una asignación definitiva
— no construir control de acceso sobre estos puntos hasta confirmarlo.

---

## 0. Prospecto / Visita sin cita (todavía sin cuenta)

*Alguien sin `ACCOUNT`, y a menudo sin ningún registro `PERSON` hasta que recepción crea
uno. No es un usuario del sistema en el sentido de tener acceso propio — se representa
únicamente a través de lo que el personal introduce en su nombre.*

- Dar sus datos de contacto y la actividad/horario deseados a recepción, convirtiéndose
  en un `WAITLIST_ENTRY` (`subject_person_id`/`contact_person_id` creados como registros
  `PERSON` mínimos si aún no existían en el sistema, `desired_course_id`, opcionalmente
  `desired_group_id`, y notas libres en `preferred_schedule_notes`)
- Recibir un enlace de `SIGNUP_INVITATION` del personal una vez que se ha encontrado un
  hueco, y completar el registro por sí mismo — esto es lo que realmente crea su
  `ACCOUNT` (si no tenía una), el vínculo `ACCOUNT_MEMBER` y la `SUBSCRIPTION` resultante
- Alternativamente, ser inscrito directamente por el personal sin pasar nunca por un
  enlace (`resolution_method = direct_assignment`) — por ejemplo, cuando el personal
  configura la facturación por teléfono

No puede: hacer nada más en el sistema — sin acceso propio, sin autoservicio, hasta que
(y a menos que) se convierta en Titular de Cuenta.

---

## 1. Titular de Cuenta / Pagador

*Una `PERSON` propietaria de una `ACCOUNT` — ya sea un socio adulto que paga su propia
cuota, o un padre/madre o tutor que paga por otras personas.*

- Registrarse y crear su propia `ACCOUNT`
- Vincular a otra `PERSON` a su `ACCOUNT` mediante `ACCOUNT_MEMBER` (p. ej., añadir a un
  hijo o hija)
- Añadir, actualizar o eliminar un `PAYMENT_METHOD` en su `ACCOUNT`
- Ver las `INVOICE` y `PAYMENT` facturadas a su `ACCOUNT`
- Crear una `SUBSCRIPTION` (inscripción) para cualquier `PERSON` vinculada a su
  `ACCOUNT`, en un `GROUP`
- Cancelar o solicitar un cambio de plan en una `SUBSCRIPTION` existente
- Firmar una `WAIVER` (exención de responsabilidad) — como `signed_by_person_id`, para
  sí mismo o para un menor vinculado (`subject_person_id`)
- Consultar los listados de `COURSE` / `GROUP` / `CLASS_SESSION` (solo lectura) para
  decidir en qué inscribirse
- **Solicitar** un intercambio o clase de recuperación puntual ("Juanito no puede venir
  el martes, ¿puede unirse a otro grupo de Judo esta semana?") — esto es una solicitud
  dirigida a Administración, no una escritura de autoservicio; el titular de la cuenta
  no puede modificar directamente la `SUBSCRIPTION`/`ATTENDANCE` del hueco de sesión de
  otra persona. *(Pregunta abierta, ver `entity-relationships.md`: si esto termina
  siendo un `WAITLIST_ENTRY`, compartiendo la misma maquinaria que la captación de
  nuevos prospectos, o si necesita un registro puntual independiente y más ligero — una
  acomodación temporal no tiene exactamente la misma forma que una nueva inscripción.)*
- Recibir notificaciones automáticas de facturación (generadas por el sistema; solo
  lectura)

No puede: tocar la `ACCOUNT`, `SUBSCRIPTION` o `PAYMENT_METHOD` de otra persona; crear o
editar `GROUP`/`COURSE`/`SUBSCRIPTION_PLAN`; crear o modificar ninguna `*_BOOKING`.

---

## 2. Socio/a (sujeto de una suscripción)

*Una `PERSON` inscrita mediante `SUBSCRIPTION`. Puede ser o no el titular de la cuenta —
un socio menor de edad normalmente no tiene acceso propio y se representa únicamente
como datos (sujeto de `ATTENDANCE`, sujeto de `WAIVER`).*

- Si es adulto y paga su propia cuota: las mismas capacidades que el Titular de Cuenta,
  para sí mismo
- Si es menor de edad: sin acciones directas en el sistema — el entrenador y
  administración actúan en su nombre
- **Opcional, si el producto llega a dar acceso limitado a menores mayores/adolescentes:**
  - Ver sus propias `SUBSCRIPTION` y el horario resultante (`GROUP` → `CLASS_SESSION`)
  - Ver su propio historial de `ATTENDANCE` y cualquier nota de progreso del entrenador

No puede: gestionar la facturación, firmar su propia `WAIVER` si es menor, ni modificar
ninguna reserva.

---

## 3. Entrenador/a

*Una `PERSON` con un registro `STAFF` donde `role = coach`. Su alcance está generalmente
limitado a los `GROUP`/`CLASS_SESSION` a los que está realmente asignado mediante
`STAFF_BOOKING`.*

- Ver su propio horario: las `CLASS_SESSION` en las que tiene un `STAFF_BOOKING`
- Ver el listado de socios (lista de `SUBSCRIPTION`) de un `GROUP` que entrena
  - Si ve el estado de facturación (`INVOICE`/`SUBSCRIPTION.status` con importes) o solo
    un indicador de elegibilidad (activo/congelado, sin detalle económico) se controla
    por grupo mediante `GROUP_VISIBILITY_SETTING.coach_sees_billing_status`, configurado
    por Administración
- Tomar y editar la `ATTENDANCE` de una `CLASS_SESSION` en la que está asignado
- Crear una `EQUIPMENT_BOOKING` puntual para su propia `CLASS_SESSION`
  ("mañana necesitamos bandas elásticas y kettlebells") — sujeta a la misma regla de
  conflicto de `EQUIPMENT.total_quantity` que cualquier otra reserva; sin excepciones
- Registrar notas de progreso/graduación sobre una `PERSON` en su listado, si el
  producto lo permite
- **`[POR DECIDIR]`** Cancelar su propia `CLASS_SESSION` (p. ej., por enfermedad
  repentina)
- **`[POR DECIDIR]`** Solicitar/asignar un entrenador sustituto (modificar su propio
  `STAFF_BOOKING` o crear uno para un compañero)
- **`[POR DECIDIR]`** Cambiar la `SPACE_BOOKING` de su propia sesión (cambio de sala
  puntual)
- **Triaje de lista de espera (confirmado como compartido con Administración, según el
  cliente):** revisar los `WAITLIST_ENTRY` abiertos de los cursos que entrena, y ayudar
  a decidir si un `GROUP`/`CLASS_SESSION` nuevo, o uno existente con hueco, puede
  absorberlos — comprobando en el proceso la disponibilidad de
  `SPACE_BOOKING`/`STAFF_BOOKING`/`EQUIPMENT_BOOKING`. Un entrenador puede proponer una
  solución; si puede *finalizarla* (crear la `SUBSCRIPTION` o enviar la
  `SIGNUP_INVITATION` él mismo) o si debe pasársela a Administración para ejecutarla
  forma parte del mismo límite entrenador/administrador aún pendiente que los puntos
  `[POR DECIDIR]` de arriba

No puede: crear o editar `COURSE`, `GROUP`, `SUBSCRIPTION_PLAN` ni precios; ver listados
o facturación de grupos que no entrena; tocar directamente `ACCOUNT`, `PAYMENT_METHOD`
o `INVOICE`; forzar un conflicto de reserva (nadie puede).

---

## 4. Administración (recepción / personal administrativo)

*Una `PERSON` con un registro `STAFF` donde `role = admin`.*

- CRUD completo sobre `PERSON`, `ACCOUNT`, `ACCOUNT_MEMBER` (crear registros, fusionar
  duplicados, reasignar a una persona a otra cuenta)
- Crear y gestionar `COURSE` y su `COURSE_REQUIREMENT` (necesidades de material por
  defecto)
- Crear y gestionar `GROUP`: cadencia, entrenador por defecto, aforo, franja de edad
- Configurar `GROUP_VISIBILITY_SETTING` por grupo (qué pueden ver los entrenadores
  asignados a ese grupo)
- Crear y gestionar `CLASS_SESSION`: programar, cancelar, ajustar `capacity_override`
- Crear y gestionar los catálogos de `SPACE` y `EQUIPMENT`, incluyendo
  `EQUIPMENT.total_quantity`
- Crear, reasignar o cancelar `SPACE_BOOKING`, `EQUIPMENT_BOOKING` y `STAFF_BOOKING` —
  así es como se resuelve realmente un conflicto de reserva (cancelando una de las
  partes, o reasignándola)
- Crear, gestionar y fijar el precio de los `SUBSCRIPTION_PLAN`
- Crear, cancelar o congelar una `SUBSCRIPTION` en nombre de **cualquier** persona —
  esta es la vía real de ejecución de una solicitud de intercambio/recuperación de un
  titular de cuenta ("mover a Juanito al grupo del jueves esta semana")
- Responder preguntas de disponibilidad consultando los `GROUP` del mismo `course_id`
  y el aforo de sus `CLASS_SESSION` frente a las reservas actuales — la respuesta
  directa a "¿puede unirse a otro grupo esta semana?"
- Disparar/supervisar las tandas de facturación; ver y reintentar `PAYMENT` fallidos;
  procesar un `PAYMENT` manual; emitir un reembolso (hasta el límite que fije el
  Propietario)
- Gestionar los registros de `WAIVER`: subir/verificar `document_ref`, reclamar firmas
  pendientes
- Generar informes que cruzan varias entidades: ingresos, ocupación, morosidad, bajas
- Crear un `WAITLIST_ENTRY` a partir de una visita sin cita o una consulta telefónica —
  creando registros `PERSON` mínimos para el sujeto/contacto si aún no están en el
  sistema
- **Triaje de lista de espera (el "sandbox"):** revisar todos los `WAITLIST_ENTRY`
  abiertos de un `COURSE` y negociar el encaje — esto es por naturaleza un ejercicio de
  planificación, no una acción CRUD aislada: contrastar los horarios deseados con la
  disponibilidad de `SPACE`/`STAFF`/`EQUIPMENT`, y unos con otros, ya que varias
  personas en lista de espera podrían justificar en conjunto abrir un `GROUP`/
  `CLASS_SESSION` nuevo
- Resolver un `WAITLIST_ENTRY` de dos maneras:
  - **Asignación directa** — crear la `SUBSCRIPTION` uno mismo, fijando
    `resolution_method = direct_assignment`, `resolved_by_staff_id`, `resolved_group_id`
  - **Enviando un enlace de inscripción** — crear una `SIGNUP_INVITATION` sobre un
    `GROUP` propuesto, dejando que el contacto complete su propio registro y
    `SUBSCRIPTION`
- Hacer seguimiento y reclamar las `SIGNUP_INVITATION` a punto de caducar o sin usar

No puede: forzar un conflicto de reserva (nadie puede); por defecto, crear/editar otras
cuentas de `STAFF` ni aprobar reembolsos por encima del umbral — eso corresponde al
Propietario, según el reparto de roles de abajo, salvo que el cliente indique lo
contrario.

---

## 5. Propietario/a

*Una `PERSON` con un registro `STAFF` donde `role = owner`. Superconjunto de
Administración.*

- Todo lo que puede hacer Administración, más:
- Crear, editar y desactivar cualquier cuenta de `STAFF`, incluyendo otros
  Administradores y Propietarios
- Aprobar reembolsos/excepciones por encima del umbral que Administración puede
  autoaprobar
- Ver informes financieros y de cumplimiento a nivel de todo el negocio
- Configurar ajustes globales del gimnasio (p. ej., multi-sede, si el negocio llega a
  expandirse)

**Nota sobre una carencia del modelo:** el ERD actual solo tiene un enum genérico
`STAFF.role` (`coach`/`admin`/`owner`) — no existe una entidad `PERMISSION` que dé
granularidad por acción. Todo lo anterior se aplica en la lógica de la aplicación
contra ese único campo, no en el propio modelo de datos. Esto es suficiente para un
gimnasio único en la v1, pero no escalará a algo como "este administrador puede emitir
reembolsos, aquel otro no" sin añadir una tabla de permisos real.

---

## 6. Sistema (automatizado)

*Sin ninguna `PERSON` detrás — tareas en segundo plano que actúan bajo una identidad de
sistema.*

- Generar `INVOICE` según el `SUBSCRIPTION_PLAN.billing_cycle` de cada una, a partir de
  las `SUBSCRIPTION` activas vinculadas a una `ACCOUNT`
- Intentar el `PAYMENT` mediante el `PAYMENT_METHOD` de la `ACCOUNT` (domiciliación SEPA,
  tarjeta, Bizum)
- Reintentar un `PAYMENT` fallido según la política de gestión de impagos; marcar la
  `SUBSCRIPTION`/`ACCOUNT` como morosa si se agotan los reintentos
  *(nota: `SUBSCRIPTION.status` actualmente solo tiene `active/frozen/cancelled` —
  probablemente haga falta un estado `delinquent` o `payment_failed` que aún no está en
  el modelo)*
- Precargar la `EQUIPMENT_BOOKING`/`SPACE_BOOKING` por defecto de una `CLASS_SESSION`
  nueva a partir de `COURSE_REQUIREMENT` y los valores por defecto del `GROUP`
- Aplicar las restricciones de exclusión de reservas en el momento de escribir (rechaza
  solapamientos en `SPACE_BOOKING`/`STAFF_BOOKING`, o una `EQUIPMENT_BOOKING` que supere
  `total_quantity`) — este es el mecanismo detrás de "nadie puede forzar una doble
  reserva", no un rol
- Enviar notificaciones automáticas a `ACCOUNT.billing_email` (nueva factura, pago
  fallido, recordatorio de renovación)
- Entregar un enlace de `SIGNUP_INVITATION` al email/teléfono del `contact_person_id` en
  cuanto el personal lo crea, y marcarlo como `expired` si pasa `expires_at` sin usarse
  — qué ocurre entonces con el `WAITLIST_ENTRY` padre sigue siendo una pregunta abierta
  (ver más abajo)

---

## Cuestiones abiertas heredadas del modelo de entidades

Estos casos de uso heredan las mismas preguntas sin resolver señaladas en
`entity-relationships.md`:

- Las acciones del entrenador marcadas `[POR DECIDIR]` arriba (cancelar su propia
  sesión, reasignar entrenador/sala) dependen de la respuesta que el cliente todavía no
  ha dado sobre el límite de decisión entre entrenador y administrador.
- Si una asistencia/recuperación puntual requiere un cambio completo de `SUBSCRIPTION`
  o puede sostenerse solo sobre `ATTENDANCE` afecta a si la "solicitud de intercambio"
  de un Titular de Cuenta termina en una nueva `SUBSCRIPTION` o en un registro puntual
  más ligero.
- Todavía no existe una entidad `PERMISSION`, así que los límites entre Administración y
  Propietario de arriba son una propuesta por defecto, no algo aplicable a nivel de
  modelo de datos hoy.
- Si un entrenador puede finalizar la resolución de una lista de espera (crear la
  `SUBSCRIPTION` / enviar la `SIGNUP_INVITATION`) o solo puede proponerla para que
  Administración la ejecute es el mismo límite entrenador/administrador aún abierto,
  ahora extendido al triaje de listas de espera.
- Qué ocurre con un `WAITLIST_ENTRY` cuando su `SIGNUP_INVITATION` caduca sin usarse —
  ¿vuelve automáticamente a `open` para un nuevo triaje, o requiere que una persona lo
  note y actúe?
- Si el escenario de "intercambio puntual" de un socio ya existente pertenece al
  `WAITLIST_ENTRY`, o necesita su propia entidad más ligera, según la nota bajo Titular
  de Cuenta más arriba.
