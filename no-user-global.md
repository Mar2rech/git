
# Usar GIT sin ser usuario Global

La forma más limpia de configurar un usuario y email de Git solo para un repositorio específico es usar la opción --local (que es la que Git aplica por defecto cuando estás dentro del repo). Esto evita modificar tu configuración global.


---

## ✅ **Configuración de Git solo para este repositorio**

1. Entra en la carpeta del repositorio:

```bash
cd /ruta/de/tu/repositorio
```

2. Configura el usuario **solo para este repo**:

```bash
git config user.name "Tu Nombre"
```

3. Configura el email **solo para este repo**:

```bash
git config user.email "tunombre@ejemplo.com"
```

Esto crea o modifica el archivo:

```
.git/config
```

sin tocar tu configuración global (`~/.gitconfig`).

---

## 🧪 Verifica que quedó bien configurado

```bash
git config --local --list
```

Deberías ver algo como:

```
user.name=Tu Nombre
user.email=tunombre@ejemplo.com
```

---

## 📝 Si quieres asegurarte de NO usar tu identidad global

Puedes comprobar tu configuración global:

```bash
git config --global --list
```

Y si quieres evitar que Git use la global por accidente, puedes desactivarla temporalmente:

```bash
git config --global --unset user.name
git config --global --unset user.email
```

*(Solo si lo necesitas.)*

---

## git push remote: Permission to Mar2rech/game-boy.git denied to Serio120. fatal: unable to access 'https://github.com/Mar2rech/game-boy.git/': The requested URL returned error: 403

Ese mensaje significa, básicamente, **“no tienes permiso para hacer push a ese repositorio”**.  
GitHub te está diciendo que el usuario autenticado (**Serio120**) no tiene acceso de escritura al repo:

```
remote: Permission to Mar2rech/game-boy.git denied to Serio120.
```

Vamos por partes para que lo entiendas y lo soluciones rápido.

---

## 🧩 ¿Qué está pasando exactamente?

GitHub ve que intentas hacer `git push` al repositorio:

```
https://github.com/Mar2rech/game-boy.git
```

Pero el usuario con el que estás autenticado es:

```
Serio120
```

Y ese usuario **no tiene permisos** (no es dueño, no es colaborador, no tiene write access).

Por eso devuelve un **403 Forbidden**.

---

## 🔍 ¿Por qué puede ocurrir?

Hay tres causas típicas:

### 1️⃣ Estás intentando subir a un repo que NO es tuyo  
Si el repo pertenece a **Mar2rech**, solo él o sus colaboradores pueden hacer push.

### 2️⃣ Estás autenticado en GitHub con otra cuenta  
Aunque configures `git config user.name` y `git config user.email`,  
**eso NO autentica tu cuenta de GitHub**.

La autenticación real la hace:

- tu token personal (PAT)
- tu sesión de GitHub Desktop
- tu sesión de Git Credential Manager

Si estás logueado como **Serio120**, GitHub usará esa identidad.

### 3️⃣ Estás usando HTTPS y Git recuerda credenciales antiguas  
Git guarda credenciales en el **Git Credential Manager** y puede estar usando una cuenta equivocada.

---

## 🛠️ ¿Cómo lo solucionas?

### ✔ Opción A: Usa la cuenta correcta para ese repo

1. Borra las credenciales guardadas:

En Windows:

- Abre **Administrador de credenciales**
- Ve a **Credenciales de Windows**
- Busca algo como:
  ```
  git:https://github.com
  ```
- Elimínalo.

2. Vuelve a hacer `git push`  
Git te pedirá login → inicia sesión con la cuenta que **sí** tenga permisos en el repo.

---

### ✔ Opción B: Pide acceso al dueño del repo  
Si el repo es de **Mar2rech**, él debe añadirte como colaborador:

GitHub → Settings → Collaborators → Add collaborator → tu usuario.

---

### ✔ Opción C: Haz un fork si no tienes permisos  
Si no te van a dar acceso:

1. Haz un **fork** del repo en tu cuenta.
2. Clona tu fork.
3. Haz push allí.

---

## 🧪 ¿Quieres que revisemos juntos qué cuenta está usando tu Git ahora mismo?

Puedes ejecutar:

```bash
git config --global user.name
git config --global user.email
```

Y también:

```bash
git credential-manager list
```

