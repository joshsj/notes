# Axios (HTTP Client)

## The usual

```js
const client = axios.create({
  baseURL: "http://site.com/api/",
  withCredentials: false,
  headers: {
    "Accept": "application/json",
    "Content-Type": "application/json",
  }
});

client.get("/data")
  .then(res => {...})
  .catch(err => {...})
```

## Interceptors

- Used to call functions upon request/response

```js
axios.interceptors.request.use(
  (config) => {
    // do something before request is sent
    return config;
  },
  (error) => {
    // do something with request error
    return Promise.reject(error);
  }
);

axios.interceptors.response.use(
  (res) => {
    // any status code 2xx
    return response;
  },
  (error) => {
    return Promise.reject(error);
  }
);
```
