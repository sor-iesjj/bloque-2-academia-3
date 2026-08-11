## Fase 6 · Los equipos de las aulas

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Proyecto Final del Bloque 2 · Boochan Academy · Kea DHCP**
> 🧭 Índice del proyecto: [[02_Indice_Proyecto]] · 🏫 El cliente: [[00_El_Cliente_Boochan_Academy]]
>
> **📦 Entrega:** tres vídeos · instantáneas `AC3 · Fase 6 terminada` de las **tres** máquinas · copias `.ova`

---

> [!important] ✍️ Este apartado son TRES vídeos y lo que produzcas va a tu repositorio
> | | |
> | :--- | :--- |
> | 📹 **3 vídeos** | `B2 · AC3 · F6 · Implementación` / `· Verificación` / `· Averías`, en la playlist `B2_Academia_3` |
> | 📝 **1 entrada de apuntes** | `b2-ac3.6-los-equipos-de-las-aulas.md` en `00_Apuntes/Trimestre_N/B2_Ubuntu_Local/`, con la estructura del **Bloque 0 · Fase 0.1.b** |
> | 💿 **Copias `.ova`** | A tu **disco externo**. Nunca a GitHub |
>
> **Identifícate al empezar cada vídeo** y pon **timestamps** en la descripción (`00:00 Presentación` y uno por paso). Los dos son corte duro en la rúbrica. Si algo de esto no lo tienes montado: **[[01_ANTES_DE_EMPEZAR]]**.

## **1 · EL ENCARGO**

> [!quote] 🗣️ Lo que te dice el cliente
> *"Esto es lo que quiero ver funcionando. Coge un ordenador del aula 1 y otro del aula 4, que están en plantas distintas, y enséñame que un chaval entra en los dos con su usuario y encuentra lo suyo. Y de paso me enseñas lo otro: que desde el ordenador del alumno no se vean los exámenes, y que desde el del profesor sí. Si eso funciona, para mí el trabajo está hecho."*

> [!info] 🎯 Lo que ha pedido de verdad, traducido
> El cliente acaba de definirte **la prueba de aceptación** del proyecto. Es lo que va a mirar antes de pagarte.
> - **Un equipo de aula** y **un equipo de profesor**, unidos al dominio.
> - **La misma persona entrando en los dos**, con sus cosas.
> - **La matriz de la Fase 5, demostrada desde el sitio donde se nota**: el listado de red de un Windows.

> [!info] 🔄 En esta variante, los equipos cogen su IP del servidor Kea DHCP
> A diferencia de la Academia con IP fija, aquí los clientes **no llevan IP estática**: se configuran con DHCP automático y la reciben del servidor Kea que montaste en la Fase 1. Las reservas que declaraste entonces —`.20` para `aula1-pc01` y `.21` para `aula4-prof`— son las que garantizan que cada equipo tenga **siempre la misma dirección**, sin configurarla a mano en Windows.
>
> El DDNS que activaste en Kea se encarga de que, además, los nombres de los equipos se registren automáticamente en el DNS del dominio. Cuando un cliente obtenga su IP, el servidor sabrá su nombre sin que tú toques nada.

> [!danger] 🛑 Qué significa "encontrar lo suyo", y qué NO
> Se cumple con **dos** cosas y solo con esas dos:
> 1. Que **sus credenciales valgan en cualquier equipo**.
> 2. Que **su carpeta personal viva en el servidor** y le llegue por red desde donde se siente.
>
> **Los perfiles móviles quedan fuera de este proyecto.** El escritorio, el fondo de pantalla y los iconos **no** tienen que viajar con el alumno: eso es otra tecnología, con otros problemas.
>
> Si te pones a montarlos, estás haciendo trabajo que nadie te ha pedido, que **no se corrige** y que te va a dar quebraderos de cabeza gratis. **El cliente quiere sus ficheros, no su fondo de pantalla.**

---

## **2 · IMPLEMENTACIÓN**

