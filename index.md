![Cover](./img/MI19_COVERslide_16-9.png)


![Title](./img/MI19_COVERslide_16-9_title.png)


# @eamodeorubio <!-- .element: style="color: yellow;text-transform: none" -->

<a href="https://www.contentful.com/" rel="nofollow" target="_blank"><img src="img/contentful.svg" style="max-width:600px;border:none;background:none;" alt="Powered by Contentful" /></a>


# Typing like a <span style="color: #294E80">Pro</span>
### (<span style="font-weight: lighter;">with</span> <img class="ts" src="./img/ts-logo-white.svg"/>)


## Why a Type System?


### Not only IDE help

![IDE Help](./img/ide-help.webp)


### Also Correctness
#### (build/assign the right data)

```typescript
              const john = {                     const john: PersonalInfo = {
                name: 'John',                      name: 'John',
                age: -30,             VS.          age: 30,
                heigth: 176                        height: 176
              }                                  }
```


### More important, readability
#### (express what your code expects)

```javascript
// How do I use this method?
function savePersonalInfo(personalInfo)
```

VS.

```typescript
interface PersonalInfo {
  name: string
  age: number
  height: number
}

function savePersonalInfo(data: PersonalInfo): Promise<void>
```


#### The more readable, the less boilerplate docs we need to write
#### Focus effort on <span class="good">high value docs</span> (use cases, best practices, design, etc.)


### Also removing boilerplate tests
#### Type checker does those for you

```javascript
describe('savePersonalInfo', () => {
  // Boilerplate, low value
  describe('validates input', () => {
    // check null, fields, etc...
  })
  it('returns a promise', () => {
    // Really?
  })
  describe('saves the data!', () => {
    //...Now I can write the tests that I'm interested in
  })
})
```


### Types gives a <em><span class="good">faster</span></em> feedback loop!
#### By detecting <span class="bad">silly mistakes</span> <em>as you type</em>


### Types gives a <em><span class="good">better DX</span></em>!
#### By <em>removing</em> the need of some <span class="bad">low value</span> tests
#### making the code much more <span class="good">boilerplate docs</span>


## A trading bot!

![Trading bot](./img/trading-bot.gif)


### Order Lifecycle

![Order lifecycle](./img/order-lifecycle.png)


### <em>Purchase</em> & <em>Sell</em> orders are different
<!-- ### Orders

* The market takes time to trade orders
* Some orders can be `failed` because:
  * The trade was not done after some specified time
  * They contained some error, like bad stock symbol
* Keep track of the "broker" `orderId` for further reference
* Two kinds: <em>purchase</em> & <em>sell</em> -->


### Purchase order
#### Creation

* Which `stock` to buy
* Maximum price the bot is willing to pay
* Maximum quantity the bot is willing to buy


### Purchase order
#### Traded

* What was the actual purchase price
* How many shares were actually puchased


### Sell order
#### Creation

* Which `stock` to sell
* Minimum sell price
* Minimum quantity the bot is willing to sell


### Sell order
#### Traded

* What was the actual sell price
* How many shares were actually sold


### What's in an order?

| Type     | Created | Pending | Failed | Traded |
| ---:      | :---:     | :---:    | :---:    | :---: |
| Purchase | `stock`, `maxPrice`, `maxQuantity` | +`brokerId` | +`error` | +`puchasePrice`, +`unitsPurchased` |
| Sell | `stock`, `minPrice`, `minQuantity` | +`brokerId` | +`error` | +`sellPrice`, +`unitsSold` |


## Naïve approach


### Start with a purchase
#### Add data from <em>created</em> state

```typescript
interface PurchaseOrder {
  stock: StockSymbol
  maxPrice: Money
  maxQuantity: Money
}
```


### Add data from the <span style="color: yellow">pending</span> state

```typescript
interface PurchaseOrder {
  stock: StockSymbol
  maxPrice: Money
  maxQuantity: Natural
  // For pending
  brokerId: ID
}
```


### There is also an <span class="bad">error</span> state

