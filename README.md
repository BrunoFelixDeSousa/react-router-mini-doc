# Mini Documentação: React Router
A mini documentação inclui:

 - Instalação e configuração básica
 - Estrutura de pastas recomendada
 - Principais componentes e hooks do React Router
 - Autenticação e rotas protegidas
 - Layouts e rotas aninhadas
 - Lazy loading para melhor performance
 - Tratamento de erros e redirecionamentos
 - Data Router (loaders e actions)
 - Técnicas avançadas como rotas modais
 - Integração com bibliotecas de UI
 - Considerações de SEO
 - Estratégias de teste
 - Exemplo de implementação completa

## 1. Instalação

Para instalar o React Router em um projeto React com TypeScript e Vite:

```bash
# Usando npm
npm install react-router-dom

# Usando yarn
yarn add react-router-dom

# Usando pnpm
pnpm add react-router-dom
```

## 2. Configuração Básica

### Estrutura de Pastas Recomendada

```
src/
├── main.tsx        # Ponto de entrada
├── App.tsx         # Componente principal
├── routes/
│   ├── index.tsx   # Configuração das rotas
│   └── paths.ts    # Constantes de caminhos
├── pages/
│   ├── Home.tsx
│   ├── About.tsx
│   └── NotFound.tsx
└── components/
    └── ...
```

### Arquivo de Constantes de Rotas (src/routes/paths.ts)

```typescript
// Arquivo contendo todas as constantes de caminhos para as rotas
// Facilita a manutenção e evita erros de digitação

export const PATHS = {
  HOME: "/",
  ABOUT: "/about",
  CONTACT: "/contact",
  USER: "/user/:id", // Rota com parâmetro
  PRODUCTS: "/products",
  PRODUCT_DETAILS: "/products/:productId",
} as const;

// Exporta os tipos para uso com TypeScript
export type AppPaths = (typeof PATHS)[keyof typeof PATHS];
```

### Configuração de Rotas Principal (src/routes/index.tsx)

```typescript
import { createBrowserRouter, RouteObject } from "react-router-dom";
import { PATHS } from "./paths";

// Importação das páginas
import App from "../App";
import Home from "../pages/Home";
import About from "../pages/About";
import Contact from "../pages/Contact";
import User from "../pages/User";
import Products from "../pages/Products";
import ProductDetails from "../pages/ProductDetails";
import NotFound from "../pages/NotFound";

// Definição das rotas com TypeScript
const routes: RouteObject[] = [
  {
    path: PATHS.HOME,
    element: <App />, // Layout principal
    errorElement: <NotFound />, // Página de erro global
    children: [
      {
        index: true, // Rota raiz
        element: <Home />,
      },
      {
        path: PATHS.ABOUT,
        element: <About />,
      },
      {
        path: PATHS.CONTACT,
        element: <Contact />,
      },
      {
        path: PATHS.USER,
        element: <User />,
      },
      {
        path: PATHS.PRODUCTS,
        element: <Products />,
      },
      {
        path: PATHS.PRODUCT_DETAILS,
        element: <ProductDetails />,
      },
    ],
  },
];

// Criação do router usando a configuração acima
export const router = createBrowserRouter(routes);
```

### Configuração do Ponto de Entrada (src/main.tsx)

```typescript
import React from "react";
import ReactDOM from "react-dom/client";
import { RouterProvider } from "react-router-dom";
import { router } from "./routes";
import "./index.css";

ReactDOM.createRoot(document.getElementById("root")!).render(
  <React.StrictMode>
    <RouterProvider router={router} />
  </React.StrictMode>
);
```

### Componente App com Layout (src/App.tsx)

```typescript
import { Outlet } from "react-router-dom";
import Navbar from "./components/Navbar";
import Footer from "./components/Footer";

function App() {
  return (
    <div className="app-container">
      <Navbar />
      <main>
        {/* O componente Outlet renderiza a rota filha atual */}
        <Outlet />
      </main>
      <Footer />
    </div>
  );
}

export default App;
```

## 3. Componentes Principais e Hooks

### Navegação com o Componente Link

```typescript
import { Link } from "react-router-dom";
import { PATHS } from "../routes/paths";

function Navbar() {
  return (
    <nav>
      <ul>
        {/* Link é usado para navegação sem recarregar a página */}
        <li>
          <Link to={PATHS.HOME}>Home</Link>
        </li>
        <li>
          <Link to={PATHS.ABOUT}>Sobre</Link>
        </li>
        <li>
          <Link to={PATHS.CONTACT}>Contato</Link>
        </li>
        <li>
          <Link to={PATHS.PRODUCTS}>Produtos</Link>
        </li>
      </ul>
    </nav>
  );
}

export default Navbar;
```

### Navegação Programática com useNavigate

```typescript
import { useNavigate } from "react-router-dom";
import { PATHS } from "../routes/paths";

function ProductCard({ product }) {
  const navigate = useNavigate(); // Hook para navegação programática

  const handleClick = () => {
    // Navegação programática para detalhes do produto
    // Substitui ':productId' pelo ID real do produto
    const productPath = PATHS.PRODUCT_DETAILS.replace(":productId", product.id);
    navigate(productPath);
  };

  return (
    <div className="product-card" onClick={handleClick}>
      <h3>{product.name}</h3>
      <p>{product.description}</p>
      <button>Ver Detalhes</button>
    </div>
  );
}

export default ProductCard;
```

### Acesso a Parâmetros de URL com useParams

```typescript
import { useParams } from "react-router-dom";
import { useEffect, useState } from "react";

// Interface para tipar os parâmetros esperados na URL
interface ProductParams {
  productId: string;
}

function ProductDetails() {
  // Recupera o parâmetro da URL
  const { productId } = useParams<ProductParams>();
  const [product, setProduct] = useState(null);

  useEffect(() => {
    // Exemplo de busca de dados baseada no parâmetro da URL
    const fetchProduct = async () => {
      try {
        const response = await fetch(`/api/products/${productId}`);
        const data = await response.json();
        setProduct(data);
      } catch (error) {
        console.error("Erro ao buscar produto:", error);
      }
    };

    fetchProduct();
  }, [productId]);

  if (!product) return <div>Carregando...</div>;

  return (
    <div className="product-details">
      <h1>{product.name}</h1>
      <p>{product.fullDescription}</p>
      <p>Preço: R${product.price}</p>
    </div>
  );
}

export default ProductDetails;
```