> [!example] 🎬 Antes de empezar
> 1. Léete el apartado entero.
> 2. Recupera **las cuatro pruebas que dejaste pendientes en la Fase 5**. Hoy se tachan.
> 3. Comprueba que el servidor está arrancado, que el dominio responde y que **Kea DHCP está activo** (`systemctl is-active kea-dhcp4-server`) **antes** de encender ningún cliente.
> 4. **Arranca la grabación e identifícate.**

**Lo que tiene que existir al final de este apartado:**

- [ ] **Dos máquinas virtuales con Windows 11 instalado por ti**, desde la ISO:
  - `aula1-pc01` → `172.20.10.20`, hará de **equipo de alumno** (planta baja)
  - `aula4-prof` → `172.20.10.21`, hará de **equipo de profesor** (primera planta)
- [ ] Las dos **obtienen su IP del servidor Kea DHCP**, sin configuración estática en Windows.
- [ ] Las dos **unidas al dominio** `ACADEMIA.LOCAL`.
- [ ] Se puede iniciar sesión en las dos **con cuentas del dominio**.
- [ ] Un alumno, al iniciar sesión, **llega a su carpeta personal** — y es la misma en los dos equipos.
- [ ] El profesorado llega a lo suyo: `material`, `examenes` y las carpetas del alumnado.
- [ ] **La matriz de la Fase 5 se cumple y se demuestra**, cliente a cliente.

> [!info] 🔄 Cómo se configura la IP en esta variante
> En Windows, la configuración de red es **DHCP automático**. Nada de IP fija. La dirección la asigna Kea desde el servidor, y la reserva que declaraste en la Fase 1 (con la MAC del equipo) es la que garantiza que siempre sea la misma.
>
> Esto significa que **antes de arrancar cada cliente por primera vez**, tienes que:
> 1. **Anotar la MAC** de la tarjeta de red virtual de esa VM.
> 2. **Verificar en el servidor** que la reserva en `/etc/kea/kea-dhcp4.conf` coincide con esa MAC.
> 3. Si no coincide, editar el JSON, validarlo con `kea-dhcp4 -t` y reiniciar el servicio.
>
> **La MAC la ves en VirtualBox**: `Configuración > Red > Avanzadas > Dirección MAC`. O desde el propio Windows con `ipconfig /all` una vez arrancado.

> [!info] 💿 Sí, las instalas tú. Y no te voy a explicar cómo
> **Instalar Windows 11 en una máquina virtual lo hiciste el curso pasado.** No es contenido de este módulo y aquí no se repite: tienes la ISO en un pendrive desde el Bloque 0 y sabes crear una VM.
>
> Solo cuatro cosas que **sí** son de este apartado, porque si las fallas te bloquean después:
>
> | | |
> | :--- | :--- |
> | **El nombre del equipo** | Ponlo **durante la instalación** o justo después. Cambiarlo cuando ya está en el dominio obliga a sacarlo y volverlo a meter |
> | **Cuenta local, no de Microsoft** | Una cuenta Microsoft en un equipo que va a un dominio es un lío innecesario. **Cuenta local, sin conectar a internet** durante la instalación |
> | **La tarjeta de red** | Del mismo tipo que le pusiste al servidor en la Fase 1. Si no se ven, no hay dominio que valga |
> | **Instantánea antes de unir** | Con Windows recién instalado, actualizado y **antes de tocar el dominio**. Es tu punto de retorno |
>
> **La segunda máquina no la instales dos veces.** Piensa: tienes una VM de Windows limpia y con instantánea, y necesitas otra igual con distinto nombre y distinta IP. Ya sabes hacer esto — y ya sabes cuál es la trampa de hacerlo mal *(la Fase 6 te la va a cobrar en el dominio)*.

> [!warning] ⚠️ Tres cosas que tiene que tener un cliente para unirse a un dominio, y solo una es obvia
> Si la unión falla, el 95 % de las veces es una de estas:
> - **A quién le pregunta el nombre.** Un cliente que resuelve nombres por internet **no encuentra el dominio**, aunque el servidor esté a un metro.
> - **La hora.** La autenticación del dominio **rechaza cualquier cosa con más de cinco minutos de desfase**. Y una máquina virtual que ha estado pausada tiene el reloj parado en el día que la pausaste.
> - **La red.** Que las dos máquinas estén en el mismo sitio y se vean.
>
> Los tres los sufriste en la Fase 8 del Bloque 2. **Compruébalos antes de intentar la unión**, no después de tres intentos fallidos.

