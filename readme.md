 ## Estrutura de Diretórios
 
 ```
 project-root/
├── src/
│   ├── controllers/
│   │   ├── auth.controller.ts
│   │   └── user.controller.ts
│   ├── middlewares/
│   │   ├── auth.middleware.ts
│   │   └── error.middleware.ts
│   ├── models/
│   │   ├── user.model.ts
│   ├── routes/
│   │   ├── auth.routes.ts
│   │   └── user.routes.ts
│   ├── services/
│   │   ├── auth.service.ts
│   │   └── user.service.ts
│   ├── utils/
│   │   ├── logger.ts
│   ├── config/
│   │   ├── database.ts
│   │   ├── env.ts
│   ├── app.ts
│   ├── server.ts
├── tests/
│   ├── auth.test.ts
│   └── user.test.ts
├── .env
├── .gitignore
├── package.json
├── tsconfig.json
└── README.md
```

Para visualizar a explicação da arquitetura [clique aqui](./explain.md)

## Exemplo de aplicação NodeJS

OBS: as dependências utilizadas no projeto são apenas a fim de exemplo.

1. Inicializando o Projeto

```
mkdir project-root
cd project-root
npm init -y
npm install express dotenv mongoose jsonwebtoken bcryptjs helmet cors morgan
npm install -D typescript ts-node @types/node @types/express @types/jsonwebtoken @types/bcryptjs @types/cors @types/morgan jest ts-jest @types/jest supertest @types/supertest
npx tsc --init
```

2. Configuração do tsconfig.json

```json
{
  "compilerOptions": {
    "target": "ES6",
    "module": "commonjs",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
  }
}
```

3. Configuração do Express (src/app.ts)

```typescript
import express from 'express';
import cors from 'cors';
import helmet from 'helmet';
import morgan from 'morgan';
import authRoutes from './routes/auth.routes';
import userRoutes from './routes/user.routes';
import { errorHandler } from './middlewares/error.middleware';
import { connectDatabase } from './config/database';

const app = express();

app.use(helmet());
app.use(cors());
app.use(morgan('dev'));
app.use(express.json());

app.use('/api/auth', authRoutes);
app.use('/api/users', userRoutes);

app.use(errorHandler);

connectDatabase();

export default app;
```

4. Inicialização do Servidor (src/server.ts)

```typescript
import app from './app';
import { config } from './config/env';

const PORT = config.PORT || 3000;

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

5. Configuração do Ambiente (src/config/env.ts)

```typescript
import dotenv from 'dotenv';

dotenv.config();

export const config = {
  PORT: process.env.PORT,
  MONGO_URI: process.env.MONGO_URI,
  JWT_SECRET: process.env.JWT_SECRET
};
```

6. Conexão com o Banco de Dados (src/config/database.ts)

```typescript
import mongoose from 'mongoose';
import { config } from './env';

export const connectDatabase = async () => {
  try {
    await mongoose.connect(config.MONGO_URI, {
      useNewUrlParser: true,
      useUnifiedTopology: true,
    });
    console.log('Database connected');
  } catch (error) {
    console.error('Database connection error:', error);
    process.exit(1);
  }
};
```

7. Middleware de Erro (src/middlewares/error.middleware.ts)

```typescript
import { Request, Response, NextFunction } from 'express';

export const errorHandler = (err: any, req: Request, res: Response, next: NextFunction) => {
  const statusCode = err.statusCode || 500;
  const message = err.message || 'Internal Server Error';
  res.status(statusCode).json({ message });
};
```

8. Exemplo de Serviço (src/services/auth.service.ts)

```typescript
import jwt from 'jsonwebtoken';
import bcrypt from 'bcryptjs';
import { config } from '../config/env';
import User from '../models/user.model';

export const register = async (email: string, password: string) => {
  const hashedPassword = await bcrypt.hash(password, 10);
  const newUser = new User({ email, password: hashedPassword });
  await newUser.save();
  return newUser;
};

export const login = async (email: string, password: string) => {
  const user = await User.findOne({ email });
  if (!user || !await bcrypt.compare(password, user.password)) {
    throw new Error('Invalid credentials');
  }
  const token = jwt.sign({ id: user._id }, config.JWT_SECRET, { expiresIn: '1h' });
  return { user, token };
};
```

9. Exemplo de Controlador (src/controllers/auth.controller.ts)

```typescript
import { Request, Response } from 'express';
import { register, login } from '../services/auth.service';

export const registerController = async (req: Request, res: Response) => {
  try {
    const user = await register(req.body.email, req.body.password);
    res.status(201).json(user);
  } catch (error) {
    res.status(400).json({ message: error.message });
  }
};

export const loginController = async (req: Request, res: Response) => {
  try {
    const { user, token } = await login(req.body.email, req.body.password);
    res.json({ user, token });
  } catch (error) {
    res.status(400).json({ message: error.message });
  }
};
```

10. Exemplo de Rotas (src/routes/auth.routes.ts)

```typescript
import { Router } from 'express';
import { registerController, loginController } from '../controllers/auth.controller';

const router = Router();

router.post('/register', registerController);
router.post('/login', loginController);

export default router;
```

11. Exemplo de Model (src/models/user.model.ts)

```typescript
import { Schema, model } from 'mongoose';

const userSchema = new Schema({
  email: { type: String, required: true, unique: true },
  password: { type: String, required: true },
}, {
  timestamps: true,
});

export default model('User', userSchema);
```

12. Logger (src/utils/logger.ts)

```typescript
import winston from 'winston';

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json()
  ),
  transports: [
    new winston.transports.Console(),
    new winston.transports.File({ filename: 'combined.log' })
  ],
});

export default logger;
```

13. Testes (tests/auth.test.ts)

```typescript
import request from 'supertest';
import app from '../src/app';
import mongoose from 'mongoose';

beforeAll(async () => {
  await mongoose.connect(process.env.MONGO_URI, {
    useNewUrlParser: true,
    useUnifiedTopology: true,
  });
});

afterAll(async () => {
  await mongoose.connection.close();
});

describe('Auth Endpoints', () => {
  it('should register a new user', async () => {
    const res = await request(app)
      .post('/api/auth/register')
      .send({
        email: 'test@example.com',
        password: '123456'
      });
    expect(res.statusCode).toEqual(201);
    expect(res.body).toHaveProperty('_id');
  });

  it('should login an existing user', async () => {
    const res = await request(app)
      .post('/api/auth/login')
      .send({
        email: 'test@example.com',
        password: '123456'
      });
    expect(res.statusCode).toEqual(200);
    expect(res.body).toHaveProperty('token');
  });
});

```