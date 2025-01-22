# Setting Up a Proxy in Axios

[![Promo](https://github.com/luminati-io/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.com/) 

This Axios proxy guide covers the following topics:

1. [Axios and Proxies](#axios-and-proxies)
2. [Using a Proxy in Axios](#using-a-proxy-in-axios)
   - [HTTP/HTTPS Proxies](#httphttps-proxies)
   - [SOCKS Proxies](#socks-proxies)
3. [Axios Proxy: Advanced Use Cases](#axios-proxy-advanced-use-cases)
   - [Setting a Proxy Globally](#setting-a-proxy-globally)
   - [Dealing With Proxy Authentication in Axios](#dealing-with-proxy-authentication-in-axios)
   - [Setting Proxies via Environment Variables](#setting-proxies-via-environment-variables)
   - [Implementing Rotating Proxies](#implementing-rotating-proxies)
4. [Conclusion](#conclusion)

## Axios and Proxies

[Axios](https://axios-http.com/) is one the most widely-used HTTP clients in the JavaScript ecosystem. It offers a Promise-based, easy-to-use, intuitive API for performing HTTP requests and dealing with custom headers, configurations, and cookies.

By routing your Axios requests through a proxy, you can mask your IP address, making it more challenging for the target server to identify and block you.

## Using a Proxy in Axios

Let's set up an HTTP, HTTPS, or SOCKS proxy in Axios. Install the `axios` npm package:

```bash
npm install axios
```

In Node.js, Axios natively supports HTTP and HTTPS proxies via the [`proxy`](https://github.com/axios/axios#request-config) config. So, if you want to use HTTP/HTTPS proxies with Axios in a Node.js application, there is nothing else todo here.

If you instead want to use a non-HTTP/S proxy, you need to rely on the [Proxy Agents](https://github.com/TooTallNate/proxy-agents) project. This provides `http.Agent` implementations to integrate Axios with proxies in different protocols:

- HTTP and HTTPS proxies: [`https-proxy-agent`](https://github.com/TooTallNate/proxy-agents/blob/main/packages/https-proxy-agent)
- SOCKS, SOCKS5, and SOCKS4: [`socks-proxy-agent`](https://github.com/TooTallNate/proxy-agents/blob/main/packages/socks-proxy-agent)
- PAC-\*: [`pac-proxy-agent`](https://github.com/TooTallNate/proxy-agents/blob/main/packages/pac-proxy-agent)

### HTTP/HTTPS Proxies

The URL of your HTTP/HTTPS proxy should look like this:

```
"<PROXY_PROTOCOL>://<PROXY_HOST>:<PROXY_PORT>"
```

- `<PROXY_PROTOCOL>` will be “http” for HTTP proxies and “https” for HTTPS proxies.
- `<PROXY_HOST>` is generally a raw IP.
- `<PROXY_PORT>` is the port the proxy server listens to.

For example, suppose this is the URL of your HTTP proxy:

```
"http://47.88.62.42:80"
```

You can set this proxy in Axios as follows:

```js
axios.get(targetURL, {

    proxy: { 

        protocol: "http", 

        host: "47.88.62.42",

        port: 80

    }

})
```

To verify that the above Axios proxy approach works, retrieve the URL of a free HTTP or HTTPS proxy server. Try this example:

```
Protocol: HTTP; IP Address: 52.117.157.155; Port: 8002
```

The complete proxy URL will be `http://52.117.157.155:8002`.

To verify that the proxy works as expected, target the [/ip](https://httpbin.io/ip) endpoint from the HTTPBin project. This public API returns the IP of the incoming request, so it should return the IP of the proxy server.

The snippet of the Node.js script will be:

```js
import axios from "axios"

async function testProxy() {

    // perform the desired request through the HTTP proxy

const response = await axios.get("https://httpbin.io/ip", {
    proxy: {  
        protocol: "http",  
        host: "52.117.157.155",
        port: 8002
    }
});

    // print the result

    console.log(response.data)

}

testProxy()
```

Execute the script, and it should log:

```js
{ "origin": "52.117.157.155" }
```

> **Warning**:\
> You will not get the same result if you run the script, because free proxy services are unreliable, slow, error-prone, data-greedy, and short-lived.

### SOCKS Proxies

If you try to set the “socks” string in the protocol field of the proxy config object, you will get the following error:

```js
AssertionError [ERR_ASSERTION]: protocol mismatch

  // ...

 {

  generatedMessage: false,

  code: 'ERR_ASSERTION',

  actual: 'dada:',

  expected: 'http:',

  operator: '=='

}
```

That is because Axios does not natively support SOCKS proxies. Add the `socks-proxy-agent` npm library to your project’s dependencies:

```bash
npm install socks-proxy-agent
```

This package allows you to connect to a SOCKS proxy server while making HTTP or HTTPS requests in Axios.

Then, import the SOCKS proxy agent implementation from the library:

```js
const SocksProxyAgent = require("socks-proxy-agent")
```

Or if you are an ESM user:

```js
import { SocksProxyAgent } from "socks-proxy-agent"
```

Suppose this is the URL of your SOCKS proxy:

```
"socks://183.88.74.73:4153"
```

> **Note**:\
> The proxy protocol can be either “socks”, “socks5”, or “socks4”.

Store it in a variable and pass it to the `SocksProxyAgent` constructor:

```js
const proxyURL = "socks://183.88.74.73:4153"

const proxyAgent = new SocksProxyAgent(proxyURL)
```

`SocksProxyAgent()` initializes an `http.Agent` instance to perform HTTP/HTTPS requests through the proxy URL.

You can now use a SOCKS proxy with Axios as follows:

```js
axios.get(targetURL, { 

    httpAgent: proxyAgent,     

    httpsAgent: proxyAgent 

})
```

`httpAgent` and `httpsAgent` define the custom agent to use when performing HTTP and HTTPS requests, respectively. In other words, the HTTP or HTTPS request made by Axios will go through the specified SOCKS proxy. In a similar way, you can use the [`https-proxy-agent`](https://www.npmjs.com/package/https-proxy-agent) npm package as an alternative way to set HTTP/HTTPS proxies in Axios.

Put it all together:

```js
import axios from "axios"

import { SocksProxyAgent } from "socks-proxy-agent"

async function testProxy() {

    // replace with the URL of your SOCKS proxy 

    const proxyURL = "socks://183.88.74.73:4153"

    // define the HTTP/HTTPS proxy agent

    const proxyAgent = new SocksProxyAgent(proxyURL)

    // perform the request via the SOCKS proxy

    const response = await axios.get("https://httpbin.io/ip", { 

        httpAgent: proxyAgent,     

        httpsAgent: proxyAgent 

    })

    // print the result

    console.log(response.data) // { "origin": "183.88.74.73" }

}

testProxy()
```

Follow the link for other examples of [how to configure a SOCKS proxy in Axios](https://writech.run/blog/how-to-use-a-socks-proxy-in-axios-6c0355a2e013/).

## Axios Proxy: Advanced Use Cases

### Setting a Proxy Globally

You can set a proxy globally by specifying it directly in an Axios instance:

```js
const axiosInstance = axios.create({

    proxy: { 

        protocol: "<PROXY_PROTOCOL>", 

        host: "<PROXY_HOST>",

        port: "<PROXY_PORT>" 

    },

    // other configs...

})
```

Or if you are a Proxy Agents user:

```js
// proxy Agent definition ...

const axiosInstance = axios.create({

    httpAgent: proxyAgent,     

    httpsAgent: proxyAgent 

})
```

Here’s how you configure Axios to use a SOCKS proxy globally:

```js
import { SocksProxyAgent } from "socks-proxy-agent";

const proxyURL = "socks://183.88.74.73:4153";

// Create a SOCKS proxy agent
const proxyAgent = new SocksProxyAgent(proxyURL);

// Create an Axios instance with the SOCKS proxy
const axiosInstance = axios.create({
    httpAgent: proxyAgent, // for HTTP requests
    httpsAgent: proxyAgent, // for HTTPS requests
    // other configs...
});
```

All requests made with `axiosInstance` will now automatically go through the specified proxy.

### Dealing With Proxy Authentication in Axios

To allow only paying users access to premium proxies, proxy providers protect them with authentication. Trying to connect to an authenticated proxy without a username and password will result in a [407 Proxy Authentication Required](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/407) error.

In particular, here is the syntax of the URL of an authenticated proxy:

```
[<PROTOCOL>://]<USERNAME>:<PASSWORD>@<HOST>[:<PORT>]
```

For example, a real-world URL to connect to an authenticated proxy might be:

```
http://admin:lK4w90MEe45YIkOpk@156.127.0.192:8391
```

In this case, the proxy URL field would be:

- `<PROTOCOL>:HTTP`
- `<HOST>:156.127.0.192`
- `<PORT>:8391`
- `<USERNAME>:admin`
- `<PASSWORD>:lK4w90MEe45YIkOpk`

To deal with proxy authentication in Axios, specify the username and password in the `authfield` of `proxy`:

```js
axios.get(targetURL, {

    proxy: { 

        protocol: "http", 

        host: "156.127.0.192",

        port: "8381",

        auth: {

            username: "admin",

            password: "lK4w90MEe45YIkOpk"

        }

    }

})
```

If you are instead a Proxy Agents user, you have two ways to deal with authentication:

1. Add the credentials directly in the proxy URL:

```js
var proxyAgent = new SocksProxyAgent("http://admin:[email protected]:8391")
```

2. Set the `username` and `password` options in a [URL](https://nodejs.org/api/url.html) object:

```js
const proxyOpts = new URL("http://156.127.0.192:8391")

proxyOpts.username = "admin"

proxyOpts.password = "lK4w90MEe45YIkOpk"

const proxyAgent = new SocksProxyAgent(proxyOpts)
```

The same approaches also work with HttpsProxyAgent.

### Setting Proxies via Environment Variables

Another way to configure a proxy globally in Axios is by setting the following environment variables:

- `HTTP_PROXY`: The URL of the proxy server to use for HTTP requests.
- `HTTPS_PROXY`: The URL of the proxy server to use for HTTPS requests.

On Linux or macOS, you can set them like this:

```bash
export HTTP_PROXY = "[<PROTOCOL>://]<USERNAME>:<PASSWORD>@<HOST>[:<PORT>]"

export HTTPS_PROXY = "[<PROTOCOL>://]<USERNAME>:<PASSWORD>@<HOST>[:<PORT>]"
```

When Axios detects these environment variables, it reads from them the proxy settings, including the credentials for authentication. Set the `proxy` field to `false` to make Axios ignore those environment variables. Keep in mind that you can also define a `NO_PROXY` env as a comma-separated list of domains that should not be proxied.

Note that the same mechanism also works when [using proxies in cURL](https://brightdata.com/blog/proxy-101/curl-with-proxies).

### Implementing Rotating Proxies

To prevent the target site from blocking your proxy's IP address, ensure that each request you perform originates from a different proxy server:

1. Define a list of objects, each containing the information to connect to a different proxy.
2. Randomly select a proxy object before each request.
3. Configure the selected proxy in Axios.

The approach outlined above assumes that you have access to a pool of reliable proxy servers, such as the [rotating proxies](https://brightdata.com/solutions/rotating-proxies) that Bright Data offers.

## Conclusion

Bright Data controls the best proxy servers in the world, serving Fortune 500 companies and over 20,000 customers. Its worldwide proxy network involves:

*   [Datacenter proxies](https://brightdata.com/proxy-types/datacenter-proxies) – Over 770,000 datacenter IPs.
*   [Residential proxies](https://brightdata.com/proxy-types/residential-proxies) – Over 72M residential IPs in more than 195 countries.
*   [ISP proxies](https://brightdata.com/proxy-types/isp-proxies) – Over 700,000 ISP IPs.
*   [Mobile proxies](https://brightdata.com/proxy-types/mobile-proxies) – Over 7M mobile IPs.

[Create a free Bright Data account](https://brightdata.com/#popup-155639) today to try our proxy servers.