> [!danger] 🛑 Si la Fase 2 quedó mal, hoy te enteras
> Aquí es donde aparece *"no se encuentra el dominio"* si el servicio se anunció por la tarjeta equivocada. **No es un problema de Windows.** Si te pasa, el diagnóstico empieza en el servidor.
>
> Y esa es la lección: **el apartado donde se manifiesta un fallo casi nunca es el apartado que lo causó.**

> [!question] 🤔 Para decir en voz alta en el vídeo
> 1. Al unir el equipo al dominio te pide credenciales. **¿De quién, y por qué de esa cuenta y no de un profesor?**
> 2. Después de unir, Windows obliga a reiniciar. **¿Por qué? ¿Qué no se aplica hasta ese reinicio?**
> 3. La carpeta personal del alumno se conecta como unidad de red. **¿Qué pasa si no la marcas como persistente?** ¿Y qué pasaría con 120 alumnos si tuvieras que conectarla a mano en cada equipo?
> 4. La pregunta buena: **¿por qué usas el nombre del servidor y no su dirección** cuando conectas la carpeta? *(Pista: tiene que ver con cómo se demuestra tu identidad, y con que una de las dos formas es más antigua y menos segura.)*
> 5. 🔴 **¿Qué ventaja tiene que los clientes reciban su IP por DHCP con Kea en vez de tenerla fija?** ¿Y qué inconveniente añade?
> 6. 🔴 Si el servidor Kea DHCP se para, **¿los equipos que ya tienen su IP asignada dejan de funcionar?** ¿Y los que acaban de arrancar?
> 7. 🔴 La pregunta de cierre: **¿quién ha registrado el nombre `aula1-pc01.academia.local` en el DNS del dominio?** ¿Lo has hecho tú o lo ha hecho otro?

---

## **3 · VERIFICACIÓN**

> [!danger] 🛑 Hoy la verificación ES el encargo
> En los apartados anteriores comprobabas tu trabajo. **Hoy demuestras lo que el cliente compró.** Este vídeo es el que le enseñarías a él.

**Las nueve pruebas del punto 8 de [[00_El_Cliente_Boochan_Academy]], una por una.** No te las voy a repetir aquí: están escritas, tienes que ir a buscarlas, y **eres tú quien decide con qué cuentas y desde qué equipo demuestras cada una.**

| Lo que se corrige de esta parte | Qué significa |
| :--- | :--- |
| **Que estén las nueve** | Ninguna se da por hecha porque "es evidente" |
| **Que uses el equipo adecuado** | Hay pruebas que solo tienen sentido desde el equipo de alumno, y otras desde el de profesor |
| **Que se vea el resultado, no solo el intento** | Un mensaje de error es un resultado. Léelo en voz alta |
| **Que interpretes** | *"Ha salido esto, y significa que se cumple la regla tal"* |

> [!warning] ⚠️ La prueba 1 es la del cliente, y necesita las DOS máquinas
> *"Un alumno entra en `aula1` y luego en `aula4` y encuentra sus cosas."*
>
> Para hacerla de verdad tienes que iniciar sesión con **el mismo alumno en los dos equipos**. Y para que sirva de algo: **deja un fichero desde uno y ábrelo desde el otro.** Eso es la demostración; entrar y salir no lo es.
>
> El equipo `aula4-prof` es "de profesor" por su papel en la academia, **no porque tenga nada que se lo impida**: es un Windows del dominio y un alumno también puede iniciar sesión ahí. Justo por eso vale como segunda aula.

> [!danger] 🛑 Las pruebas de invisibilidad son las que no pudiste hacer en la Fase 5
> `examenes` para un alumno y `secretaria` para un profesor **no pueden aparecer en el listado de red**. Ni aparecer.
>
> **Enseña el listado completo en pantalla**, no solo el intento de abrir. La diferencia entre *"no puedo entrar"* y *"ni siquiera está"* es todo el requisito 3E del cliente, y solo se ve así.

