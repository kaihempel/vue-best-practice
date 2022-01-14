author: Kai Hempel
summary: Workshop: Vue best practice
id: getting-started
categories: Frontend development
environments: JS
status: Published

# Workshop: Vue best practice

## Einführung

Vue.js biete viele möglichkeiten interaktive Frontends zu implementieren. Mit wachsender Komplexität sind einige dieser 
Wege zu bevorzugen. Andere sollten gemieden werden.

In diesem Workshop werden folgende Konzepte näher besprochen:
- Simple Eingabe Elemente
- Vuex Store
- Lifecycle Events
- Memory managment

#### Hinweis:
Dieser Workshop basiert auf der Vue Version 2. 

### Umgang mit diesem Workshop

Negative
: **ROT markiert:**
Fortgeschrittene Implementierungen, solltet ihr euch nur damit befassen, falls ich flotter oder erfahrener unterwegs seid! Zuerst die einfachen Aufgaben erledigen, danach die Fortgeschritten, andernfalls kommt ihr mit der Zeit von 60 Minuten nicht hin!

Positive
: **Grün markiert:**
Erstellen/Implementierung notwendig. (Ihr müsst hier selber überlegen und neuen Code schreiben)

Positive
: **Gelb markiert:**
Anpassungen notwendig (Ihr müsst hier selber überlegen und bestehenden Code anpassen)

## Simple Eingabe Elemente

### Komunikation zwischen Komponenten
Die Kommunikation zwischen Komponente geschieht mit hilfe von "Properties" (Parent -> Child) 
und Customs-Events (Child -> Parent).


