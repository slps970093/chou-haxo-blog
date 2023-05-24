---
title: Nuxt 3 在 docker 建置開發環境，並搭配 HMR 
date: 2023-05-24 18:26:49
tags: [docker, nuxt3, traefik]
---
## trefik v2 設定

**/etc/traefik.yml** 
```
entryPoints:
  web:
    address: ":80"

  websecure:
    address: ":443"

api:
  dashboard: true
  insecure: true

tls:
  certificates:
    - certFile: "/certificates/lara-media-web-local/domain.cert"
      keyFile: "/certificates/lara-media-web-local/domain.key"

http:
  middlewares:
    test-redirectscheme:
      redirectScheme:
        scheme: https

log:
  filePath: "/logs/traefik.log"

accessLog:
  filePath: "/logs/access.log"

providers:
  docker:
    endpoint: "unix:///var/run/docker.sock"
```


## nuxt 3 專案設定

這邊主要有幾個重點

1. 將 npm run dev 修改為可以走 80 (可選/若不修改需指定loadbalancer.server.port 80--> 你指定的 PORT)
2. HMR 指定成 websocket 模式 (port 為 22300 可自訂)

### 教學

1. 填寫 **Dockerfile** 這邊我用我自己寫的當範本
我將工作目錄設定在 **/var/www/html**
```
FROM node:18.16-alpine3.16

WORKDIR /var/www/html
RUN npm install -g typescript ts-node ts-node-dev
# https://stackoverflow.com/questions/67903114/javascript-heap-out-of-memory-in-docker-image-run
ENV NODE_OPTIONS=--max_old_space_size=2048
```

2. 填寫 **docker-compose.yml**
```
version: "3.0"

services:
  nuxt-server:
    build:
      context: docker/image/dev
    volumes:
      - .:/var/www/html
      # https://dev.to/nklsw/how-to-create-a-dockerized-nuxt-3-development-environment-1p0a
      - /var/www/html/node_modules
    networks:
      - traefik-network
      - web-director-network
    command: sh -c "npm ci &&npm run dev-container-traefik"
    expose:
      - 22300
    labels:
      - "traefik.docker.network=traefik-network"
      # 首頁
      - traefik.http.routers.web-director-nuxt3.service=web-director-nuxt3
      - traefik.http.routers.web-director-nuxt3.rule=Host(`web-director.local`)
      - traefik.http.services.web-director-nuxt3.loadbalancer.server.port=80
      # 對應 /etc/traefik.yml 裡面的 entrypoint 要告訴 traefik 進入點
      - traefik.http.routers.web-director-nuxt3.entrypoints=web 
      # 首頁 https
      - traefik.http.routers.web-director-nuxt3-https.service=web-director-nuxt3-https
      - traefik.http.routers.web-director-nuxt3-https.entrypoints=websecure
      - traefik.http.routers.web-director-nuxt3-https.rule=Host(`web-director.local`)
      - traefik.http.services.web-director-nuxt3-https.loadbalancer.server.port=80
      - traefik.http.routers.web-director-nuxt3-https.tls=true
      # HMR Support
      # https://github.com/nuxt/nuxt/issues/12748#issuecomment-1397234718
      - traefik.http.routers.web-director-nuxt3-vitesocket.service=web-director-nuxt3-vitesocket
      # 由於 hmr port 是 22300 所以要轉
      - traefik.http.services.web-director-nuxt3-vitesocket.loadbalancer.server.port=22300
      - traefik.http.routers.web-director-nuxt3-vitesocket.rule=Host(`web-director.local`) && PathPrefix(`/_nuxt/hmr/`)
      - traefik.http.routers.web-director-nuxt3-vitesocket.entrypoints=web,websecure
      - traefik.http.routers.web-director-nuxt3-vitesocket.tls=true

networks:
  web-director-network:
    driver: bridge
  traefik-network:
    external: true
```

3. 修改 **nuxt.config.ts** 增加下面的

```
vite: {
    server: {
        hmr: {
            protocol: 'wss',
            port: 22300,
            clientPort: 443,    // 用戶指定走的 port
            path: 'hmr/',
            timeout: 3,
        }
    }
},
```
**完整範本**
```
export default defineNuxtConfig({
    modules: [
        '@pinia/nuxt',
        '@pinia-plugin-persistedstate/nuxt',
        '@nuxtjs/i18n',
    ],
    runtimeConfig: {
        public: {
            API_FRONTEND_DOMAIN: ( process.env.API_FRONTEND_DOMAIN ?? "http://api.frontend.ezlive.local"),
            API_ADMIN_DOMAIN: ( process.env.API_ADMIN_DOMAIN ?? "http://api.admin.ezlive.local"),
            API_DOMAIN: ( process.env.API_DOMAIN ?? "http://api-ezlive.local"),
            LIVE_SOCKET_IO_DOMAIN: ( process.env.LIVE_SOCKET_IO_DOMAIN ?? "http://localhost:8888" )
        }
    },
    vite: {
        server: {
            hmr: {
                protocol: 'wss',
                port: 22300,        // 對外PORT
                clientPort: 443,    // 用戶端連結 PORT
                path: 'hmr/',
                timeout: 3,
            }
        }
    },
})

```
## 參考資料
[How to create a dockerized Nuxt 3 development environment](https://dev.to/nklsw/how-to-create-a-dockerized-nuxt-3-development-environment-1p0a)
[Exclude a Sub-Folder When Adding a Volume to Docker](https://www.baeldung.com/ops/docker-exclude-sub-folder-when-adding-volume)
[Nuxt development server continiously restarts with --https](https://github.com/nuxt/nuxt/issues/12748#issuecomment-1397234718)
[Nuxt Configuration Reference](https://nuxt.com/docs/api/configuration/nuxt-config#nuxt-configuration-reference)