Si quieres, dime qué te sale y te digo exactamente qué tocar.


## A ver, explico la situación:Serio120 es el usuario habitual del equipo, y tiene configurado el git en su equipo con sus credenciales. Yo solo utilizo el equipo para subir un repositorio a mi github, ¿Como puedo hacerlo sin eliminar la configuración de Serio120?

Perfecto, ahora sí: 

**puedes subir tu repo a tu GitHub desde un equipo que ya está configurado con otra cuenta (Serio120) sin romper nada de su configuración**.  
Solo necesitas **aislar tu autenticación** para esta tarea.

Aquí tienes **la forma correcta, limpia y sin tocar nada de Serio120**.

---

# ✅ Opción recomendada: usar un token personal (PAT) solo para tu repo  
Esta es la forma más segura y NO afecta al usuario Serio120.

### 1️⃣ Crea un token en tu GitHub  
En tu cuenta → Settings → Developer settings → Personal access tokens → Tokens (classic)  
Crea uno con permisos:

- `repo`

Copia el token.

---

### 2️⃣ Clona tu repo usando tu usuario directamente en la URL  
Ejemplo:

```bash
git clone https://TU_USUARIO@github.com/TU_USUARIO/tu-repo.git
```

Así fuerzas a Git a pedir credenciales **para tu usuario**, no para Serio120.

---

### 3️⃣ Cuando Git pida contraseña → pega tu token  
Git lo guardará como credencial separada.  
No toca las credenciales de Serio120.

---

# 🧼 ¿Y si ya tienes el repo clonado por Serio120?  
No pasa nada. Solo cambia la URL remota para que Git sepa que debe usar tu usuario:

```bash
git remote set-url origin https://TU_USUARIO@github.com/TU_USUARIO/tu-repo.git
```

Luego haz:

```bash
git push
```

Git te pedirá usuario/contraseña → pones tu usuario y tu token.

Esto **no elimina ni modifica** las credenciales de Serio120.

---

# 🛡️ ¿Por qué esto funciona sin romper nada?
- Git Credential Manager guarda **una credencial por URL exacta**.  
- Si tú usas `https://TU_USUARIO@github.com/...`, Git lo trata como una entrada distinta.
- La configuración global de Serio120 (`git config --global`) no se toca.
- Su sesión de GitHub Desktop tampoco se toca.

---

# 🧠 Alternativa: usar SSH (aún más limpio)
Si quieres aislar totalmente tu identidad:

1. Genera una clave SSH solo para ti:

```bash
ssh-keygen -t ed25519 -C "tu-email"
```

2. Añade la clave pública a tu GitHub.
3. Configura el remote:

```bash
git remote set-url origin git@github.com:TU_USUARIO/tu-repo.git
```

SSH usará **tu clave**, no la de Serio120.

---

# 🎯 Conclusión  
Sí puedes usar ese equipo sin romper nada.  
La clave es **no borrar las credenciales de Serio120**, sino **usar una URL o método de autenticación que identifique tu usuario**.

---

Si quieres, dime:

- tu usuario de GitHub  
- si ya tienes el repo clonado o no  

Y te digo exactamente qué comando usar en tu caso.

## Si, tengo el repo clonado y solo quiero subir la actualización. mi usuario es Mar2rech

Para tu caso —solo necesitas hacer **push a tu propio repo desde un equipo ajeno**— el token puede ser **lo más minimalista posible**.

Aquí va la respuesta clara y sin rodeos.

---

# ✅ Scopes mínimos que debes marcar para un token classic

Para poder hacer:

- `git push`
- `git pull`
- `git clone`
- `git fetch`

…solo necesitas **un único scope**:

### ✔ **`repo`**

Ese scope incluye:

- acceso de lectura y escritura a tus repositorios privados y públicos  
- permisos suficientes para hacer push  
- nada más

No necesitas marcar nada más.

---

# 🛡️ ¿Por qué solo `repo`?

Porque:

- `repo` ya engloba todo lo necesario para operaciones Git  
- no da permisos administrativos  
- no afecta a settings de tu cuenta  
- no da acceso a organizaciones ni facturación  
- es el mínimo seguro para tu caso

---

# 🚫 Scopes que NO debes marcar

No marques:

- `admin:repo_hook`
- `workflow`
- `admin:org`
- `gist`
- `delete_repo`
- `user`
- `notifications`
- ni ningún otro

