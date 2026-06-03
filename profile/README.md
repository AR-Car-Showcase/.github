# AR Car Showcase

An immersive **Augmented Reality Car Showcase** platform that lets users browse, customize, and experience vehicles in their real-world environment. Built with **React Native**, **Spring Boot**, and **Blender** for real-time 3D model generation.

---

## 🚀 Platform Overview

AR Car Showcase (ARCS) combines mobile AR technology with cloud-based 3D model generation to deliver an interactive car shopping experience. Users can browse the full car catalog, customize colors and materials in a 3D studio, and project their customized vehicle into their real-world environment using Augmented Reality.

---

## 🏗️ System Architecture

```
┌─────────────────────────────────────────────────────────┐
│          Mobile Client (React Native / Expo)            │
│  • 3D Customization Studio (React Three Fiber)          │
│  • AR Projection (ViroReact - ARCore/ARKit)             │
│  • Authentication & User Preferences                     │
└────────────────┬────────────────────────────────────────┘
                 │
                 │ HTTPS (api.arcarshowcase.com)
                 ↓
┌─────────────────────────────────────────────────────────┐
│           Cloudflare (DNS + SSL + Security)             │
└────────────────┬────────────────────────────────────────┘
                 │
                 ↓
┌─────────────────────────────────────────────────────────┐
│         Nginx Reverse Proxy (Oracle Cloud VM 1)         │
└────────────────┬────────────────────────────────────────┘
                 │
                 ↓
┌─────────────────────────────────────────────────────────┐
│              Spring Boot API Server (VM 1)              │
│  • RESTful API (Car Catalog, Auth, Customization)      │
│  • JWT Authentication                                    │
│  • Recommendation Engine                                │
│  • PostgreSQL Database (Docker)                         │
│  • Redis Cache (Docker)                                 │
│  • Python Recommendation Service (Docker)               │
└────────────────┬────────────────────────────────────────┘
                 │
                 │ Internal HTTP Request
                 ↓
┌─────────────────────────────────────────────────────────┐
│        Blender 3D Generation Service (VM 2)             │
│  • Python/Flask API                                     │
│  • Blender Headless Rendering                          │
│  • Oracle Object Storage Integration                    │
│  • OCI Instance Principal Authentication                │
└────────────────┬────────────────────────────────────────┘
                 │
                 │ Upload Generated Models
                 ↓
┌─────────────────────────────────────────────────────────┐
│         Oracle Cloud Object Storage (ar-models)         │
│  • models/ (base vehicle models)                        │
│  • generated/ (user customizations)                     │
│  • Public bucket with pre-authenticated requests        │
└─────────────────────────────────────────────────────────┘
```

---

## 🛠️ Tech Stack

### Frontend (Mobile)
- **React Native** 0.81.5 with **Expo** SDK 54
- **ViroReact** for AR functionality (ARCore/ARKit)
- **React Three Fiber** + **Three.js** for 3D model viewer
- **TypeScript** for type safety
- **Expo Router** for file-based navigation

### Backend (Server)
- **Spring Boot** 3.x (Java 17) with Maven
- **PostgreSQL** for persistence
- **Redis** for caching and session management
- **Spring Security** + JWT for authentication
- **Python FastAPI** for recommendation service
- **Docker Compose** for containerization

### 3D Processing
- **Python Flask** API
- **Blender** headless mode for .glb generation
- Material and color manipulation via Blender Python API

### Infrastructure
- **Oracle Cloud** Free Tier (2 Ubuntu VMs)
- **Nginx** reverse proxy with Let's Encrypt SSL
- **Cloudflare** for DNS, SSL termination, and security
- **Oracle Object Storage** for static assets and generated models
- **OCI Instance Principals** for secure cloud resource access

---

## ✨ Features

