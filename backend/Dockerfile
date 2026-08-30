# --- ETAPA 1: Construcción (Build) ---
FROM node:20-alpine AS builder

# Habilitar pnpm
RUN corepack enable && corepack prepare pnpm@latest --activate

WORKDIR /app

# Copiar archivos de dependencias y esquema de base de datos
COPY package.json pnpm-lock.yaml ./
COPY prisma ./prisma/

# Instalar dependencias
RUN pnpm install --frozen-lockfile

# Copiar el resto del código fuente del backend
COPY . .

# Generar el cliente de Prisma y compilar TypeScript
RUN pnpm prisma generate
RUN pnpm run build || npx tsc

# --- ETAPA 2: Producción (Runner) ---
FROM node:20-alpine AS runner

WORKDIR /app

# Habilitar pnpm en la etapa final
RUN corepack enable && corepack prepare pnpm@latest --activate

# Copiar manifiesto e instalar solo dependencias de producción
COPY package.json pnpm-lock.yaml ./
RUN pnpm install --prod --frozen-lockfile

# Copiar artefactos compilados y motor de Prisma desde la etapa builder
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/prisma ./prisma

EXPOSE 3000

# Ejecución directa optimizada
CMD ["node", "dist/index.js"]
