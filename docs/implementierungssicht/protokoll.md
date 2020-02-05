# Protokoll [Version 1]

## Events

```json
{
	"version": number,
	"id": "uuid",
	"src": "str", // ORDER, STOCK, KITCHEN, TRACKING, DELIVERY, SERVING, PAYMENT
	"type": {
		"context": "str", // ORDER, DISH
		"event": "str"
			// ORDER: PLACED, IN_PROGRESS, FINISHED, IN_DELIVERY, DELIVERED, DELIVERY_FAILED, PICKED_UP, PAYMENT_COMPLETED
			// DISH: IN_PROGRESS, FINISHED, IN_SERVING, SERVED
	},
	"object": "uuid",
	"payload": { }
}
```

### Payloads

#### ORDER.PLACED
```json
{
	"type": "str", // INHOUSE, DELIVERY
	"dishes": [
		{
			"id": "uuid",
			"number": number,
			"name": "str",
			"price": number, // cent
			"currency": "str", // ISO 4217-code
		}
	],
	"customer": {
		"id": "uuid",
		"last_name"?: "str",
		"first_name"?: "str"
	},
	"address"?: {
		"line_1": "str",
		"line_2": "str",
		"line_3": "str",
		"zip": number,
		"city": "str",
		"country": "str" // ISO 3166-alpha-3
	},
	"table"?: number
}
```

#### ORDER.IN_DELIVERY / DELIVERED / DELIVERY_FAILED
```json
{
	"driver": number
}
```

#### ORDER.PAYMENT_COMPLETED
```json
{
	"method": "str",
	"amount": number // cent
}
```

#### DISH.IN_PROGRESS / FINISHED
```json
{
	"cook": number
}
```

#### DISH.IN_SERVING / SERVED
```json
{
	"waiter": number
}
```

## Errors

```json
{
	"version": number,
	"id": "uuid",
	"src": "str",
	"error": {
		"errno": number,
		"message": "str"
	},
	"event": "uuid"
}
```

---

# RPC

Abrufen der Lager-Menge einer Zutat:
```
StockService.getAmountInStock(ingredientId: number): number
```

Abrufen der nötigen Zutaten für ein Gericht:
```
MenuService.getIngredientsForDish(dishId: number): [{ ingredientId: number, amount: number }]
```

StorageAmount.proto
```proto
syntax = "proto3";

service StorageAmount {
    rpc GetAmountInStock (IngredientId) returns (Amount);
}

message IngredientId {
    string  ingredientId = 1;
}

message Amount {
    int32 amount = 1;
}
```

---

# REST

### Bestell-Service

**POST /order**

consumes:
```
<order-placed-payload>
```

returns: id of placed order
```
{
  "id": "7514eb16-836c-4736-be9e-a9e47531dad2"
}
```

### Tracking-Service

**GET /order?id=[id]**

returns: current state of order
```json
{
	<order-placed-payload>
	"events": [ { <event> } ],
	"stages": [ { "stage": "PLACED", "entered": "datetime" } ],
	"dishes": [
		{
			<dish>,
			"events": [ { <event> } ],
			"stages": [ { "stage": "IN_PROGRESS", "entered": "datetime" } ]
		}
	]
}
```

#### Websocket

**/order/subscribe?order=[id]**

Emits: "event"

### Storage-Service

Ingredient:
```json
{
    "id": "5e31d8e6c3971b00017db519",
    "category": "Gemüse",
    "name": "Birne",
    "amount": 32,
    "reseverations": 0,
    "unit": "Stück"
}
```

**GET /api/storage/getingredient/:oid**

consumes: oid -> Ingredient Id

returns: ingredient
```json
<ingredient>
```

**GET /api/storage/getingredients**

returns: list of ingredients
```json
{
    [
        { <ingredient> }
    ]
}
```

**PUT /api/storage/putingredient**

Neu erstellen:

consumes: ingredient
```json
<ingredient> (ohne id)
```
returns: ingredient
```json
<ingredient>
```

Bestehendes editieren:

consumes: ingredient
```json
<ingredient>
```
returns: 200, wenn erfolgreich editiert

**DELETE /api/storage/deleteingredient**

consumes: ingredient
```json
<ingredient>
```
returns: ingredient
```json
<ingredient>
```
