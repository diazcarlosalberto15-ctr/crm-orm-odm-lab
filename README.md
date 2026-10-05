# CRM ORM/ODM Lab

API REST de un CRM básico que combina un ORM (Sequelize + PostgreSQL) y un ODM (Mongoose + MongoDB).

## Stack

- Node.js 22, Express 5, CommonJS
- Sequelize + PostgreSQL 16 (`User`, `Company`, `Contact`)
- Mongoose + MongoDB 7 (`Activity`)
- Jest + Supertest
- GitHub Codespaces, Dev Containers, Docker Compose
- Supervisor (`npm run dev`)

## Arquitectura

```text
GitHub Codespace
│
├── app       Node.js 22  ──┬── Sequelize ──> postgres (PostgreSQL)
│                           └── Mongoose  ──> mongo    (MongoDB)
├── postgres
└── mongo
```

La aplicación se conecta por nombre de servicio (`postgres`, `mongo`). Las credenciales de desarrollo llegan como variables de entorno definidas en `.devcontainer/docker-compose.yml` (ver `.env.example`).

## Iniciar el Codespace

1. En GitHub: **Code → Codespaces → Create codespace on main**.
2. Espera a que se levanten los tres servicios (`app`, `postgres`, `mongo`). `postCreateCommand` ejecuta `npm install`.

## Instalar dependencias

```bash
npm install
```

## Seed y reset

```bash
npm run seed    # inserta datos deterministas (3 users, 4 companies, 8 contacts, 10 activities)
npm run reset   # elimina y recrea tablas/base de datos y vuelve a sembrar
```

## Iniciar la API

```bash
npm start       # node ./bin/www
npm run dev     # supervisor ./bin/www
```

Servidor en el puerto `3000` (variable `PORT`).

## Pruebas

```bash
npm test
```

Cada suite restablece PostgreSQL y MongoDB antes de ejecutarse y cierra las conexiones al terminar.

## Endpoints

| Método | Ruta | Descripción |
|---|---|---|
| GET | `/health` | Health check |
| GET | `/users` | Listar usuarios |
| GET | `/users/:id` | Obtener usuario |
| POST | `/users` | Crear usuario |
| PUT | `/users/:id` | Actualizar usuario |
| DELETE | `/users/:id` | Eliminar usuario |
| GET | `/companies` | Listar compañías (`?industry=`) |
| GET | `/companies/:id` | Obtener compañía |
| POST | `/companies` | Crear compañía |
| PUT | `/companies/:id` | Actualizar compañía |
| DELETE | `/companies/:id` | Eliminar compañía |
| GET | `/contacts` | Listar contactos |
| GET | `/contacts/:id` | Obtener contacto |
| POST | `/contacts` | Crear contacto |
| PUT | `/contacts/:id` | Actualizar contacto |
| DELETE | `/contacts/:id` | Eliminar contacto |
| GET | `/activities` | Listar actividades (`?type=`) |
| GET | `/activities/:id` | Obtener actividad |
| POST | `/activities` | Crear actividad |
| PUT | `/activities/:id` | Actualizar actividad |
| DELETE | `/activities/:id` | Eliminar actividad |

Los errores se devuelven como JSON: `{ "error": "Contact not found" }`.

## Respuestas
** 1. Dos motores **
En el activities.js para poder guardar metadatos variables según el registro se utiliza una db de tipo NoSQL ya que es mas flexible que una SQL.
Mientras que contacts.js y companies.js necesitan un esquema mas rigido que evite registros inconsistentes.

** 2. ORM vs ODM **
Un ORM abstrae bases de datos SQL mediante tablas y filas usando Sequelize. Un ODM abstrae las bases de datos NoSQL en colecciones y documentos usando mongoose, Su principal diferencia es que ORM maneja esquemas rígidos con llaves foraneas y ODM esquemas dinamicos JSON y BSON

** 3. Configuracion por variables de entorno **
En la raiz del proyecto se crea un archivo .env para evitar subir credenciales o llaves a github. Los nombres de los host que usa la app para conectarse son posgres en el caso de Sequelize y mongodb://mongo:27017/crm en el caso de mongoose. No son localhost por que corren en contenedores independientes aislados.

** 4. Asociaciones. **
Es una Relacion de 1 a muchos la llave foránea es companyId que esta alojada en la tabla de contacts y el alias as: 'contacts' sirve para definir el nombre con el que se accede a los contactos desde company

** 5. Eager loading **
Hacer consultas separadas genera dos peticiones a la db, mientras que el include solo hace una sola petición, Es preferible usar el include ya que solo hace un viaje a la db y entega la estructura JSON ya procesada

** 6. Instancia vs consulta** 
El enfoque por instancia modifica el objeto y retorna la instancia del registro actualizado mientras que Model.update ejecuta la instrucción sin lectura previa retornando solo un arreglo con el numero de filas afectadas.

** 7. Esquema flexible **
Se utiliza el tipo mongoose.Schema.Types.Mixed. Esto permite guardar cualquier objeto sin restricciones en CALL, EMAIL o MEETING, su desventaja es el perder las validaciones automáticas de tipos de datos en la db.

** 8. Sin ref **
No se puede usar ref o populate porque esas funciones de moongose solo sirven dentro de mongoDB y no pueden cruzar hacia posgreSQL , si un User o Contact se elimina en posgreSQL, el id almacenado en mongoDB querada como un registro huérfano

** 9. Documento actualizado **
Antes retomaba el documento con los datos viejos previos a la modificación y fue corregido agregando la opción {new: true, runValidators: true} para forzar la devolución del documento modificado y validar los datos.

** 10. Pruebas de comportamiento **
probar el comportamiento verifica lo que la API devuelve en lugar de como esta implementado internamente. La ventaja de esto es que permite refactorizar o cambiar la lógica del código sin romper las pruebas

** 11. Repetibilidad **
setup.js limpia y restablece las bases de datos e inserta datos semilla antes y después de cada suite. Esto asegura un entorno limpio para que las pruebas siempre arrojen los mismos resultados

** 12. Tu experiencia **
El reto mas difícil fue el 5 por la configuración del alias as: 'contacts', Las pruebas jest al mostrar expected vs recived fueron de gran ayuda para identificar si la respuesta devolvia el objeto antes de actualizarse.

## Evidencia

![Evidencia 9 test de 9](images/Evidencia%20npm%20test.png)