No los necesitas para subir código.

---

# 🎯 Resumen rápido

| Acción | Scope necesario |
|-------|-----------------|
| Hacer push | `repo` |
| Hacer pull | `repo` |
| Clonar | `repo` |
| Nada más | ❌ |

---

Si quieres, te digo exactamente cómo usar ese token en tu push final para que Git no toque nada de Serio120.

## vale tengo otro problema: Le meti el comando: git remote set-url origin https://Mar2rech@github.com/Mar2rech/game-boy.git antes de generar el token, me salio la ventana y cancelé, ahora intento que salga otra vez la ventana y parece que no sale

Perfecto, Nicolás. Esto que te está pasando es **muy típico**: Git ya guardó una credencial fallida o incompleta cuando cancelaste la ventana, y ahora **no vuelve a mostrarla** porque cree que ya tiene una credencial válida para esa URL.

La buena noticia: **se arregla en 10 segundos**.

---

# ✅ Solución: borrar SOLO la credencial asociada a tu URL  
No afecta a Serio120, porque Git guarda credenciales **por URL exacta**.

### 🔧 Pasos:

### **1️⃣ Abre el Administrador de credenciales de Windows**
- Pulsa **Inicio**
- Escribe: **Administrador de credenciales**
- Entra en **Credenciales de Windows**

### **2️⃣ Busca la entrada relacionada con GitHub**
Suele llamarse:

```
git:https://Mar2rech@github.com
```

o

```
git:https://github.com
```

### **3️⃣ Elimínala**
Haz clic → **Quitar**

---

# 🔄 4️⃣ Intenta de nuevo el push

En tu repo:

```bash
git push
```

Ahora **sí** debe aparecer la ventana de login otra vez.

Pon:

- Usuario → `Mar2rech`
- Contraseña → tu token

---

# 🧠 ¿Por qué no afecta a Serio120?

Porque sus credenciales están guardadas bajo:

```
git:https://github.com
```

o bajo su propio usuario.

Tú estás usando:

```
https://Mar2rech@github.com/...
```

→ Es una entrada distinta.  
→ No se pisa.  
→ No se borra.  
→ No se toca.

---

# 🧪 Si quieres comprobar qué credenciales tiene Git guardadas

En PowerShell:

```bash
git credential-manager list
```

---

Si quieres, dime qué entradas ves en el Administrador de credenciales y te digo exactamente cuál borrar.

## no aparece esa, solo aparece la de Serio120

Perfecto, Nicolás. Si **no aparece ninguna credencial asociada a tu usuario**, significa que Git **sí guardó algo**, pero no en el Administrador de credenciales de Windows, sino en **Git Credential Manager**, que a veces no muestra entradas “parciales” o fallidas.

La solución sigue siendo sencilla: **forzar a Git a olvidar la credencial que guardó para esa URL**.

Aquí tienes las 3 formas efectivas de hacerlo. Te doy las más seguras primero.

---

# ✅ OPCIÓN 1 — Forzar a Git a borrar la credencial desde consola  
Ejecuta esto dentro del repo:

```bash
git credential-manager erase
```

Cuando te pida datos, introduce exactamente esto:

```
protocol=https
host=github.com
username=Mar2rech
```

Y luego presiona **Enter** dos veces.

Esto borra **solo** la credencial asociada a tu usuario, sin tocar la de Serio120.

---

# ✅ OPCIÓN 2 — Forzar a Git a pedir credenciales ignorando las guardadas  
Ejecuta:

```bash
git push https://Mar2rech@github.com/Mar2rech/game-boy.git
```

Esto obliga a Git a tratarlo como una URL nueva y debería disparar la ventana de login.

---

# ✅ OPCIÓN 3 — Ver todas las credenciales que Git tiene guardadas  
Para confirmar qué está pasando:

```bash
git credential-manager list
```

Si aparece algo como:

```
https://Mar2rech@github.com
```

→ Esa es la que hay que borrar.

---

# 🧠 ¿Por qué no aparece en el Administrador de credenciales?

Porque Git Credential Manager **no siempre guarda las credenciales en Windows Credential Store**.  
A veces las guarda en su propio backend interno, especialmente si la ventana se cerró a medias.

Por eso no ves nada nuevo, solo la de Serio120.

---

# 🚀 Después de borrar la credencial

Vuelve a hacer:

```bash
git push
```