- 🔭 **Augmented Reality** – Project 3D car models into your real environment
- 🎨 **Real-Time Customization** – Modify body color, rims, interior, and more
- 🚗 **Full Car Catalog** – Browse vehicles with filtering by brand, fuel type, body type, and budget
- 🤖 **AI Recommendations** – Personalized car suggestions based on preferences
- 🔐 **Secure Authentication** – JWT-based login with email verification via OTP
- 💾 **Personal Showroom** – Save and manage your customized designs
- 🌐 **Cross-Platform** – Supports Android and iOS

---

## 📦 Repositories

| Repository | Description |
|---|---|
| [**AR-Car-Showcase**](https://github.com/AdepuSriCharan/AR-Car-Showcase) | React Native mobile application |
| [**AR-Car-Showcase-Server**](https://github.com/AdepuSriCharan/AR-Car-Showcase-Server) | Spring Boot backend + Blender service |

---

## 🚀 Deployment Architecture

### Infrastructure

**Oracle Cloud Free Tier** — 2 Ubuntu VMs in the same VCN and subnet for secure private networking.

#### VM 1: Core Backend
- Spring Boot API server
- PostgreSQL database (Docker)
- Redis cache (Docker)
- Python recommendation service (Docker)
- Nginx reverse proxy with Let's Encrypt SSL

#### VM 2: 3D Generation Service
- Blender headless rendering
- Python Flask API
- Oracle Object Storage integration
- OCI Instance Principal authentication

### Networking

- **Domain**: `arcarshowcase.com` managed via **Cloudflare**
- **API Endpoint**: `api.arcarshowcase.com` (proxied through Cloudflare)
- **SSL**: Let's Encrypt certificates with Cloudflare in Full (Strict) mode
- **Internal Communication**: VMs communicate via private IPs within the VCN

### Request Flow

```
User → Cloudflare → api.arcarshowcase.com → Nginx → Spring Boot (8080)
                                                           ↓
                                            Internal Request to VM 2
                                                           ↓
                                                  Blender Service (5000)
                                                           ↓
                                           Upload to Oracle Object Storage
                                                           ↓
                                              Return public model URL
```

### Storage

**Oracle Object Storage** — Bucket `ar-models`
- `models/` — Base vehicle 3D models (.glb files)
- `generated/` — User-customized models created by Blender service
- Public bucket with pre-authenticated requests for mobile client access

### Authentication

**OCI Instance Principals** configured with:
- Dynamic Groups for VM-based service authentication
- IAM Policies for read/write access to Object Storage
- No API keys or credentials stored on servers

### Memory Optimization

Added **4 GB swap file** on VM 1 to improve stability for the Spring Boot application running on a 1 GB RAM instance.

### Email

**Custom Domain Email** — `contact@arcarshowcase.com` configured for user support and notifications.

---

## 🔄 3D Model Generation Workflow

1. User customizes car in mobile app (selects colors/materials)
2. App sends request to Spring Boot API
3. Spring Boot forwards request to Blender Service (VM 2)
4. Blender Service:
   - Downloads base model from Oracle Object Storage
   - Applies material and color changes using Blender Python API
   - Exports customized model as .glb file
   - Uploads generated model to `ar-models/generated/`
5. Returns public URL to Spring Boot
6. Spring Boot returns URL to mobile app
7. App loads customized model in AR viewer

---

## 🔐 Security

- **JWT Authentication** for API access
- **Spring Security** with role-based access control
- **HTTPS/TLS** enforced via Let's Encrypt + Cloudflare
- **OCI Instance Principals** eliminate credential storage
- **Redis** for secure session management
- **Email Verification** via OTP for new account signup

---

## 🌐 Learn More

For detailed setup instructions, API documentation, and architecture diagrams, see the individual repository READMEs:
- **Mobile App**: [AR-Car-Showcase](https://github.com/AdepuSriCharan/AR-Car-Showcase)
- **Backend Server**: [AR-Car-Showcase-Server](https://github.com/AdepuSriCharan/AR-Car-Showcase-Server)

---

## 📄 License

MIT License — See individual repositories for details.
