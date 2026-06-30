# Builder Design Pattern

It’s particularly useful in situations where:

- An object has many optional fields, and most callers only need a subset.
- You want to avoid telescoping constructors or long parameter lists.
- The object must be assembled through multiple steps, possibly in a specific order.

Without a builder, developers often end up with overloaded constructors or a wide set of setters. For example, a User object might include fields like name, email, phone, address, and preferences. As the number of fields grows, the API becomes harder to use correctly, easier to misuse, and more difficult to maintain.

## The Problem: Building Complex HttpRequest Objects

Each HttpRequest can contain a mix of required and optional fields:

- URL (required)
- HTTP Method (e.g., GET, POST, PUT, defaults to GET)
- Headers (optional, multiple key-value pairs)
Query Parameters (optional, multiple key-value - pairs)
- Request Body (optional, typically for POST/PUT)
- Timeout (optional, default to 30 seconds)

But as the number of optional fields increases, so does the complexity of object construction.



<details>
<summary>Python</summary>

```python
class HttpRequestTelescoping:
    def __init__(self, url, method="GET", headers=None, query_params=None, body=None, timeout=30000):
        self.url = url
        self.method = method
        self.headers = headers if headers is not None else {}
        self.query_params = query_params if query_params is not None else {}
        self.body = body
        self.timeout = timeout

        print(f"HttpRequest Created: URL={url}, "
              f"Method={method}, "
              f"Headers={len(self.headers)}, "
              f"Params={len(self.query_params)}, "
              f"Body={body is not None}, "
              f"Timeout={timeout}")

    # Optional: add getter methods if needed

if __name__ == "__main__":
    req1 = HttpRequestTelescoping("https://api.example.com/data")

    req2 = HttpRequestTelescoping(
        "https://api.example.com/submit",
        "POST",
        None,
        None,
        '{"key":"value"}'
    )

    req3 = HttpRequestTelescoping(
        "https://api.example.com/config",
        "PUT",
        {"X-API-Key": "secret"},
        None,
        "config_data",
        5000
    )
```

</details>

<details>
<summary>C++</summary>

```cpp
class HttpRequestTelescoping {
private:
    string url;
    string method;
    map<string, string> headers;
    map<string, string> queryParams;
    string body;
    int timeout;

public:
    HttpRequestTelescoping(const string& url)
        : HttpRequestTelescoping(url, "GET") {}

    HttpRequestTelescoping(const string& url, const string& method)
        : HttpRequestTelescoping(url, method, {}) {}

    HttpRequestTelescoping(const string& url, const string& method,
                           const map<string, string>& headers)
        : HttpRequestTelescoping(url, method, headers, {}) {}

    HttpRequestTelescoping(const string& url, const string& method,
                           const map<string, string>& headers,
                           const map<string, string>& queryParams)
        : HttpRequestTelescoping(url, method, headers, queryParams, "") {}

    HttpRequestTelescoping(const string& url, const string& method,
                           const map<string, string>& headers,
                           const map<string, string>& queryParams,
                           const string& body)
        : HttpRequestTelescoping(url, method, headers, queryParams, body, 30000) {}

    HttpRequestTelescoping(const string& url, const string& method,
                           const map<string, string>& headers,
                           const map<string, string>& queryParams,
                           const string& body, int timeout)
        : url(url), method(method), headers(headers),
          queryParams(queryParams), body(body), timeout(timeout) {
        cout << "HttpRequest Created: URL=" << url
             << ", Method=" << method
             << ", Headers=" << headers.size()
             << ", Params=" << queryParams.size()
             << ", Body=" << (!body.empty())
             << ", Timeout=" << timeout << endl;
    }

    // Optional: Getters can be added here
};

int main() {
    HttpRequestTelescoping req1("https://api.example.com/data");

    HttpRequestTelescoping req2(
        "https://api.example.com/submit",
        "POST",
        {},
        {},
        "{\"key\":\"value\"}"
    );

    HttpRequestTelescoping req3(
        "https://api.example.com/config",
        "PUT",
        {{"X-API-Key", "secret"}},
        {},
        "config_data",
        5000
    );

    return 0;
}
```

</details>
<br/>


## What’s Wrong with This Approach?

- **Error-Prone**: 
Clients must pass null for optional parameters they do not want to set, increasing the risk of bugs. 

## What is the Builder Pattern

wo ideas define the pattern:

1. **Step-by-step construction**: Instead of passing everything to a constructor at once, you set each field through individual method calls. You only call the methods for the fields you need.

2. **Fluent interface**: Each setter method returns the builder itself, allowing you to chain calls into a single readable expression that ends with `build()`.

