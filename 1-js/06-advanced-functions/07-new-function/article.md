
# The "new Function" syntax

C'è un modo in più per creare una funzione. E' raramente usato, ma talvolyta non c'è alternativa.

## Syntax

La sintassi per creare una funzione:

```js
let func = new Function ([arg1, arg2, ...argN], functionBody);
```

La funzione è creata con gli argomenti `arg1...argN` e la data `functionBody`.

E' più facile da capire se si guarda l'esempio. Ecco una funzione con 2 argomenti:

```js run
let sum = new Function('a', 'b', 'return a + b');

alert( sum(1, 2) ); // 3
```

E qua c'è una funzione senza argomenti, solo con il corpo della funzione:

```js run
let sayHi = new Function('alert("Hello")');

sayHi(); // Hello
```

Ciò che rende diverso questo metodo dagli altri è che la funzione è creata letteralmente da una stringa, che è eseguita quando il codice viene eseguito.

Tutte le altre maniere di dichiarare una funzione richiedevano di scrivere la funzione direttamente nel codice.

Ma il pezzo di codice `new Function` permette di trasformare qualsiasi stringa in una funzione. Per esempio, possiamo ricevere una nuova funzione dal server e poi eseguirla:

```js
let str = ... receive the code from a server dynamically ...

let func = new Function(str);
func();
```

E' usato in casi veramente specifici, come quando riceviamo codice da un server, o quando compiliamo dinamicamente una funzione da un template, in web-applications più complesse.

## Closure

Di solito, una funzione ricorda dov'è stata creata nella proprietà `[[Environment]]`. Si riferisce all'ambiente lessicale nel quale è stata creata (abbiamo trattato questo argomento nel capitolo <info:closure>).

Ma quando una funzione è creata usando `new Function`, il suo `[[Environment]]` cerca la funzione non nell'ambiente lessicale, ma nell'ambiente globale.

Quindi, una funzione del genere non deve avere accesso a variabili esterne, ma solo alle variabili globali.

```js run
function getFunc() {
  let value = "test";

*!*
  let func = new Function('alert(value)');
*/!*

  return func;
}

getFunc()(); // error: value is not defined
```

Comparalo con la maniera tradizionale:

```js run
function getFunc() {
  let value = "test";

*!*
  let func = function() { alert(value); };
*/!*

  return func;
}

getFunc()(); // *!*"test"*/!*, from the Lexical Environment of getFunc
```

Questa feature può apparire strana, ma a livello pratico è molto utile.

Immagina che dobbiamo creare una funzione partendo da una stringa. Il codice di quella funzione non è conosciuta al tempo di stesura del programma (per questo non usiamo funzioni nella maniera classica), ma verrà conosciuto quando il codice verrà eseguito. Potremmo riceverlo dal server o da un'altra fonte.

La nostra funzione ha bisogno di interagire con il codice principale.

Cosa succederebbe se tu potessi accedere alle variabili esterne?

Il problema è che prima di Javascript sia stato pubblicato, è stato compresso usando un *minifier* -- un programma speciale che restringe il codice rimuovendo commenti extra, spazi e -- soprattutto, rinomina le variabili locali con nomi più corti.

Per esempio, se una funzione ha `ler userName`, minifier lo rimpiazza con `let a` (oppure un'altra lettera se questa è già stata presa), e lo fa dappertutto. Di solito è una cosa sicura da fare, perchè la variabile è locale, niente al di fuori della funzione può accederci. E dentro la funzione, minifier rimpiazza tutte le menzioni di esso. I minifiers sono smart, analizzano la struttura del codice, così non fanno danni. Non sono solo semplici trova-e-rimpiazza.

Quindi se `new Function` avesse accesso a variabili esterne, non sarebbe in grado di trovate `userName`.

**Se `new Function` avesse accesso a variabili esterne, avrebbe problemi con il minifier.**

Inoltre, un codice del genere sarebbe architetturalmente brutto e incline ad errori.

To pass something to a function, created as `new Function`, we should use its arguments.
Per passare qualcosa ad una funzione, creata come `new Function`, dovremmo usare i suoi argomenti.

## Summary

The syntax:

```js
let func = new Function ([arg1, arg2, ...argN], functionBody);
```

Per ragioni storiche, gli argomenti possono essere anche dati come una lista separata da virgole.

Queste tre stringhe di codice significano la stessa cosa:

```js
new Function('a', 'b', 'return a + b'); // basic syntax
new Function('a,b', 'return a + b'); // comma-separated
new Function('a , b', 'return a + b'); // comma-separated with spaces
```

Le funzioni create con `new Function`, hanno `[[Environment]]` che si riferisce all'ambiente lessicale globale, non quello esterno. Quindi, non possono usare variabili esterne. Ma questa è una buona cosa, perchè assicura la mancanza di errori. Passare parametri esplicitamente è un metodo migliore architettonicamente e non causa problemi con i minifiers.