---

### **3.5 · 🔴 VERIFICACIÓN EXTRA: Kea DHCP y DDNS**

> [!info] 🎯 Estas comprobaciones son exclusivas de la Academia con Kea
> No estaban en la Academia con IP fija porque allí la IP se configuraba a mano en cada cliente. Aquí el DHCP es una pieza más de la infraestructura y **se verifica como tal**.

| # | Tiene que quedar demostrado | Cómo se comprueba |
| :--- | :--- | :--- |
| K1 | Que **los dos clientes han obtenido su IP del servidor Kea** | `cat /var/lib/kea/kea-leases4.csv` o `kea-admin lease-list` |
| K2 | Que la IP que tienen **es exactamente la de su reserva** | Comparar el CSV de leases con el JSON de `/etc/kea/kea-dhcp4.conf` |
| K3 | Que la configuración de Kea **sigue siendo válida** | `kea-dhcp4 -t /etc/kea/kea-dhcp4.conf` |
| K4 | 🔴 Que **los nombres de los clientes aparecen en el DNS del dominio** | `host aula1-pc01.academia.local` y `host aula4-prof.academia.local` desde el servidor |
| K5 | Que esos nombres **resuelven a las IP correctas** | Comparar la salida del DNS con `172.20.10.20` y `172.20.10.21` |
| K6 | Que el servicio Kea DHCP **está activo y habilitado** | `systemctl is-active kea-dhcp4-server` y `systemctl is-enabled kea-dhcp4-server` |

> [!danger] 🛑 La K4 es la prueba que cierra el círculo
> El cliente de la Fase 2 dijo *"que las cuentas estén en un sitio"*. Eso era el dominio. Pero para que un Windows encuentre ese dominio, necesita resolver el nombre del servidor. Y para eso necesita un DNS.
>
> **El DNS del dominio lo montaste en la Fase 2.** Pero los nombres de los clientes —`aula1-pc01` y `aula4-prof`— no aparecieron ahí hasta que Kea los registró mediante DDNS. Eso pasó **en este mismo apartado**, cuando los clientes arrancaron por primera vez y obtuvieron su IP.
>
> Si la K4 falla, el DDNS no está funcionando. Revisa `"dhcp-ddns"` en `/etc/kea/kea-dhcp4.conf` y que el servicio de nombres del dominio acepte actualizaciones.

> [!tip] 💡 Cómo se enseña K1 y K2 en el vídeo
> Muestra el CSV de leases con `cat /var/lib/kea/kea-leases4.csv`. Tiene que verse:
> - Una línea para `aula1-pc01` con IP `172.20.10.20`.
> - Una línea para `aula4-prof` con IP `172.20.10.21`.
>
> Y luego haz `grep` sobre el JSON de Kea para enseñar las dos reservas:
> ```bash
> grep -A4 '"hostname": "aula1-pc01"' /etc/kea/kea-dhcp4.conf
> grep -A4 '"hostname": "aula4-prof"' /etc/kea/kea-dhcp4.conf
> ```
>
> La IP del CSV y la IP de la reserva en el JSON **tienen que coincidir**. Si no, algo no está bien — y el fallo más probable es que la MAC declarada en la reserva no coincida con la MAC real del cliente.

> [!example] 🤖 Y el verificador, en el servidor
> ```bash
> sudo ./verificar_ac3_proyecto.sh
> ```
> Del bloque `A` al `G`, todo en verde. **Pero fíjate en lo que dice el final del informe**: hay cosas que el script declara que no puede comprobar. Esas son exactamente las que acabas de demostrar tú desde Windows.
>
> **Un script no sustituye a una prueba real. Complementa.**

---

## **4 · LABORATORIO DE AVERÍAS**

> [!danger] 🛑 Requisito: instantánea de las TRES máquinas
> Hoy hay tres VM en juego. Si rompes un cliente y no tienes punto de retorno **de ese cliente**, te toca reinstalar Windows.

