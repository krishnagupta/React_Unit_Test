## React Unit Test Cases

### How to mock specific module function in jest?
Mocking a function generally is very easy in jest via jest.fn(). However, if you run into the following scenario which one function in the module is calling another function in the same module, it could be tricky.
For example, you have two functions in a module file.
```
// helper.js
export function getFullName(firstName, lastName) {
  return firstName + ' ' + lastName;
}
export function greet(firstName, lastName) {
  const fullName = getFullName(firstName, lastName);
  return `Hey, ${fullName}`;
}
```
When you are trying to write your jest unit test to test the function greet(), you would assume that you can just mock the getFullName() inside function greet().
```
import { getFullName, greet } from './helper';
describe('greet', () => {
  it('should return greet message with full name', () => {
    getFullName = jest.fn().mockReturnValue('mock full name');
    const result = greet('QJ', 'Li');
    expect(result).toBe('Hey, mock full name');
  });
});
```
However, the result is not what you expect like above. The reason being, getFullName function in the helper.js is a local scope function and override it directly via jest.fn() won’t work here.
There are a few solutions here:

1.) Store all export functions to an object and export by default
Instead of using export function to export each function, do the following:
```
// helper.js
function getFullName(firstName, lastName) {
  return firstName + ' ' + lastName;
}
function greet(firstName, lastName) {
  const fullName = exportFunctions.getFullName(firstName, lastName);
  return `Hey, ${fullName}`;
}
const exportFunctions = {
  getFullName,
  greet
};
export default exportFunctions;
```
The trick here is to use exportFunctions.getFullName() in greet() instead of calling getFullName() directly. In this case, in your unit test, you can mock the getFullName() via jest.fn()
```
import helpers from './helper';
describe('greet', () => {
  it('should return greet message with full name', () => {
    helpers.getFullName = jest.fn().mockReturnValue('mock full name');
    const result = helpers.greet('QJ', 'Li');
    expect(result).toBe('Hey, mock full name');
  });
});
```

### Testing Component Lifecycle
```
it('componentWillReceiveProps', async () => {
    const component = renderShallow(<NewHearingInstrument {...props} />)
    const instance = component.instance()
    const spy = jest.spyOn(instance, 'prefillEditPage')
    await instance.componentWillReceiveProps(props)
    expect(spy).toHaveBeenCalled();
  })
  ```
  
  ### Testing function having arguement as another function
  ```
  it('should call onSelectLessonSet', () => {
    const component = renderShallow(<EditLessonSetPreference {...props} />).instance()
    const setFieldValue = jest.fn()
    component.onSelectLessonSet(123, setFieldValue)
    expect(setFieldValue).toHaveBeenCalled()
  })
  ```