Y ahora sí debería aparecer la ventana de login para que pongas:

- Usuario → `Mar2rech`
- Contraseña → tu token

---

Si quieres, dime qué te sale al ejecutar:

```bash
git credential-manager list
```

y te digo exactamente cuál borrar.

## Me funciono este comando:git push https://Mar2rech@github.com/Mar2rech/game-boy.git ;Este otro sin embargo falló: git credential-manager list

Perfecto, Nicolás.  
Lo que te pasó tiene todo el sentido: **Git Credential Manager (GCM)** cambió hace un tiempo y **ya no incluye el comando `list`**, por eso te salió ese mensaje. No es un error tuyo, simplemente esa función ya no existe en versiones recientes.

Lo importante es que **el comando que sí funciona (`git push https://Mar2rech@github.com/...`) resolvió el problema**, porque forzó a Git a tratar la URL como nueva y a pedir credenciales frescas.

Aun así, te explico por qué pasó y qué comandos sí existen hoy.

---

# 🧠 Por qué `git credential-manager list` ya no existe
Las versiones nuevas de Git Credential Manager (GCM Core) **eliminaron el comando `list`** por motivos de seguridad.  
Ahora solo existen:

- `get`
- `store`
- `erase`
- `configure`
- `unconfigure`
- `diagnose`

Por eso te salió:

```
'list' was not matched.
```

No es que esté roto: **simplemente ya no forma parte del programa**.

---

# 🧹 Cómo listar credenciales hoy (si algún día lo necesitas)
Como GCM ya no permite listar, la única forma de ver credenciales es:

### ✔ En Windows Credential Manager  
(pero solo si GCM decidió guardarlas ahí)

### ✔ O usando `git credential-manager get` manualmente  
Ejemplo:

```bash
git credential-manager get
```

Y cuando te pida datos, introduces:

```
protocol=https
host=github.com
```

Esto te devuelve **la credencial que GCM usaría**, si existe.

---

# 🎉 Lo importante: ya solucionaste el problema
El comando que usaste:

```bash
git push https://Mar2rech@github.com/Mar2rech/game-boy.git
```

funcionó porque:

- Forzó a Git a ignorar cualquier credencial previa.
- Obligó a GCM a pedir usuario/token nuevos.
- No tocó la configuración de Serio120.
- No necesitaste borrar nada manualmente.

Ahora ya puedes trabajar con tu repo sin interferir con el usuario habitual del equipo.

---

Si quieres, te explico cómo dejar configurado el remote para que no tengas que escribir la URL larga cada vez.

## No gracias, por hoy esta bien la clase de git-github.Solo una ultima necesidad. He utilizado estos 2 comandos:git config --global --unset user.name git config --global --unset user.email como puedo reestablecer lo borrado?

Perfecto, Nicolás. Esto tiene arreglo rápido y sin dolor.

Cuando ejecutaste:

```
git config --global --unset user.name
git config --global --unset user.email
```

lo único que hiciste fue **borrar esos valores de la configuración global**.  
Para restaurarlos, simplemente tienes que volver a ponerlos.

---

# ✅ Cómo restaurar la configuración global de Serio120

Solo necesitas volver a ejecutar:

```bash
git config --global user.name "Serio120"
git config --global user.email "el-email-de-serio120@loquesea.com"
```

Obviamente, sustituye el email por el que él tenía configurado.

---

# 🧪 ¿Cómo saber qué valores tenía antes?

Si Serio120 quiere recuperar exactamente lo que tenía, puede mirar:

### ✔ 1. Su archivo global de Git (si aún existe)
En Windows:

```
C:\Users\Serio120\.gitconfig
```

Ábrelo con un editor de texto.  
Si está vacío, es porque lo borraste con `unset`.

### ✔ 2. Su configuración en GitHub Desktop (si lo usa)
GitHub Desktop muestra el nombre y email configurados.

### ✔ 3. Su configuración en GitHub (perfil)
A veces la gente usa el mismo email que en GitHub:

GitHub → Settings → Emails

---

# 🎯 Resumen rápido

Para restaurar:

```bash
git config --global user.name "Serio120"
git config --global user.email "email@ejemplo.com"
```

Y listo, queda exactamente como antes.

---

Si quieres, te ayudo a comprobar qué valores tiene ahora mismo su `.gitconfig` para dejarlo perfecto.