### Acesso a Query Parameters com useSearchParams

```typescript
import { useSearchParams } from "react-router-dom";
import { useEffect, useState } from "react";

function Products() {
  // Hook para manipular query parameters (?category=electronics&sort=price)
  const [searchParams, setSearchParams] = useSearchParams();
  const [products, setProducts] = useState([]);

  // Recupera valores dos query parameters
  const category = searchParams.get("category") || "";
  const sort = searchParams.get("sort") || "name";
  const page = Number(searchParams.get("page") || "1");

  // Função para atualizar filtros
  const updateFilters = (newFilters) => {
    // Constrói novos query parameters preservando valores existentes
    const updatedParams = new URLSearchParams(searchParams);

    Object.entries(newFilters).forEach(([key, value]) => {
      if (value) {
        updatedParams.set(key, value);
      } else {
        updatedParams.delete(key);
      }
    });

    // Atualiza a URL com os novos parâmetros
    setSearchParams(updatedParams);
  };

  useEffect(() => {
    // Exemplo de busca de dados com filtros
    const fetchProducts = async () => {
      try {
        const queryString = searchParams.toString();
        const response = await fetch(`/api/products?${queryString}`);
        const data = await response.json();
        setProducts(data);
      } catch (error) {
        console.error("Erro ao buscar produtos:", error);
      }
    };

    fetchProducts();
  }, [searchParams]); // Refaz a busca quando os parâmetros mudam

  return (
    <div className="products-page">
      <h1>Produtos</h1>

      <div className="filters">
        <select value={category} onChange={(e) => updateFilters({ category: e.target.value })}>
          <option value="">Todas as categorias</option>
          <option value="electronics">Eletrônicos</option>
          <option value="clothing">Roupas</option>
        </select>

        <select value={sort} onChange={(e) => updateFilters({ sort: e.target.value })}>
          <option value="name">Nome (A-Z)</option>
          <option value="price_asc">Preço (menor - maior)</option>
          <option value="price_desc">Preço (maior - menor)</option>
        </select>
      </div>

      <div className="product-list">
        {products.map((product) => (
          <ProductCard key={product.id} product={product} />
        ))}
      </div>

      <div className="pagination">
        <button disabled={page === 1} onClick={() => updateFilters({ page: page - 1 })}>
          Anterior
        </button>
        <span>Página {page}</span>
        <button onClick={() => updateFilters({ page: page + 1 })}>Próxima</button>
      </div>
    </div>
  );
}

export default Products;
```

### Acesso à Localização Atual com useLocation

```typescript
import { useLocation } from "react-router-dom";

function AnalyticsWrapper({ children }) {
  const location = useLocation(); // Acesso às informações da rota atual

  useEffect(() => {
    // Exemplo: enviar evento de analytics quando a rota muda
    const sendPageView = () => {
      const { pathname, search } = location;
      console.log(`Página visitada: ${pathname}${search}`);

      // Exemplo com serviço de analytics
      // analytics.sendPageView({
      //   page: pathname,
      //   queryParams: search
      // });
    };

    sendPageView();
  }, [location]); // Executa quando a localização muda

  return <>{children}</>;
}

export default AnalyticsWrapper;
```

## 4. Rotas Protegidas e Autenticação

### Componente de Rota Protegida com Redirecionamento

```typescript
import { Navigate, Outlet, useLocation } from "react-router-dom";
import { PATHS } from "../routes/paths";
import { useAuth } from "../hooks/useAuth"; // Hook personalizado para autenticação

function ProtectedRoute() {
  const { isAuthenticated } = useAuth();
  const location = useLocation();

  // Se não estiver autenticado, redireciona para login
  // O state permite lembrar a rota original para retornar após login
  if (!isAuthenticated) {
    return <Navigate to="/login" state={{ from: location.pathname }} replace />;
  }

  // Se estiver autenticado, renderiza o conteúdo da rota
  return <Outlet />;
}

export default ProtectedRoute;
```

### Configuração de Rotas Protegidas

```typescript
// Adicionando ao arquivo routes/index.tsx

const routes: RouteObject[] = [
  // ... rotas públicas

  // Rotas protegidas agrupadas sob um layout comum
  {
    element: <ProtectedRoute />, // Componente que verifica autenticação
    children: [
      {
        path: "/dashboard",
        element: <Dashboard />,
      },
      {
        path: "/profile",
        element: <Profile />,
      },
      {
        path: "/settings",
        element: <Settings />,
      },
    ],
  },
];
```

### Redirecionamento após Login

```typescript
import { useNavigate, useLocation } from "react-router-dom";
import { useState } from "react";
import { useAuth } from "../hooks/useAuth";

function Login() {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const { login } = useAuth();
  const navigate = useNavigate();
  const location = useLocation();

  // Recupera a rota original de onde o usuário veio
  const from = location.state?.from || "/dashboard";

  const handleSubmit = async (e) => {
    e.preventDefault();

    try {
      await login(email, password);
      // Redireciona de volta para a página original após login
      navigate(from, { replace: true });
    } catch (error) {
      console.error("Erro ao fazer login:", error);
    }
  };

  return (
    <div className="login-page">
      <h1>Login</h1>
      <form onSubmit={handleSubmit}>
        <div>
          <label htmlFor="email">Email:</label>
          <input type="email" id="email" value={email} onChange={(e) => setEmail(e.target.value)} required />
        </div>
        <div>
          <label htmlFor="password">Senha:</label>
          <input
            type="password"
            id="password"
            value={password}
            onChange={(e) => setPassword(e.target.value)}
            required
          />
        </div>
        <button type="submit">Entrar</button>
      </form>
    </div>
  );
}

export default Login;
```

## 5. Rotas Aninhadas e Layouts

### Exemplo de Rotas Aninhadas com Layout

