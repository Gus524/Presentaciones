# Presentación: Inteligencia Artificial como Copiloto (El Desarrollador al Mando)

Esta es la presentación técnica e instructiva de la **Sesión 9**, diseñada para guiar al equipo de desarrollo en la aceleración de la migración de **9 sistemas legacy** a Clean Architecture / Hexagonal (.NET 10 + Angular 22) manteniendo intactas las reglas de arquitectura y calidad del proyecto.

---

## 🚀 Requisitos Previos

- **Runtime / Package Manager:** [Bun](https://bun.sh/) (v1.0+)

---

## 🛠️ Cómo Ejecutar la Presentación

### 1. Instalación de Dependencias
Ejecuta el siguiente comando en la raíz del proyecto para instalar las dependencias con **Bun**:

```bash
bun install
```

### 2. Iniciar el Servidor de Desarrollo Slidev
Para abrir la presentación interactiva inmediatamente en tu navegador web:

```bash
bun run dev
```

O alternativamente usando `bunx`:

```bash
bunx slidev --open
```

La presentación estará disponible en `http://localhost:3030`.

---

## 📋 Comandos Disponibles

- `bun run dev` — Inicia el servidor local de Slidev con hot reload y abre el navegador.
- `bun run build` — Compila la presentación a una aplicación SPA estática lista para producción.
- `bun run export` — Exporta la presentación a formato PDF o imágenes PNG.

---

## 📂 Estructura del Proyecto

- `slides.md` — Archivo principal con las 18 slides de la presentación en sintaxis nativa de Slidev.
- `styles/index.css` — Estilos globales personalizados, tipografía Inter/Fira Code, contenedores para diagramas Mermaid y tarjetas glassmorphism.
- `package.json` — Configuración del proyecto y dependencias de Slidev gestionadas con Bun.
- `docs/` — Documentación técnica detallada de la arquitectura agéntica implementada en el proyecto.
