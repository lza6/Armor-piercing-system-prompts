# fetch() - Web API Reference

The `fetch()` method starts the process of fetching a resource from the network, returning a promise that is fulfilled once the response is available.

## Syntax

```javascript
fetch(resource)
fetch(resource, options)
```

### Parameters

**resource**

This defines the resource that you wish to fetch. This can be:

- A string or any other object with a stringifier, including a URL object, providing the URL of the resource you want to fetch.
- A `Request` object.

**options** (optional)

An object containing any custom settings you want to apply to the request. The possible options are:

| Option | Type | Description |
|--------|------|-------------|
| `method` | `string` | The request method, e.g., `GET`, `POST`, `PUT`, `DELETE` |
| `headers` | `Headers` or `Object` | Any headers you want to add to your request |
| `body` | `Blob`, `BufferSource`, `FormData`, `URLSearchParams`, `string`, or `ReadableStream` | Body of the request |
| `mode` | `string` | The mode of the request: `cors`, `no-cors`, `same-origin` |
| `credentials` | `string` | Controls what browsers do with credentials: `omit`, `same-origin`, `include` |
| `cache` | `string` | How the request will interact with the browser's HTTP cache |
| `redirect` | `string` | How to handle a redirect response: `follow`, `error`, `manual` |
| `referrer` | `string` | Specifies the referrer of the request |
| `referrerPolicy` | `string` | Specifies the referrer policy |
| `integrity` | `string` | Contains the subresource integrity value of the request |
| `keepalive` | `boolean` | Used to allow the request to outlive the page |
| `signal` | `AbortSignal` | An `AbortSignal` object to communicate with a fetch request and abort it |
| `priority` | `string` | Specifies the priority of the fetch request relative to other requests: `high`, `low`, `auto` |

### Return Value

A `Promise` that resolves to a `Response` object.

### Exceptions

- `AbortError` `DOMException` - The request was aborted due to a call to the `AbortController.abort()` method.
- `TypeError` - Can occur for the following reasons:
  - The resource URL contains credentials
  - The scheme is not `http` or `https`
  - The `Request.mode` is `navigate`

## Examples

### Basic GET Request

```javascript
async function fetchData() {
  try {
    const response = await fetch("https://api.example.com/data");
    if (!response.ok) {
      throw new Error(`HTTP error! Status: ${response.status}`);
    }
    const data = await response.json();
    console.log(data);
  } catch (error) {
    console.error("Fetch failed:", error);
  }
}
```

### POST Request with JSON

```javascript
async function postData(url, data) {
  const response = await fetch(url, {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.stringify(data),
  });

  return response.json();
}

const result = await postData("https://api.example.com/users", {
  name: "Jane Doe",
  email: "jane@example.com",
});
```

### Uploading a File

```javascript
async function upload(file) {
  const formData = new FormData();
  formData.append("file", file);

  const response = await fetch("https://api.example.com/upload", {
    method: "POST",
    body: formData,
  });

  return response.json();
}
```

### Aborting a Fetch

```javascript
const controller = new AbortController();
const signal = controller.signal;

setTimeout(() => controller.abort(), 5000);

try {
  const response = await fetch("https://api.example.com/slow", { signal });
  const data = await response.json();
  console.log(data);
} catch (err) {
  if (err.name === "AbortError") {
    console.log("Request was aborted");
  } else {
    throw err;
  }
}
```

### Checking Response Status

```javascript
async function fetchWithStatus(url) {
  const response = await fetch(url);

  switch (response.status) {
    case 200:
      return response.json();
    case 404:
      throw new Error("Resource not found");
    case 401:
      throw new Error("Unauthorized");
    case 500:
      throw new Error("Server error");
    default:
      throw new Error(`Unexpected status: ${response.status}`);
  }
}
```

### Streaming Response

```javascript
async function streamResponse(url) {
  const response = await fetch(url);
  const reader = response.body.getReader();
  const decoder = new TextDecoder();

  while (true) {
    const { done, value } = await reader.read();
    if (done) break;
    console.log(decoder.decode(value, { stream: true }));
  }
}
```

## Browser Compatibility

| Browser | Version |
|---------|---------|
| Chrome | 42+ |
| Firefox | 39+ |
| Safari | 10.1+ |
| Edge | 14+ |
| Opera | 29+ |
| Node.js | 18+ (built-in) |

## Specifications

- [Fetch Standard](https://fetch.spec.whatwg.org/#fetch-method)
- [MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/API/fetch)

## See Also

- [Headers](https://developer.mozilla.org/en-US/docs/Web/API/Headers)
- [Request](https://developer.mozilla.org/en-US/docs/Web/API/Request)
- [Response](https://developer.mozilla.org/en-US/docs/Web/API/Response)
- [Using Fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch)
