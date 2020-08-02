## Tutorial de como funciona Phaser.

Vamos a ir comentando las diez partes de como hacer un juego con Phaser 3 utilizando la parte de mijuego.html.

Ahi dentro esta tbn el script de javascript, asi que no lo vamos a hacer con archivos aparte. Todo a pelo con HTML.
___

### Parte 1

Es una pequena introduccion a lo que es la interfaz tipica de un proyecto con Phaser. 

Es interesante ver que tienes el tamano de pantalla ahi definido y que se puede elegir el tipo de juego que quieres tener, si quieres que sea web, canvas o auto que automaticamente te elige la que sea.

Vemos que todo se va a quedar guardado en una variable llamada game.

```js
    var config = {
        type: Phaser.AUTO,
        width: 800,
        height: 600,
        scene: {
            preload: preload,
            create: create,
            update: update
        }
    };

    var game = new Phaser.Game(config);

    function preload ()
    {
    }

    function create ()
    {
    }

    function update ()
    {
    }
```
___

### Parte 2

Lo primero que vamos a hacer aqui es meter todas las imagenes que vamos a utilizar al menos en la escena principal.

Estamos insertando tanto imagenes como sprites del personaje. Interesantemente le podemos poner un frameWidth y un frameHeight al personaje por ejemplo, mientras que eso se le define a los fondos u otras imagenes en la parte de create. IMPORTANTE: identificador antes de url.

```js
function preload ()
{
    this.load.image('sky', 'assets/sky.png');
    this.load.image('ground', 'assets/platform.png');
    this.load.image('star', 'assets/star.png');
    this.load.image('bomb', 'assets/bomb.png');
    this.load.spritesheet('dude', 
        'assets/dude.png',
        { frameWidth: 32, frameHeight: 48 }
    );
}
```
Luego con la funcion crear metemos tanto el cielo como la estrella.
```js
  function create ()
    {
        
        this.add.image(400, 300, 'sky');
        this.add.image(400, 300, 'star');
    }
```
El orden, como veremos, es imporante. Va por capas, y si pones primero la estrella, luego el cielo te la tapa.

___

### Parte 3

Vamos a crearle unas cuantas plataformas al juego.

Para ello, tenemos que declarar una variable antes de load `(let platforms;)` y luego utilizar una serie de comandos. 


```js
    function create ()
    {
    this.add.image(400, 300, 'sky');

    platforms = this.physics.add.staticGroup();

    platforms.create(400, 568, 'ground').setScale(2).refreshBody();

    platforms.create(600, 400, 'ground');
    platforms.create(50, 250, 'ground');
    platforms.create(750, 220, 'ground');
    }
```
De ahi lo mas importante es la tercera linea con lo de this.physics.add.staticGroup(). En la parte 4 vemos la explicacion de la plataforma.

Ademas, le vamos a meter la propiedad de physics para que metamos la fisica, y mas concretamente la gravedad, en juego.

```js
    var config = {
        type: Phaser.AUTO,
        width: 1000,
        height: 700,
        physics: {
        default: 'arcade',
        arcade: {
            gravity: { y: 300 },
            debug: false
        }
    },
```
---
### Parte 4 - Las plataformas

Estamos utilizando las fisicas de tipo arcade si recordamos que lo anadimos mas arriba. Hay dos tipos de physics bodies:
- **dinamicos or dynamic body**- un cuerpo que se puede mover y que tiene por ello velocidad o aceleracion. Puede rebotar y colisionar con otros objetos. Ademas, dicha colision esta influenciada por la masa de los cuerpos de los otros elementos.
- **estaticos or Static Body**-  es un elemento que tan solo tiene posicion y tamano. No le puedes meter velocidad, no le influencia la gravedad, y si algo colisiona con el, no se mueve. Es lo que solemos utilizar para el ground o las plataformas.

```js
platforms = this.physics.add.staticGroup();
```

En todo esto entran en juego los grupos, con lo cual te puedes ahorrar mucho trabajo, ya que una vez le digas a un parent element lo que es, automaticamente sus hijos seran physics enabled children, es decir, habran heredado la fisica.

Cuando importamos las imagenes, creamos el 'ground' y eso es precisamente lo que creamos en la escena. Primero decimos que todo lo que cree platform va a ser estatico y luego automaticamente creamos los ground con platforms.create.

#### MIND BLOWING

Lo interesante del primer ejemplo es que por ejemplo tenemos el ground que per se es 400x32. 
Lo colocamos en una posicion y se coge su centro. 

