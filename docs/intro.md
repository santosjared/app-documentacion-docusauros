---
sidebar_position: 1
---

# Tutorial para microservicios aplicaciones

### ¿Qué nececitas?

- [Node.js](https://nodejs.org/en/download/) version v18.15.0 o para arriba:
- Haber descargado el **sistema central backend**

## Pasos a seguir
### Paso 1
Descarga el sistema principal **sistema central backend** del siguiente repositorio github(https://github.com) o por link del repositorio de servidor local GitLab gitlab(http://10.10.214.219:3300/sistema-central-backend.get)
- ejecutal el siguiente comando: para github
```bash
git clone https://github.com/santosjared/ssistema-central-backend.git
```
comando para clonar del repositorio del servidor local GitLab
```bash
git clone http://10.10.214.219:3300/sistema-central-backend.get
```
### Paso 2
- ingresa al achivo sistema-central-backend
```bash
cd sistema-central-backend
```
### Paso 3
- En este archivo clona las **configuraciones de aplicaciones** de los sguientes repositorios github(https://github.com/santosjared/apps.git) o del repositorio del servidor local gitlab(http://10.10.214.219:3300/santosjared/apps.git)

- ejecutal el siguiente comando: para github
```bash
git clone https://github.com/santosjared/apps.git
```
comando para clonar del repositorio del servidor local GitLab
```bash
git clone http://10.10.214.219:3300/santosjared/apps.git
```
### Paso 4
- Instala las dependencias del sistema 
```bash
npm install
```
### paso 5
#### Start your site

Run the development server:

```bash
npm run start:dev
```
La aplicacion arrancará en http://localhost:3100/api
## Codigo de documentacion de microservicios aplicaciones 
link del repositorio:
https://github.com/santosjared/app-documentacion-docusauros.git
Los comando para instalar en tu computador local ejecuta los siguintes comnado 
- Clona del repositorio de github
```bash
git clone https://github.com/santosjared/app-documentacion-docusauros.git
```
- Instala las dependencias 
```bash
npm install
```
- Arranca el proyecto

```bash
npm run start
```
**Organizacion:** FUNDACION DE SOFTWARE LIBRE
**Autor:** Santos Machaca Lopez