```typescript
// Em routes/index.tsx

const routes: RouteObject[] = [
  {
    path: PATHS.HOME,
    element: <App />,
    children: [
      // Rotas públicas
      {
        index: true,
        element: <Home />,
      },
      {
        path: PATHS.ABOUT,
        element: <About />,
      },

      // Rotas do painel administrativo com layout próprio
      {
        path: "admin",
        element: <AdminLayout />,
        children: [
          {
            index: true,
            element: <AdminDashboard />,
          },
          {
            path: "users",
            element: <AdminUsers />,
          },
          {
            path: "settings",
            element: <AdminSettings />,
          },
        ],
      },
    ],
  },
];
```

### Layout de Admin com Menu Lateral

```typescript
import { Outlet, NavLink } from "react-router-dom";

function AdminLayout() {
  return (
    <div className="admin-layout">
      <aside className="sidebar">
        <h2>Admin</h2>
        <nav>
          <ul>
            <li>
              {/* NavLink adiciona classe 'active' automaticamente quando a rota está ativa */}
              <NavLink
                to="/admin"
                className={({ isActive }) => (isActive ? "active-link" : "")}
                end // Importante: só fica ativo quando for exatamente esta rota
              >
                Dashboard
              </NavLink>
            </li>
            <li>
              <NavLink to="/admin/users" className={({ isActive }) => (isActive ? "active-link" : "")}>
                Usuários
              </NavLink>
            </li>
            <li>
              <NavLink to="/admin/settings" className={({ isActive }) => (isActive ? "active-link" : "")}>
                Configurações
              </NavLink>
            </li>
          </ul>
        </nav>
      </aside>

      <main className="admin-content">
        {/* Renderiza o componente da rota filha atual */}
        <Outlet />
      </main>
    </div>
  );
}

export default AdminLayout;
```

## 6. Lazy Loading (Carregamento Preguiçoso)

O carregamento preguiçoso permite dividir seu aplicativo em pedaços menores e carregar apenas o que é necessário, melhorando o desempenho.

### Implementação de Lazy Loading

```typescript
import { lazy, Suspense } from "react";
import { createBrowserRouter } from "react-router-dom";
import { PATHS } from "./paths";

// Componente de layout principal é carregado imediatamente
import App from "../App";

// Página de carregamento
import LoadingPage from "../components/LoadingPage";

// Carregamento preguiçoso dos componentes de página
const Home = lazy(() => import("../pages/Home"));
const About = lazy(() => import("../pages/About"));
const Contact = lazy(() => import("../pages/Contact"));
const Products = lazy(() => import("../pages/Products"));
const ProductDetails = lazy(() => import("../pages/ProductDetails"));
const NotFound = lazy(() => import("../pages/NotFound"));

// Rotas de admin com lazy loading
const AdminLayout = lazy(() => import("../layouts/AdminLayout"));
const AdminDashboard = lazy(() => import("../pages/admin/Dashboard"));
const AdminUsers = lazy(() => import("../pages/admin/Users"));
const AdminSettings = lazy(() => import("../pages/admin/Settings"));

const routes = [
  {
    path: PATHS.HOME,
    element: <App />,
    children: [
      {
        index: true,
        element: (
          // Suspense mostra um fallback enquanto o componente é carregado
          <Suspense fallback={<LoadingPage />}>
            <Home />
          </Suspense>
        ),
      },
      {
        path: PATHS.ABOUT,
        element: (
          <Suspense fallback={<LoadingPage />}>
            <About />
          </Suspense>
        ),
      },
      // Outras rotas com o mesmo padrão...

      // Rotas de admin agrupadas com seu próprio Suspense
      {
        path: "admin",
        element: (
          <Suspense fallback={<LoadingPage />}>
            <AdminLayout />
          </Suspense>
        ),
        children: [
          {
            index: true,
            element: <AdminDashboard />,
          },
          {
            path: "users",
            element: <AdminUsers />,
          },
          {
            path: "settings",
            element: <AdminSettings />,
          },
        ],
      },
    ],
  },
];

export const router = createBrowserRouter(routes);
```

### Componente de Loading

```typescript
import React from "react";

function LoadingPage() {
  return (
    <div className="loading-container">
      <div className="spinner"></div>
      <p>Carregando...</p>
    </div>
  );
}

export default LoadingPage;
```

## 7. Tratamento de Erros

### Página de Erro Personalizada

```typescript
import { useRouteError, isRouteErrorResponse } from "react-router-dom";

function ErrorPage() {
  const error = useRouteError();

  // Verifica se é um erro de rota (404, etc)
  if (isRouteErrorResponse(error)) {
    return (
      <div className="error-page">
        <h1>{error.status}</h1>
        <h2>{error.statusText}</h2>
        <p>{error.data?.message || "Algo deu errado."}</p>
        <button onClick={() => window.history.back()}>Voltar</button>
      </div>
    );
  }

  // Erro genérico
  return (
    <div className="error-page">
      <h1>Oops!</h1>
      <h2>Ocorreu um erro inesperado</h2>
      <p>Desculpe pelo inconveniente. Por favor, tente novamente mais tarde.</p>
      <button onClick={() => window.history.back()}>Voltar</button>
    </div>
  );
}

export default ErrorPage;
```

### Configuração de Tratamento de Erros

```typescript
// Em routes/index.tsx

const routes: RouteObject[] = [
  {
    path: PATHS.HOME,
    element: <App />,
    errorElement: <ErrorPage />, // Tratamento de erro global
    children: [
      // ... rotas
    ],
  },
];
```

## 8. Redirecionamentos

### Configuração de Redirecionamentos

```typescript
import { Navigate } from "react-router-dom";
import { PATHS } from "./paths";

const routes: RouteObject[] = [
  // ...outras rotas

  // Redirecionamento simples
  {
    path: "/old-about",
    element: <Navigate to={PATHS.ABOUT} replace />,
  },

  // Redirecionamento com preservação de parâmetros
  {
    path: "/users/:id",
    element: <Navigate to="/user/:id" replace />,
  },

  // Redirecionamento para a página inicial para rotas não encontradas
  {
    path: "*",
    element: <Navigate to={PATHS.HOME} replace />,
  },
];
```