```typescript
interface PurchaseOrder {
  stock: StockSymbol
  maxPrice: Money
  maxQuantity: Natural
  // For pending
  brokerId: ID
  // For failed
  errorMsg: string
}
```


### And <span class="good">traded</span>💸

```typescript
interface PurchaseOrder {
  stock: StockSymbol
  maxPrice: Money
  maxQuantity: Natural
  // For pending
  brokerId: ID
  // For failed
  errorMsg: string
  // For traded
  purchasePrice: Money
  unitsPurchased: Natural
}
```


### Orders are in one state at a time

```typescript
interface PurchaseOrder {
  stock: StockSymbol
  maxPrice: Money
  maxQuantity: Natural
  // For pending
  brokerId?: ID
  // For failed
  errorMsg?: string
  // For traded
  purchasePrice?: Money
  unitsPurchased?: Natural
}
```


### Which state is the order in?

```typescript
enum OrderStatus {
  CREATED,
  PENDING,
  FAILED,
  TRADED
}
```


### Which state is the order in?

```typescript
interface PurchaseOrder {
  status: OrderStatus
  stock: StockSymbol
  maxPrice: Money
  maxQuantity: Natural
  // For pending
  brokerId?: ID
  // For failed
  errorMsg?: string
  // For traded
  purchasePrice?: Money
  unitsPurchased?: Natural
}
```


### But there can be two kind of orders

```typescript
enum OrderKind {
  PURCHASE,
  SELL
}
```


### Purchase and sell orders have different data for some states

```typescript
interface Order {
  kind: OrderKind
  status: OrderStatus
  stock: StockSymbol
  minPrice?: Money
  maxPrice?: Money
  minQuantity?: Natural
  maxQuantity?: Natural
  brokerId?: ID
  errorMsg?: string
  sellPrice?: Money
  purchasePrice?: Money
  unitsPurchased?: Natural
  unitsSold?: Natural
}
```


### Non-sense order

```typescript
// No compiler complains
// Do you get some IDE help here?
const nonSensicalOrder: Order = {
  kind: OrderKind.PURCHASE,
  status: OrderStatus.TRADED,
  stock: 'CodeMotion',
  errorMsg: 'not traded',
  purchasePrice: 300,
  unitsSold: 49
}
```


### This is the help from your IDE

![No IDE help](./img/no-ide-help.webp)


### Same, same with broker messages

```typescript
interface BrokerMsg {
  brokerId: ID
  errorMsg?: string
  sellPrice?: Money
  purchasePrice?: Money
  unitsPurchased?: Natural
  unitsSold?: Natural
}
```


### Order updates

```typescript
// Business logic
declare function updateOrder(current: Order, msg: BrokerMsg): Order
```


### Non-sense update

```typescript
const whoKnowsWhatIsThis: Order = updateOrder(
  nonSensicalOrder,
  { brokerId: '1', sellPrice: 1000 } // Non-sensical broker msg
)
// What we just did here?
```


### Another non-sense update

```typescript
// Arguments correct, but:
// * Cannot trade a non-pending order! It is not yet in the broker
// * Cannot apply a successful sell to a purchase order
const whoKnowsWhatIsThis: Order = updateOrder(
  aCorrectJustCreatedPurchaseOrder,
  aCorrectSuccessfulSellBrokerMsg
)
// Should we throw or what?
```


### A correct update... 
#### ...but we don't know anything of the result!

```typescript
// Whe know it's a successful purchase by the name.
// But the compiler does not know and cannot help us!
const successfulPurchase: Order = updateOrder(
  aCorrectPendingPurchaseOrder,
  aCorrectSuccessfulPurchaseBrokerMsg
)
```


### Is <img class="ts" src="./img/ts-logo-white.svg"/> useful after all?

