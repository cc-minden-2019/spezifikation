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

### Error-Codes

TODO

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