---

### **AVERÍA 1 · El reloj**

> [!bug] 🔨 Qué tienes que romper
> En el **equipo de aula**, para el servicio que mantiene la hora en su sitio y **adelanta el reloj diez minutos**.
>
> Luego cierra sesión e intenta entrar con una cuenta del dominio.

> [!question] 🤔 Predice antes de ejecutar
> 1. ¿Seguirá el equipo viendo al servidor? ¿Y resolviendo su nombre?
> 2. ¿Qué mensaje crees que dará al intentar iniciar sesión?
> 3. **¿Va a mencionar la hora ese mensaje?**

**Lo que tienes que hacer tú:** diagnosticarlo desde el síntoma. Y **copiar el mensaje de error tal cual**, porque la gracia de esta avería está ahí: en lo poco que se parece el mensaje a la causa.

> [!success] 🎯 Lo que se te evalúa aquí
> Que sepas que **la autenticación de un dominio depende de la hora**, y que un desfase de minutos tira abajo un sistema que por lo demás está perfecto.
>
> Y el caso real que te vas a encontrar: **una máquina virtual que estuvo pausada tres semanas**. Al reanudarla, su reloj cree que sigue siendo el día que la pausaste. Si un lunes nadie puede entrar y el viernes iba todo bien, esto es lo primero que se mira.

---

### **AVERÍA 2 · La carpeta que reaparece**

> [!bug] 🔨 Qué tienes que romper
> En el **servidor**, quita la opción que hace que un recurso publicado **no aparezca** en el listado de red de quien no tiene acceso. Quítala solo de `examenes`.
>
> Haz copia del fichero antes, **valida antes de aplicar** y aplica el cambio.

> [!question] 🤔 Predice antes de ejecutar
> 1. Después de esto, ¿podrá un alumno **entrar** en `examenes`?
> 2. ¿Podrá **verla**?
> 3. **¿Habrá algún comando en el servidor que te diga que algo va mal?**

**Lo que tienes que hacer tú:** comprobarlo **desde el equipo de aula**, con un alumno, mirando el listado de red. Y en el servidor, buscar qué comprobación lo detecta —hay una— y cuáles no lo detectan, que son casi todas.

> [!danger] 🤯 Fíjate en lo que NO ha pasado
> **Nada.** El acceso sigue perfectamente denegado: el alumno no puede abrir la carpeta. Los permisos son correctos, la lista de accesos es correcta, el servicio funciona.
>
> Lo único que has roto es que ahora **el alumno sabe que la carpeta existe**. Y eso, desde el servidor, **no se puede comprobar por ningún medio**.

> [!success] 🎯 Lo que se te evalúa aquí
> Que **denegar el acceso y ocultar la existencia son dos capas distintas**, y que la segunda importa más de lo que parece: los nombres de las carpetas son información.
>
> Y una idea que te vas a encontrar toda tu vida profesional: **hay configuraciones que no se pueden verificar desde donde se escriben.** Por eso la Fase 5 te dejó cuatro pruebas anotadas en vez de fingir que estaba todo comprobado.

---

### **AVERÍA 3 · 🔴 La reserva que no encaja**

> [!bug] 🔨 Qué tienes que romper
> En el servidor, edita `/etc/kea/kea-dhcp4.conf` y **cambia la MAC** de la reserva de `aula1-pc01` por una inventada. Valida el JSON, reinicia el servicio Kea y **reinicia también el equipo `aula1-pc01`** *(el de alumno)*.
>
> Haz copia del JSON antes de tocarlo. Siempre.

> [!question] 🤔 Predice antes de ejecutar
> 1. ¿Se quejará `kea-dhcp4 -t` del cambio? *(La sintaxis es válida; la MAC es inventada pero bien formada.)*
> 2. Cuando el cliente reinicie y pida IP, **¿qué IP va a recibir?**
> 3. ¿Seguirá funcionando el DDNS para este equipo?

**Lo que tienes que hacer tú:** comparar la IP que recibe el cliente **antes y después** del cambio, y explicar por qué Kea le ha dado la que le ha dado. En el CSV de leases se ve perfectamente.