```javascript
// Is it much more safer than plain JS?
// Note that plain JS is cheaper, no need to write types!
const nonSensicalOrder = {
  kind: PURCHASE,
  status: TRADED,
  stock: 'CodeMotion',
  errorMsg: 'not traded',
  purchasePrice: 300,
  unitsSold: 49
}

const nonSensicalBrokerMsg = { brokerId: '1', sellPrice: 1000 }

const whoKnowsWhatIsThis = updateOrder(nonSensicalOrder, nonSensicalBrokerMsg)
```


# 🤔 💣
# 🙈 😕


## Leveraging <img class="ts" src="./img/ts-logo-white.svg"/> type system
#### (Use <img class="ts" src="./img/ts-logo-white.svg"/> like a <span style="color: #294E80">Pro</span>)


### Discrimated unions
#### Example: Result type

```typescript
interface Failure {
  ok: false // Single values are also types!
  error: string
}

interface Ok {
  ok: true
  data: any
}

type Result = Ok | Failure
```


### Discrimated unions
#### Type refinement

```typescript
declare function update(sql: UpdateSQL): Result;

const updated = update(makUsersAbove18AsLegalAge);

if(updated.ok) {
  // Here TS knows that updated.result exists
  if(updated.data > 0) { // Fails, updated.data type is any!
    // Even if it were not to fail
    // ok.data may not be a number as we thought
    console.log(`Updated ${ok.data} users`)
  } else {
    console.log('No rows updated')
  }
} else {
  // Here TS knows that updated.error exists and it's a string
  console.log(`Something went wrong: ${ok.error}`)
}
```


### Parametric Types (a.k.a. Generics)
#### Functions that take types and return types

```typescript
// "Ok" now takes another type R and returns a new Type
interface Ok<R> {
  ok: true
  data: R
}

// So "Result" must take a parameter type also
type Result<R> = Ok<R> | Failure
```


### Parametric Types (a.k.a. Generics)

```typescript
declare function update(sql: UpdateSQL): Result<number>;

const updated = update(makUsersAbove18AsLegalAge)

// All safe now, the IDE will help us also
if(updated.ok) {
  if(updated.data > 0) {
    console.log(`Updated ${ok.data} users`)
  } else {
    console.log('No rows updated')
  }
} else {
  console.log(`Something went wrong: ${ok.error}`)
}
```


### Conditional Types
#### If this <span style="color: yellow">type</span> then that <span style="color: green">type</span>

```typescript
enum EventKind {
  LOGIN,
  LOGIN_SUCCESS,
  // More types...
}

interface LoginPayload {
  username: string
  password: string
}

interface LoginSuccessPayload {
  token: string
}
```


### Conditional Types
#### If this <span style="color: yellow">type</span> then that <span style="color: green">type</span>

```typescript
type EventPayload<K extends EventKind> =
  K extends EventKind.LOGIN ? LoginPayload :
  K extends EventKind.LOGIN_SUCCESS ? LoginSuccessPayload :
  // ... K extends EventKind.OTHER_KIND ? OtherKindPayload :
  never

type Event<K extends EventKind> = { kind: K } & EventPayload<K>  

declare function dispatch<K extends EventKind>(
  eventKind: K,
  payload: EventPayload<K>
): Promise<void>;

// Safe & IDE help
dispatch(EventKind.LOGIN_SUCCESS, {
  token: '8012466df09f6c6a7a6a2b7df41e355489dff0a3'
})
```


### Back to the orders!

* The data in an order depends on two things
  * The order kind (sell vs. purchase) determines initial details
  * The status (created, pending, failed, traded) determines the current state data


### Order(Kind, Status) = Details(Kind) + State(Status)

```typescript
type Order<K extends OrderKind, S extends OrderStatus> = {
  details: OrderDetails<K>
  state: OrderState<S> 
}
```


### Details(Kind)

```typescript
interface PurchaseOrderDetails {
  kind: OrderKind.PURCHASE
  stock: StockSymbol
  maxPrice: Money
  maxQuantity: Natural
}

interface SellOrderDetails {
  kind: OrderKind.SELL
  stock: StockSymbol
  minPrice: Money
  minQuantity: Natural
}

type OrderDetails<O extends OrderKind> =
  O extends OrderKind.PURCHASE ? PurchaseOrderDetails :
  O extends OrderKind.SELL ? SellOrderDetails :
  never
```


