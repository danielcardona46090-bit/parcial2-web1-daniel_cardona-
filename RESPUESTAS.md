# Parcial 2 · Respuestas de razonamiento

Nombre:Daniel Cardona 

Son cuatro preguntas de 5 puntos sobre código que ya conoces. Para cada una:

1. Lee el código y escribe primero lo que **crees** que pasa.
2. Compruébalo: pégalo en la consola de DevTools (F12 → Console) con tu página del parcial abierta en Live Server. En esa consola sí se puede usar `await` fuera de una función.
3. Escribe tu respuesta debajo de cada letra: (a), (b), (c)…

Se califica la explicación, no solo el resultado: con dos o tres frases por letra es suficiente. Escríbela con tus palabras, porque te la pueden preguntar.

## P1 · ¿En qué orden se imprime? (5 puntos)

**Lo viste en:** la función `cargar` del Explorador (semana 8): «await pausa ESTA función, no el navegador».

```js
console.log('1');
setTimeout(() => console.log('2'), 0);
const cargar = async () => {
  console.log('3');
  const respuesta = await fetch('https://rickandmortyapi.com/api/episode/1');
  console.log('4');
};
cargar();
console.log('5');
```

**(a)** Los cinco números en el orden en que salen en la consola. (2 pts)

Respuesta:

1, 3, 5, 2, 4

**(b)** Por qué el `3` sale **antes** que el `5`, si `cargar` es una función `async`. (2 pts)

Respuesta:

Cuando se llama a `cargar()`, una función `async` se ejecuta de forma normal y síncrona hasta el primer `await`. El `console.log('3')` está antes del `await`, así que corre enseguida dentro de la llamada `cargar()`, y solo después el código sigue con el `console.log('5')`. Lo que se pausa es lo que viene después del `await`, no el inicio de la función.

**(c)** Por qué el `5` no espera a que termine el `fetch`, y por qué el `2` y el `4` salen al final. (1 pt)

Respuesta:

Al llegar al `await fetch(...)`, `cargar` se pausa y devuelve el control al código que la llamó, así que el `5` se imprime sin esperar la red. El `2` (callback de `setTimeout`, aunque sea con 0 ms) y el `4` (lo que sigue al `await`) esperan a que termine todo el código síncrono y pasan por la cola de eventos; el `4` además depende de que la respuesta llegue de internet, por eso sale de último.

## P2 · Un 404 que no llega al catch (5 puntos)

**Lo viste en:** el `pedirJSON` del Explorador (semana 8): «fetch solo falla si NO pudo hablar con el servidor».

El episodio 9999 no existe: la API responde con código **404** y el cuerpo `{"error":"Episode not found"}`.

```js
try {
  const respuesta = await fetch('https://rickandmortyapi.com/api/episode/9999');
  const datos = await respuesta.json();
  console.log('Episodio:', datos.name);
} catch (error) {
  console.log('Falló:', error.message);
}
```

**(a)** Qué imprime la consola, exactamente. (1 pt)

Respuesta:

`Episodio: undefined`

**(b)** Por qué **no** entra al `catch`, aunque el episodio no existe. Pista: piensa en la diferencia entre «el servidor respondió con un 404» y «no hubo respuesta» (por ejemplo, sin internet). (2 pts)

Respuesta:

`fetch` solo rechaza su promesa cuando no logra hablar con el servidor (sin internet, DNS caído, etc.). Un 404 sí es una respuesta HTTP válida: la promesa se resuelve con `respuesta.ok === false`. Además `respuesta.json()` funciona porque `{"error":"Episode not found"}` es JSON válido; como no hay excepción no entra al `catch`, y `datos.name` no existe, por eso da `undefined`.

**(c)** La línea que falta para que sí entre al `catch`: escríbela e indica en qué parte del código va. Luego escribe qué imprimiría la consola con esa línea. (2 pts)

Respuesta:

Hay que revisar `respuesta.ok` y lanzar el error: `if (!respuesta.ok) throw new Error(`HTTP ${respuesta.status}`);`. Va justo después del `await fetch(...)` y antes de `respuesta.json()`. La consola imprimiría: `Falló: HTTP 404`.

## P3 · LocalStorage sin JSON (5 puntos)

**Lo viste en:** `js/favoritos.js` del Explorador (semana 8): «localStorage solo guarda TEXTO».