## 9. Hooks Avançados

### Criando um Hook Personalizado para Parâmetros de URL

```typescript
import { useParams, useNavigate, useLocation } from "react-router-dom";

// Hook para lidar com parâmetros de URL tipados
export function useTypedParams<T extends Record<string, string>>() {
  return useParams<T>();
}

// Hook para gerenciar histórico de navegação
export function useHistoryStack(limit = 5) {
  const location = useLocation();
  const navigate = useNavigate();

  // Estado para armazenar histórico recente
  const [history, setHistory] = useState<string[]>([]);

  useEffect(() => {
    setHistory((prev) => {
      // Adiciona caminho atual ao histórico
      const updated = [...prev, location.pathname];
      // Limita o tamanho do histórico
      return updated.slice(-limit);
    });
  }, [location.pathname, limit]);

  // Função para voltar um número específico de páginas
  const goBack = (steps = 1) => {
    const target = history[history.length - 1 - steps];
    if (target) {
      navigate(target);
    } else {
      // Volta para a página inicial se não houver histórico suficiente
      navigate("/");
    }
  };

  return { history, goBack };
}
```

### Hook para Ações de Formulário

```typescript
import { useNavigate, useSubmit, useActionData } from "react-router-dom";

function ContactForm() {
  const navigate = useNavigate();
  const submit = useSubmit();
  const actionData = useActionData(); // Dados retornados pela action do formulário

  const handleSubmit = (event) => {
    event.preventDefault();
    const formData = new FormData(event.target);

    // Envia os dados para a action definida na rota
    submit(formData, {
      method: "post",
      action: "/contact",
    });
  };

  return (
    <form onSubmit={handleSubmit}>
      {/* Exibe erros ou mensagens de sucesso */}
      {actionData?.errors && (
        <div className="form-errors">
          {Object.values(actionData.errors).map((error) => (
            <p key={error}>{error}</p>
          ))}
        </div>
      )}

      {actionData?.success && (
        <div className="success-message">
          <p>Mensagem enviada com sucesso!</p>
        </div>
      )}

      <div>
        <label htmlFor="name">Nome:</label>
        <input type="text" id="name" name="name" required />
      </div>

      <div>
        <label htmlFor="email">Email:</label>
        <input type="email" id="email" name="email" required />
      </div>

      <div>
        <label htmlFor="message">Mensagem:</label>
        <textarea id="message" name="message" rows={5} required></textarea>
      </div>

      <div className="form-actions">
        <button type="button" onClick={() => navigate(-1)}>
          Cancelar
        </button>
        <button type="submit">Enviar</button>
      </div>
    </form>
  );
}

export default ContactForm;
```

## 10. Data Router (Novidades do React Router v6.4+)

### Loaders e Actions

```typescript
// src/routes/index.tsx
import { createBrowserRouter } from "react-router-dom";
import { PATHS } from "./paths";

// Importações necessárias
import App from "../App";
import Products from "../pages/Products";
import ProductDetails from "../pages/ProductDetails";
import Contact from "../pages/Contact";

// Funções loaders e actions
import { productsLoader, productDetailLoader } from "../loaders/productLoaders";
import { contactAction } from "../actions/contactActions";

const routes = [
  {
    path: PATHS.HOME,
    element: <App />,
    children: [
      // ... outras rotas

      // Rota com loader para carregar dados
      {
        path: PATHS.PRODUCTS,
        element: <Products />,
        loader: productsLoader, // Carrega dados antes de renderizar
      },

      // Rota com loader e errorElement específico
      {
        path: PATHS.PRODUCT_DETAILS,
        element: <ProductDetails />,
        loader: productDetailLoader,
        errorElement: <ProductError />, // Tratamento de erro específico
      },

      // Rota com action para formulário
      {
        path: PATHS.CONTACT,
        element: <Contact />,
        action: contactAction, // Processa envio de formulário
      },
    ],
  },
];

export const router = createBrowserRouter(routes);
```

### Implementação de um Loader

```typescript
// src/loaders/productLoaders.ts
import { LoaderFunctionArgs } from "react-router-dom";
import { getProducts, getProductById } from "../api/productApi";

// Loader para lista de produtos
export async function productsLoader({ request }: LoaderFunctionArgs) {
  // Acesso aos query parameters
  const url = new URL(request.url);
  const category = url.searchParams.get("category");
  const sort = url.searchParams.get("sort") || "name";
  const page = Number(url.searchParams.get("page") || "1");

  try {
    // Chama API com os parâmetros extraídos
    const productsData = await getProducts({ category, sort, page });

    return {
      products: productsData.items,
      pagination: {
        total: productsData.total,
        currentPage: page,
        totalPages: Math.ceil(productsData.total / productsData.perPage),
      },
    };
  } catch (error) {
    // Tratamento de erro
    console.error("Erro ao carregar produtos:", error);
    throw new Response("Erro ao carregar produtos", { status: 500 });
  }
}

// Loader para detalhes do produto
export async function productDetailLoader({ params }: LoaderFunctionArgs) {
  const { productId } = params;

  try {
    if (!productId) {
      throw new Response("ID do produto não fornecido", { status: 400 });
    }

    const product = await getProductById(productId);

    if (!product) {
      throw new Response("Produto não encontrado", { status: 404 });
    }

    return { product };
  } catch (error) {
    console.error("Erro ao carregar produto:", error);
    throw error instanceof Response ? error : new Response("Erro ao carregar produto", { status: 500 });
  }
}
```

### Implementação de uma Action

