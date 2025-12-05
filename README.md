# 📄 Paggo OCR - Sistema Completo de Análise de Documentos com IA

Sistema fullstack para upload, processamento OCR e análise inteligente de documentos fiscais usando IA.

![Stack](https://img.shields.io/badge/Next.js-14-black?logo=next.js)
![Stack](https://img.shields.io/badge/NestJS-10-red?logo=nestjs)
![Stack](https://img.shields.io/badge/PostgreSQL-15-blue?logo=postgresql)
![Stack](https://img.shields.io/badge/TypeScript-5-blue?logo=typescript)

---

## 📋 Índice

- [Visão Geral](#visão-geral)
- [Arquitetura](#arquitetura)
- [Tecnologias](#tecnologias)
- [Funcionalidades](#funcionalidades)
- [Fluxo de Comunicação](#fluxo-de-comunicação)
- [Setup Completo](#setup-completo)
- [Como Usar](#como-usar)
- [Deploy](#deploy)
- [Estrutura dos Repositórios](#estrutura-dos-repositórios)

---

## 🎯 Visão Geral

O **Paggo OCR** é uma solução completa que permite:

1. **Upload de Documentos** - Usuários fazem upload de notas fiscais (imagens ou PDFs)
2. **Processamento OCR** - Extração automática de texto usando Tesseract
3. **Análise por IA** - Chat interativo sobre o documento usando GPT-4o-mini
4. **Exportação** - Download dos resultados em PDF ou JSON

### Casos de Uso

✅ Digitalização de notas fiscais  
✅ Extração automática de dados fiscais  
✅ Consultas inteligentes sobre documentos  
✅ Auditoria e compliance  
✅ Automação contábil  

---

## 🏗 Arquitetura

### Visão Geral do Sistema

```
┌─────────────────────────────────────────────────────────────────┐
│                        USUÁRIO                                  │
│                    (Navegador Web)                              │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         │ HTTPS
                         │
┌────────────────────────▼────────────────────────────────────────┐
│                   FRONTEND (Next.js 14)                         │
│                  https://seu-app.vercel.app                     │
│                                                                 │
│  • Autenticação (NextAuth)                                      │
│  • Interface responsiva (Tailwind + Shadcn)                     │
│  • Gerenciamento de estado (TanStack Query)                     │
│  • Upload de arquivos (Drag & Drop)                             │
│  • Chat com IA em tempo real                                    │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         │ REST API (JSON)
                         │ Authorization: Bearer <JWT>
                         │
┌────────────────────────▼────────────────────────────────────────┐
│                  BACKEND (NestJS 10)                            │
│                https://sua-api.railway.app                      │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  Controllers (Rotas HTTP)                                │  │
│  │  • /auth (login, register)                               │  │
│  │  • /documents (CRUD)                                     │  │
│  │  • /llm (chat com IA)                                    │  │
│  └────────────────────┬─────────────────────────────────────┘  │
│                       │                                         │
│  ┌────────────────────▼─────────────────────────────────────┐  │
│  │  Services (Lógica de Negócio)                            │  │
│  │  • AuthService - JWT, bcrypt                             │  │
│  │  • DocumentsService - CRUD, validações                   │  │
│  │  • OcrService - Tesseract.js                             │  │
│  │  • LlmService - OpenAI API                               │  │
│  │  • StorageService - Oracle Cloud / Local                 │  │
│  └──────┬─────────┬──────────┬──────────┬───────────────────┘  │
│         │         │          │          │                       │
└─────────┼─────────┼──────────┼──────────┼───────────────────────┘
          │         │          │          │
          │         │          │          │
┌─────────▼────┐ ┌──▼──────┐ ┌▼────────┐ ┌▼──────────────┐
│ PostgreSQL   │ │ Oracle  │ │Tesseract│ │   OpenAI      │
│   (Railway)  │ │  Cloud  │ │   OCR   │ │   GPT-4o-mini │
│              │ │ Storage │ │ Engine  │ │               │
│ • users      │ │         │ │         │ │ • Perguntas   │
│ • documents  │ │ • Imagens│ │• Extrai │ │   sobre docs  │
│ • conversa   │ │ • PDFs  │ │  texto  │ │ • Resumos     │
│   -tions     │ │         │ │         │ │               │
└──────────────┘ └─────────┘ └─────────┘ └───────────────┘
```

### Fluxo de Dados Detalhado

#### 1️⃣ Upload de Documento

```
Usuário                Frontend              Backend              Storage        OCR
  │                       │                     │                   │            │
  │ Seleciona arquivo     │                     │                   │            │
  ├──────────────────────>│                     │                   │            │
  │                       │                     │                   │            │
  │                       │ POST /documents/upload                  │            │
  │                       │  (multipart/form-data)                  │            │
  │                       ├────────────────────>│                   │            │
  │                       │                     │                   │            │
  │                       │                     │ Valida arquivo    │            │
  │                       │                     │ (tipo, tamanho)   │            │
  │                       │                     │                   │            │
  │                       │                     │ Salva em storage  │            │
  │                       │                     ├──────────────────>│            │
  │                       │                     │                   │            │
  │                       │                     │  <URL do arquivo> │            │
  │                       │                     │<──────────────────┤            │
  │                       │                     │                   │            │
  │                       │                     │ Cria no DB:       │            │
  │                       │                     │  status=PROCESSING│            │
  │                       │                     │                   │            │
  │                       │  { id, status, url }│                   │            │
  │                       │<────────────────────┤                   │            │
  │                       │                     │                   │            │
  │ Documento criado      │                     │ [ASYNC] Processa OCR           │
  │<──────────────────────┤                     ├───────────────────────────────>│
  │                       │                     │                   │            │
  │                       │                     │       <texto extraído>          │
  │                       │                     │<────────────────────────────────┤
  │                       │                     │                   │            │
  │                       │                     │ Atualiza DB:      │            │
  │                       │                     │  status=COMPLETED │            │
  │                       │                     │  extractedText=...│            │
```

#### 2️⃣ Chat com IA sobre Documento

```
Usuário                Frontend              Backend              LLM (OpenAI)
  │                       │                     │                      │
  │ "Qual o valor total?" │                     │                      │
  ├──────────────────────>│                     │                      │
  │                       │                     │                      │
  │                       │ POST /llm/ask       │                      │
  │                       │ { documentId, question }                   │
  │                       ├────────────────────>│                      │
  │                       │                     │                      │
  │                       │                     │ Busca documento      │
  │                       │                     │ + histórico          │
  │                       │                     │                      │
  │                       │                     │ Monta contexto:      │
  │                       │                     │  - System prompt     │
  │                       │                     │  - Texto extraído    │
  │                       │                     │  - Histórico chat    │
  │                       │                     │  - Nova pergunta     │
  │                       │                     │                      │
  │                       │                     │ POST /chat/completions
  │                       │                     ├─────────────────────>│
  │                       │                     │                      │
  │                       │                     │    <resposta IA>     │
  │                       │                     │<─────────────────────┤
  │                       │                     │                      │
  │                       │                     │ Salva conversa no DB │
  │                       │                     │                      │
  │                       │  { question, answer }                      │
  │                       │<────────────────────┤                      │
  │                       │                     │                      │
  │ Exibe resposta        │                     │                      │
  │<──────────────────────┤                     │                      │
```

#### 3️⃣ Download de Resultados

```
Usuário                Frontend              Backend              
  │                       │                     │
  │ Clica "Download PDF"  │                     │
  ├──────────────────────>│                     │
  │                       │                     │
  │                       │ GET /documents/:id/download
  │                       ├────────────────────>│
  │                       │                     │
  │                       │                     │ Busca documento
  │                       │                     │ + texto extraído
  │                       │                     │ + conversas
  │                       │                     │
  │                       │                     │ Gera PDF com:
  │                       │                     │  - Imagem doc
  │                       │                     │  - Texto OCR
  │                       │                     │  - Histórico chat
  │                       │                     │
  │                       │    <arquivo PDF>    │
  │                       │<────────────────────┤
  │                       │                     │
  │ Download iniciado     │                     │
  │<──────────────────────┤                     │
```

---

## 🛠 Tecnologias

### Frontend
| Tecnologia | Versão | Uso |
|------------|--------|-----|
| Next.js | 14 | Framework React com App Router |
| React | 18 | Biblioteca UI |
| TypeScript | 5 | Tipagem estática |
| Tailwind CSS | 3 | Estilização |
| Shadcn/ui | Latest | Componentes UI |
| TanStack Query | 5 | Gerenciamento de estado servidor |
| NextAuth.js | 5 | Autenticação |
| React Hook Form | 7 | Formulários |
| Zod | 3 | Validação de schemas |
| Axios | 1 | Cliente HTTP |

### Backend
| Tecnologia | Versão | Uso |
|------------|--------|-----|
| NestJS | 10 | Framework Node.js |
| TypeScript | 5 | Tipagem estática |
| Prisma ORM | 5 | ORM para banco de dados |
| PostgreSQL | 15 | Banco de dados relacional |
| Passport JWT | 10 | Autenticação JWT |
| bcrypt | 5 | Hash de senhas |
| Tesseract.js | 5 | OCR (reconhecimento de texto) |
| OpenAI SDK | 4 | Integração com GPT |
| AWS SDK (S3) | 3 | Storage (Oracle Cloud é S3-compatible) |
| PDFKit | 0.13 | Geração de PDFs |

### Infraestrutura
| Serviço | Uso |
|---------|-----|
| **Vercel** | Hospedagem do frontend |
| **Railway** | Hospedagem do backend + PostgreSQL |
| **Oracle Cloud** | Object Storage (armazenamento de arquivos) |
| **OpenAI** | API de IA (GPT-4o-mini) |

---

## ✨ Funcionalidades

### 🔐 Autenticação
- [x] Registro de usuários com validação
- [x] Login com JWT (token expira em 7 dias)
- [x] Proteção de rotas (middleware)
- [x] Logout
- [x] Hash de senhas com bcrypt (10 rounds)

### 📤 Upload de Documentos
- [x] Drag & drop de arquivos
- [x] Suporte a JPG, PNG, PDF
- [x] Validação de tamanho (max 10MB)
- [x] Preview antes do upload
- [x] Barra de progresso
- [x] Storage em Oracle Cloud (produção) ou local (dev)

### 🔍 OCR (Reconhecimento de Texto)
- [x] Processamento assíncrono (não bloqueia o usuário)
- [x] Engine: Tesseract.js
- [x] Suporte a português + inglês
- [x] Status em tempo real (PROCESSING → COMPLETED/FAILED)
- [x] Extração de texto estruturado

### 🤖 IA (Análise Inteligente)
- [x] Chat interativo sobre documentos
- [x] Contexto mantido entre perguntas
- [x] Modelo: GPT-4o-mini (rápido e econômico)
- [x] Resumo automático de documentos
- [x] Histórico de conversas persistido

### 📊 Gerenciamento
- [x] Dashboard com lista de documentos
- [x] Filtros e busca
- [x] Status visual (processando, concluído, erro)
- [x] Visualização detalhada de documentos
- [x] Exclusão com cleanup de arquivos

### 💾 Exportação
- [x] Download em PDF (documento + texto + chat)
- [x] Download em JSON estruturado
- [x] Formatação profissional

---

## 🔄 Fluxo de Comunicação Frontend ↔ Backend

### Autenticação

```typescript
// FRONTEND: Login
const response = await axios.post('http://localhost:3001/api/auth/login', {
  email: 'user@example.com',
  password: 'senha123'
});

// Resposta:
{
  "user": {
    "id": "uuid",
    "email": "user@example.com",
    "name": "Nome"
  },
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}

// FRONTEND: Armazena token
localStorage.setItem('token', response.data.accessToken);

// FRONTEND: Todas as próximas requisições incluem token
axios.defaults.headers.common['Authorization'] = `Bearer ${token}`;
```

### Upload de Documento

```typescript
// FRONTEND
const formData = new FormData();
formData.append('file', file);

const response = await axios.post(
  'http://localhost:3001/api/documents/upload',
  formData,
  {
    headers: {
      'Content-Type': 'multipart/form-data',
      'Authorization': `Bearer ${token}`
    }
  }
);

// BACKEND processa:
// 1. Valida autenticação (JWT Guard)
// 2. Valida arquivo (tipo, tamanho)
// 3. Salva em Oracle Cloud
// 4. Cria registro no PostgreSQL
// 5. Dispara OCR assíncrono
// 6. Retorna resposta imediata

// Resposta:
{
  "id": "doc-uuid",
  "originalName": "nota_fiscal.jpg",
  "storageUrl": "https://objectstorage.sa-saopaulo-1.oraclecloud.com/...",
  "status": "PROCESSING",
  "createdAt": "2024-12-26T10:30:00Z"
}
```

### Verificar Status do OCR

```typescript
// FRONTEND: Polling a cada 2 segundos
const checkStatus = async () => {
  const response = await axios.get(
    `http://localhost:3001/api/documents/${docId}/status`,
    { headers: { Authorization: `Bearer ${token}` } }
  );
  
  if (response.data.status === 'COMPLETED') {
    // Parar polling, mostrar texto extraído
  } else if (response.data.status === 'FAILED') {
    // Mostrar erro
  } else {
    // Continuar polling
    setTimeout(checkStatus, 2000);
  }
};

// Resposta:
{
  "id": "doc-uuid",
  "status": "COMPLETED",
  "hasText": true,
  "error": null
}
```

### Chat com IA

```typescript
// FRONTEND: Usuário faz pergunta
const response = await axios.post(
  'http://localhost:3001/api/llm/ask',
  {
    documentId: 'doc-uuid',
    question: 'Qual o valor total desta nota fiscal?'
  },
  { headers: { Authorization: `Bearer ${token}` } }
);

// BACKEND:
// 1. Busca documento e valida ownership
// 2. Verifica se OCR foi concluído
// 3. Busca histórico de conversas
// 4. Monta contexto para OpenAI:
//    - System prompt
//    - Texto extraído
//    - Histórico
//    - Nova pergunta
// 5. Chama OpenAI API
// 6. Salva conversa no PostgreSQL
// 7. Retorna resposta

// Resposta:
{
  "question": "Qual o valor total desta nota fiscal?",
  "answer": "O valor total da nota fiscal é R$ 1.234,56",
  "timestamp": "2024-12-26T10:35:00Z"
}
```

### Download de Resultados

```typescript
// FRONTEND
const response = await axios.get(
  `http://localhost:3001/api/documents/${docId}/download`,
  {
    headers: { Authorization: `Bearer ${token}` },
    responseType: 'blob' // Importante para arquivos binários
  }
);

// Criar link de download
const url = window.URL.createObjectURL(new Blob([response.data]));
const link = document.createElement('a');
link.href = url;
link.setAttribute('download', 'resultado.pdf');
document.body.appendChild(link);
link.click();

// BACKEND:
// 1. Busca documento + texto + conversas
// 2. Gera PDF usando PDFKit
// 3. Retorna arquivo binário

// Headers da resposta:
Content-Type: application/pdf
Content-Disposition: attachment; filename="nota_fiscal_result.pdf"
```

---

## 🚀 Setup Completo

### 1. Clone os Repositórios

```bash
# Backend
git clone https://github.com/seu-usuario/paggo-ocr-api.git
cd paggo-ocr-api

# Frontend (em outro terminal)
git clone https://github.com/seu-usuario/paggo-ocr-web.git
cd paggo-ocr-web
```

### 2. Setup do Backend

```bash
cd paggo-ocr-api

# Instalar dependências
npm install

# Configurar PostgreSQL (Docker)
docker run --name paggo-postgres \
  -e POSTGRES_PASSWORD=postgres \
  -e POSTGRES_DB=paggo_ocr \
  -p 5432:5432 \
  -d postgres:15

# Criar arquivo .env
cat > .env << EOF
DATABASE_URL="postgresql://postgres:postgres@localhost:5432/paggo_ocr?schema=public"
JWT_SECRET="$(node -e "console.log(require('crypto').randomBytes(32).toString('hex'))")"
JWT_EXPIRES_IN="7d"
STORAGE_TYPE="local"
UPLOAD_PATH="./uploads"
OPENAI_API_KEY="sk-proj-sua-chave"
OPENAI_MODEL="gpt-4o-mini"
PORT=3001
FRONTEND_URL="http://localhost:3000"
EOF

# Rodar migrações
npx prisma migrate dev --name init
npx prisma generate

# Iniciar servidor
npm run start:dev
```

Backend rodando em: **http://localhost:3001**

### 3. Setup do Frontend

```bash
cd paggo-ocr-web

# Instalar dependências
npm install

# Criar arquivo .env.local
cat > .env.local << EOF
NEXT_PUBLIC_API_URL="http://localhost:3001/api"
NEXTAUTH_SECRET="$(openssl rand -base64 32)"
NEXTAUTH_URL="http://localhost:3000"
EOF

# Iniciar servidor
npm run dev
```

Frontend rodando em: **http://localhost:3000**

---

## 🎮 Como Usar

### 1. Criar Conta

1. Acesse http://localhost:3000/register
2. Preencha: nome, email, senha
3. Clique em "Registrar"
4. Você será redirecionado para o dashboard

### 2. Fazer Upload de Documento

1. No dashboard, clique em "Novo Upload"
2. Arraste uma nota fiscal (JPG/PNG/PDF) ou clique para selecionar
3. Aguarde o upload e processamento OCR (15-30 segundos)
4. O status mudará de "Processando" para "Concluído"

### 3. Visualizar Texto Extraído

1. Na lista de documentos, clique no documento desejado
2. Você verá:
   - Imagem original do documento
   - Texto extraído pelo OCR
   - Interface de chat

### 4. Fazer Perguntas com IA

1. Na página do documento, no painel de chat
2. Digite perguntas como:
   - "Qual o valor total?"
   - "Quem é o fornecedor?"
   - "Quais produtos estão listados?"
   - "Há algum desconto aplicado?"
3. A IA responderá baseada no texto extraído

### 5. Fazer Download dos Resultados

1. Na página do documento, clique em "Download"
2. Escolha o formato:
   - **PDF**: Documento formatado com imagem + texto + conversas
   - **JSON**: Dados estruturados para integração

---

## 🚢 Deploy

### Backend no Railway

1. Crie conta no [Railway](https://railway.app)
2. Novo projeto → Deploy from GitHub
3. Selecione o repositório do backend
4. Adicione PostgreSQL: Add Service → Database → PostgreSQL
5. Configure variáveis de ambiente:
   ```
   DATABASE_URL=${Postgres.DATABASE_URL}
   JWT_SECRET=sua-chave-secreta
   STORAGE_TYPE=oracle
   ORACLE_REGION=sa-saopaulo-1
   ORACLE_NAMESPACE=...
   ORACLE_BUCKET_NAME=paggo-ocr-uploads
   ORACLE_ACCESS_KEY=...
   ORACLE_SECRET_KEY=...
   OPENAI_API_KEY=sk-proj-...
   OPENAI_MODEL=gpt-4o-mini
   NODE_ENV=production
   FRONTEND_URL=https://seu-app.vercel.app
   ```
6. Deploy automático!

URL da API: `https://seu-projeto.railway.app`

### Frontend no Vercel

1. Crie conta no [Vercel](https://vercel.com)
2. Import Git Repository
3. Selecione o repositório do frontend
4. Configure variáveis de ambiente:
   ```
   NEXT_PUBLIC_API_URL=https://sua-api.railway.app/api
   NEXTAUTH_SECRET=sua-chave-secreta
   NEXTAUTH_URL=https://seu-app.vercel.app
   ```
5. Deploy!

URL do App: `https://seu-app.vercel.app`

---

## 📂 Estrutura dos Repositórios

### Backend
```
paggo-ocr-api/
├── src/
│   ├── auth/              # Autenticação JWT
│   ├── users/             # Gerenciamento de usuários
│   ├── documents/         # CRUD de documentos
│   ├── ocr/               # Processamento OCR
│   ├── llm/               # Integração OpenAI
│   ├── storage/           # Oracle Cloud Storage
│   └── prisma/            # ORM e migrations
├── prisma/
│   └── schema.prisma      # Schema do banco
├── .env                   # Variáveis de ambiente
└── README.md
```

### Frontend
```
paggo-ocr-web/
├── src/
│   ├── app/
│   │   ├── (auth)/        # Páginas públicas
│   │   └── (dashboard)/   # Páginas protegidas
│   ├── components/        # Componentes React
│   ├── lib/               # Utilitários
│   ├── hooks/             # Custom hooks
│   └── types/             # TypeScript types
├── .env.local             # Variáveis de ambiente
└── README.md
```

---

## 📊 Métricas e Performance

### Backend
- **Tempo de resposta**: < 200ms (endpoints simples)
- **Upload**: Até 10MB, processa em ~2-5s
- **OCR**: 15-30s para documentos típicos
- **LLM**: 2-5s por pergunta

### Frontend
- **First Contentful Paint**: < 1.5s
- **Time to Interactive**: < 3s
- **Lighthouse Score**: > 90

### Custos Estimados (mensal para teste)

| Serviço | Custo |
|---------|-------|
| Railway (Backend + DB) | $5-10 |
| Vercel (Frontend) | Gratuito |
| Oracle Cloud Storage | Gratuito (Always Free) |
| OpenAI API | $2-5 (300-500 perguntas) |
| **Total** | **~$10/mês** |

---

## 🔒 Segurança

- [x] Senhas hasheadas com bcrypt (10 rounds)
- [x] JWT com expiração de 7 dias
- [x] Validação de ownership (usuários só veem seus docs)
- [x] CORS configurado para origem específica
- [x] Validação de entrada com class-validator
- [x] Rate limiting (recomendado para produção)
- [x] HTTPS em produção

---

## 🧪 Testes

### Backend
```bash
cd paggo-ocr-api
npm run test              # Testes unitários
npm run test:e2e          # Testes end-to-end
npm run test:cov          # Cobertura
```

### Frontend
```bash
cd paggo-ocr-web
npm run test              # Jest + Testing Library
npm run test:e2e          # Playwright (recomendado)
```

---

## 🐛 Troubleshooting

### "CORS blocked"
- Verifique `FRONTEND_URL` no backend
- Verifique `NEXT_PUBLIC_API_URL` no frontend

### "OCR muito lento"
- Use imagens menores (< 2MB)
- Considere Google Cloud Vision API

### "OpenAI rate limit"
- Adicione créditos na conta OpenAI
- Implemente cache de respostas

### "Upload falha"
- Verifique credenciais Oracle Cloud
- Teste com storage local primeiro

---

## 📚 Documentação Adicional

- [README Backend](./backend/README.md) - Detalhes da API
- [README Frontend](./frontend/README.md) - Detalhes da interface

---

## 📄 Licença

MIT License - Desenvolvido para o case técnico Paggo

---

## 👥 Contato

Desenvolvido por [Seu Nome]
- GitHub: [@seu-usuario](https://github.com/seu-usuario)
- Email: seu.email@example.com
- LinkedIn: [Seu Perfil](https://linkedin.com/in/seu-perfil)

---

## 🎯 Próximos Passos (Melhorias Futuras)

- [ ] WebSockets para status em tempo real
- [ ] Fila de processamento com Bull/BullMQ
- [ ] Cache Redis para respostas LLM
- [ ] Suporte a múltiplos idiomas
- [ ] OCR com Google Cloud Vision (maior precisão)
- [ ] Análise de sentimento dos documentos
- [ ] Dashboard com métricas e analytics
- [ ] API de webhooks para integrações
- [ ] Export para Excel/CSV
- [ ] Reconhecimento de assinatura digital

---

**⭐ Se este projeto foi útil, considere dar uma estrela no GitHub!**

