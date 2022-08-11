# types-ddd documentation

## About the lib

The library was created to support developers in developing domain-rich applications.

Full documentation.
Version 3.x

This lib use as core [rich-domain](https://www.npmjs.com/package/rich-domain)

## Documentation

### Result

What is Result:

`Result` is a class that encapsulates the result of an operation and stores the success or failure state without throws the application.

#### Interface and Generic Types

- A = `Payload` optional default `void`
- B = `Error` optional default `string`
- C = `MetaData` optional default `{}`

```ts

IResult<A, B, C>

```

Example how to use generic types.
First let's create our interfaces to use as generic type.
- The type of data to be retrieved can be any type you want.


```tS

interface IData {
	data: string;
}

interface IError {
	message: string;
}

interface IMeta {
	arg: number;
}

```

Now let's implement a function that return the result below

```ts

IResult<IData, IError, IMeta>

```
So let's implement that on a simple function.

```ts

const isPair = (value: number):IResult<IData, IError, IMeta> => {

	const isPairValue = value % 2 === 0;
	const metaData = { arg: value };
	
	if (isPairValue) {
		
		// success payload 
		const payload = { data: `${value} is pair` };

		// return success
		return Result.Ok(payload, metaData);
	}

	// failure payload 
	const error = { message: `${value} is not pair` };

	// return failure
	return Result.fail(error, metaData);
};


```
Here we have a function that returns success if the value informed is even and returns failure if it is odd.

Success Case

```ts

const result = isPair(42);

console.log(result.isOk());

> true

console.log(result.value());

> 'Object { data: "42 is pair" }'

console.log(result.metaData());

> 'Object { arg: 42 }'

console.log(result.error());

> null

```
Failure Case

```ts

const result = isPair(43);

console.log(result.isFail());

> true

console.log(result.error());

> 'Object { message: "43 is not pair" }'

console.log(result.metaData());

> 'Object { arg: 43 }'

console.log(result.value());

> null

```

#### Void

The most simple void success example.

Let's see the same example using void.

```ts

const checkPair = (value: number): Result<void> => {

	const isPair = value % 2 === 0;

	// success case
	if(isPair) return Result.Ok(); 
	
	// failure case
	return Result.fail('not pair');
}

```
Using the function as success example

```ts

const result: IResult<void> = checkPair(42);

console.log(result.isOk());

> true

console.log(result.isFail());

> false

console.log(result.error());

> null

console.log(result.value());

> null

console.log(result.metaData());

> 'Object {}'

```

Fail example

```ts

const result: IResult<void> = checkPair(43);

console.log(result.isFail());

> true

console.log(result.isOk());

> false

console.log(result.error());

> "not pair"

console.log(result.value());

> null

console.log(result.metaData());

> 'Object {}'

```

#### toObject method
you can get a summarized object with the properties of an instance of a `Result`

```ts

console.log(result.toObject())

> Object
`{
	"data": null, 
	"error": "not pair", 
	"isFail": true, 
	"isOk": false, 
	"metaData": Object {}
 }`

```

#### Hooks

In the instances of a Result there are two hooks that allow the execution of a command according to the state.

```ts

class Command implements ICommand<void, void> {
	execute(): void {
		console.log("running command ...");
	}
}

const myCommand = new Command();

const result = Result.Ok();

result.execute(myCommand).on('Ok');

> "running command ..."

```

You might also want to pass arguments to the command

```ts

class Command implements ICommand<string, void> {
	execute(error: string): void {
		console.log(error);
	}
}

const myCommand = new Command();

const result = Result.fail('something went wrong');

result.execute(myCommand).withData(result.error()).on('fail');

> "something went wrong"

```

#### Combine

You can use the static `combine` function of `Result` to check many instances if any are failed it will return the instance with error state.

Success example 

```ts

const resultA = Result.Ok();
const resultB = Result.Ok();
const resultC = Result.Ok();

const result = Result.combine([resultA, resultB, resultC]);

console.log(result.isOk());

> true

```
Failure example 

```ts

const resultA = Result.Ok();
const resultB = Result.Fail('oops err');
const resultC = Result.Ok();

const result = Result.combine([resultA, resultB, resultC]);

console.log(result.isOk());

> false

console.log(result.error());

> 'oops err'

```

---


### ValueObject

What is value object:

- Are only identified by their values, not by their ids (for example money is a value object as long as we are not tracking individual banknotes, if we need to track individual banknotes then it should be a banknote entity)
- Can be used to measure or describe things (name, description, amount, height, date, time, range, address, etc.)
- You can combine other value types that usually go together into a new value object type, like address (city, street, country, postal code) or ...range, or ...type
- Prefer to put the behavior on value objects rather than on entities because value objects are immutable and do not have side effects (like changing their state or changing the state of any entity)
- Can be part of an entity
- Should be immutable, behaviors should not change the state of a value object, but can rather create a new value object (should act similar to C# strings, structs, ints, and other value types)
- Can be persisted but only as part of an entity, not individually.


#### Simple Value Object.

Value objects extend to `ValueObject` class have private constructor and public static method called `create`.
The `create` method receives the props which by default is an object with the key `value`.

the value object below is a base example without any kind of validation

```ts

import { IResult, Result, ValueObject } from "types-ddd";

export interface NameProps {
	value: string;
}

export class Name extends ValueObject<NameProps>{
	private constructor(props: NameProps) {
		super(props);
	}

	public static create(props: NameProps): IResult<Name> {
		return Result.Ok(new ProductName(props));
	}
}

export default Name;

```

Now that we have defined our value object class, we can create an instance.

the `create` method returns an instance of Name encapsulated by the `Result`, so it is important to always assess whether the result is a success before getting the value.

```ts

const result = Name.create({ value: 'Jane' });

console.log(result.isOk());

> true

const name = result.value();

console.log(name.get('value'));

> "Jane"

```

Once we have an instance of a value object, we can use some methods that the library makes available.

By default setters are enabled

```ts

name.set('value').to('John');

console.log(name.get('value'));

> "John"

```

When you use the `set` or `change` function to modify the state, each change is saved in a history

```ts

console.log(name.history().count());

> 2

// back to old value on history
name.history().back();

console.log(name.get('value'));

> "Jane"

```

> **I don't advise you to use state change of a value object. Create a new one instead of changing its state. However the library will leave that up to you to decide.**

To disable the setters of a value object use the parameters below in the super.
This property disables the set function of the value object.

```ts

	constructor(props: NameProps){
		super(props, { disableSetters: true })
	}

```

Now when trying to change the value using `set` or `change` it will not be modified.

```ts

console.log(name.get('value'));

> "John"

name.set('value').to('James');

console.log(name.get('value'));

> "John"

```

#### Using validation

Validation before create instance.
A validator instance is available in the "Value Object" domain class.

```ts

import { IResult, Result, ValueObject } from "types-ddd";

export interface NameProps {
	value: string;
}

export class Name extends ValueObject<NameProps>{
	private constructor(props: NameProps) {
		super(props);
	}

	public static isValidProps({ value }: NameProps): boolean {
		const { string } = this.validator;
		return string(value).hasLengthBetween(3, 30);
	}

	public static create(props: NameProps): IResult<Name> {
		const message = 'name must have length min 3 and max 30 char';

		if (!this.isValidProps(props)) return Result.fail(message);

		return Result.Ok(new ProductName(props));
	}
}

export default Name;

```

Now when you try to instantiate a name, the value will be checked and if it doesn't meet the validation requirements, a `Result` will be returned with an error state.

```ts

const empty = '';

const result = Name.create({ value: empty });

console.log(result.isFail());

> true

console.log(result.error());

> "name must have length min 3 and max 30 char"

console.log(result.value());

> null

```

#### Validation before set

The `isValidProps` Method validates properties when creating a new instance, but which method validates before modifying a value?
For this there is the method `validation`

The validation method takes two arguments, the first the `key` of props and the second the `value`.
So when calling the `set` or `change` function, this method will be called automatically to validate the value, if it doesn't pass the validation, the value is not changed.

> There must be a validation for each "props" key

```ts

	validation<Key extends keyof Props>(value: Props[Key], key: Key): boolean {
		
		const { number } = this.validator;

		const options: IPropsValidation<Props> = {
			value: (value: number) => number.isBetween(0, 130),
		} 

		return options[key](value);
	};

```

In case your value object has only one attribute you can simply use the already created static validation method.
Let's see a complete example as below

```ts

import { IResult, Result, ValueObject } from "types-ddd";

export interface NameProps {
	value: string;
}

export class Name extends ValueObject<NameProps>{
	private constructor(props: NameProps) {
		super(props);
	}

	validation(value: string): boolean {
		return Name.isValidProps({ value });
	}

	public static isValidProps({ value }: NameProps): boolean {
		const { string } = this.validator;
		return string(value).hasLengthBetween(3, 30);
	}

	public static create(props: NameProps): IResult<Name> {
		const message = 'name must have length min 3 and max 30 char';

		if (!this.isValidProps(props)) return Result.fail(message);

		return Result.Ok(new ProductName(props));
	}
}

export default Name;

```

Let's test the instance and the validation method.
Value is not modified if it does not pass validation.

```ts

const result = Name.create({ value: 'Jane' });

console.log(result.isOk());

> true

const name = result.value();

console.log(name.get('value'));

> "Jane"

const empty = '';

name.set('value').to(empty);

console.log(name.get('value'));

> "Jane"

name.set('value').to("Jack");

console.log(name.get('value'));

> "Jack"

```

---

### Entity

Info here ...

---

### Aggregate

Info here ...