```typescript
// src/actions/contactActions.ts
import { ActionFunctionArgs } from "react-router-dom";
import { sendContactMessage } from "../api/contactApi";

export async function contactAction({ request }: ActionFunctionArgs) {
  // Extrai dados do formulário da requisição
  const formData = await request.formData();
  const name = formData.get("name") as string;
  const email = formData.get("email") as string;
  const message = formData.get("message") as string;

  // Validação básica no servidor
  const errors: Record<string, string> = {};

  if (!name || name.trim().length < 3) {
    errors.name = "Nome deve ter pelo menos 3 caracteres";
  }

  if (!email || !email.includes("@")) {
    errors.email = "Email inválido";
  }

  if (!message || message.trim().length < 10) {
    errors.message = "Mensagem deve ter pelo menos 10 caracteres";
  }

  // Se houver erros, retorna-os
  if (Object.keys(errors).length > 0) {
    return { errors };
  }

  try {
    // Envia dados para a API
    await sendContactMessage({ name, email, message });

    // Retorna sucesso
    return { success: true };
  } catch (error) {
    console.error("Erro ao enviar mensagem:", error);
    return { errors: { _form: "Falha ao enviar mensagem. Tente novamente." } };
  }
}
```

### Uso de Loader e Action em Componentes

```typescript
import { useLoaderData, useNavigation } from "react-router-dom";

function Products() {
  // Dados carregados pelo loader
  const { products, pagination } = useLoaderData();

  // Estado de navegação (idle, loading, submitting)
  const navigation = useNavigation();
  const isLoading = navigation.state === "loading";

  return (
    <div className="products-page">
      <h1>Produtos</h1>

      {/* Indicador de carregamento */}
      {isLoading ? (
        <div className="loading-indicator">Carregando...</div>
      ) : (
        <>
          <div className="product-list">
            {products.map((product) => (
              <ProductCard key={product.id} product={product} />
            ))}
          </div>

          {/* Paginação */}
          <div className="pagination">
            <button
              disabled={pagination.currentPage === 1}
              onClick={() => {
                /* lógica de paginação */
              }}
            >
              Anterior
            </button>
            <span>
              Página {pagination.currentPage} de {pagination.totalPages}
            </span>
            <button
              disabled={pagination.currentPage === pagination.totalPages}
              onClick={() => {
                /* lógica de paginação */
              }}
            >
              Próxima
            </button>
          </div>
        </>
      )}
    </div>
  );
}

export default Products;
```

## 11. Roteamento Avançado

### Rotas com Modal

```typescript
import { useLocation, useNavigate, Outlet } from "react-router-dom";

function AppWithModals() {
  const location = useLocation();
  const navigate = useNavigate();

  // Verifica se há estado backgroundLocation, usado para modais
  const background = location.state?.backgroundLocation;

  // Fecha o modal voltando para a rota anterior
  const closeModal = () => navigate(-1);

  return (
    <>
      {/* Renderiza a página de fundo */}
      {background && <Outlet context={{ background }} />}

      {/* Se temos um background, estamos em uma rota modal */}
      {background && (
        <div className="modal-container" onClick={closeModal}>
          <div className="modal-content" onClick={(e) => e.stopPropagation()}>
            {/* Renderiza o componente da rota atual como modal */}
            <button className="close-button" onClick={closeModal}>
              ×
            </button>
            <Outlet />
          </div>
        </div>
      )}

      {/* Sem background, renderiza normalmente */}
      {!background && <Outlet />}
    </>
  );
}

export default AppWithModals;
```

### Configuração de Rotas com Modal

```typescript
// src/routes/index.tsx

const routes = [
  {
    path: "/",
    element: <AppWithModals />,
    children: [
      {
        index: true,
        element: <Home />,
      },
      {
        path: "products",
        element: <Products />,
      },
      {
        path: "products/:productId",
        element: <ProductDetails />,
      },
      {
        // Esta rota pode ser acessada como modal ou página completa
        path: "quick-view/:productId",
        element: <ProductQuickView />,
      },
    ],
  },
];
```

### Navegação para o Modal

```typescript
import { Link, useLocation } from "react-router-dom";

function ProductItem({ product }) {
  const location = useLocation();

  return (
    <div className="product-item">
      <h3>{product.name}</h3>
      <div className="product-actions">
        {/* Link normal para página de detalhes */}
        <Link to={`/products/${product.id}`}>Ver Detalhes</Link>

        {/* Link para modal, preservando a localização atual como background */}
        <Link to={`/quick-view/${product.id}`} state={{ backgroundLocation: location }}>
          Visualização Rápida
        </Link>
      </div>
    </div>
  );
}
```

## 12. Scroll Restoration

### Configuração do Scroll para o Topo ao Navegar

```typescript
import { useEffect } from "react";
import { useLocation } from "react-router-dom";

function ScrollToTop() {
  const { pathname } = useLocation();

  useEffect(() => {
    // Rola para o topo quando a rota muda
    window.scrollTo(0, 0);
  }, [pathname]);

  return null; // Este componente não renderiza nada
}

// Uso no App.tsx
function App() {
  return (
    <>
      <ScrollToTop />
      <div className="app-container">
        <Navbar />
        <main>
          <Outlet />
        </main>
        <Footer />
      </div>
    </>
  );
}
```

### Restauração de Scroll Avançada

```typescript
import { useEffect, useRef } from "react";
import { useLocation } from "react-router-dom";

function ScrollRestoration() {
  const { pathname } = useLocation();
  const scrollPositions = useRef({}); // Armazena posições de scroll por rota

  // Salva posição de scroll ao sair da rota
  useEffect(() => {
    const savePosition = () => {
      scrollPositions.current[pathname] = window.scrollY;
    };

    // Salva antes de navegar para outra rota
    window.addEventListener("beforeunload", savePosition);

    return () => {
      savePosition();
      window.removeEventListener("beforeunload", savePosition);
    };
  }, [pathname]);

  // Restaura posição de scroll ao voltar para uma rota
  useEffect(() => {
    if (scrollPositions.current[pathname]) {
      // Pequeno timeout para garantir que o DOM esteja pronto
      const timer = setTimeout(() => {
        window.scrollTo(0, scrollPositions.current[pathname]);
      }, 100);

      return () => clearTimeout(timer);
    } else {
      // Se é uma nova rota, scroll para o topo
      window.scrollTo(0, 0);
    }
  }, [pathname]);

  return null;
}
```

## 13. Dicas de Performance

### Memoização de Handlers de Eventos

