# studio-tools
> A set of sui-studio usable tools.

```sh
$ npm install @s-ui/studio-tools --save
```

# Domain builder

> Domain builder has the purpose of giving to you the possibility of mocking some of non implemented domain use cases meanwhile your team are developing them.

# How it works

### Base initialization:

```js
import { DomainBuilder } from '@s-ui/studio-tools'
import myDomain from 'domain'


const domain = DomainBuilder.extend({ myDomain }).build()

```

### Mocking use cases

> To mock an use case you need to call two functions: 'for' and 'respondWith'
>
> Let's suppose that we want to mock an use case that isn't already implemented. Its name is 'get_products':

```js
import { DomainBuilder } from '@s-ui/studio-tools'
import myDomain from 'domain'

const getProductsResponse = {
  success: ['pineapple', 'apple', 'strawberry', 'coffee']
}
const domain = DomainBuilder.extend({ myDomain }).for({useCase: 'get_products'}).respondWith(getProductsResponse).build()


// Execute the use case and check if everything works
domain.get('current_user_use_case').execute().then((products) => {
  console.log(products) // ['pineapple', 'apple', 'strawberry', 'coffee']
})
```


### Mocking the configuration

```js
DomainBuilder.extend(
  {
    domain,
    config: 'mocked-config'
  })
```

### Forcing an error throw

```js
import { DomainBuilder } from '@s-ui/studio-tools'
import myDomain from 'domain'

const getProductsError = {
  fail: 'Unexpected error :('
}
const domain = DomainBuilder.extend({ myDomain }).for({useCase: 'get_products'}).respondWith(getProductsError).build()


// Execute the use case and check if everything works
domain.get('current_user_use_case').execute().then((products) => {
  // Never will be fired
}).catch((e) => {
  console.log(e) // Unexpected error :(
})
```

### Using Sinon spies for testing

> You can use Sinon spies to test that your use cases are being called correctly and verify their behavior. This is particularly useful when you want to mock a response while also tracking how the function was called.

```js
import { DomainBuilder } from '@s-ui/studio-tools'
import sinon from 'sinon'
import myDomain from 'domain'

// Create a spy that returns a specific response
const spy = sinon.spy(() => {
  return 'spied-response'
})

// Build the domain with the spied use case
const domain = DomainBuilder
  .extend({ domain: myDomain })
  .for({ useCase: 'get_products' })
  .respondWith({ success: spy })
  .build()

// Execute the use case
domain.get('get_products').execute().then((result) => {
  console.log(result) // 'spied-response'
  
  // Verify the spy was called
  sinon.assert.calledOnce(spy)
  
  // You can also check call arguments, call count, etc.
  console.log(spy.callCount) // 1
  console.log(spy.firstCall.args) // Arguments passed to the spy
})
```

This approach allows you to:
- Track how many times a use case was called
- Verify the arguments passed to the use case
- Assert on the call order when multiple use cases are involved
- Combine mocking with behavior verification in your tests

For a complete example, see the [test implementation](https://github.com/SUI-Components/sui/blob/main/packages/sui-studio-utils/src/test/common/domainBuilderSpec.js).



> THINGS TO KEEP IN MIND: 

> You CAN'T mock a use case if already exists on the domain. This means that we can ONLY mock use cases that doesn't exist on the domain 



# I18N

> Function with the purpose of set our locales on rosseta.

# How it works
The library accepts two types of flow
1. Usecase given translations.
2. Object given translations.

### Usecase given translations:

The locales are getted using a usecase of a domain. You pass the usecase not the domain.

```js
import { i18n } from '@s-ui/studio-tools'
import myDomain from 'domain'


i18n({ literalsUseCase: myDomain.get('get_literals_from_backend') }).then((rossetaInstance) => {
  rossetaInstance.t('myLocaleName');
})



```

### Object given translations:
 
The locales are getted by an object argument. No call is done to any use case.

Dictionary should be formed with this format:

```js
    'es-ES': { // Or your iso language
      'LOGIN': 'INICIAR SESIÓN', // your locale names
      'SIGNUP': 'CREAR UNA CUENTA'
    }
```

```js
import { i18n } from '@s-ui/studio-tools'
import myDomain from 'domain'
import myLocalesDictionary from '../../utils/dictionary' // Or wherever you have your locales object.

i18n({ dictionary: myLocalesDictionary }).then((rossetaInstance) => {
  rossetaInstance.t('myLocaleName');
})

```

#### IN BOTH CASES the function returns a PROMISE.


## Configuration

The function comes with a little customization feature. You can customizate:
1. Culture - default es-ES
2. Currency - default EUR


Send custom config is as easy as put a config property on your object arguments:

```js
import { i18n } from '@s-ui/studio-tools'
import myDomain from 'domain'
import myLocalesDictionary from '../../utils/dictionary' // Or wherever you have your locales object.

i18n({
        dictionary: myLocalesDictionary,
        config: {
          currency: 'EUR',
          culture: 'es-ES'
        }
}).then((rossetaInstance) => {
  rossetaInstance.t('myLocaleName');
})

```


