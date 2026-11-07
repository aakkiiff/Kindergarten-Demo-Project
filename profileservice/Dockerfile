FROM node:22-alpine
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm i
COPY . .
EXPOSE 5004
CMD ["node", "src/server.js"]