## Implementing Builder
<details>
<summary>Python</summary>

```python
class HttpRequest:
    def __init__(self, builder):
        self.url = builder._url
        self.method = builder._method
        self.headers = dict(builder._headers)  # defensive copy
        self.query_params = dict(builder._query_params)
        self.body = builder._body
        self.timeout = builder._timeout

    def __str__(self):
        return (f"HttpRequest(url='{self.url}', method='{self.method}', "
                f"headers={self.headers}, query_params={self.query_params}, "
                f"body='{self.body}', timeout={self.timeout})")

    class Builder:
        def __init__(self, url):
            self._url = url  # required
            self._method = "GET"
            self._headers = {}
            self._query_params = {}
            self._body = None
            self._timeout = 30000

        def method(self, method):
            self._method = method
            # same Builder object (self), allowing method chaining.
            return self

        def add_header(self, key, value):
            self._headers[key] = value
            return self

        def add_query_param(self, key, value):
            self._query_params[key] = value
            return self

        def body(self, body):
            self._body = body
            return self

        def timeout(self, timeout):
            self._timeout = timeout
            return self

        def build(self):
            return HttpRequest(self)


if __name__ == "__main__":
    # Simple GET request
    get = HttpRequest.Builder("https://api.example.com/users") \
        .build()

    # POST with body and custom timeout
    post = HttpRequest.Builder("https://api.example.com/users") \
        .method("POST") \
        .add_header("Content-Type", "application/json") \
        .body('{"name":"Alice","email":"alice@example.com"}') \
        .timeout(5000) \
        .build()

    # Authenticated PUT with query parameters
    put = HttpRequest.Builder("https://api.example.com/config") \
        .method("PUT") \
        .add_header("Authorization", "Bearer token123") \
        .add_header("Content-Type", "application/json") \
        .add_query_param("env", "production") \
        .add_query_param("version", "2") \
        .body('{"feature_flag":true}') \
        .timeout(10000) \
        .build()

    print(get)
    print(post)
    print(put)
```

</details>

<details>
<summary>C++</summary>

```cpp
class HttpRequest {
private:
    string url;
    string method;
    map<string, string> headers;
    map<string, string> queryParams;
    string body;
    int timeout;

    // Private constructor
    HttpRequest(const string& url, const string& method,
                const map<string, string>& headers,
                const map<string, string>& queryParams,
                const string& body, int timeout)
        : url(url), method(method), headers(headers),
          queryParams(queryParams), body(body), timeout(timeout) {}

public:
    string getUrl() const {
        return url;
    }
    string getMethod() const {
        return method;
    }

    void print() const {
        cout << "HttpRequest{url='" << url << "', method='" << method
             << "', headers=" << headers.size()
             << ", queryParams=" << queryParams.size()
             << ", body='" << body << "', timeout=" << timeout << "}" << endl;
    }

    class Builder {
    private:
        string url;
        string method = "GET";
        map<string, string> headers;
        map<string, string> queryParams;
        string body;
        int timeout = 30000;

    public:
        explicit Builder(const string& url) : url(url) {}

        Builder& setMethod(const string& m) {
            method = m;
            return *this;
        }

        Builder& addHeader(const string& key, const string& value) {
            headers[key] = value;
            return *this;
        }

        Builder& addQueryParam(const string& key, const string& value) {
            queryParams[key] = value;
            return *this;
        }

        Builder& setBody(const string& b) {
            body = b;
            return *this;
        }

        Builder& setTimeout(int t) {
            timeout = t;
            return *this;
        }

        HttpRequest build() const {
            return HttpRequest(url, method, headers, queryParams, body, timeout);
        }
    };
};

int main() {
    // Simple GET request
    HttpRequest get = HttpRequest::Builder("https://api.example.com/users")
        .build();

    // POST with body and custom timeout
    HttpRequest post = HttpRequest::Builder("https://api.example.com/users")
        .setMethod("POST")
        .addHeader("Content-Type", "application/json")
        .setBody("{\"name\":\"Alice\",\"email\":\"alice@example.com\"}")
        .setTimeout(5000)
        .build();

    // Authenticated PUT with query parameters
    HttpRequest put = HttpRequest::Builder("https://api.example.com/config")
        .setMethod("PUT")
        .addHeader("Authorization", "Bearer token123")
        .addHeader("Content-Type", "application/json")
        .addQueryParam("env", "production")
        .addQueryParam("version", "2")
        .setBody("{\"feature_flag\":true}")
        .setTimeout(10000)
        .build();

    get.print();
    post.print();
    put.print();

    return 0;
}
```

</details>
<br/>