```js
    platforms.create(400, 568, 'ground').setScale(2).refreshBody();
```
Lo interesante que tiene el ejemplo anterior es que se colocaria el objeto al final del todo sin ocupar todo el ancho y tal, pero al meterle el `setScale por 2` se convierte en un objeto de 800x64, lo cual nos cubre perfectamente lo que necesitamos. Luego le tenemos que meter lo de `refreshBody()` de forma obligatoria porque estamos cambiando el tamano de un cuerpo fisico estatico.

---
### Parte 5 - Meter al jugador

Vamos a insertar todo esto para crear al jugador y a las propiedades que tiene.

```js
player = this.physics.add.sprite(100, 450, 'dude');

player.setBounce(0.2);
player.setCollideWorldBounds(true);

this.anims.create({
    key: 'left',
    frames: this.anims.generateFrameNumbers('dude', { start: 0, end: 3 }),
    frameRate: 10,
    repeat: -1
});

this.anims.create({
    key: 'turn',
    frames: [ { key: 'dude', frame: 4 } ],
    frameRate: 20
});

this.anims.create({
    key: 'right',
    frames: this.anims.generateFrameNumbers('dude', { start: 5, end: 8 }),
    frameRate: 10,
    repeat: -1
});
```

Ahora vamos a ir explicando poco a poco cada cosa.

La primera parte del codigo se encarga de crear el personaje que hemos cargado de preload.

```js
player = this.physics.add.sprite(100, 450, 'dude');

player.setBounce(0.2);
player.setCollideWorldBounds(true);
```

Como lo hemos cargado con this.physics.add.sprite, va a ser dinamico por defecto.
- El setBounce(0.2) hace que rebote ligeramente cada vez que aterrice en algun lado.
- El setCollideWorldBounds(true) lo que le dice es que no puede salirse de los bordes del juego, que en este caso son los 800x600.


El personaje se introduce con un sprite por una razon. Dentro tiene animation frames, lo cual es importante para luego las direcciones.
![personaje](./assets/dude.png)

Luego creamos los movimientos y eso se hace con la siguiente estructura. El framerate es la velocidad y el repeat: -1 le indica que los cuatro numeros de las animaciones en el caso de izquierda y derecha se repitan de forma loopica.

```js
this.anims.create({
    key: 'right',
    frames: this.anims.generateFrameNumbers('dude', { start: 5, end: 8 }),
    frameRate: 10,
    repeat: -1
});
```

IMPORTANTE: en este momento el jugador todavia no se mueve, solo le hemos dado indicaciones de como lo tiene que hacer.

### PARTE 6 - METIENDO LA FISICA EN ESTO

Hay muchos sistemas para meterle fisica a los videojuegos con Phaser. Tenemos Arcade Physics, impact Physics y Matter.js Physics, pero para hacer las cosas sencillas, vamos a seguir utilizando Arcade Physics, que es ligerito y perfecto para mobile browsers.

Le podemos meter gravedad a un objeto simplemente manipulandolo con la propiedad body.
```js 
player.body.setGravityY(300)
```

Ahora lo que tenemos que hacer es crear un `Collider Object`, que no es mas que una funcion que nos permite hacer colisionar dos entidades, en este caso, el jugador con las plataformas. Es genial porque como todas las plataformas estan juntas en un mismo grupo, no hay que ir de una en una.
```js
this.physics.add.collider(player,platforms);
```

PART 7 - CONTROLLING THE PLAYER WITH THE KEYBOARD

Buenas noticias, no tenemos que hacer lo tipico de addEventListener('keydown', keyDownHandler) anymore ya que Phaser tiene una funcion a mano muy mona que nos activa las 4 flechas principales.
```js
cursors = this.input.keyboard.createCursorKeys();
```

y luego poner en el update loop lo siguiente:
```js
if (cursors.left.isDown)
{
    player.setVelocityX(-160);

    player.anims.play('left', true);
}
else if (cursors.right.isDown)
{
    player.setVelocityX(160);

    player.anims.play('right', true);
}
else
{
    player.setVelocityX(0);

    player.anims.play('turn');
}

if (cursors.up.isDown && player.body.touching.down)
{
    player.setVelocityY(-330);
}
```

Es muy sencillo de explicar, tiene todo el sentido del mundo.

---
### PARTE 8 - Vamos a meterle objetos a la cosa

El codigo que vamos a ver en esta parte es el de las estrellas. En lugar de crear un grupo estatico como habiamos hecho con las plataformas antes, vamos a crear ahora un grupo dinamico: las estrellas.

ANTES
```js
platforms = this.physics.add.staticGroup();
platforms.create(400, 568, 'ground').setScale(2).refreshBody(); 
```

