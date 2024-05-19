# TypeScript's Global Scope

```.ts
globalThis.myFunc = () => true;
globalThis.myVar = 1;
```
We will get `Cannot find name 'myVar'.ts(2304)`
```.ts
expect(myFunc()).toBe(true);
expect(myVar).toBe(1);
```
How to fix that?
You will need use `declare global {}.`
<details>
<summary>Solution</summary>

```.ts
declare global {
  function mySolutionFunc(): boolean;
  var mySolutionVar: number;
}
```
</details>

```.ts
window.makeGreeting = () => "Hello!";
window.makeGreeting();
```

Property 'makeGreeting' does not exist on type `'Window & typeof globalThis'.ts(2339)`
```.ts
declare global {
  interface Window {
    makeGreetingSolution: () => string;
  }
}
```
Some other ways we can use `types.d.ts`
```.ts
interface Window {
  makeGreeting: () => string;
}

function myFunc(): boolean;
var myVar: number;
```

How do we overwrite type from external libs
Example:
Here's what getAnimatingState from the `fake-animation-lib` looks like:
```.ts
export const getAnimatingState = (): string => {
  if (Math.random() > 0.5) {
    return "before-animation";
  }

  if (Math.random() > 0.5) {
    return "animating";
  }

  return "after-animation";
};
```

```.ts
import { getAnimatingState } from "fake-animation-lib";
const animatingState = getAnimatingState();
type Example = typeof animatingState;

// We want Example type to be "before-animation" | "animating" | "after-animation"

```
Solution:
We can create a definition type `something.d.ts`

```.ts
declare module "fake-animation-lib-solution" {
  export type AnimatingState =
    | "before-animation"
    | "animating"
    | "after-animation";
  export function getAnimatingState(): AnimatingState;
}

```

# Filtering with Type Predicates
```.ts
export const values = ["a", "b", undefined, "c", undefined];

const filteredValues = values.filter((value) => Boolean(value));
```
When hover on filteredValues type
`const filteredValues: (string | undefined)[]`
How to use type predicate:
<details>
<summary>Solution</summary>

```.ts
const filteredValues = values.filter((value) => Boolean(value)) as string[];
const filteredValues = values.filter((value): value is string =>
  Boolean(value),
);
```
</details>

# Assertion Function

How to assert or type guards for a variable

```.ts
interface User {
  id: string;
  name: string;
}

interface AdminUser extends User {
  role: "admin";
  organisations: string[];
}

interface NormalUser extends User {
  role: "normal";
}

function assertUserIsAdmin(user: NormalUser | AdminUser) {
  if (user.role !== "admin") {
    throw new Error("Not an admin user");
  }
}

const example = (user: NormalUser | AdminUser) => {
    assertUserIsAdmin(user);
    console.log(user); // user is still NormalUser or AdminUser
};
```
We can use type predicate like:
<details>
<summary>Solution</summary>
  
```.ts
function userIsAdmin(user: NormalUser | AdminUser): user is AdminUser {
  return user.role === 'admin';
}

const example = (user: NormalUser | AdminUser) => {
    if(userIsAdmin(user)) {
      //user is Admin now
    }
};
```
</details>

Or use assertion function https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-7.html#assertion-functions
```.ts
function assertUserIsAdmin(
  user: NormalUser | AdminUser,
): asserts user is AdminUser {
  if (user.role !== "admin") {
    throw new Error("Not an admin user");
  }
}
```
# Classes as Types and Values

```.ts
class CustomError extends Error {
  constructor(message: string, public code: number) {
    super(message);
    this.name = "CustomError";
  }
}

// How do we type the 'error' parameter?
const handleCustomError = (error: unknown) => {
  console.error(error.code);

  type test = Expect<Equal<typeof error.code, number>>;
};
```

# Classes with Type Predicate
We expect `form.isInvalid()` is only return string as errors, but it return string or undefined like:

```.ts
class Form<TValues> {
  error?: string;

  constructor(
    public values: TValues,
    private validate: (values: TValues) => string | void,
  ) {}

  isInvalid(): {
    const result = this.validate(this.values);

    if (typeof result === "string") {
      this.error = result;
      return true;
    }

    this.error = undefined;
    return false;
  }
}

const form = new Form(
  {
    username: "",
    password: "",
  },
  (values) => {
    if (!values.username) {
      return "Username is required";
    }

    if (!values.password) {
      return "Password is required";
    }
  },
);

if (form.isInvalid()) {
  type test1 = Expect<Equal<typeof form.error, string>>; // Wrong here
} else {
  type test2 = Expect<Equal<typeof form.error, string | undefined>>;
}

```
We will predicate `isInvalid` function is correct type form & { error: string } with
Type Predicate syntax for a function:

<details>
<summary>Solution</summary>
  
```.ts
isInvalid(): this is Form<TValues> & { error: string } {
  const result = this.validate(this.values);

  if (typeof result === "string") {
    this.error = result;
    return true;
  }

  this.error = undefined;
  return false;
}
```
</details>

# Assertion Function and Classes
How do we predicate a type in a function?

```.ts
interface User {
  id: string;
}

export class SDK {
  loggedInUser?: User;

  constructor(loggedInUser?: User) {
    this.loggedInUser = loggedInUser;
  }

  // How do we type this assertion function?
  assertIsLoggedIn() {
    if (!this.loggedInUser) {
      throw new Error("Not logged in");
    }
  }

  createPost(title: string, body: string) {
    type test1 = Expect<Equal<typeof this.loggedInUser, User | undefined>>;

    this.assertIsLoggedIn();

    type test2 = Expect<Equal<typeof this.loggedInUser, User>>; //expect this is only User
  }
}
```
Type predicate
<details>
<summary>Solution</summary>

```.ts
isLoggedInUser(): this is SDK & { loggedInUser: User } {
  if (!this.loggedInUser) {
    return false
  }
  return true
}

//OR

assertIsLoggedIn(): asserts this is SDK & { loggedInUser: User } {
  if (!this.loggedInUser) {
    throw new Error("Not logged in");
  }
}
```
</details>

# Where do external types come from?
- `lib.d.ts` : types for JS from TS team
- `lib.dom.d.ts`: types for DOM from other team outside TS (Mozilla)
- `@type/react`, `@type/node`, `@type/lodash` types from libs - type definitions - use for some project that did not use TS
  but have separate @type for their user like React, Express: https://github.com/DefinitelyTyped/DefinitelyTyped/tree/master/types
- Types shipped within the lib: vite, vue, jest, react-query

# Extract Types to Extend an external libs
We can not modify this file
```.ts
// inside fake-external-lib/fetches.ts
export const fetchUser = async (id: string) => {
  return {
    id,
    firstName: "John",
    lastName: "Doe",
  };
};

```

```.ts
/**
 * We're using a function from fake-external lib, but we need
 * to extend the types. Extract the types below.
 */

type ParametersOfFetchUser = unknown;

type ReturnTypeOfFetchUserWithFullName = unknown;

export const fetchUserWithFullName = async (
  ...args: ParametersOfFetchUser
): Promise<ReturnTypeOfFetchUserWithFullName> => {
  const user = await fetchUser(...args);
  return {
    ...user,
    fullName: `${user.firstName} ${user.lastName}`,
  };
};
```

<details>
<summary>Solution</summary>

```.ts
type ParametersOfFetchUser = Parameters<typeof fetchUser>;

type ReturnTypeOfFetchUserWithFullName = Awaited<
  ReturnType<typeof fetchUser>
> & { fullName: string };

export const fetchUserWithFullName = async (
  ...args: ParametersOfFetchUser
): Promise<ReturnTypeOfFetchUserWithFullName> => {
  const user = await fetchUser(...args);
  return {
    ...user,
    fullName: `${user.firstName} ${user.lastName}`,
  };
};
```
</details>

# Passing Type Arguments with Lodash
```.ts
interface User {
  name: string;
  age: number;
}

const groupByAge = (array: unknown[]) => {
  const grouped = _.groupBy(array, "age");

  return grouped;
};

const result = groupByAge([
  {
    name: "John",
    age: 20,
  },
  {
    name: "Jane",
    age: 20,
  },
  {
    name: "Mary",
    age: 30,
  },
]);
```
collection.d.ts
```.ts
groupBy<T>(collection: List<T> | null | undefined, iteratee?: ValueIteratee<T>): Dictionary<T[]>;
```

Pass Type Arguments
```.ts
const groupByAge = <T>(array: T[]) => {
  const grouped = _.groupBy<T>(array, "age");

  return grouped;
};

// type for result
const result: _.Dictionary<{
    name: string;
    age: number;
}[]>
```

Pass Generic type for reduce function
```.ts
interface Person {
  name: string;
  age: number;
}

const persons: Person[] = [
  { name: 'John', age: 30 },
  { name: 'Alice', age: 45 },
];

const ageByPerson = persons.reduce<Record<string, string>>((result, person) => ({
  ...result,
  [person.age]: [...result[person.age] ?? [], person.age]
}), {});

console.log(ageByPerson);
```