```typescript
import { useCallback } from "react";
import { useNavigate } from "react-router-dom";

function NavigationMenu() {
  const navigate = useNavigate();

  // Memoiza o handler para evitar re-renderizações desnecessárias
  const handleNavigation = useCallback(
    (path) => {
      navigate(path);
    },
    [navigate]
  );

  return (
    <nav>
      <button onClick={() => handleNavigation("/")}>Home</button>
      <button onClick={() => handleNavigation("/about")}>Sobre</button>
      <button onClick={() => handleNavigation("/products")}>Produtos</button>
      <button onClick={() => handleNavigation("/contact")}>Contato</button>
    </nav>
  );
}
```

### Preload de Componentes

```typescript
import { lazy, useEffect } from "react";
import { useLocation, Link } from "react-router-dom";

// Componentes com lazy loading
const Home = lazy(() => import("../pages/Home"));
const About = lazy(() => import("../pages/About"));
const Products = lazy(() => import("../pages/Products"));

function App() {
  const { pathname } = useLocation();

  // Função para pré-carregar componentes
  const preloadPage = (page) => {
    // Simula um import() para acionar o carregamento do chunk
    switch (page) {
      case "home":
        import("../pages/Home");
        break;
      case "about":
        import("../pages/About");
        break;
      case "products":
        import("../pages/Products");
        break;
      default:
        break;
    }
  };

  return (
    <div className="app">
      <nav>
        <Link to="/" onMouseEnter={() => preloadPage("home")}>
          Home
        </Link>
        <Link to="/about" onMouseEnter={() => preloadPage("about")}>
          Sobre
        </Link>
        <Link to="/products" onMouseEnter={() => preloadPage("products")}>
          Produtos
        </Link>
      </nav>

      <main>
        <Outlet />
      </main>
    </div>
  );
}
```

## 14. Melhores Práticas e Padrões

### Padrão de Organização de Diretórios

```
src/
├── main.tsx                   # Ponto de entrada com RouterProvider
├── App.tsx                    # Layout principal
├── routes/
│   ├── index.tsx              # Configuração central de rotas
│   └── paths.ts               # Constantes de caminhos
├── layouts/
│   ├── AdminLayout.tsx        # Layout para seção admin
│   └── DashboardLayout.tsx    # Layout para dashboard
├── pages/
│   ├── Home.tsx               # Páginas públicas
│   ├── About.tsx
│   ├── Contact.tsx
│   ├── products/
│   │   ├── ProductsPage.tsx   # Página de listagem
│   │   └── ProductDetails.tsx # Página de detalhes
│   └── admin/
│       ├── Dashboard.tsx      # Páginas administrativas
│       └── Settings.tsx
├── components/
│   ├── shared/                # Componentes compartilhados
│   │   ├── Navbar.tsx
│   │   └── Footer.tsx
│   └── product/               # Componentes específicos
│       └── ProductCard.tsx
├── hooks/
│   ├── useAuth.ts             # Hook de autenticação
│   └── useScrollTop.ts        # Hook para scroll
├── loaders/                   # Funções de carregamento
│   └── productLoaders.ts
├── actions/                   # Ações de formulário
│   └── contactActions.ts
└── context/
    └── AuthContext.tsx        # Contexto de autenticação
```

### Usando Custom Hooks para Lógica de Roteamento

```typescript
// src/hooks/useRouteUtils.ts
import { useParams, useNavigate, useLocation } from "react-router-dom";
import { PATHS } from "../routes/paths";

export function useRouteUtils() {
  const params = useParams();
  const navigate = useNavigate();
  const location = useLocation();

  // Gera uma URL com parâmetros substituídos
  const createUrl = (path, params = {}) => {
    let url = path;
    Object.entries(params).forEach(([key, value]) => {
      url = url.replace(`:${key}`, String(value));
    });
    return url;
  };

  // Navega para uma rota usando constantes e parâmetros
  const navigateTo = (pathKey, params = {}, options = {}) => {
    const path = PATHS[pathKey];
    if (!path) {
      console.error(`Path key "${pathKey}" not found`);
      return;
    }
    const url = createUrl(path, params);
    navigate(url, options);
  };

  // Navega para uma rota modal
  const openInModal = (pathKey, params = {}) => {
    const path = PATHS[pathKey];
    if (!path) {
      console.error(`Path key "${pathKey}" not found`);
      return;
    }
    const url = createUrl(path, params);
    navigate(url, { state: { backgroundLocation: location } });
  };

  return {
    params,
    navigate,
    location,
    createUrl,
    navigateTo,
    openInModal,
  };
}
```

### Exemplo de Uso do Hook Personalizado

```typescript
import { useRouteUtils } from "../hooks/useRouteUtils";

function ProductList({ products }) {
  const { navigateTo, openInModal } = useRouteUtils();

  return (
    <div className="product-list">
      {products.map((product) => (
        <div key={product.id} className="product-item">
          <h3>{product.name}</h3>
          <div className="actions">
            {/* Usando o hook para navegação */}
            <button onClick={() => navigateTo("PRODUCT_DETAILS", { productId: product.id })}>Ver Detalhes</button>

            {/* Abrindo em modal */}
            <button onClick={() => openInModal("PRODUCT_QUICK_VIEW", { productId: product.id })}>
              Visualização Rápida
            </button>
          </div>
        </div>
      ))}
    </div>
  );
}
```

## 15. Integração com Bibliotecas de UI

### Exemplo com Material UI