### State(Status)

```typescript
interface OrderCreated {
  status: OrderStatus.CREATED
}
interface OrderPending {
  status: OrderStatus.PENDING
  brokerId: ID
}
interface OrderFailed {
  status: OrderStatus.FAILED
  brokerId: ID
  errorMsg: string
}
interface OrderTraded {
  status: OrderStatus.TRADED
  brokerId: ID
  sellPrice?: Money // Ooops, those `?` again!!
  purchasePrice?: Money
  unitsPurchased?: Natural
  unitsSold?: Natural
}
```


### State(Status, Kind)
#### Better

```typescript
interface PurchaseOrderTraded {
  status: OrderStatus.TRADED
  brokerId: ID
  purchasePrice: Money
  unitsPurchased: Natural
}

interface SellOrderTraded {
  status: OrderStatus.TRADED
  brokerId: ID
  sellPrice: Money
  unitsSold: Natural
}

type OrderTraded<K extends OrderKind> =
  K extends OrderKind.PURCHASE ? PurchaseOrderTraded :
  K extends OrderKind.SELL ? SellOrderTraded :
  never
```


### Order(Kind, Status) = Details(Kind) + State(Status, Kind)

```typescript
type OrderState<S extends OrderStatus, K extends OrderKind> =
  S extends OrderStatus.CREATED ? OrderCreated :
  S extends OrderStatus.PENDING ? OrderPending :
  S extends OrderStatus.FAILED ? OrderFailed :
  S extends OrderStatus.TRADED ? OrderTraded<K> :
  never

type Order<K extends OrderKind, S extends OrderStatus> = {
  details: OrderDetails<K>
  state: OrderState<S, K> 
}
```


### Broker messages

```typescript
interface Acked {
  brokerId: ID
}
interface Failed {
  errorMsg: ID
}
interface Purchased {
  purchasePrice: Money
  unitsPurchased: Natural
}
interface Sold {
  sellPrice: Money
  unitsSold: Natural
}
type Traded<K extends OrderKind> = 
  K extends OrderKind.PURCHASE ? Purchased :
  K extends OrderKind.SELL ? Sold :
  never
type BrokerEvent<K extends OrderKind> = Acked | Failed | Traded<K>
```


## But now, broker messages and states have duplicated fields
### Need to refactor somehow 🤔


### DRY Messages & States
#### Intersection types!

```typescript
type OrderPending =
  { status: OrderStatus.PENDING } & Acked

type OrderFailed =
  { status: OrderStatus.FAILED } & Acked & Failed

// We can even remove PurchaseOrderTraded and SellOrderTraded now!
type OrderTraded<K extends OrderKind> =
  { status: OrderStatus.TRADED } & Acked & Traded<K>
```


### Creating orders

```typescript
// Type safe, IDE help, non-verbose factory functions
declare function createOrder<
  K extends OrderKind,
  D extends OrderDetails<K>
>(
  details: D
): Order<K, OrderStatus.CREATED>
```


### Business logic

```typescript
declare function acceptOrder<K extends OrderKind>(
  order: Order<K, OrderStatus.CREATED>,
  event: Acked
): Order<K, OrderStatus.PENDING>;

declare function rejectOrder<K extends OrderKind>(
  order: Order<K, OrderStatus.PENDING>,
  event: Failed
): Order<K, OrderStatus.FAILED>;

declare function fulfilOrder<K extends OrderKind>(
  order: Order<K, OrderStatus.CREATED>,
  event: Traded<K>
): Order<K, OrderStatus.PENDING>;
```


### IDE Help
#### Try at home!

```typescript
const newSell = createOrder({
  kind: OrderKind.SELL,
  minQuantity: 4,
  minPrice: 300,
  stock: 'UTF'
})

const acceptedSell = acceptOrder(newSell, {
  brokerId: '1'
})

const successfulSell = fulfilOrder(acceptedSell, {
  unitsSold: 5,
  sellPrice: 330
})
```


