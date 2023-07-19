# chatgpt-web
Pure Javascript ChatGPT demo based on OpenAI API

A pure JavaScript implementation of the ChatGPT project, based on the OpenAI API.

Deploy an HTML file to use it.

Supports features such as copy/update/refresh session, voice input, text-to-speech, and many [custom options](#custom-options).

Supports session search, dark mode, custom avatars, [PWA application](#pwa-application), API quota display, and more.

Reference projects:
[markdown-it](https://github.com/markdown-it/markdown-it),
[highlight.js](https://github.com/highlightjs/highlight.js),
[github-markdown-css](https://github.com/sindresorhus/github-markdown-css),
[chatgpt-html](https://github.com/slippersheepig/chatgpt-html),
[markdown-it-copy](https://github.com/ReAlign/markdown-it-copy),
[markdown-it-texmath](https://github.com/goessner/markdown-it-texmath),
[awesome-chatgpt-prompts-zh](https://github.com/PlexPt/awesome-chatgpt-prompts-zh)

![Example](https://raw.githubusercontent.com/xqdoo00o/chatgpt-web/main/example.png)

## Demo

[Online Preview](https://xqdoo00o.github.io/chatgpt-web/) (Requires configuration of OpenAI API endpoint and API key)

## Usage
### **Note: When deploying a reverse proxy interface, please pay attention to the [supported regions](https://platform.openai.com/docs/supported-countries) by OpenAI. Deploying on servers in unsupported regions may result in account suspension! It is recommended to configure HTTPS, as transmitting the API key in plain text over the public network using HTTP is highly insecure!**
___
- **Deploy HTML Only**

    Deploy the `index.html` file using any HTTP server. Open the webpage settings, enter the API key, and specify the OpenAI API endpoint.

    - If you can access `api.openai.com` normally, enter `https://api.openai.com/`.
    
    - If you cannot access `api.openai.com` normally, enter the reverse proxy address (you can use [Cloudflare Worker](https://github.com/xqdoo00o/openai-proxy) or similar). Note: The reverse proxy interface response must include the cross-origin header `Access-Control-Allow-Origin`.
    
- **Deploy HTML and OpenAI Reverse Proxy Interface Together**

    **Note: The server must be able to access `api.openai.com`, and there is no need to set the OpenAI API endpoint in the webpage.**
    
    - Using Nginx, the example configuration is as follows:

        ```
        map $http_authorization $gptauth {
            default  $http_authorization;
            # Configure the default API key after "Bearer ". If you only allow setting the API key on the webpage, please delete the next line.
            ""       "Bearer sk-your-token";
        }
        server {
            listen       80;
            server_name  example.com;
            # Enable gzip compression for the openai interface. Compression works well for large amounts of repetitive text and saves server-side bandwidth.
            gzip  on;
            gzip_min_length 1k;
            gzip_types text/event-stream;

            # If you need to deploy in a website subdirectory, such as "example.com/chatgpt", configure as follows:
            # location ^~ /chatgpt/v1 {
            location ^~ /v1 {
                proxy_pass https://api.openai.com/v1;
                proxy_set_header Host api.openai.com;
                proxy_ssl_name api.openai.com;
                proxy_ssl_server_name on;
                proxy_set_header Authorization $gptauth;
                proxy_pass_header Authorization;
                # Streaming transfer, buffering must be turned off, otherwise it will hang. Must be configured!!!
                proxy_buffering off;
            }
            # Keep consistent with the path of the reverse proxy interface above
            # location /chatgpt {
            location / {
                alias /usr/share/nginx/html/;
                index index.html;
            }
        }
        ```
        If the server cannot access `api.openai.com` normally, you can use socat for reverse proxy and HTTP proxy. Change the `proxy_pass` configuration to:
        ```
        proxy_pass https://127.0.0.1:8443/v1;
        ```
        And start socat:
        ```
        socat TCP4-LISTEN:8443,reuseaddr,fork PROXY:http-proxy-address:api.openai.com:443,proxyport=http-proxy-port
        ```
    - Using Caddy, which can automatically generate HTTPS certificates. The example configuration is as follows:

        ```
        yourdomain.example.com {
            reverse_proxy /v1/* https://api.openai.com {
                header_up Host api.openai.com
                header_up Authorization "{http.request.header.Authorization}"
                header_up Authorization "Bearer sk-your-token"
            }

            file_server / {
                root /var/wwwroot/chatgpt-web
                index index.html
            }
        }
        ```
        **Caddy 2.6.5 and later versions support the `https_proxy` and `http_proxy` environment variables. If the server cannot access `api.openai.com` normally, you can set the proxy environment variables first.**

## PWA Application
Deploy the files [icon.png](https://raw.githubusercontent.com/xqdoo00o/chatgpt-web/main/icon.png), [manifest.json](https://raw.githubusercontent.com/xqdoo00o/chatgpt-web/main/manifest.json), and [sw.js](https://raw.githubusercontent.com/xqdoo00o/chatgpt-web/main/sw.js) in the same directory as `index.html` to enable PWA application support.

**Note: If you rename `index.html`, you also need to modify `./index.html` in the `sw.js` file.**

**After deploying the PWA application, if you update the HTML file, you need to update `sw.js` at the same time, otherwise the update may not take effect.**

## Custom Options

- Left sidebar support: session search, create/rename/delete (session/folder), light/dark/auto theme mode, export/import/reset session and settings data, display API quota, display local storage.

- Optional GPT model: default is gpt-3.5. To use the gpt-4 model, you need to apply through OpenAI's form or use [ChatGPT-to-API](https://github.com/xqdoo00o/ChatGPT-to-API) to simulate the web ChatGPT for API usage (gpt-4 requires a Plus account).

- Optional OpenAI API endpoint: If you deploy a reverse proxy using Nginx or Caddy, you can leave this field empty.

- Optional API key: By default, it is not set. **If you want to use a custom API key in the webpage settings, it is recommended to configure HTTPS for the reverse proxy interface. Transmitting the API key in plain text over the public network using HTTP is highly insecure.**

- Optional user avatar: You can modify it to any image URL.

- Optional system roles: By default, it is not enabled. There are four preset roles, and they are dynamically loaded from [awesome-chatgpt-prompts-zh](https://github.com/PlexPt/awesome-chatgpt-prompts-zh).

- Optional role personality: By default, it is set to "flexible and creative," corresponding to the `top_p` parameter in the API documentation.

- Optional answer quality: By default, it is set to "balanced," corresponding to the `temperature` parameter in the API documentation.

- Modify typewriter speed: By default, it is set to "fast." The larger the value, the faster the speed.

- Allow continuous conversation: By default, it is enabled. Including context information in the conversation will increase API costs.

- Allow long replies: By default, it is disabled. **Enabling it may increase API costs and lose most of the context. For some replies that require a "continue" to be complete, there is no need to send "continue" when this option is enabled.**

- Select voice: By default, it is set to Bing Speech. Supports Azure Speech and system speech, which can be set separately for question and answer voices.

- Volume: By default, it is set to maximum.

- Speaking rate: By default, it is set to normal.

- Pitch: By default, it is set to normal.

- Allow continuous text-to-speech: By default, it is enabled. Continuously reads all the conversations until the end.

- Allow automatic text-to-speech: By default, it is disabled. Automatically reads new replies. **(On iOS, you need to enable "Settings - Auto-Play Video Previews," and on Safari for Mac, you need to enable "Settings for This Website - Allow All Auto-Play").**

- Support voice input: By default, it is set to recognize Mandarin Chinese. Long press the voice button to modify the recognition options. **Voice recognition requirements: Use a browser based on the Chrome kernel + HTTPS webpage or local webpage.** If clicking the voice button does not respond, it may be due to microphone permission not being granted or no microphone device being installed.