AHORA // Por defecto si ponemos group es dinamico. Y queremos un grupo dinamico porque las estrellas se mueven.
```js
stars = this.physics.add.group({
    key: 'star',
    repeat: 11,
    setXY: { x: 12, y: 0, stepX: 70 }
});

stars.children.iterate(function (child) {

    child.setBounceY(Phaser.Math.FloatBetween(0.4, 0.8));

});
```
Vamos a explicar un poco todo ese codigo.
En el caso de las estrellas hemos visto que primero te dice que objeto es con la propiedad `key('star')`, `repeat value` esta puesto en 11, luego vamos a crear 12 estrellas porque repite 1 11 veces. 
Luego tenemos una propiedad `setXY` que lo que hace es decirte la posicion inicial de la primera estrella, y la distancia entre una y otra. 

La otra pieza de codigo lo que hace es crear una iteracion que consiste que todos los hijos de stars, es decir, cada uno de los elementos, vaya a tener un setBounce ranging from 0.4 y 0.8.

Hasta este momento todo es jiji jaja, pero las estrellas se van de la pantalla porque no les hemos dicho que pueden colisionar con un elemento estatico, como son nuestras plataformas.

```js
this.physics.add.collider(stars, platforms);
```


#### COLLECTING STARS

Aqui viene una funcion chunga, que consiste en poner al jugador, el objeto, la funcion que hace que desaparezca el body de la estrella, un null que no se lo que es y un this.
```js
this.physics.add.overlap(player, stars, collectStar, null, this);
```

Luego le decimos a Phaser que chequee en todo momento si hay dicho overlap con la funcion collectStar que deciamos antes. IMPORTANTE: despues de upload, por fuera.
```js
function collectStar (player, star)
{
    star.disableBody(true, true);
}
```

### PARTE 9 - UN CONTADOR

Le vamos a meter un contador, primero creamos que la variable score empiece en cero y luego creamos la variable scoreText.

```js
var score = 0;
var scoreText;
```

Eso lo anadimos antes de upload si quiera. Luego en create vamos a meter lo siguiente:
```js
scoreText = this.add.text(16, 16, 'score: 0', { fontSize: '32px', fill: '#000' });
```

Y finalmente en la funcion de collectStar le vamos a meter la actualizacion de la puntuacion.
```js
function collectStar (player, star)
{
    star.disableBody(true, true);

    score += 10;
    scoreText.setText('Score: ' + score);
}
```

---
### PARTE 10 y FINAL - METER BOMBAS QUE NOS HAGAN LA VIDA DIFICIL

Vamos a crear lo siguiente: una vez recojamos todas las estrellas, caera una bomba que si nos toca nos mata. Luego volveran a caer las estrellas y lo que tenemos que hacer es recoger el maximo numero posible sin morir.

Lo primero es crear el grupo de las bombas y meterle un par de collider.

```js
bombs = this.physics.add.group();

this.physics.add.collider(bombs, platforms);

this.physics.add.collider(player, bombs, hitBomb, null, this);

```

Despues de eso tenemos que crear actually la function hitBomb
```js
function hitBomb (player, bomb)
{
    this.physics.pause();

    player.setTint(0xff0000);

    player.anims.play('turn');

    gameOver = true;
}
```

Y finalmente vamos a crear una bomba, para ello tenemos que modificar la funcion de collectStar, porque queremos que la bomba salga cuando no haya ninguna estrella disponible. En el siguiente codigo empieza la magia:

```js
function collectStar (player, star)
{
    star.disableBody(true, true);

    score += 10;
    scoreText.setText('Score: ' + score);

    if (stars.countActive(true) === 0)
    {
        stars.children.iterate(function (child) {

            child.enableBody(true, child.x, 0, true, true);

        });

        var x = (player.x < 400) ? Phaser.Math.Between(400, 800) : Phaser.Math.Between(0, 400);

        var bomb = bombs.create(x, 16, 'bomb');
        bomb.setBounce(1);
        bomb.setCollideWorldBounds(true);
        bomb.setVelocity(Phaser.Math.Between(-200, 200), 20);

    }
}
```

Lo nuevo es a partir del if condicional que lo que hace es que si se cumple la condicion de que no quedan estrellas restantes (eso hace countActive) entonces:
- vuelve a lanzar o activar los cuerpos de las estrellas, reasignando su yposition to cero, lo que hace que vuelvan a salir.
- se crea una variable aleatoria llamada `x` que luego vamos a utilizar para que la misma bomba salga en una posicion aleatoria. Siempre en una posicion aleatoria pero alejada del jugador.

```js
var x = (player.x < 400) ? Phaser.Math.Between(400, 800) : Phaser.Math.Between(0, 400);
```

Por ultimo creamos la bomba diciendole que bomb es hijo de bombs y que tenga una posicion x aleatoria, que no pueda botar, que no pueda salirse del mundo, y que tenga una velocidad aleatoria.

El resultado: al principio tenemos una bombita que te toca las pelotas todo el rato, pero luego se van sumando y es complicado!