## <img class="ts" src="./img/ts-logo-white.svg"/> in the Wild Frontier


## Data enters our system <em style="color: yellow">without</em> <img class="ts" src="./img/ts-logo-white.svg"/> checks!


### What's the type of data coming from JS?

```typescript
// index.ts

// This is what we export in our `pet` library
export interface Pet {
  name: string
  age: number
}

export interface PetBehaviour {
  manners: PetManners
  tricks: PetTrick[]
}

// Don't ever do this!
// This function signature is not only incorrect but unsafe!
export function train(pet: Pet): PetBehaviour {
  // ...
}
```


### Later in a JS consumer

```javascript
// third-party-user.js

import { train } from 'pet-trainer'

// This is legal JS, no TS compiler here!
const petBehaviour = train({
  name: ['Fido', 'the', 'dog'],
  age: '1y'
})
```


### What's the type of data coming from JS?

```typescript
// Really, we don't know what JS can send as an input!
export function train(pet: unknown): PetBehaviour {
  // ...
}
```


### `unknown` vs. `any`

* `unknown` is more restrictive that `any`
* `any` accepts any operation and field access
* `unknown` does not, forces us to refine the type first

```typescript
anyValue.a, anyValue[1], anyValue.trim(), anyValue() // OK

unknownValue.a, unknownValue[1], unknownValue.trim(), unknownValue() // ERRORs
```


### Type guards
#### Reusable type refinement!

```typescript
// Is it object like?
const isObject = (x: unknown): x is Record<string, any> =>
  typeof x === 'object' &&
  x !== null

export interface Pet {
  name: string
  age: number
}

// Is it a pet?
export const isPet = (x: unknown): x is Pet =>
  isObject(x) &&
  typeof x.name === 'string' &&
  typeof x.age === 'number'
```


### Explicitly separate the core logic

```typescript
// index.ts

// Real implementation hidden in another module
import { Pet, PetBehaviour, isPet, ops } from './core'

// This is what consumer code sees
export function train(pet: unknown): PetBehaviour {
  if(isPet(pet)) {
    return ops.train(pet) // ops.train(pet: Pet): PetBehaviour
  }
  throw new Error('That is not a pet')
}
```


### Better experience for TS users

```typescript
import { Pet, PetBehaviour, isPet, ops } from './core'

// Re-export types and typeguards for TS consumers
export { Pet, PetBehaviour, isPet } from './core'

// Overload will help IDE to give TS consumers hints
export function train(pet: Pet): boolean;
export function train(pet: unknown): PetBehaviour {
  if(isPet(pet)) {
    return ops.train(pet) // ops.train(pet: Pet): PetBehaviour
  }
  throw new Error('That is not a pet')
}
```


### What about HTTP servers?


```typescript
// http/routes/train.ts
import { Pet, isPet, ops } from '../../core'
import { errorTracker } from 'famous-error-tracker'
import { db } from '../../db'

export function mount(router: Router): void {
  router.put('/train', async ctx => {
    const pet: unknown = ctx.request.body
    if(!isPet(pet)) {
      ctx.response.status = 400
      return
    }
    const newBehaviour = ops.train(pet)
    const dbResult = await db.save(newBehaviour)
    if(!dbResult.ok) {
      errorTracker.error(dbResult.error)
      ctx.response.status = 500
      return 
    }
    ctx.response.status = 200
    ctx.response.body = newBehaviour
  })
}
```


### Sanitization

* Type guards must always take `unknown` as input
* Provide type guards for all the types that represent inputs
* Any published function in a library must only take `unknown` inputs 
* Any function reading data (db, files, http routes, etc) must check it using type guards
* Give a nice experience to TS users by providing overloads in the public API


## There is more in <img class="ts" src="./img/ts-logo-white.svg"/>!
### Index types, Mapped types, type assertions, etc.


## Any time for questions??
