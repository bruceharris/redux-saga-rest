# redux-saga-rest

[![npm version](https://img.shields.io/npm/v/redux-saga-rest.svg?style=flat-square)](https://www.npmjs.com/package/redux-saga-rest)

`redux-saga-rest` is a thin wrapper around the Fetch API that integrates with [redux-saga](https://github.com/yelouafi/redux-saga) and supports request/response middleware.

## Installation

```sh
# dependencies
yarn add redux redux-saga isomorphic-fetch

yarn add redux-saga-rest
```

## Usage Example

#### `api.js`

```javascript
import { put, select } from 'redux-saga/effects';
import { API } from 'redux-saga-rest';

import * as selectors from './selectors';
import * as actions from './actions';

const authMiddleware = () => function* (req, next) {

    // request middleware
    const user = yield select(selectors.user);
    const headers = req.headers || new Headers();
    headers.set('Authorization', `Bearer ${user.token}`);

    // retrieve the response
    const res = yield next(new Request(req, { headers }));

    // response middleware
    if (res.status === 401) {
        yield put(actions.logout());
    }

    // return the response
    return res;
};

export const auth = new API('/api/')
    .use(authMiddleware());
```

**TODO:** Describe the middleware application order.

#### `sagas.js`

```javascript
import { takeEvery, put } from 'redux-saga/effects';

import * as constants from './constants';
import * as actions from './actions';
import { auth } from './api';

function* watchUpdateProfile() {
    yield takeEvery(constants.UPDATE_PROFILE, function* (action) {
        const res = yield auth.patch('/profile/', action.payload);
        
        if (res.ok) {
            yield put(actions.updateProfileSuccess());
        } else {
            yield put(actions.updateProfileFailure());
        }
    });
}

export default function* () {
    yield [
        watchUpdateProfile(),
    ];
};
```
