# Preview Server

> Author: Charley

LayaAir IDE includes a built-in HTTP / HTTPS preview server for local project preview and debugging.

Through the preview service, developers can precisely control the preview service's access method, security policy, and multi-language testing behavior to adapt to different scenarios such as individual development, team collaboration, and multi-project parallel development.

## 1. Host Address (`liveServerHost`)

`liveServerHost` is used to specify the host address bound by the preview server.

**When this value is an empty string, the IDE will automatically obtain the current device's local IP address as the service address**, which is the most common and hassle-free configuration.

In actual development, whether to explicitly specify this parameter usually depends on the preview service's access range. If only debugging in the local browser, set it to `localhost` to avoid being accessed by other devices on the LAN with higher security. When needing to access the preview page on mobile phones, tablets, or other computers, set it to the current device's IP address in the LAN.

In multi-network interface environments (e.g., simultaneously connected to wired and wireless networks), explicitly specifying `liveServerHost` can also avoid the IDE automatically selecting an unavailable network interface, causing the "service started but cannot access" problem.

**Type and Default Value**

- Type: String
- Default: `""` (automatically use local IP)

## 2. HTTP Port (`liveServerPort`)

`liveServerPort` is used to specify the HTTP access port for the preview service. The IDE will listen on this port when starting the preview service and provide project resources to the browser through this port.

The default port is `18090`, which doesn't need modification in most cases. However, when another program on the local machine has occupied this port, or when needing to run multiple LayaAir projects simultaneously for comparison testing, you need to configure different port numbers for different projects to avoid conflicts.

After modifying the port, the preview access format is:

```
http://[host]:[liveServerPort]
```

If preview service startup fails and prompts that the port is occupied, it can usually be solved by changing the port or closing the program occupying the port.

**Type and Default Value**

- Type: Number
- Default: `18090`
- Range: `80 – 65535`

## 3. HTTPS Port (`liveServerSecurePort`)

`liveServerSecurePort` is used to configure the HTTPS preview service access port. When the project involves browser capabilities that must be used in an HTTPS environment (such as WebGPU, WebXR, etc.), testing can be performed through this port.

Similar to the HTTP port, this configuration mainly affects access method and doesn't change the project's own logic behavior. The IDE uses a self-signed certificate, so when accessing the HTTPS preview address in a browser, security warnings are normal and only for development and testing phases.

HTTPS preview address format:

```
https://[host]:[liveServerSecurePort]
```

**Type and Default Value**

- Type: Number
- Default: `18091`
- Range: `80 – 65535`

## 4. Protect Source Map (`protectSourcemap`)

`protectSourcemap` is used to control whether to apply access protection to Source Map files. Source Map describes the mapping relationship between compiled code and source code, which can significantly improve problem location efficiency during debugging, but may also expose source code structure.

In the development phase, disabling this option allows directly viewing source code and setting breakpoints through browser developer tools. In testing or external preview environments, it's usually recommended to keep it enabled by default to prevent `.map` files from being directly accessed.

Whether to enable this option is essentially a trade-off between "debugging convenience" and "source code security."

**Type and Default Value**

- Type: Boolean
- Default: `true`

## 5. Test Language (`testLanguage`)

`testLanguage` is used to specify the language environment used during preview, mainly for multi-language project testing needs. Through this configuration, you can quickly switch the language used for preview without modifying code or resource configuration.

This parameter is particularly useful when verifying multi-language text display, layout adaptation, and language resource completeness. For example, text length differences in different languages can affect UI layout. Switching test languages can detect problems early.

This value is usually filled with standard language codes, such as `zh-CN`, `en-US`, etc. When not set, the project default language is used.

**Type and Default Value**

- Type: String
- Default: None
