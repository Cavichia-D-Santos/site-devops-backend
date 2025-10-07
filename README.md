# 📘 README — Criando sua Primeira API com TypeScript e Express

## 🎯 Objetivo
Neste guia você vai aprender a criar uma **API REST** simples utilizando **Node.js**, **Express** e **TypeScript**.  
Ao final, você terá um projeto rodando localmente que responde a requisições HTTP.

---

## 🚀 Passo a passo

### 1️⃣ Criar a pasta do projeto
```bash
mkdir api-typescript
cd api-typescript
```

### 2️⃣ Iniciar o projeto Node.js
```bash
npm init -y
```
> Isso cria o arquivo `package.json`, que guarda as informações e dependências do projeto.

---

### 3️⃣ Instalar dependências
**Dependência principal (produção):**
```bash
npm install express
```

**Dependências de desenvolvimento:**
```bash
npm install -D typescript ts-node-dev @types/node @types/express
```

- `typescript`: compilador TS  
- `ts-node-dev`: roda o projeto em TS sem precisar compilar manualmente  
- `@types/...`: fornece tipagem para Node.js e Express  

---

### 4️⃣ Criar o arquivo de configuração do TypeScript
```bash
npx tsc --init
```

Edite o arquivo `tsconfig.json` para ter as opções abaixo:
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true
  }
}
```

---

### 5️⃣ Estruturar o projeto
Crie as pastas principais:
```bash
mkdir src src/routes src/controllers
```

A estrutura ficará assim:
```
src/
 ├── controllers/
 │    └── UserController.ts
 ├── routes/
 │    └── userRoutes.ts
 └── server.ts
```

---

### 6️⃣ Configurar scripts no `package.json`
Abra o arquivo `package.json` e adicione os scripts:

```json
"scripts": {
  "dev": "ts-node-dev --respawn src/server.ts",
  "build": "tsc",
  "start": "node dist/server.js"
}
```

- `dev`: roda a API em modo desenvolvimento  
- `build`: compila os arquivos TS para JS  
- `start`: roda o projeto já compilado  

---

### 7️⃣ Criar o servidor (server.ts)

Crie o arquivo `src/server.ts` com o seguinte código:

```ts
import express from "express";
import userRoutes from "./routes/userRoutes";

const app = express();
app.use(express.json());

// Rotas
app.use("/users", userRoutes);

const PORT = 3000;
app.listen(PORT, () => {
  console.log(`🚀 Server running at http://localhost:${PORT}`);
});
```

---

### 8️⃣ Criar as rotas (userRoutes.ts)

Crie o arquivo `src/routes/userRoutes.ts`:

```ts
import { Router } from "express";
import { UserController } from "../controllers/UserController";

const router = Router();
const userController = new UserController();

router.get("/", (req, res) => userController.getAll(req, res));
router.get("/:id", (req, res) => userController.getById(req, res));
router.post("/", (req, res) => userController.create(req, res));
router.put("/:id", (req, res) => userController.update(req, res));
router.delete("/:id", (req, res) => userController.delete(req, res));

export default router;
```

---

### 9️⃣ Criar o controller (UserController.ts)

Crie o arquivo `src/controllers/UserController.ts`:

```ts
import { Request, Response } from "express";

interface User {
  id: number;
  name: string;
  email: string;
}

export class UserController {
  private users: User[] = [];
  private idCounter = 1;

  getAll(req: Request, res: Response): Response {
    return res.json(this.users);
  }

  getById(req: Request, res: Response): Response {
    const id = Number(req.params.id);
    const user = this.users.find(u => u.id === id);
    return user ? res.json(user) : res.status(404).json({ message: "User not found" });
  }

  create(req: Request, res: Response): Response {
    const { name, email } = req.body;
    const newUser: User = { id: this.idCounter++, name, email };
    this.users.push(newUser);
    return res.status(201).json(newUser);
  }

  update(req: Request, res: Response): Response {
    const id = Number(req.params.id);
    const { name, email } = req.body;
    const user = this.users.find(u => u.id === id);
    if (!user) return res.status(404).json({ message: "User not found" });

    user.name = name ?? user.name;
    user.email = email ?? user.email;
    return res.json(user);
  }

  delete(req: Request, res: Response): Response {
    const id = Number(req.params.id);
    this.users = this.users.filter(u => u.id !== id);
    return res.status(204).send();
  }
}
```

---

### 🔟 Rodar a API
Execute o comando:

```bash
npm run dev
```

Se tudo deu certo, você verá no terminal:
```
🚀 Server running at http://localhost:3000
```

---

## 📚 Testando a API

Use [Postman](https://www.postman.com/) ou [Insomnia](https://insomnia.rest/) para enviar requisições:

- `GET http://localhost:3000/users` → lista usuários  
- `POST http://localhost:3000/users` → cria usuário (body JSON: `{ "name": "Maria", "email": "maria@email.com" }`)  
- `GET http://localhost:3000/users/1` → busca usuário com ID 1  
- `PUT http://localhost:3000/users/1` → atualiza usuário (body JSON: `{ "name": "Maria Silva" }`)  
- `DELETE http://localhost:3000/users/1` → remove usuário  

---