```typescript
import { useNavigate } from "react-router-dom";
import { AppBar, Toolbar, Typography, Button, Drawer, List, ListItem, ListItemText, ListItemIcon } from "@mui/material";
import { Home, Info, ShoppingCart, ContactMail } from "@mui/icons-material";
import { PATHS } from "../routes/paths";

function AppHeader() {
  const navigate = useNavigate();
  const [drawerOpen, setDrawerOpen] = useState(false);

  const menuItems = [
    { text: "Home", icon: <Home />, path: PATHS.HOME },
    { text: "Sobre", icon: <Info />, path: PATHS.ABOUT },
    { text: "Produtos", icon: <ShoppingCart />, path: PATHS.PRODUCTS },
    { text: "Contato", icon: <ContactMail />, path: PATHS.CONTACT },
  ];

  const handleNavigation = (path) => {
    navigate(path);
    setDrawerOpen(false);
  };

  return (
    <>
      <AppBar position="static">
        <Toolbar>
          <Typography variant="h6" style={{ flexGrow: 1 }}>
            Minha Loja
          </Typography>

          {/* Links de navegação para desktop */}
          <div className="desktop-nav">
            {menuItems.map((item) => (
              <Button color="inherit" key={item.text} onClick={() => handleNavigation(item.path)}>
                {item.text}
              </Button>
            ))}
          </div>

          {/* Botão de menu para mobile */}
          <Button color="inherit" className="mobile-menu-button" onClick={() => setDrawerOpen(true)}>
            Menu
          </Button>
        </Toolbar>
      </AppBar>

      {/* Drawer de navegação para mobile */}
      <Drawer anchor="left" open={drawerOpen} onClose={() => setDrawerOpen(false)}>
        <List>
          {menuItems.map((item) => (
            <ListItem button key={item.text} onClick={() => handleNavigation(item.path)}>
              <ListItemIcon>{item.icon}</ListItemIcon>
              <ListItemText primary={item.text} />
            </ListItem>
          ))}
        </List>
      </Drawer>
    </>
  );
}

export default AppHeader;
```

## 16. Considerações de SEO

### Head Management com React Helmet

```typescript
import { Helmet } from "react-helmet-async";
import { useLoaderData } from "react-router-dom";

function ProductDetails() {
  const { product } = useLoaderData();

  return (
    <div className="product-details">
      <Helmet>
        <title>{product.name} | Minha Loja</title>
        <meta name="description" content={product.description.substring(0, 160)} />
        <meta name="keywords" content={`${product.category}, ${product.tags.join(", ")}`} />
        <link rel="canonical" href={`https://minhaloja.com/produtos/${product.id}`} />
        <meta property="og:title" content={`${product.name} | Minha Loja`} />
        <meta property="og:description" content={product.description.substring(0, 160)} />
        <meta property="og:image" content={product.imageUrl} />
        <meta property="og:url" content={`https://minhaloja.com/produtos/${product.id}`} />
        <meta property="og:type" content="product" />
      </Helmet>

      <h1>{product.name}</h1>
      <p className="product-description">{product.description}</p>
      <div className="product-price">
        <span>R$ {product.price.toFixed(2)}</span>
      </div>

      {/* Resto do conteúdo da página */}
    </div>
  );
}

export default ProductDetails;
```

### Provedor de Helmet

```typescript
import { HelmetProvider } from "react-helmet-async";
import { RouterProvider } from "react-router-dom";
import { router } from "./routes";

function Root() {
  return (
    <HelmetProvider>
      <RouterProvider router={router} />
    </HelmetProvider>
  );
}

ReactDOM.createRoot(document.getElementById("root")!).render(
  <React.StrictMode>
    <Root />
  </React.StrictMode>
);
```

## 17. Testes

### Teste Unitário de um Componente com React Router

```typescript
import { render, screen } from "@testing-library/react";
import { BrowserRouter } from "react-router-dom";
import userEvent from "@testing-library/user-event";
import Navbar from "../components/Navbar";

// Helper para envolver componentes com router para teste
const renderWithRouter = (ui, { route = "/" } = {}) => {
  window.history.pushState({}, "Test page", route);

  return {
    user: userEvent.setup(),
    ...render(ui, { wrapper: BrowserRouter }),
  };
};

describe("Navbar Component", () => {
  test("renderiza links de navegação", () => {
    renderWithRouter(<Navbar />);

    expect(screen.getByText(/home/i)).toBeInTheDocument();
    expect(screen.getByText(/sobre/i)).toBeInTheDocument();
    expect(screen.getByText(/produtos/i)).toBeInTheDocument();
    expect(screen.getByText(/contato/i)).toBeInTheDocument();
  });

  test("navegação funciona ao clicar nos links", async () => {
    const { user } = renderWithRouter(<Navbar />);

    await user.click(screen.getByText(/sobre/i));
    expect(window.location.pathname).toBe("/about");

    await user.click(screen.getByText(/produtos/i));
    expect(window.location.pathname).toBe("/products");
  });
});
```

### Testes de Integração com Rotas

```typescript
import { render, screen } from "@testing-library/react";
import { RouterProvider, createMemoryRouter } from "react-router-dom";
import { routes } from "../routes";
import userEvent from "@testing-library/user-event";

describe("Integração de Rotas", () => {
  test("navegação entre páginas", async () => {
    // Cria um router de memória iniciando na home
    const router = createMemoryRouter(routes, {
      initialEntries: ["/"],
    });

    const { user } = { user: userEvent.setup(), ...render(<RouterProvider router={router} />) };

    // Verifica se estamos na home
    expect(screen.getByRole("heading", { name: /bem-vindo/i })).toBeInTheDocument();

    // Navega para a página sobre
    await user.click(screen.getByText(/sobre/i));

    // Verifica se estamos na página sobre
    expect(screen.getByRole("heading", { name: /sobre nós/i })).toBeInTheDocument();

    // Navega para a página de produtos
    await user.click(screen.getByText(/produtos/i));

    // Verifica se estamos na página de produtos
    expect(screen.getByRole("heading", { name: /nossos produtos/i })).toBeInTheDocument();
  });

  test("rota não encontrada exibe 404", async () => {
    // Cria um router de memória iniciando em uma rota que não existe
    const router = createMemoryRouter(routes, {
      initialEntries: ["/rota-que-nao-existe"],
    });

    render(<RouterProvider router={router} />);

    // Verifica se a página 404 é exibida
    expect(screen.getByText(/página não encontrada/i)).toBeInTheDocument();
  });

  test("rota protegida redireciona para login quando não autenticado", async () => {
    // Mock do hook de autenticação para simular usuário não autenticado
    jest.mock("../hooks/useAuth", () => ({
      useAuth: () => ({ isAuthenticated: false }),
    }));

    // Cria um router de memória iniciando em uma rota protegida
    const router = createMemoryRouter(routes, {
      initialEntries: ["/dashboard"],
    });

    render(<RouterProvider router={router} />);

    // Verifica se fomos redirecionados para a página de login
    expect(screen.getByRole("heading", { name: /login/i })).toBeInTheDocument();
  });
});
```

## 18. Exemplo de Implementação Completa

### Estrutura Final de um Projeto

```
src/
├── main.tsx
├── App.tsx
├── assets/
├── components/
│   ├── layout/
│   │   ├── Navbar.tsx
│   │   ├── Footer.tsx
│   │   └── ErrorBoundary.tsx
│   └── ui/
│       ├── LoadingSpinner.tsx
│       └── Modal.tsx
├── context/
│   └── AuthContext.tsx
├── hooks/
│   ├── useAuth.ts
│   └── useRouteUtils.ts
├── layouts/
│   ├── MainLayout.tsx
│   └── AdminLayout.tsx
├── pages/
│   ├── Home.tsx
│   ├── About.tsx
│   ├── Contact.tsx
│   ├── Login.tsx
│   ├── NotFound.tsx
│   ├── products/
│   │   ├── ProductsPage.tsx
│   │   ├── ProductDetails.tsx
│   │   └── ProductForm.tsx
│   └── admin/
│       ├── Dashboard.tsx
│       └── UserManagement.tsx
├── routes/
│   ├── index.tsx
│   └── paths.ts
├── services/
│   ├── api.ts
│   └── productService.ts
├── styles/
│   └── global.css
└── utils/
    └── formatters.ts
