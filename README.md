# Шпаргалка по Vue 3 Options api, React и React Native (оба functional component)

# Оглавление

- [Vue 3 Options API](#vue-3-options-api)
  - [Пример компонента](#9-пример-компонента)
  - [Управление состоянием (State management)](#10-управление-состоянием-state-management)
  - [Окружение и библиотеки](#11-окружение-и-библиотеки)

- [React (Functional Components)](#react-functional-components)
  - [Пример компонента](#7-пример-компонента)
  - [Управление состоянием (State management)](#8-управление-состоянием-state-management)
  - [Окружение и библиотеки](#9-окружение-и-библиотеки)

- [React Native (Functional Components)](#react-native-functional-components)
  - [Пример компонента](#4-пример-компонента)
  - [Управление состоянием (State management)](#8-управление-состоянием-state-management-1)
  - [Стилизация](#9-стилизация)
  - [Окружение и библиотеки](#10-окружение-и-библиотеки)


## Vue 3 Options API

### Особенности

- **Декларативные шаблоны** с директивами (`v-model`, `v-for` и т. д.), а не JSX
- **Двусторонняя привязка** по умолчанию (`v-model`) без дополнительных библиотек
- **Реактивность через Proxy**, отслеживает чтение/запись полей без явных геттеров/сеттеров
- **Options API** разделяет код на блоки (`data`, `methods`, `computed`), не смешивая логику

### 1. **`data()`** – возвращает объект реактивного состояния;

в Options API `data` обязательно должна быть функцией, чтобы при повторном использовании компонента каждый экземпляр
имел свой собственный стейт. Vue оборачивает возвращённый объект через Proxy для отслеживания изменений.

   ```js
   export default {
    data() {
        return {
            count: 0,
            user: {name: 'Alice', age: 30}
        };
    }
}
   ```

### 2. **`methods`** – обычные функции для обработки событий и действий;

Методы определяются в одном объекте, а при вызове внутри шаблона автоматически привязываются к контексту компонента (
`this`). Методы не кэшируются.

   ```js
   export default {
    data() {
        return {count: 0};
    },
    methods: {
        increment() {
            this.count++;
        },
        greet(name) {
            alert(`Hello, ${name}`);
        }
    }
}
   ```

### 3. **`computed`** – вычисляемые свойства с автоматическим кэшированием;

Вычисляемые свойства пересчитываются только при изменении зависимостей, что позволяет оптимизировать рендер. В отличие
от методов, кэшируются до следующего изменения.

   ```js
   export default {
    data() {
        return {count: 5};
    },
    computed: {
        doubleCount() {
            return this.count * 2;
        },
        welcomeMessage() {
            return `Count is ${this.count}`;
        }
    }
}
   ```

### 4. **`watch`** – отслеживание изменений реактивных данных;

В `watch` можно отслеживать как простые поля, так и глубоко вложенные объекты (`deep: true`). Полезно для асинхронных
операций, реагирующих на изменение данных.

   ```js
   export default {
    data() {
        return {searchQuery: ''};
    },
    watch: {
        searchQuery: {
            handler(newVal, oldVal) {
                this.fetchResults(newVal);
            },
            immediate: true,
            deep: false
        }
    },
    methods: {
        fetchResults(q) {
            // API‑запрос по q
        }
    }
}
   ```

### 5. **`lifecycle hooks`** – ключевые этапы: `beforeCreate`, `created`, `beforeMount`, `mounted`, `beforeUpdate`,
   `updated`, `beforeUnmount`, `unmounted`;

Позволяют выполнять код на разных стадиях: инициализация, добавление в DOM, обновление, удаление.

   ```js
   export default {
    created() {
        console.log('Component instance created, data ready');
    },
    mounted() {
        console.log('Component mounted to DOM');
    },
    beforeUnmount() {
        console.log('Cleanup before destroy');
    }
}
   ```

### 6. **Шаблон и директивы** – декларативный HTML с привязками:

    * `v-if` / `v-else`
    * `v-for="item in items" :key="item.id"`
    * `v-model` для двусторонней привязки
    * `@click="handler"` (сокращение для `v-on:click`)
    * `:name="value"` (сокращение для `v-bind:name`)

   директивы преобразуют атрибуты в реактивные слушатели и привязки. `@` и `:` — синтаксический сахар.

   ```html
   <template>
     <button @click="increment">+1</button>
     <input v-model="count" type="number" />
     <child-component :name="user.name" @custom-event="onCustomEvent" />
   </template>
   ```

### 7. **Передача данных между компонентами**

    * **Родитель → Дочерний**:

      ```html
      <Child :msg="parentMsg" @reply="handleReply" />
      ```
    * **В дочернем компоненте** объявляем `props` и эмитим событие:

      ```js
      export default {
        props: ['msg'],      // принимает
        methods: {
          reply() {          // отправляет
            this.$emit('reply', { from: 'child' });
          }
        }
      }
      ```

### 8. **`$emit`** – отправка событий из дочернего в родительский компонент;

В дочернем компоненте вызывают `this.$emit('eventName', payload)`. В родителе ловят через атрибут
`@event-name="handler"`.

   ```js
   // Child.vue
export default {
    methods: {
        notifyParent() {
            this.$emit('update', {count: this.count});
        }
    }
}
   ```

   ```html
   <!-- Parent.vue -->
<Child @update="onChildUpdate"/>
   ```

   ```js
   methods: {
    onChildUpdate(payload)
    {
        console.log('From child:', payload);
    }
}
   ```

### 9. **Пример компонента**

   ```html
   <!-- Parent.vue -->
   <template>
     <div>
       <Child :msg="parentMsg" @reply="handleReply" />
     </div>
   </template>
   <script>
   import Child from './Child.vue';
   export default {
     components: { Child },
     data() { return { parentMsg: 'Hello Child' }; },
     methods: {
       handleReply(data) {
         console.log('От ребенка:', data);
       }
     }
   };
   </script>

   <!-- Child.vue -->
   <template>
     <div>
       <p>Message: {{ msg }}</p>
       <button @click="reply()">Reply</button>
     </div>
   </template>
   <script>
   export default {
     props: ['msg'],
     methods: {
       reply() { this.$emit('reply', { from: 'child' }); }
     }
   };
   </script>
   ```

### 10. **Управление состоянием (State management)**

* **MobX (с mobx-vue-lite)**

  ```js
  import { reactive } from 'vue';
  import { useLocalObservable } from 'mobx-vue-lite';

  export default {
    setup() {
      const store = useLocalObservable(() => ({
        count: 0,
        increment() { this.count++; }
      }));
      return { store };
    }
  };
  ```

* **Redux (с Redux Toolkit)**

  ```js
  // store.js
  import { configureStore, createSlice } from '@reduxjs/toolkit';
  const counter = createSlice({
    name: 'counter',
    initialState: 0,
    reducers: { increment: state => state + 1 }
  });
  export const { increment } = counter.actions;
  export default configureStore({ reducer: counter.reducer });
  ```

  ```js
  // main.js
  import { createApp } from 'vue';
  import { createRedux } from 'vue-redux';
  import store from './store';

  const app = createApp(App);
  app.use(createRedux(store));
  app.mount('#app');
  ```

### 11. **Окружение и библиотеки**

- **Сборка и CLI**
    - Vite (рекомендован)
    - Vue CLI
- **Роутинг**
    - Vue Router
- **State‑менеджмент**
    - Pinia (новый рекомендованный)
    - Vuex
- **SSR / Meta‑framework**
    - Nuxt.js
- **UI‑фреймворки**
    - Vuetify
    - Element Plus
    - BootstrapVue
- **Формы и валидация**
    - VeeValidate
    - VueUse (набор утилит)

---

## React (Functional Components)

### Особенности

- **JSX‑синтаксис**: компоненты описываются в JavaScript, нет отдельного шаблона
- **Virtual DOM** и диффинг для минимального обновления реального DOM
- **Хуки** (`useState`, `useEffect`, `useContext` и т. д.) вместо классовых компонентов
- **Однонаправленный поток данных**: `props` вниз, события/колбэки вверх
- **Обширная экосистема**: рендеринг на сервере (Next.js), мобильные (React Native), статическую генерацию
- **Функциональный стиль** и композиция логики через кастом‑хуки

### 1. **`useState`** – локальный стейт и его обновление;

`const [value, setValue] = useState(init)`. `setValue` может принимать функцию-обновление, основанную на предыдущем
стейте.

   ```jsx
   function Counter() {
    const [count, setCount] = useState(0);
    const increment = () => setCount(prev => prev + 1);
    return <button onClick={increment}>{count}</button>;
}
   ```

### 2. **`useEffect`** – побочные эффекты после рендера;

эффекты запускаются после отрисовки. Очистка (`return`) вызывается перед следующим эффектом или размонтированием.

   ```jsx
   useEffect(() => {
    document.title = `Count: ${count}`;
    return () => console.log('Cleanup');
}, [count]);
   ```

### 3. **`props`** – неизменяемые входные данные;

передавайте примитивы и объекты, деструктурируйте в аргументах. Для валидации используйте PropTypes или TypeScript.

   ```jsx
   function Greeting({name = 'Guest'}) {
    return <h1>Hello, {name}</h1>;
}
   ```

### 4. **События** (`onClick`, `onChange`);

обработчики получают синтетическое событие. Используйте `e.preventDefault()` и `e.stopPropagation()` для контроля.

   ```jsx
   <form onSubmit={e => {
    e.preventDefault();
    submit();
}}>...</form>
   ```

### 5. **JSX** – разметка внутри JavaScript;

допускаются выражения `{}` и условный рендер через `&&` или тернарный оператор. Нельзя возвращать несколько корневых
узлов без фрагмента.

   ```jsx
   return (
    <>
        {items.length ? items.map(...) : <p>No items</p>}
    </>
);
   ```

### 6. **Передача данных между компонентами**

    * **Родитель → Дочерний:**

      ```jsx
      <Child msg={parentMsg} onReply={handleReply} />
      ```
    * **В дочернем:**

      ```jsx
      function Child({ msg, onReply }) {
        // принимает msg, чтобы отправить – вызывает onReply(...)
      }
      ```

### 7. **Пример компонента**

   ```jsx
   // Parent.jsx
   import React, { useState } from 'react';
   import Child from './Child';
   export default function Parent() {
     const [parentMsg] = useState('Hello Child');
     const handleReply = data => console.log('От ребенка:', data);
     return <Child msg={parentMsg} onReply={handleReply} />;
   }

   // Child.jsx
   import React from 'react';
   export default function Child({ msg, onReply }) {
     return (
       <div>
         <p>Message: {msg}</p>
         <button onClick={() => onReply({ from: 'child' })}>
           Reply
         </button>
       </div>
     );
   }
   ```

### 8. **Управление состоянием (State management)**

* **MobX (с mobx-react-lite)**

  ```jsx
  import { makeAutoObservable } from 'mobx';
  import { observer } from 'mobx-react-lite';

  class Store {
    count = 0;
    constructor() { makeAutoObservable(this); }
    increment() { this.count++; }
  }
  const store = new Store();

  const Counter = observer(() => (
    <button onClick={() => store.increment()}>{store.count}</button>
  ));
  ```

* **Redux (с Redux Toolkit)**

  ```js
  // store.js
  import { configureStore, createSlice } from '@reduxjs/toolkit';
  const counterSlice = createSlice({
    name: 'counter',
    initialState: { value: 0 },
    reducers: { increment: state => { state.value++ } }
  });
  export const { increment } = counterSlice.actions;
  export const store = configureStore({ reducer: { counter: counterSlice.reducer } });
  ```

  ```jsx
  // App.jsx
  import { Provider, useSelector, useDispatch } from 'react-redux';

  function Counter() {
    const count = useSelector(state => state.counter.value);
    const dispatch = useDispatch();
    return <button onClick={() => dispatch(increment())}>{count}</button>;
  }

  export default function App() {
    return (
      <Provider store={store}>
        <Counter />
      </Provider>
    );
  }
  ```

### 9. **Окружение и библиотеки**

- **Сборка и стартовые шаблоны**
    - Create React App
    - Vite (React шаблон)
- **Роутинг**
    - React Router
- **State‑менеджмент**
    - Redux (+ Redux Toolkit)
    - MobX
    - Recoil
- **SSR / Meta‑framework**
    - Next.js
    - Gatsby
- **UI‑библиотеки**
    - Material‑UI (MUI)
    - Ant Design
    - Chakra UI
    - Semantic UI React
- **Запросы и кеширование**
    - React Query
    - SWR
- **Формы и валидация**
    - Formik
    - React Hook Form
- **Утилиты и анимации**
    - Framer Motion
    - Styled‑Components / Emotion

---

## React Native (Functional Components)

### Особенности

- **Нативные UI‑компоненты** (не веб‑элементы) под iOS и Android
- **Единый код на JS/JSX** кроссплатформенный, с возможностью платформо‑специфичных файлов
- **Стили в JS** через объекты и Flexbox вместо CSS
- **Мост («bridge»)** между JS‑потоком и нативным, влияет на производительность и задержки
- **Поддержка жестов и анимаций** через `react-native-gesture-handler` и `reanimated`
- **Доступ к нативным API** (камера, геолокация) через модули-плагины

### 1. **Основные компоненты** – `View`, `Text`, `Image`, `ScrollView`;

Стили создаются через `StyleSheet.create({ ... })`. Нет HTML – только собственные компоненты.

   ```jsx
   import {View, Text, Image, StyleSheet} from 'react-native';

function Profile() {
    return (
        <View style={styles.container}>
            <Image source={{uri: 'https://...'}} style={styles.avatar}/>
            <Text style={styles.name}>User Name</Text>
        </View>
    );
}

const styles = StyleSheet.create({
    container: {alignItems: 'center', padding: 16},
    avatar: {width: 100, height: 100, borderRadius: 50},
    name: {fontSize: 18, marginTop: 8}
});
   ```

### 2. **`useState`, `useEffect`** – те же хуки, но эффекты могут слушать системные события;

Часто используют для регистрации `BackHandler` или подписок на push‑уведомления.

### 3. **Навигация** – `react-navigation`:

   ```jsx
   import { NavigationContainer } from '@react-navigation/native';
   import { createStackNavigator } from '@react-navigation/stack';
   const Stack = createStackNavigator();

   function App() {
     return (
       <NavigationContainer>
         <Stack.Navigator>
           <Stack.Screen name="Home" component={HomeScreen} />
           <Stack.Screen name="Profile" component={ProfileScreen} />
         </Stack.Navigator>
       </NavigationContainer>
     );
   }
   ```

### 4. **Пример компонента**

   ```jsx
   // Parent.jsx
   import React, { useState } from 'react';
   import { View } from 'react-native';
   import Child from './Child';

   export default function Parent() {
     const [parentMsg] = useState('Hello Mobile Child');
     const handleReply = data => console.log('От ребенка:', data);
     return (
       <View>
         <Child msg={parentMsg} onReply={handleReply} />
       </View>
     );
   }

   // Child.jsx
   import React from 'react';
   import { View, Text, Button } from 'react-native';

   export default function Child({ msg, onReply }) {
     return (
       <View>
         <Text>Message: {msg}</Text>
         <Button title="Reply" onPress={() => onReply({ from: 'child' })} />
       </View>
     );
   }
   ```

### 5. **Flexbox** – основной способ верстки;

`flexDirection`, `justifyContent`, `alignItems`, `flex`.

### 6. **Платформенные API** – `Platform`, `Dimensions`;

Условный рендер под iOS/Android, получение размеров экрана.

### 7. **Жесты** – `react-native-gesture-handler`, `react-native-reanimated`;

Для сложных анимаций и взаимодействий.

### 8. **Управление состоянием (State management)**

* **MobX**

  ```jsx
  import { makeAutoObservable } from 'mobx';
  import { observer } from 'mobx-react-lite';

  class Store {
    count = 0;
    constructor() { makeAutoObservable(this); }
    increment() { this.count++; }
  }
  const store = new Store();

  const Counter = observer(() => (
    <View><Text onPress={() => store.increment()}>{store.count}</Text></View>
  ));
  ```

* **Redux (Redux Toolkit)**
  *(аналогично React — настройка `configureStore`, оборачивание `<Provider>` и использование `useSelector`/`useDispatch`
  в компонентах.)*

### 9. **Стилизация**

- **`StyleSheet.create`**
    - Основной способ: создаёте объект стилей и ссылаетесь по ключу.
    - Оптимизация: StyleSheet собирает неизменяемые стили в отдельный пул.
   ```jsx
   import { StyleSheet, View, Text } from 'react-native';

   const styles = StyleSheet.create({
     container: {
       flex: 1,
       padding: 16,
       backgroundColor: '#fff',
     },
     title: {
       fontSize: 24,
       fontWeight: 'bold',
       marginBottom: 8,
     },
     subtitle: {
       fontSize: 16,
       color: '#666',
     },
   });

   export default function MyComponent() {
     return (
       <View style={styles.container}>
         <Text style={styles.title}>Заголовок</Text>
         <Text style={styles.subtitle}>Подзаголовок</Text>
       </View>
     );
   }

- **Inline‑стили**

    * Быстрое переопределение или динамические значения.
    * Не так оптимизированы, выполняются при каждом рендере.

   ```jsx
   <View style={{ margin: 10, backgroundColor: isActive ? 'green' : 'gray' }}>
     <Text style={{ color: '#fff' }}>Inline Styled</Text>
   </View>
   ```

- **Композиция стилей**

    * Можно передавать массив: стили применяются по порядку, последний перекрывает предыдущие.

   ```jsx
   <Text style={[styles.baseText, isError && styles.errorText]}>
     Сообщение
   </Text>

   // в StyleSheet:
   baseText: { fontSize: 16 },
   errorText: { color: 'red' },
   ```

- **Псевдоклассы / активные состояния**

    * Для `Touchable*` компонентов можно менять стили через `activeOpacity` или `underlayColor`.
    * Для более сложных эффектов — использовать `Pressable` с функцией для `style`:

   ```jsx
   import { Pressable, Text } from 'react-native';

   <Pressable
     style={({ pressed }) => [
       styles.button,
       pressed && styles.buttonPressed
     ]}
     onPress={onPress}
   >
     <Text style={styles.buttonText}>Press Me</Text>
   </Pressable>

   // styles:
   button: { padding: 12, backgroundColor: '#0066cc' },
   buttonPressed: { backgroundColor: '#005bb5' },
   buttonText: { color: '#fff', textAlign: 'center' },
   ```

- **Тема и глобальные стили**

    * Можно вынести общие цвета, размеры и шрифты в отдельный объект и импортировать в компоненты:

   ```js
   // theme.js
   export const theme = {
     colors: {
       primary: '#0066cc',
       background: '#fff',
       text: '#333',
     },
     spacing: {
       s: 8,
       m: 16,
       l: 24,
     },
   };
   ```

   ```jsx
   import { theme } from './theme';
   const styles = StyleSheet.create({
     container: { flex: 1, padding: theme.spacing.m, backgroundColor: theme.colors.background },
     text: { color: theme.colors.text },
   });
   ```

- **Адаптивная верстка**

    * Реагировать на размеры экрана через `Dimensions` или хук `useWindowDimensions`.
    * Можно менять стили динамически:

   ```jsx
   import { useWindowDimensions, StyleSheet } from 'react-native';

   export default function Responsive() {
     const { width } = useWindowDimensions();
     const isLarge = width > 600;
     const styles = StyleSheet.create({
       text: {
         fontSize: isLarge ? 24 : 16,
       },
     });
     return <Text style={styles.text}>Responsive Text</Text>;
   }
   ```

---

### 10. **Окружение и библиотеки**

- **Сборка и рантайм**
    - Expo (managed / bare workflow)
    - React Native CLI
- **Навигация**
    - React Navigation
    - React Native Navigation (Wix)
- **State‑менеджмент**
    - Redux (+ Redux Toolkit)
    - MobX
    - Recoil
- **UI‑библиотеки**
    - React Native Elements
    - NativeBase
    - React Native Paper
- **Анимации и жесты**
    - react-native-reanimated
    - react-native-gesture-handler
- **Формы и валидация**
    - Formik
    - React Hook Form (+ native adapters)
- **Нативные API / Хранилище**
    - @react-native-async-storage/async-storage
    - Expo‑модули (Camera, Location, SecureStore и т. д.)  
