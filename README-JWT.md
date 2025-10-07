# 🔐 Implementando JWT e Autenticação na API

## 🎯 Objetivo
Este guia mostra como implementar **autenticação JWT** na API e integrar com o **Swagger** para endpoints protegidos.

---

## 🚀 Passo a passo

### 1️⃣ Instalar dependências JWT
```bash
npm install jsonwebtoken bcryptjs
npm install -D @types/jsonwebtoken @types/bcryptjs
```

### 2️⃣ Criar middleware de autenticação
Crie `src/middlewares/auth.ts`:

```ts
import { Request, Response, NextFunction } from 'express';
import jwt from 'jsonwebtoken';

const JWT_SECRET = process.env.JWT_SECRET || 'sua-chave-secreta';

export interface AuthRequest extends Request {
  userId?: number;
}

export const authMiddleware = (req: AuthRequest, res: Response, next: NextFunction) => {
  const token = req.header('Authorization')?.replace('Bearer ', '');

  if (!token) {
    return res.status(401).json({ message: 'Token não fornecido' });
  }

  try {
    const decoded = jwt.verify(token, JWT_SECRET) as { userId: number };
    req.userId = decoded.userId;
    next();
  } catch (error) {
    return res.status(401).json({ message: 'Token inválido' });
  }
};
```

### 3️⃣ Criar controller de autenticação
Crie `src/controllers/AuthController.ts`:

```ts
import { Request, Response } from 'express';
import jwt from 'jsonwebtoken';
import bcrypt from 'bcryptjs';

const JWT_SECRET = process.env.JWT_SECRET || 'sua-chave-secreta';

interface LoginUser {
  id: number;
  email: string;
  password: string;
  name: string;
}

export class AuthController {
  private users: LoginUser[] = [
    {
      id: 1,
      email: 'admin@teste.com',
      password: '$2a$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi', // password
      name: 'Admin'
    }
  ];

  async login(req: Request, res: Response): Promise<Response> {
    const { email, password } = req.body;

    if (!email || !password) {
      return res.status(400).json({ message: 'Email e senha são obrigatórios' });
    }

    const user = this.users.find(u => u.email === email);
    if (!user) {
      return res.status(401).json({ message: 'Credenciais inválidas' });
    }

    const isValidPassword = await bcrypt.compare(password, user.password);
    if (!isValidPassword) {
      return res.status(401).json({ message: 'Credenciais inválidas' });
    }

    const token = jwt.sign({ userId: user.id }, JWT_SECRET, { expiresIn: '24h' });

    return res.json({
      token,
      user: {
        id: user.id,
        name: user.name,
        email: user.email
      }
    });
  }

  async register(req: Request, res: Response): Promise<Response> {
    const { name, email, password } = req.body;

    if (!name || !email || !password) {
      return res.status(400).json({ message: 'Todos os campos são obrigatórios' });
    }

    const existingUser = this.users.find(u => u.email === email);
    if (existingUser) {
      return res.status(400).json({ message: 'Email já cadastrado' });
    }

    const hashedPassword = await bcrypt.hash(password, 10);
    const newUser: LoginUser = {
      id: this.users.length + 1,
      name,
      email,
      password: hashedPassword
    };

    this.users.push(newUser);

    const token = jwt.sign({ userId: newUser.id }, JWT_SECRET, { expiresIn: '24h' });

    return res.status(201).json({
      token,
      user: {
        id: newUser.id,
        name: newUser.name,
        email: newUser.email
      }
    });
  }
}
```

### 4️⃣ Criar rotas de autenticação
Crie `src/routes/authRoutes.ts`:

```ts
import { Router } from "express";
import { AuthController } from "../controllers/AuthController";

const router = Router();
const authController = new AuthController();

/**
 * @swagger
 * components:
 *   securitySchemes:
 *     bearerAuth:
 *       type: http
 *       scheme: bearer
 *       bearerFormat: JWT
 */

/**
 * @swagger
 * /api/auth/login:
 *   post:
 *     summary: Fazer login
 *     tags: [Auth]
 *     requestBody:
 *       required: true
 *       content:
 *         application/json:
 *           schema:
 *             type: object
 *             required:
 *               - email
 *               - password
 *             properties:
 *               email:
 *                 type: string
 *                 example: admin@teste.com
 *               password:
 *                 type: string
 *                 example: password
 *     responses:
 *       200:
 *         description: Login realizado com sucesso
 *         content:
 *           application/json:
 *             schema:
 *               type: object
 *               properties:
 *                 token:
 *                   type: string
 *                 user:
 *                   type: object
 *       401:
 *         description: Credenciais inválidas
 */
router.post("/login", (req, res) => authController.login(req, res));

/**
 * @swagger
 * /api/auth/register:
 *   post:
 *     summary: Registrar novo usuário
 *     tags: [Auth]
 *     requestBody:
 *       required: true
 *       content:
 *         application/json:
 *           schema:
 *             type: object
 *             required:
 *               - name
 *               - email
 *               - password
 *             properties:
 *               name:
 *                 type: string
 *               email:
 *                 type: string
 *               password:
 *                 type: string
 *     responses:
 *       201:
 *         description: Usuário criado com sucesso
 *       400:
 *         description: Dados inválidos
 */
router.post("/register", (req, res) => authController.register(req, res));

export default router;
```

### 5️⃣ Proteger rotas de usuários
Atualize `src/routes/userRoutes.ts`:

```ts
import { Router } from "express";
import { UserController } from "../controllers/UserController";
import { authMiddleware } from "../middlewares/auth";

const router = Router();
const userController = new UserController();

// Aplicar middleware de autenticação em todas as rotas
router.use(authMiddleware);

/**
 * @swagger
 * /api/users:
 *   get:
 *     summary: Lista todos os usuários
 *     tags: [Users]
 *     security:
 *       - bearerAuth: []
 *     responses:
 *       200:
 *         description: Lista de usuários
 *       401:
 *         description: Token não fornecido ou inválido
 */
router.get("/", (req, res) => userController.getAll(req, res));

// ... demais rotas com security: - bearerAuth: []
```

### 6️⃣ Atualizar servidor
Modifique `src/server.ts`:

```ts
import express from "express";
import swaggerUi from 'swagger-ui-express';
import { swaggerSpec } from './config/swagger';
import userRoutes from "./routes/userRoutes";
import authRoutes from "./routes/authRoutes";

const app = express();
app.use(express.json());

// Swagger
app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerSpec));

// Rotas
app.use("/api/auth", authRoutes);
app.use("/api/users", userRoutes);

const PORT = 3000;
app.listen(PORT, () => {
  console.log(`🚀 Server running at http://localhost:${PORT}`);
  console.log(`📚 Swagger docs at http://localhost:${PORT}/api-docs`);
});
```

### 7️⃣ Criar pasta middlewares
```bash
mkdir src/middlewares
```

### 8️⃣ Configurar variável de ambiente
Crie `.env`:
```
JWT_SECRET=minha-chave-super-secreta-jwt-2024
```

Instale dotenv:
```bash
npm install dotenv
```

---

## 🔑 Testando a autenticação

### 1. Fazer login
```http
POST http://localhost:3000/api/auth/login
Content-Type: application/json

{
  "email": "admin@teste.com",
  "password": "password"
}
```

### 2. Usar token nas requisições
```http
GET http://localhost:3000/api/users
Authorization: Bearer SEU_TOKEN_AQUI
```

---

## 📚 Testando no Swagger

1. Acesse `http://localhost:3000/api-docs`
2. Faça login no endpoint `/api/auth/login`
3. Copie o token retornado
4. Clique no botão **"Authorize"** no topo
5. Digite: `Bearer SEU_TOKEN`
6. Agora pode testar endpoints protegidos

---

## 🛡️ Segurança

- **Tokens expiram** em 24h
- **Senhas são criptografadas** com bcrypt
- **Middleware valida** todos os tokens
- **Variáveis de ambiente** para chaves secretas