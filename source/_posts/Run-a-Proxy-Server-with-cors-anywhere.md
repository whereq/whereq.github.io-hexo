---
title: Run a Proxy Server with cors-anywhere
date: 2024-12-19 12:14:20
category:
- Node.js
- Proxy
- CORS
tags:
- Node.js
- Proxy
- CORS
---

#### 1. **Install Node.js**
   - Ensure you have Node.js installed on your machine. You can download it from [nodejs.org](https://nodejs.org/).
   - Verify the installation by running:
     ```bash
     node -v
     npm -v
     ```

#### 2. **Create a New Directory for Your Proxy Server**
   - Create a new directory for your proxy server:
     ```bash
     mkdir cors-proxy-server
     cd cors-proxy-server
     ```

#### 3. **Initialize a New Node.js Project**
   - Initialize a new Node.js project:
     ```bash
     npm init -y
     ```
   - This will create a `package.json` file.

#### 4. **Install `cors-anywhere`**
   - Install the `cors-anywhere` package:
     ```bash
     npm install cors-anywhere
     ```

#### 5. **Create the Proxy Server Script**
   - Create a new file named `server.js` in the project directory:
     ```bash
     touch server.js
     ```
   - Add the following code to `server.js`:

---

### `server.js` (JavaScript Implementation)

Here’s the complete implementation of the proxy server in **JavaScript**:

```javascript
const cors_proxy = require("cors-anywhere");

// Define the host and port for the proxy server
const host = "127.0.0.1"; // Localhost
const port = 8080; // Port to run the proxy

// Create and start the proxy server
cors_proxy
    .createServer({
        originWhitelist: [], // Allow all origins (you can restrict this to specific domains)
        requireHeader: ["origin", "x-requested-with"], // Require these headers
        removeHeaders: ["cookie", "cookie2"], // Remove sensitive headers
    })
    .listen(port, host, () => {
        console.log(`CORS Anywhere proxy server is running on http://${host}:${port}`);
    });
```

---

### 6. **Run the Proxy Server**
   - Start the proxy server:
     ```bash
     node server.js
     ```
   - You should see the following output:
     ```
     CORS Anywhere proxy server is running on http://127.0.0.1:8080
     ```

---

### Using the Proxy Server in Your Application

Once the proxy server is running, you can use it in your application to bypass CORS restrictions.

#### Example: Fetching Data in a React Application

```javascript
const proxyUrl = "http://127.0.0.1:8080/"; // Your local proxy server
const apiUrl = "https://dog.ceo/api/breeds/list/all"; // The API you want to access

fetch(proxyUrl + apiUrl)
    .then((response) => response.json())
    .then((data) => console.log(data))
    .catch((error) => console.error("Error:", error));
```

---

### Summary of Steps

1. Install Node.js and initialize a new project.
2. Install the `cors-anywhere` package.
3. Create a `server.js` file with the proxy server implementation.
4. Run the proxy server using `node server.js`.
5. Use the proxy server in your application by prefixing your API URL with the proxy URL.

---

### Notes

- **Security**: By default, the proxy server allows all origins (`originWhitelist: []`). You can restrict this to specific domains for better security.
- **Production**: Running a local proxy server is fine for development, but for production, consider deploying the proxy server to a cloud service (e.g., Heroku, Vercel) or using a backend server as a middleman.
- **Rate Limits**: Some APIs may have rate limits or restrictions on the number of requests you can make. Be mindful of this when using a proxy.

This setup will allow you to bypass CORS restrictions and make API requests from your application during development.

