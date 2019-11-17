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
 componentWillReceiveProps(nextProps, nextState) {
    if (
      !this.state.isPagePrefilled &&
      !nextProps.PatientQuery.loading &&
      !nextProps.DeviceModelsQuery.loading &&
      !nextProps.DeviceModelsFeaturesQuery.loading &&
      !_.isEmpty(nextProps.Customer)
    ) {
      this.prefillEditPage(nextProps)
    }
  }
  ```
  Test :-
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
### Async/Await Mock Value Return
```
import {callAnotherFunction} from '../../../utils';

  export const handler = async (event, context, callback) => {

  const {emailAddress, emailType} = event.body;
  console.log("**** GETTING HERE = 1")
  const sub = await callAnotherFunction(emailAddress, emailType);
  console.log("**** Not GETTING HERE = 2", sub) // **returns undefined**

  // do something else here
  callback(null, {success: true, returnValue: sub})

}
```
Test :-
```
import { callAnotherFunction } from '../../../utils';

callAnotherFunction.mockImplementation(() => Promise.resolve('someValue'));

test('test' , done => {
  const testData = {
    body: {
      emailAddress: 'email',
      emailType: 'type
    }
  };

  function callback(dataTest123) {
    expect(dataTest123).toBe({success: true, returnValue: 'someValue');
    done();
  }

  handler(testData, null, callback);
});
```

```
biblia/js/search.js
const axios = require("axios");
const googleBooksUrl ="https://www.googleapis.com/books/v1/volumes";
const keys = require("../config/keys");
const search = {
  fetchBooks: (term, index = 0) =>
    axios.get(googleBooksUrl, {
      params: {
        q: term,
        startIndex: index,
        key: keys.googleBooksApiKey
      }
    })
    .then(response => {
      return response;
    })
    .catch(err => err.response)
 ```
 
Test :-

```
const mockAxios = require("axios");
const search = require("../js/search");
const keys = require("../config/keys");
const searchResponse = require("../__fixtures__/searchResponse");
test("fetches results from google books api", () => {
  mockAxios.get.mockImplementationOnce(() =>
    Promise.resolve(searchResponse.raw)
  );
  let term = "tolkein";
  let index = 0;
return search.fetchBooks(term, index).then(response => {
    expect(response).toEqual(searchResponse.raw);
  });
});
```
### Mocking JS Current Date
```
const literallyJustDateNow = () => Date.now();

test('It should call and return Date.now()', () => {
  const realDateNow = Date.now.bind(global.Date);
  const dateNowStub = jest.fn(() => 1530518207007);
  global.Date.now = dateNowStub;

  expect(literallyJustDateNow()).toBe(1530518207007);
  expect(dateNowStub).toHaveBeenCalled();

  global.Date.now = realDateNow;
});
```

### Testing ComponetWillUnmount
```
it('should reset SupportId after componentWillUnmount is called', () => {
    const component = renderShallow(<ProfileTuning {...props} />)
    const componentWillUnmount = jest.spyOn(component.instance(), 'componentWillUnmount')
    component.unmount()
    expect(componentWillUnmount).toHaveBeenCalled()
  })
```

### should call handleSubmit function on submit
```
it('should call handleSubmit function on submit', () => {
    const wrapper = shallow(<Search toggleAlert={jest.fn()} />)
    const spy = jest.spyOn(wrapper.instance(), 'handleSubmit')
    wrapper.instance().forceUpdate()
    wrapper.find('.btn').simulate('click')
    expect(spy).toHaveBeenCalled()
    spy.mockClear()
  })
  
  OR
  
  it('should call handleSubmit function on submit', () => {
    const wrapper = shallow(<Search toggleAlert={jest.fn()} />)
    wrapper.instance().handleSubmit = jest.fn()
    wrapper.update()
    wrapper.find('.btn').simulate('click')
    expect(wrapper.instance().handleSubmit).toHaveBeenCalled()
  })
```

### Should catch exception while Fetching Feature for devices
```
it('Should catch exception while Fetching Feature for devices', () => {
    const getHdDeviceModelsFeatures = jest.fn(() => {
      throw TypeError()
    })
    instance.fetchFeaturesForDevices({ leftModelId: '0', rightModelId: '0' })
    expect(getHdDeviceModelsFeatures).toThrow(TypeError)
  })
```