```js
const vistos = [1, 5, 8];
localStorage.setItem('vistos', vistos);
const leido = localStorage.getItem('vistos');
console.log(leido, typeof leido);      // (a)
console.log(leido.length);             // (b)
const recuperado = JSON.parse(leido);  // (c)
```

**(a)** Qué imprime la línea marcada (a): el valor y su tipo. (1 pt)

Respuesta:

`1,5,8 string`. `localStorage.setItem` convierte el array a texto con `toString()`, que une los elementos con comas, y `getItem` siempre devuelve un string.

**(b)** Qué imprime la línea marcada (b), y por qué da ese número. (1 pt)

Respuesta:

`5`. Es la longitud del texto `"1,5,8"`: cinco caracteres (tres dígitos y dos comas), no la cantidad de elementos del array.

**(c)** Qué pasa en la línea marcada (c): ¿funciona o sale un error? Si sale un error, cuál y por qué. (1 pt)

Respuesta:

Sale un error: `SyntaxError` (por ejemplo `Unexpected non-whitespace character after JSON at position 1`, según el navegador). `"1,5,8"` no es JSON válido: `JSON.parse` lee el `1` como número y luego encuentra una coma que no esperaba; para ser un array tendría que ser `"[1,5,8]"`.

**(d)** La segunda línea corregida, para que `recuperado` vuelva a ser el array `[1, 5, 8]`. (1 pt)

Respuesta:

`localStorage.setItem('vistos', JSON.stringify(vistos));` (así `JSON.parse(leido)` devuelve `[1, 5, 8]`).

**(e)** Por qué en tu `leerVistos` el `JSON.parse` va dentro de un `try/catch`: qué podría pasarle a la app si no estuviera. (1 pt)

Respuesta:

Porque lo guardado puede estar dañado o editado a mano (por ejemplo `{roto`), y `JSON.parse` lanza un `SyntaxError` con texto inválido. Sin `try/catch` el error saldría de `leerVistos()` al arrancar `app.js`, el módulo se detendría y la app no cargaría. Con `try/catch` se devuelve `[]` y la app arranca normal.

## P4 · Botones que dejan de funcionar (5 puntos)

**Lo viste en:** la delegación en la grilla del Explorador (semana 7, paso 04). Está explicada en el README, en R6: «¿Qué es la delegación?».

Un compañero escribió este código. Al cargar la página, el primer clic en «Marcar como visto» funciona; pero después de ese clic, o después de escribir una letra en el buscador, **ningún botón responde**.

```js
render();
document.querySelectorAll('[data-id]').forEach((boton) => {
  boton.addEventListener('click', () => {
    vistos = alternarId(vistos, Number(boton.dataset.id));
    guardarVistos(vistos);
    render();
  });
});
```

**(a)** Qué les pasó a los botones. Pista: fíjate en qué hace `render()` con el contenido de `#lista` cada vez que se llama, y en cuántas veces se ejecuta el `forEach`. (2 pts)

Respuesta:

`render()` reemplaza todo el contenido de `#lista` con `innerHTML`: destruye los botones viejos y crea botones nuevos, que no tienen listener. El `forEach` se ejecutó una sola vez, al inicio, y solo puso listener a los botones que existían entonces. Tras el primer clic (o al escribir en el buscador) `render()` los reemplaza y los nuevos no responden.

**(b)** El código reescrito con **delegación**: un solo listener en `#lista` que funcione con cualquier botón, aunque `render()` lo haya creado después. Se revisan tres cosas, 1 punto cada una: que el listener esté en `#lista` y no en los botones; que use `closest('[data-id]')` (o algo equivalente) para saber qué botón se presionó; y que no haga nada si el clic no fue en un botón. (3 pts)

Respuesta:

El listener va en `#lista`, que nunca se borra: el clic burbujea desde el botón hasta ahí y `closest('[data-id]')` identifica el botón presionado. Si el clic no fue en un botón, `closest` devuelve `null` y la función termina sin hacer nada.

```js
lista.addEventListener('click', (evento) => {
  const boton = evento.target.closest('[data-id]');
  if (!boton) return; // el clic no fue en un botón de tarjeta
  vistos = alternarId(vistos, Number(boton.dataset.id));
  guardarVistos(vistos);
  render();
});
render();
```