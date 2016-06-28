layout: true
background-image: url(img/wood2.jpg)

---
class: center, middle
name: Mongoose!
# Mongoose
## Mejora tu relación con MongoDB
Una pequeña charla por [Carlos González](http://caal-15.github.io)
.footnote[.monred[~] [Sitio de Mongoose](http://mongoosejs.com/)]

---
class: left

# ¿Qué es Mongoose?

.big[.mid-paragraph[.monred[Mongoose] es un ODM (Object Document Mapper) para Node.js,
que significa en pocas palabras un "traductor" de Objetos de Javascript a
Documentos de MongoDB y viceversa!]]

---
class: left

# ¿Por Qué Usarlo?

.big[.mid-paragraph[La respuesta corta es: para ahorrarte horas de frustración,
pero además de eso, .monred[Mongoose] tiene muchas ventajas, entre ellas, la más valiosa
en mi opinión es la transparencia, simplemente sigues escribiendo JavaScript!]]

---
# Mucha Charla poco coding ¿Qué vamos a hacer hoy?

.big[En esta pequeña charla introductoria les pretendo mostrar:
* Esquemas de .monred[Mongoose].
* Queries  la DB y métodos integrados.
* Métodos por instancia y a nivel de Esquema.
* Poblaciones.
* Hooks.
* Un pequeño test Con [Express](http://expressjs.com/).]

---
# Pero primero lo primero ¿Cómo la instalo?

.big[Para instalar mongoose, solo hace falta instalarlo como cualquier otro
módulo de node:]

`npm install mongoose`

## .monred[Y... ¿Cómo me conecto?]

.big[Abrir la conexión a la base de datos es muy sencillo, si mongoDB
está instalado localmente:]

```javascript
var mongoose = require('mongoose');
mongoose.connect('mongodb://localhost/awesome-db-name')
```
.footnote[.monred[~] Recuerda cerrar la conexión a la db cuando termines con
`mongoose.connection.close()`]

---
class: center, middle

# Esquemas

---
# All you need is love... or __JSON__.
## .monred[Definiendo un Esquema:]

.big[Luego de requerir nuestro recién instalado módulo de .monred[Mongoose] lo
primero que se quiere hacer usualmente es definir un esquema, y para hacerlo
solo necesitamos de JSON!]

```javascript
var humanSchema = mongoose.Schema({
  name: {type: String, maxlength: 30},
  likesGameOfThrones: {type: Boolean, default: true}
})
```
.footnote[.monred[~] El esquema superior solo es demostrativo, definiremos otro
en la charla]

---
# Cualquiera puede ser un modelo!
## .monred[Modelando nuestro esquema:]

.big[Bien, ya tenemos nuesto esquema, ahora que hacemos con el?, pues lo
modelamos:]

```javascript
module.exports = mongoose.model('human', humanSchema)
```

.big[Y eso es todo, nuestro modelo está listo para usarse!]

---
# ¿Creíste que era fácil?, se pone aún más fácil

.big[.monred[Mongoose] provee gran cantidad de funciones prediseñadas:]

```javascript
var Human = require('./models/human.js')
Human.create(data, cb)
Human.find(cb)
Human.findById(id, cb)
Human.findByIdAndUpdate(id, {$set: data}, cb)
Human.findByIdAndRemove(id, cb)
```
.big[Donde cb es la función callback, id es el id generado por mongo y data es...
lo adivinaste!, un __JSON__, como:]

```javascript
var data = {name: 'Carlos'}
```

---
# Encuentra lo busca tu corazón, o no...
## .monred[Queries un poco mas avanzados:]

.big[La operación find de .monred[Mongoose] soporta por supuesto querying mucho
mas avanzado usando... sip, __JSON__ y operadores nativos de MongoDB:]

```javascript
var query = {
  age: {$gte: 20, $lte: 30},
  sex: 'F',
  likesGameOfThrones: true
}

Human.find(query, cb(err, humans))
```

.footnote[.monred[~] Me gusta Game of Thrones.]

---
class: center, middle

# Métodos

---
# Un método solo para ti
## .monred[Métodos de instancia]

.big[Pertenecen a cada una de las instancias del modelo. Se pueden definir así:]

```javascript
humanSchema.methods.isAwesome = function () {
  return this.likesGameOfThrones && this.likesCats
}
mongoose.model('human', humanSchema)
```
.big[Y para llamar el método:]

```javascript
Human.findOne({}, function (err, human) {
  // Is this human awesome?
  console.log(human.isAwesome());
})
```

---
# Un método para el pueblo
## .monred[Métodos estáticos]

.big[Los métodos estáticos funcionan al nivel del modelo, no de las instancias
del mismo, se definen así:]

```javascript
humanSchema.statics.findAwesomePeople = function (cb) {
  this.find({likesGameOfThrones: true, likesCats: true}, cb(err, humans))
}
```

.big[Y se llama desde el modelo mismo:]

```javascript
var Human = require('./models/human.js')
Human.findAwesomePeople(cb)
```

---
class: center, middle

# Poblaciones

---
# Pero ¿Que será de nuestra relación?
## .monred[Trabajando con poblaciones]

.big[Para algunas relaciones entre modelos y queries se necesita algo similar a
un "join", y para eso existe populate:]

```javascript
var petSchema = mongoose.Schema({
  name: String,
  howYouActuallyCallIt: String,
  human: {type: String, ref: 'human'}
})

petSchema.statics.findPetsAndHumans = function (cb) {
  this.find()
    .populate('human')
    .exec(cb)
}
```

---
class: center, middle
# Hooks

---
# Antes y Después
## .monred[Utilizando pre y post hooks]

.big[Útiles para realizar ciertas acciones antes o después de ciertos eventos en
la DB:]

```javascript
petSchema.pre('save', function (next) {
  Human.findByIdAndUpdate(this.human, {$inc: pets}, function (err, human) {
    if (err || !human) console.log('No pet added :(');
    else console.log('You have a new Pet :D!');
  })
  next()
})

petSchema.post('save', function (pet) {
  console.log('The newly created pet name is ' + pet.name)
})
```
---
class: center, middle
# Junto a Express

---
# Más Transparente que el agua
## .monred[Trabajando con Frameworks]

.big[Como es de esperarse, .monred[Mongoose] trabaja de maravilla con
frameworks de Node.js, démosle una mirada a express:]

```javascript
app.post('/', function (req, res) {
  Human.create(req.body, function (err, human) {
    if (err) res.json({msg: 'Failure'})
    else res.json({msh: 'OK', data: human})
  })
})
```
