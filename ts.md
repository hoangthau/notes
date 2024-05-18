# TypeScript's Global Scope

```
globalThis.myFunc = () => true;
globalThis.myVar = 1;
```
We will get `Cannot find name 'myVar'.ts(2304)`
```
expect(myFunc()).toBe(true);
expect(myVar).toBe(1);
```
How to fix that?
You will need use `declare global {}.`

```
declare global {
  function mySolutionFunc(): boolean;
  var mySolutionVar: number;
}
```

```
window.makeGreeting = () => "Hello!";
window.makeGreeting();
```

Property 'makeGreeting' does not exist on type `'Window & typeof globalThis'.ts(2339)`
```
declare global {
  interface Window {
    makeGreetingSolution: () => string;
  }
}
```
Some other ways we can use `types.d.ts`
```
interface Window {
  makeGreeting: () => string;
}

function myFunc(): boolean;
var myVar: number;
```

# Filtering with Type Predicates
```
export const values = ["a", "b", undefined, "c", undefined];

const filteredValues = values.filter((value) => Boolean(value));
```
When hover on filteredValues type
`const filteredValues: (string | undefined)[]`
How to use type predicate:
```
const filteredValues = values.filter((value) => Boolean(value)) as string[];
const filteredValues = values.filter((value): value is string =>
  Boolean(value),
);
```
# Assertion Function

How to assert or type guards for a variable

```
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

```
function userIsAdmin(user: NormalUser | AdminUser): user is AdminUser {
  return user.role === 'admin';
}

const example = (user: NormalUser | AdminUser) => {
    if(userIsAdmin(user)) {
      //user is Admin now
    }
};
```

Or use assertion function https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-7.html#assertion-functions
```
function assertUserIsAdmin(
  user: NormalUser | AdminUser,
): asserts user is AdminUser {
  if (user.role !== "admin") {
    throw new Error("Not an admin user");
  }
}
```