```

### App.tsx - Componente Principal

```typescript
import { Suspense } from "react";
import { Outlet } from "react-router-dom";
import { AuthProvider } from "./context/AuthContext";
import { HelmetProvider } from "react-helmet-async";
import Navbar from "./components/layout/Navbar";
import Footer from "./components/layout/Footer";
import LoadingSpinner from "./components/ui/LoadingSpinner";
import ErrorBoundary from "./components/layout/ErrorBoundary";

function App() {
  return (
    <HelmetProvider>
      <AuthProvider>
        <ErrorBoundary>
          <div className="app-container">
            <Navbar />
            <main className="main-content">
              <Suspense fallback={<LoadingSpinner />}>
                <Outlet />
              </Suspense>
            </main>
            <Footer />
          </div>
        </ErrorBoundary>
      </AuthProvider>
    </HelmetProvider>
  );
}

export default App;
```

### Configuração Final das Rotas

```typescript
// src/routes/index.tsx
import { lazy } from "react";
import { createBrowserRouter, Outlet, RouterProvider } from "react-router-dom";
import { PATHS } from "./paths";

// Layout e páginas principais
import App from "../App";
import MainLayout from "../layouts/MainLayout";
import ErrorPage from "../pages/NotFound";
import ProtectedRoute from "../components/ProtectedRoute";

// Carregamento preguiçoso de componentes
const Home = lazy(() => import("../pages/Home"));
const About = lazy(() => import("../pages/About"));
const Contact = lazy(() => import("../pages/Contact"));
const Login = lazy(() => import("../pages/Login"));

// Páginas de produtos
const Products = lazy(() => import("../pages/products/ProductsPage"));
const ProductDetails = lazy(() => import("../pages/products/ProductDetails"));
const ProductForm = lazy(() => import("../pages/products/ProductForm"));

// Páginas admin
const AdminLayout = lazy(() => import("../layouts/AdminLayout"));
const Dashboard = lazy(() => import("../pages/admin/Dashboard"));
const UserManagement = lazy(() => import("../pages/admin/UserManagement"));

// Loaders e Actions
import { productsLoader, productDetailLoader } from "../services/productService";
import { contactAction } from "../services/contactService";

const routes = [
  {
    element: <App />,
    errorElement: <ErrorPage />,
    children: [
      {
        element: <MainLayout />,
        children: [
          {
            index: true,
            element: <Home />,
          },
          {
            path: PATHS.ABOUT,
            element: <About />,
          },
          {
            path: PATHS.CONTACT,
            element: <Contact />,
            action: contactAction,
          },
          {
            path: PATHS.LOGIN,
            element: <Login />,
          },
          {
            path: PATHS.PRODUCTS,
            element: <Products />,
            loader: productsLoader,
          },
          {
            path: PATHS.PRODUCT_DETAILS,
            element: <ProductDetails />,
            loader: productDetailLoader,
          },
        ],
      },
      // Rotas protegidas
      {
        element: <ProtectedRoute />,
        children: [
          {
            path: PATHS.PRODUCT_NEW,
            element: <ProductForm />,
          },
          {
            path: PATHS.PRODUCT_EDIT,
            element: <ProductForm />,
            loader: productDetailLoader,
          },
          {
            path: PATHS.ADMIN,
            element: <AdminLayout />,
            children: [
              {
                index: true,
                element: <Dashboard />,
              },
              {
                path: PATHS.ADMIN_USERS,
                element: <UserManagement />,
              },
            ],
          },
        ],
      },
    ],
  },
  // Fallback para rotas não encontradas
  {
    path: "*",
    element: <ErrorPage />,
  },
];

export const router = createBrowserRouter(routes);

// Componente para inicialização do router
export function Routes() {
  return <RouterProvider router={router} />;
}
```

## Conclusão

O React Router é uma biblioteca poderosa e flexível para gerenciamento de rotas em aplicações React. Esta mini-documentação cobriu os principais aspectos desde a instalação básica até técnicas avançadas de roteamento. Alguns pontos importantes a destacar:

1. **Organização é crucial**: Manter uma estrutura clara de arquivos e constantes de rotas melhora significativamente a manutenção.

2. **TypeScript oferece segurança**: Aproveite o TypeScript para tipagem de parâmetros, rotas e dados carregados.

3. **Performance importa**: Utilize lazy loading, prefetching e memoização para otimizar a experiência do usuário.

4. **Data Router simplifica**: As novas funcionalidades do React Router v6.4+ (loaders e actions) reduzem drasticamente o código boilerplate.

5. **Padrões consistentes**: Desenvolva hooks personalizados e componentes reutilizáveis para manter consistência no código.