> [!important] ✍️ La pregunta de esta avería
> **¿Qué ha fallado exactamente?** No vale decir "la reserva": di qué propiedad de la reserva ha dejado de cumplirse —la misma que justifica que uses DHCP con reservas en vez de IP fija— y qué consecuencias tiene para el resto de servicios que dependen de que ese equipo tenga siempre `.20`.

> [!success] 🎯 Lo que se te evalúa aquí
> Que entiendas que **una reserva DHCP solo funciona si el identificador coincide**. Si la MAC no encaja, Kea trata la petición como la de un cliente desconocido y le asigna una IP del rango dinámico — si lo hay. Si no, se queda sin IP.
>
> Lo traicionero de esta avería es que **el cliente arranca, tiene IP y navega**. Todo parece normal. Pero su dirección ya no es la que espera la academia, y los servicios que dependen de ella —como el DNS o las reglas de acceso que puedan filtrar por IP— dejan de funcionar para ese equipo **sin que nadie se entere hasta que algo no responde**.

---

## **5 · PUNTO DE CONTROL**

> [!warning] ⚠️ Hoy son tres máquinas, y el orden importa
> **Apaga primero los clientes y el servidor al final.** Un cliente apagado a lo bruto mientras tiene la carpeta de red conectada puede dejar ficheros a medias.

Instantáneas: **`AC3 · Fase 6 terminada`** en las tres VM, todas apagadas de verdad.

```
SOR/Bloque_2/Proyecto_Academia/Fase 6/
    ├── B2-AC3-F6-servidor.ova
    ├── B2-AC3-F6-aula1-pc01.ova
    └── B2-AC3-F6-aula4-prof.ova
```

> [!question] 🔮 ¿Y si el disco externo se te queda corto?
> Va a pasar: seis copias del servidor más dos Windows ocupan mucho. **No borres a lo loco.**
>
> La regla que se defiende: conserva siempre **la Fase 2** *(el dominio, que es lo que no se reconstruye copiando comandos)*, **la Fase 5** *(la política entera)* y **la última**. Lo demás es negociable.
>
> **Decidir qué copia se borra es trabajo de administrador.** Di en el vídeo con qué criterio has decidido tú.

### ✅ Antes de pasar a la Fase 7

- [ ] Los tres vídeos subidos, **con identificación**.
- [ ] Las **nueve pruebas** del cliente demostradas, cada una desde el equipo que le corresponde.
- [ ] 🔴 Las **cuatro pruebas pendientes de la Fase 5**, tachadas.
- [ ] La prueba de movilidad hecha **con fichero de por medio**, no solo iniciando sesión.
- [ ] 🔴 Las **seis comprobaciones Kea (K1-K6)** enseñadas en pantalla.
- [ ] 🔴 La avería 3 grabada: reserva rota, IP cambiada, diagnóstico desde el CSV de leases.
- [ ] Verificador del `A` al `G` en verde.
- [ ] Instantáneas y `.ova` de **las tres máquinas**.

---

> [!summary] 🎓 Qué has demostrado en este apartado
> Que sabes integrar clientes en un dominio y —lo que de verdad importa— **demostrarle al cliente que tiene lo que pidió**, con una prueba por regla y desde el sitio donde se nota.
>
> Que la autenticación de un dominio **depende de la hora**, y que un desfase tumba un sistema perfecto sin decirte por qué.
>
> Que **hay cosas que solo se ven desde el cliente**. Tu servidor puede estar impecable y la protección estar a medias.
>
> Y lo nuevo de esta academia: que **el DHCP con Kea y el DDNS trabajan juntos** para que un equipo arranque, obtenga su IP, registre su nombre en el DNS y esté listo para el dominio **sin que tú toques nada en el cliente**. Has comprobado las leases, las reservas y los registros DNS, y has visto qué pasa cuando una reserva no encaja.
>
> **Siguiente:** [[Fase_7_AC3_La_Entrega]] — funciona. Ahora hay que cerrarlo y entregarlo.
