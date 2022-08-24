# Context

Es frecuente tener algunos _state_ compartidor por partes totalmente separadas
de nuestra aplicación. Algunos ejemplos típicos son el usuario conectado en
un determinado momento, o el modal o diálogo que queremos mostrar desde
cualquier sitio. En estos casos, pasarlo como _props_ por toda la aplicación
puede resultar demasiado engorroso. Para evitarlo, tenemos la opción de usar
el contexto.

## Usando context

Para crear un contexto utilizamos `createContext`, y podemos pasarle un valor
inicial si es necesario. El valor que nos devuelve incluye un campo `Provider`
que podemos utilizar para introducir un valor al _context_. Por último, para
acceder al valor del context utilizamos `useContext`, pasándole el contexto
al que queremos acceder.

Es muy frecuente crear un _provider_ para nuestro contexto que integre el
_state_ donde almacenaremos el dato, con el _Context.Provider_ que lo haga
llegar a donde haga falta, y un hook custom para acceder al contexto más
fácilmente. Usando estas dos herramientas, podemos _ocultar_ por completo el
uso de context, facilitando el uso sin preocuparse de la implementación.

```jsx
import { createContext, useContext, useState } from 'react'

const UserContext = createContext()

export function UserProvider({ children }) {
  const [user, setUser] = useState(null)
  return (
    <UserContext.Provider value={[user, setUser]}>
      {children}
    </UserContext.Provider>
  )
}

export const useUser = () => useContext(UserContext)
```

## Token

El uso de un context con los datos del usuario actual, como en el ejemplo
anterior, es una herramienta muy cómoda para hacer sencillo acceder al token
de sesión desde cualquier sitio. Si estamos haciendo uso de un hook tipo
`useFetch`, este hook puede acceder al contexto para leer al token sin tener
que programar nada en cada lugar que se use.

```jsx
function useFetch(url) {
  const [data, setData] = useState()
  const [user] = useUser()
  useEffect(() => {
    const opts = {}
    if (user?.token) opts.headers = { 'Authorization': 'Bearer ' + user.token }
    fetch(url, opts)
      .then(res => res.json())
      .then(json => setData(json))
  }, [url, user])
  return data
}
```

Para las llamadas desde callbacks no es posible hacerlo de este modo, porque no
podemos llamar al hook de context desde el event handler. Sería necesario
llamarlo antes de definir el handler, y utilizar el valor desde el handler.
Esto puede encapsularse a través de una función de orden superior, que nos
devuelva la función a utilizar desde el handler:

```jsx
function useCreatePost() {
  const [user] = useUser()
  return (post) => {
    const opts = {}
    if (user?.token) opts.headers = { 'Authorization': 'Bearer ' + user.token }
    return fetch('...', opts)
      .then(res => res.json())
  }
}

/*
Uso:
  const post = useCreatePost()
  const handleSubmit = async e => {
    e.preventDefault()
    const resultado = await post(myPost)
  }
*/
```

## A continuación...

El _context_ es una herramienta muy potente, aunque algo compleja de seguir en
una aplicación grande, especialmente si aparecen muchos contextos diferentes.
En estos casos puede ser mejor opción utilizar una herramienta para la gestión
del estado, como puede ser [Redux](./11-redux.md).
