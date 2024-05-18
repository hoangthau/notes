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




