# WhatsApp Web Reverse Proxy with Nginx on Fly.io

This project implements a reverse proxy using Nginx to access WhatsApp Web through your own application hosted on Fly.io.

# Why?

Well, let's say that your network does not allow you to access it, and you need to.
So, reverse proxy!

## What This Project Does

This application creates a reverse proxy that redirects requests from your custom domain to WhatsApp Web (`web.whatsapp.com`). Essentially, when someone accesses your application, Nginx forwards the request to WhatsApp Web and returns the response.

### Limitations and Considerations

> **Important Note**: WhatsApp Web implements various security measures to prevent proxying and embedding, including CORS protections, framing controls, and origin checks. This solution may face significant technical limitations and there's no guarantee of full functionality due to these protections.

## Prerequisites

- A [Fly.io](https://fly.io) account
- Basic command line knowledge (optional, as we'll be using the web interface)

## How to Deploy on Fly.io

### 1. File Structure

Create the following file structure in your project:

```
your-project/
├── Dockerfile
└── nginx.conf
```

### 2. Nginx Configuration

Create the `nginx.conf` file with the following content:

```nginx
server {
    listen 80;
    server_name _;

    location / {
        proxy_pass https://web.whatsapp.com;
        proxy_set_header Host web.whatsapp.com;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto https;
        proxy_set_header User-Agent $http_user_agent;
        
        # Additional headers
        proxy_set_header Accept "*/*";
        proxy_set_header Accept-Language "en-US,en;q=0.9";
        proxy_set_header Cookie $http_cookie;
        proxy_set_header Origin "https://web.whatsapp.com";
        proxy_set_header Referer "https://web.whatsapp.com/";
        
        # WebSocket configuration
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        
        # Enhanced SSL configuration
        proxy_ssl_server_name on;
        proxy_ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
        proxy_ssl_verify off;
        proxy_ssl_session_reuse on;
        
        # Increased timeouts
        proxy_connect_timeout 120s;
        proxy_read_timeout 120s;
        proxy_send_timeout 120s;
        
        # Disable cache
        proxy_no_cache 1;
        proxy_cache_bypass 1;
        add_header Cache-Control no-cache;
        
        # Try to bypass framing protections
        add_header X-Frame-Options "SAMEORIGIN";
        
        # Buffer settings
        proxy_buffer_size 128k;
        proxy_buffers 4 256k;
        proxy_busy_buffers_size 256k;
    }
}
```

### 3. Create Dockerfile

Create a `Dockerfile` with the following content:

```dockerfile
FROM nginx:alpine

# Remove default configuration
RUN rm /etc/nginx/conf.d/default.conf

# Copy your custom configuration
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Expose port 80
EXPOSE 80

# Start Nginx
CMD ["nginx", "-g", "daemon off;"]
```

### 4. Deploy on Fly.io

#### Using the Web Interface

1. Log in to your [Fly.io dashboard](https://fly.io/dashboard)
2. Click on "Create App" or "Launch an App"
3. Choose "Upload Files" method
4. Upload your `Dockerfile` and `nginx.conf`
5. Follow the prompts to configure your app:
   - Choose a name for your app
   - Select a region
   - Configure resources as needed
6. Complete the deployment process

#### Using the Fly CLI (Alternative)

If you prefer using the command line:

```bash
# Install Fly CLI if you haven't already
curl -L https://fly.io/install.sh | sh

# Login to Fly
fly auth login

# Launch a new app in the current directory
fly launch

# Deploy your app
fly deploy
```

### 5. Monitor Your Application

1. Go to your Fly.io dashboard
2. Click on your application
3. Navigate to the "Logs" tab to see real-time logs
4. Check for any error messages, particularly 502 Bad Gateway errors

## Troubleshooting

### Common Issues

1. **502 Bad Gateway Error**:
   - This usually indicates that Nginx cannot establish a connection with WhatsApp Web
   - Check your Nginx logs for more specific error messages
   - WhatsApp may be blocking the proxy request due to security measures

2. **Default Nginx Page**:
   - If you see the default "Welcome to nginx" page, your configuration is not being loaded
   - Make sure your `nginx.conf` is correctly placed and the Dockerfile is properly removing the default configuration

3. **WhatsApp Security Measures**:
   - WhatsApp Web may detect that it's being accessed through a proxy and refuse to function properly
   - You might see loading screens or authentication failures

## Alternative Approaches

If you're trying to integrate with WhatsApp, consider these more reliable alternatives:

1. **WhatsApp Business API**: Official solution for business integration
2. **WhatsApp Business Cloud API**: API for sending and receiving messages
3. **Third-party services**: Services like Twilio offer WhatsApp integration

## Legal Considerations

Using WhatsApp Web through a proxy may violate WhatsApp's Terms of Service. This project is for educational purposes only. Use at your own risk.

## Resources

- [Fly.io Documentation](https://fly.io/docs/)
- [Nginx Documentation](https://nginx.org/en/docs/)
- [WhatsApp Business API](https://developers.facebook.com/docs/whatsapp/api/reference)