#### Properties
- [https://vuejs.org/v2/guide/components-props.html](https://vuejs.org/v2/guide/components-props.html)

#### Customs Events
- [https://vuejs.org/v2/guide/components-custom-events.html](https://vuejs.org/v2/guide/components-custom-events.html)

#### Model binding
- [https://vuejs.org/v2/guide/forms.html](https://vuejs.org/v2/guide/forms.html)

### Simple Input Komponente
```js
<template>
    <div class="mein-tolles-design">
        <input :message="message" @input="message = $event.target.value" />
    </div>
</template>

<script>
export default MyInput {
    data: {
        return {
            message: 'test'
        }       
    }
}
</script>
```

Das Eingabe Feld hat ein tolles Design, kann aber in der Applikation noch nicht wie ein normales Eingabefeld genutzt werden.
Dafür muss die übergeordnete Komponente den Wert "message" setzen können. Ebenso muss eine Änderung dieser Komponente übergeben werden. 

```js
<template>
    <div class="mein-tolles-design">
        <input :value="message" @input="message = $event.target.value" />
    </div>
</template>

<script>
export default MyInput {
    props: ['message']
}
</script>
```

Jetzt kommt es allerdings zu einem Fehler, da eine "Propertie" nicht geändert werden darf!
Wir brauchen eine Methode, welche die Event Daten weiter reicht.

```js
<template>
    <div class="mein-tolles-design">
        <input :value="message" @input="handleInputEvent" />
    </div>
</template>

<script>
export default MyInput {
    props: ['message'],
    methods: {
        handleInputEvent($e) {
            this.$emit('input', $e.target.value);    
        }       
    }
}
</script>
```

Wenn "Propertie" den Namen "value" hat und das Event den Namen "input" kann auch die Vereinfachung "v-model" genutzt werden. 
So kann die übergeordnete Komponente eine Variable direkt an die Komponente binden. 
So verhalten sich die eigenen Eingabefelder ähnlich wie native HTML Elemente.

## Computed properties

Ein sinnvolle Methode um Zustände abzubilden sind "computed properties". Gerade zu Template Steuerung bieten sich diese an. 
Computed properties werden von Vue "cached" nach dem sie ausgewertet werden. Insofern muss bei der Implementierung darauf geachtet werden,
dass sie bei einer Zustandsänderung erneut ausgewertet werden können. Bei komplexen Datentypen wie Objekte und Arrays gibt es Einschränkungen
(leider nicht durch YouTube Premium abgedeckt), welche beachtet werden müssen. 

```js
<template>
    <div class="mein-tolles-design">
        <ul v-for="(value, index) in dataList">
            <li :key="index">
                {{value}}
            </li> 
        </ul>    
    </div>
</template>

<script>
export default MyInput {
    data: {
        return {
            dataList: []
        }       
    }
}
</script>
```

Soll bei einer leeren Liste ein alternativ Text dargestellt werden:

```js
<template>
    <div class="mein-tolles-design">
        <ul 
            v-if="dataList.length > 0"
            v-for="(value, index) in dataList">
            <li :key="index">
                {{value}}
            </li> 
        </ul>
        <div v-else>
            Keine Daten!
        </div>    
    </div>
</template>
```

Alternativ kann die Prüfung im Template Code durch eine "computed propertie" ersetzt werden. Dadurch wird eine bessere Lesbarkeit erreicht. 
Des Weiteren können komplexere Auswertungen viel leichter und verständlicher umgesetzt werden.

```js
<template>
    <div class="mein-tolles-design">
        <ul 
            v-if="hasData"
            v-for="(value, index) in dataList">
            <li :key="index">
                {{value}}
            </li> 
        </ul> 
        <div v-else>
            Keine Daten!
        </div>
    </div>
</template>

<script>
export default MyInput {
    data: {
        return {
            dataList: []
        }       
    },
    computed: {
        hasData() {
            return this.dataList.length > 0;    
        }       
    }
}
</script>
```

## Vuex Store

Komplexe Zustände des gesamt Systems sollten über den Vuex Store abgebildet werden. 

- [https://vuex.vuejs.org/](https://vuex.vuejs.org/)

Ebenso empfiehlt sich dieser für eine Komplexere und Applikationsweite Kommunikation. Oft hat man das Problem, dass Komponenten aktualisiert werden sollen, welche nicht direkt durch ein Event erreicht werden können. 
Oft findet man im Web dafür den Lösungsansatz des Eventbus. Einer separaten Vue Instanz, welche nur zum abarbeiten von Events genutzt werden soll. Dadurch können die Eventhandler in ganz anderen Komponenten implementiert werden, welche keinen direkten Zusammenhang zu den emitierenden Komponenten haben.
Leider ergeben sich daraus folgenden Probleme:
- Schwerer zu Debuggen
- Eventhandler müssen gezielt wieder entfernt werden
- Entfernen Schwierig wenn einige Komponenten noch länger vorhanden sind

Alternativ kann auch der Store für Temporäre Zustände genutzt werden.

### Verschiedene Store operationen

Im Vuex Store gibt es drei Arten von Daten Interaktionen:
- Getter
- Mutations
- Actions

Getter sind im Grunde "computed properties", welche den aktuellen Zustand des Systems nach aussen repräsentieren. 
Wichtig ist hier, dass das Prinzip der "computed properties" verstanden wurde. Auch bei den Gettern gelten die entsprechenden Einschränkungen.

Mutations setzten neue Werte! Mutations **müssen** immer synchron sein! Für Asynchrone Operationen sind die Actions vorgesehen.

Actions sollen auch Werte ändern, sind aber, wie oben erwähnt, für die Asynchron Operationen vorgesehen. 
Ein Beispiel dafür sind Backend Operationen über einen Ajax Call.

Ein Store Beispiel:
```js
const store = new Vuex.Store({
  state: {
      list: []
  },
  mutations: {
      addItem (state, item) {
         state.list.push(item);
      },
      setList (state, list) {
         state.list = list;
      }
  },
  actions: {
      loadItems (context) {
          ajaxCall().then(data => {
              context.commit('setList', data.list);
          });
      }
  }
})
```

Den Store in Vue einbinden:
```js
new Vue({
  el: '#app',
  store: store,
})
```

Ausführen einer Mutation:
```js
store.commit('addItem', item);

// Oder in vue via this.$store

this.$store.commit('addItem', item);
```

Ausführen einer Action:
```js
store.dispatch('loadItems');

// Oder in vue via this.$store

this.$store.dispatch('loadItems');
```

Zugriff auf den Store (state):
```js
store.state.list;

// Oder in vue via this.$store

this.$store.state.list;
```

Oder mithilfe der getter:
```js
...
getters: {
    // ...
    itemCount: (state) => {
        return state.list.length
    }
}
...

store.getters.itemCount;

// Oder in vue via this.$store

this.$store.getters.itemCount;
```

Getters, Mutations und Actions können auch importiert werden und dann innerhalb der Vue Komponente direkt benutzt werden.

## Lifecycle Events

tbd

## Memory managment

tbd
